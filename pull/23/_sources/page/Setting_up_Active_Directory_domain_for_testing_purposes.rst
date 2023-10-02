Setting_up_Active_Directory_domain_for_testing_purposes
=======================================================



Setting up Active Directory domain for testing purposes
=======================================================

Microsoft offers pre-installed Windows Server 2008 R2 Enterprise Edition
x64 for evaluation purposes.

You can download these virtual machine images in VHD format, convert
them to QCOW2 format suitable for KVM and run original VMs under Linux.

Commands in following text are shown with ``typewriter font``.



Before you start
----------------

You will need:

-  at least 16 GB of free space in filesystem (/tmp/vhd is used as
   example)
-  unar or unrar utility

   -  unar is available on Fedora 20 and later
   -  If you cannot find unar:

      -  unrar is not a part of standard Fedora distribution, you needed
         to install it from `RPM Fusion <http://rpmfusion.org/>`__
      -  ``sudo yum localinstall -y``\ ```http://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-stable.noarch.rpm`` <http://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-stable.noarch.rpm>`__
      -  ``sudo yum install -y unrar``

-  qemu-img utility

   -  ``sudo yum install -y qemu-img``

-  KVM, libvirt, QEMU and Virtual Machine Manager

   -  ``sudo yum install -y kvm libvirt qemu virt-manager``



Download and unpack original images
-----------------------------------

For reference and manual download you can see `original Microsoft's page
with detailed
information <http://www.microsoft.com/en-us/download/details.aspx?id=2227>`__.

For fast download run the following commands in terminal: '''

::

   mkdir -p /tmp/vhd
   cd /tmp/vhd
   wget http://download.microsoft.com/download/5/4/C/54C15FA1-B3AA-4A8B-B26C-47C3BA7A20E0/WS2008R2Fullx64Ent.part01.exe
   wget http://download.microsoft.com/download/5/4/C/54C15FA1-B3AA-4A8B-B26C-47C3BA7A20E0/WS2008R2Fullx64Ent.part02.rar
   wget http://download.microsoft.com/download/5/4/C/54C15FA1-B3AA-4A8B-B26C-47C3BA7A20E0/WS2008R2Fullx64Ent.part03.rar
   unar WS2008R2Fullx64Ent.part01.exe
   # OR if unar is not available
   unrar x WS2008R2Fullx64Ent.part01.exe

'''



Convert original VHD image for KVM
----------------------------------

'''

::

   cd "/tmp/vhd/WS2008R2Fullx64Ent/WS2008R2Fullx64Ent/Virtual Hard Disks"
   qemu-img convert -p -f vpc -O qcow2 WS2008R2Fullx64Ent.vhd WS2008R2Fullx64Ent.qcow2

'''

Conversion can take approximately up to 20 minutes. Progress indicator
will show < 10 % for a long time and then it finishes.

Output file (WS2008R2Fullx64Ent.qcow2) should have approximately same
size as original file (WS2008R2Fullx64Ent.vhd).



Define a new virtual machine for AD domain controller
-----------------------------------------------------

-  Run virtual machine manager: ``sudo virt-manager``
-  Click on "create a new virtual machine" icon

.. figure:: Virtman-add-mach1.png
   :alt: Virtman-add-mach1.png

   Virtman-add-mach1.png

-  Enter arbitraty machine id (it is internal to KVM/libvirt) and select
   "Import existing disk image" option

.. figure:: Virtman-add-mach2.png
   :alt: Virtman-add-mach2.png

   Virtman-add-mach2.png

-  Enter path to the disk image:
   ``/tmp/vhd/WS2008R2Fullx64Ent/WS2008R2Fullx64Ent/Virtual Hard Disks/WS2008R2Fullx64Ent.qcow2``
-  Set "OS type" to "Windows" and "Version" to "Microsoft Windows Server
   2008"

.. figure:: Virtman-add-mach5.png
   :alt: Virtman-add-mach5.png

   Virtman-add-mach5.png

-  In next dialog check "Customize configuration before install"

.. figure:: Virtman-add-mach6-highlight.png
   :alt: Virtman-add-mach6-highlight.png

   Virtman-add-mach6-highlight.png

-  In the list on left side select "Disk 1" and under "Advanced options"
   change "Storage format" to "qcow2"

.. figure:: Virtman-add-mach7-highlight.png
   :alt: Virtman-add-mach7-highlight.png

   Virtman-add-mach7-highlight.png

-  **(optional)** If you want to connect to AD from physical network,
   then select appropriate (physical) "Source device" under "NIC"

.. figure:: Virtman-add-mach8-cut.png
   :alt: Virtman-add-mach8-cut.png

   Virtman-add-mach8-cut.png

-  Click on "Begin Installation" button

.. figure:: Virtman-add-mach9-cut.png
   :alt: Virtman-add-mach9-cut.png

   Virtman-add-mach9-cut.png

The new VM will be started automatically.



The first start
---------------

Be patient. Windows will start after few minutes. You has to go through
Next-Next-Finish wizard with language and license.

After the licence wizard you will be forced to change password for
"Administrator". It has to contain at least 6 characters from at least
three character classes. Password "ADMIN4lab" should work. **Important
note**: Passwords for computer Administrator and Active Directory
administrator are not same.



Creating a new Active Directory domain
--------------------------------------

Stef Walter's blog post `How to create an Active Directory domain to
test
against <http://stef.thewalter.net/how-to-create-active-directory-domain.html>`__
contains detailed instructions. Please follow it from step 5 further.