========================
Upload and manage images
========================

A virtual machine image, referred to in this document simply
as an image, is a single file that contains a virtual disk that
has a bootable operating system installed on it. Images are used
to create virtual machine instances within the cloud. For information
about creating image files, see the `OpenStack Virtual Machine
Image Guide <http://docs.openstack.org/image-guide/content/>`_.

Depending on your role, you may have permission to upload and manage
virtual machine images. Operators might restrict the upload and
management of images to cloud administrators or operators only. If you
have the appropriate privileges, you can use the dashboard to upload and
manage images in the admin project.

.. note:: You can also use the :command:`glance` and :command:`nova`
   command-line clients or the Image service and Compute APIs to
   manage images.

Upload an image
~~~~~~~~~~~~~~~

Follow this procedure to upload an image to a project:

#. Log in to the dashboard.

#. From the CURRENT PROJECT on the :guilabel:`Project` tab, select the
   appropriate project.

#. On the :guilabel:`Project` tab, click :guilabel:`Images`.

#. Click :guilabel:`Create Image`.

   The :guilabel:`Create An Image` dialog box appears.

#. Enter the following values:

   +-------------------------------+---------------------------------+
   | :guilabel:`Name`              | Enter a name for the image.     |
   +-------------------------------+---------------------------------+
   | :guilabel:`Description`       | Enter a brief description of    |
   |                               | the image.                      |
   +-------------------------------+---------------------------------+
   | :guilabel:`Image Source`      | Choose the image source from    |
   |                               | the dropdown list. Your choices |
   |                               | are :guilabel:`Image Location`  |
   |                               | and :guilabel:`Image File`.     |
   +-------------------------------+---------------------------------+
   | :guilabel:`Image File` or     | Based on your selection for     |
   | :guilabel:`Image Location`    | :guilabel:`Image Source`, you   |
   |                               | either enter the location URL   |
   |                               | of the image in the             |
   |                               | :guilabel:`Image Location`      |
   |                               | field, or browse for the image  |
   |                               | file on your file  system and   |
   |                               | add it.                         |
   +-------------------------------+---------------------------------+
   | :guilabel:`Format`            | Select the image format (for    |
   |                               | example, QCOW2) for the image.  |
   +-------------------------------+---------------------------------+
   | :guilabel:`Architecture`      | Specify the architecture. For   |
   |                               | example, ``i386`` for a 32-bit  |
   |                               | architecture or ``x86_64`` for  |
   |                               | a 64-bit architecture.          |
   +-------------------------------+---------------------------------+
   | :guilabel:`Minimum Disk (GB)` | Leave these fields empty.       |
   | and                           |                                 |
   | :guilabel:`Minimum RAM (MB)`  |                                 |
   +-------------------------------+---------------------------------+
   | :guilabel:`Public`            | Select this check box to make   |
   |                               | the image public to all users   |
   |                               | with access to the current      |
   |                               | project.                        |
   +-------------------------------+---------------------------------+
   | :guilabel:`Protected`         | Select this check box to ensure |
   |                               | that only users with            |
   |                               | permissions can delete the      |
   |                               | image.                          |
   +-------------------------------+---------------------------------+

#. Click :guilabel:`Create Image`.

   The image is queued to be uploaded. It might take some time before
   the status changes from Queued to Active.

Update an image
~~~~~~~~~~~~~~~

Follow this procedure to update an existing image.

#. Log in to the dashboard.

#. From the CURRENT PROJECT on the :guilabel:`Project` tab, select the
   appropriate project.

#. On the :guilabel:`Project` tab, click :guilabel:`Images`.

#. Select the image that you want to edit.

#. In the :guilabel:`Actions` column, click :guilabel:`More` and then
   select :guilabel:`Edit` from the list.

#. In the :guilabel:`Update Image` dialog box, you can perform the
   following actions:

   -  Change the name of the image.

   -  Select the :guilabel:`Public` check box to make the image public.

   -  Clear the :guilabel:`Public` check box to make the image private.

#. Click :guilabel:`Update Image`.

Delete an image
~~~~~~~~~~~~~~~

Deletion of images is permanent and **cannot** be reversed. Only users
with the appropriate permissions can delete images.

#. Log in to the dashboard.

#. From the CURRENT PROJECT on the :guilabel:`Project` tab, select the
   appropriate project.

#. On the :guilabel:`Project` tab, click :guilabel:`Images`.

#. Select the images that you want to delete.

#. Click :guilabel:`Delete Images`.

#. In the :guilabel:`Confirm Delete Image` dialog box, click
   :guilabel:`Delete Images` to confirm the deletion.
