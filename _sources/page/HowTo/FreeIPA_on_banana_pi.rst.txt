FreeIPA_on_banana_pi
====================

**FreeIPA on banana pi**

--------------

Provided by: Markus Roth

Description
-----------

This How To describes how to install FreeIPA on a single board computer
with ARM CPU like the banana pi or raspberry pi.

Prerequisites
-------------

The system is installed with fedora 21. Because it will be used as a
home server, the graphical desktop is not required. For this the default
runlevel is set to multiuser.target.

``systemctl set-default multi-user.target``

The next step is to enter the FQDN (full qualified domain name) in the
hosts file. For this purpose, there should be the following in the file
/etc/hosts entry:

``192.168.1.1  hostname.example.com  hostname``



Installation of FreeIPA
-----------------------

First, the FreeIPA package has to be installed.

``yum install freeipa-server bind bind-dyndb-ldap``

or as group:

``yum groupinstall “FreeIPA Server”``

all the necessary packages are installed.

In order to finish the setup of the IPA server an adjustment must be
made in the source code. The reason for this is that the certificate
service requires about 10 minutes to start on the banana Pi. There is a
variable set with a wait time of 300s, so the setup cancel.

The adjustment is made in the file:

``/usr/lib/python2.7/site-packages/ipalib/constants.py``

the variable 'startup_timeout', 300 has to be customized.

I changed this value to 900.

Now, the setup is performed as described in the documentation of
FreeIPA. At the setup step

``[8/27]: starting certificate server instance``

you should give 10 minutes the certificate service to start.

`Category:How to <Category:How_to>`__ `Category:Draft
documentation <Category:Draft_documentation>`__