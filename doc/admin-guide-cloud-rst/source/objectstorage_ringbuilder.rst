============
Ring-builder
============

Use the swift-ring-builder utility to build and manage rings. This
utility assigns partitions to devices and writes an optimized Python
structure to a gzipped, serialized file on disk for transmission to the
servers. The server processes occasionally check the modification time
of the file and reload in-memory copies of the ring structure as needed.
If you use a slightly older version of the ring, one of the three
replicas for a partition subset will be incorrect because of the way the
ring-builder manages changes to the ring. You can work around this
issue.

The ring-builder also keeps its own builder file with the ring
information and additional data required to build future rings. It is
very important to keep multiple backup copies of these builder files.
One option is to copy the builder files out to every server while
copying the ring files themselves. Another is to upload the builder
files into the cluster itself. If you lose the builder file, you have to
create a new ring from scratch. Nearly all partitions would be assigned
to different devices and, therefore, nearly all of the stored data would
have to be replicated to new locations. So, recovery from a builder file
loss is possible, but data would be unreachable for an extended time.

Ring data structure
~~~~~~~~~~~~~~~~~~~
The ring data structure consists of three top level fields: a list of
devices in the cluster, a list of lists of device ids indicating
partition to device assignments, and an integer indicating the number of
bits to shift an MD5 hash to calculate the partition for the hash.

Partition assignment list
~~~~~~~~~~~~~~~~~~~~~~~~~
This is a list of ``array('H')`` of devices ids. The outermost list
contains an ``array('H')`` for each replica. Each ``array('H')`` has a
length equal to the partition count for the ring. Each integer in the
``array('H')`` is an index into the above list of devices. The partition
list is known internally to the Ring class as ``_replica2part2dev_id``.

So, to create a list of device dictionaries assigned to a partition, the
Python code would look like::

  devices = [self.devs[part2dev_id[partition]] for
  part2dev_id in self._replica2part2dev_id]

That code is a little simplistic because it does not account for the
removal of duplicate devices. If a ring has more replicas than devices,
a partition will have more than one replica on a device.

``array('H')`` is used for memory conservation as there may be millions
of partitions.

Replica counts
~~~~~~~~~~~~~~
To support the gradual change in replica counts, a ring can have a real
number of replicas and is not restricted to an integer number of
replicas.

A fractional replica count is for the whole ring and not for individual
partitions. It indicates the average number of replicas for each
partition. For example, a replica count of 3.2 means that 20 percent of
partitions have four replicas and 80 percent have three replicas.

The replica count is adjustable.

Example::

  $ swift-ring-builder account.builder set_replicas 4
  $ swift-ring-builder account.builder rebalance

You must rebalance the replica ring in globally distributed clusters.
Operators of these clusters generally want an equal number of replicas
and regions. Therefore, when an operator adds or removes a region, the
operator adds or removes a replica. Removing unneeded replicas saves on
the cost of disks.

You can gradually increase the replica count at a rate that does not
adversely affect cluster performance.

For example::

  $ swift-ring-builder object.builder set_replicas 3.01
  $ swift-ring-builder object.builder rebalance
  <distribute rings and wait>...

  $ swift-ring-builder object.builder set_replicas 3.02
  $ swift-ring-builder object.builder rebalance
  <distribute rings and wait>...

Changes take effect after the ring is rebalanced. Therefore, if you
intend to change from 3 replicas to 3.01 but you accidentally type
2.01, no data is lost.

Additionally, the ``swift-ring-builder X.builder create`` command can now
take a decimal argument for the number of replicas.

Partition shift value
~~~~~~~~~~~~~~~~~~~~~
The partition shift value is known internally to the Ring class as
``_part_shift``. This value is used to shift an MD5 hash to calculate
the partition where the data for that hash should reside. Only the top
four bytes of the hash is used in this process. For example, to compute
the partition for the :file:`/account/container/object` path using Python::

  partition = unpack_from('>I',
  md5('/account/container/object').digest())[0] >>
  self._part_shift

For a ring generated with part\_power P, the partition shift value is
``32 - P``.

Build the ring
~~~~~~~~~~~~~~
The ring builder process includes these high-level steps:

#. The utility calculates the number of partitions to assign to each
   device based on the weight of the device. For example, for a
   partition at the power of 20, the ring has 1,048,576 partitions. One
   thousand devices of equal weight each want 1,048.576 partitions. The
   devices are sorted by the number of partitions they desire and kept
   in order throughout the initialization process.

   .. note::

      Each device is also assigned a random tiebreaker value that is
      used when two devices desire the same number of partitions. This
      tiebreaker is not stored on disk anywhere, and so two different
      rings created with the same parameters will have different
      partition assignments. For repeatable partition assignments,
      ``RingBuilder.rebalance()`` takes an optional seed value that
      seeds the Python pseudo-random number generator.

#. The ring builder assigns each partition replica to the device that
   requires most partitions at that point while keeping it as far away
   as possible from other replicas. The ring builder prefers to assign a
   replica to a device in a region that does not already have a replica.
   If no such region is available, the ring builder searches for a
   device in a different zone, or on a different server. If it does not
   find one, it looks for a device with no replicas. Finally, if all
   options are exhausted, the ring builder assigns the replica to the
   device that has the fewest replicas already assigned.

   .. note::

      The ring builder assigns multiple replicas to one device only if
      the ring has fewer devices than it has replicas.

#. When building a new ring from an old ring, the ring builder
   recalculates the desired number of partitions that each device wants.

#. The ring builder unassigns partitions and gathers these partitions
   for reassignment, as follows:

   - The ring builder unassigns any assigned partitions from any
     removed devices and adds these partitions to the gathered list.
   - The ring builder unassigns any partition replicas that can be
     spread out for better durability and adds these partitions to the
     gathered list.
   - The ring builder unassigns random partitions from any devices that
     have more partitions than they need and adds these partitions to
     the gathered list.

#. The ring builder reassigns the gathered partitions to devices by
   using a similar method to the one described previously.

#. When the ring builder reassigns a replica to a partition, the ring
   builder records the time of the reassignment. The ring builder uses
   this value when it gathers partitions for reassignment so that no
   partition is moved twice in a configurable amount of time. The
   RingBuilder class knows this configurable amount of time as
   ``min_part_hours``. The ring builder ignores this restriction for
   replicas of partitions on removed devices because removal of a device
   happens on device failure only, and reassignment is the only choice.

These steps do not always perfectly rebalance a ring due to the random
nature of gathering partitions for reassignment. To help reach a more
balanced ring, the rebalance process is repeated until near perfect
(less than 1 percent off) or when the balance does not improve by at
least 1 percent (indicating we probably cannot get perfect balance due
to wildly imbalanced zones or too many partitions recently moved).
