**Introduction**

--------------

These tools are offering opportunity to easily try out FreeIPA in
virtual machines environment. And it could be done within a few clicks.
Whole process is divided into two parts, each covered by one script.

First part - covered by **ipa-base-prepare** script - takes care of
preparing base images. The base image is common for all virtual machines
and thus reduce time needed to create whole testing environment.

Second part is covered by **ipa-demo** script. It takes base image,
creates the virtual machines and installs and configures the FreeIPA on
all machines.

The environment consists of three virtual machines (one server and two
clients) by default, but the number of clients could by set to higher
one.

**Requirements**

--------------

Both scripts were tested on Fedora 15 x86_64 and RHEL 6.1 x86_64
systems. These packages are required:

| ``libvirt-0.8.8-7``
| ``qemu-0.14.0-8``
| ``qemu-system-0.14.0-8``
| ``qemu-kvm-0.14.0-8``
| ``qemu-img-0.14.0-8``
| ``python-virtinst-0.500.6-2``
| ``openssh-clients-5.6p1-34``

(Optionally you can install virt-manager and/or virt-viewer)

Hardware (this configuration allowed us to run smoothly all three VMs as
well as use system):

| ``10GB of disk free space``
| ``4 GB RAM``
| ``CPU dual-core with virtualization support``

One virtual machine is supposed to use one CPU and 1GB RAM. Disk images
has growing size with maximum size set to 10GB.

**Get It**

--------------

Both scripts with all necessary files are located in git repository
`here <https://github.com/ohamada/demo>`__. You can download them
through web interface or using git:

``git clone https://github.com/ohamada/demo.git``

**Easiest way to try out**

--------------

Now we'll show the easiest way to try out FreeIPA:

-  Get the scripts and extract them into a directory.

| ``git clone https://github.com/ohamada/demo.git``
| ``cd demo``

-  You must be root to run the scripts:

``su``

-  Prepare base image. It's strongly recommended to specify Fedora
   repository. You should choose the repository from `Fedora mirrors
   list <http://mirrors.fedoraproject.org/publiclist/Fedora/15/x86_64/>`__
   that is the nearest one to your current location. Just remember that
   the selected repository address must end with '.../x86_64/os'
   (example:
   'http://dl.fedoraproject.org/pub/fedora/linux/releases/15/Fedora/x86_64/os/'):

``./ipa-base-prepare.sh --createbase --repo $selected_repository ``

-  Prepare installation image:

``./ipa-base-prepare.sh --installipa``

-  Create virtual machines and prepare whole environment (disk images
   will be saved in /var/lib/libvirt/images)

``./ipa-demo.sh``

-  When the installation finishes, it prints out all necessary
   instruction on how to connect to virtual machines

-  As a next step you can try to set up MediaWiki to run against
   FreeIPA. This tutorial should help you: `Setting up MediaWiki to run
   against FreeIPA <Setting_up_MediaWiki_to_run_against_FreeIPA>`__.

**ipa-base-prepare**

--------------

Ipa-base-prepare can create base image, update it and prepare image that
has freeipa-server installed and is ready to be used by ipa-demo script
for creating virtual machines. It generates own ssh key which is then
used in further steps for running commands on virtual machines. Created
base images are saved in directory called 'archive'. File names of base
images are in this format 'f15-ipa-demo-base.$date.qcow2'. $date is used
for discovering most actual base image. Base image ready for
installation is called 'ipa-ready-image'. Script arguments:

Mandatory arguments:

-  *--createbase* - create base image
-  *--updatebase* - update base image to contain actual packages
-  *--installipa* - prepare base image with installed FreeIPA
-  *--repo* - specify Fedora repository. Using this option is strongly
   recommended because the default one is very slow. You should choose
   the repository from `Fedora mirrors
   list <http://mirrors.fedoraproject.org/publiclist/Fedora/15/x86_64/>`__
   that is the nearest one to your current location. Just remember that
   the selected repository address must end with .../x86_64/os - for
   example
   ` <http://dl.fedoraproject.org/pub/fedora/linux/releases/15/Fedora/x86_64/os/>`__\ http://dl.fedoraproject.org/pub/fedora/linux/releases/15/Fedora/x86_64/os/

Optional arguments:

-  *--imgdir* - specify different directory for lookup/storing of base
   images
-  *--sshkey* - specify ssh key to be used by script instead of newly
   generated key
-  *--base* - specify base image. If you want to update base image or
   prepare installation base image from base image of your own choice.
-  *-h, --help* - prints out help

Usage:

-  Create base image, use existing ssh key and other than default
   repository

``./ipa-base-prepare.sh --createbase --repo http://dl.fedoraproject.org/pub/fedora/linux/releases/15/Fedora/x86_64/os/ --sshkey /home/user/.ssh/key_rsa``

**ipa-demo**

--------------

Ipa-demo takes image called 'ipa-ready-image.qcow2' and uses it as a
base image for creating virtual machines. Images of virtual machines are
saved in '/var/lib/libvirt/images/' directory by default, but user can
specify his own directory. Script also needs private ssh key to access
virtual machines in order to run necessary installation scripts. He
seeks for the key in 'cert' subdirectory of his working directory, but
user can specify which key to use. Script arguments are:

-  *--base* - specify base image. If you want to update base image or
   prepare installation base image from base image of your own choice.
-  *--sshkey* - specify ssh key to be used by script
-  *--imgdir* - specify directory for storing disk images, directory
   must exist (by default /var/lib/libvirt/images)
-  *--clients* - specify number of clients (by default 2)
-  *-h, --help* - prints out help

Usage:

-  Ipa-demo takes specified base image and ssh key and creates five
   virtual machines (one server and four clients) whose disk images are
   stored in 'images' subdirectory

``./ipa-demo.sh --base /mnt/storage/ipa-ready-image.qcow2 --sshkey /home/user/.ssh/key_rsa --imgdir images --clients 4``
