SUDO_Integration_for_AIX
========================

Introduction
============

| First a little background information regarding sudo on AIX:
| As you might or might not know, sudo can be installed on AIX from
  various packages coming from various sources. IBM provides its own
  version of the sudo package through the `AIX toobox for Linux
  applications
  initiative <http://www-03.ibm.com/systems/power/software/aix/linux/>`__,
  but there are several [newer/better] packages out there.

The problem with integrating sudo on AIX with freeIPA is that most sudo
packages compiled for AIX are **NOT** compiled with LDAP support.
Although `sudo’s documentation states that sudo supports different LDAP
implementations <http://www.sudo.ws/sudo/readme_ldap.html>`__, other
than OpenLDAP, I suppose it doesn’t work well with AIX’s LDAP filesets.
That’s just my guess why most sudo packages for AIX aren’t compiled with
LDAP support.

The good news is that `Michel
Perzl <https://twitter.com/michaelperzl>`__, has successfully compiled a
sudo package with LDAP support, however it is compiled against OpenLdap
and not AIX’s LDAP fileset. This means that you'll have to install
OpenLdap as a dependency on you AIX LPAR.



Important Notices
-----------------

| **I cannot assure you that the procedures described on this article
  will work 100% of the time across all the AIX installations out
  there.**
| Please proceed with caution and preferably **try this out in a
  non-production environment before doing this in your production AIX
  environment.**

What I can assure you: None of my LPARs have presented LDAP/Kerberos
authentication or SSH issues (performed with IBM's filesets) after the
execution of this procedure. So from my perspective its safe to say
that:

#. The OpenLDAP package being installed with the following RPM package
   will not interfere with other LDAP packages, such as the IBM LDAP
   fileset, you might have installed on yout AIX LPAR.
#. The OpenSSL package being installed with the following RPM pacakge
   will not interfere with OpenSSL packages, such as the IBM OpenSSL
   fileset, you might have installed on yout AIX LPAR.

Compatibility
-------------

I have successfully implemented the procedures described on this article
on AIX LPARs runnning the following versions of AIX:

-  5.3
-  6.1
-  7.1

I have not tested the procedures described on this article on VIO LPARs
and I advise you all against it as it might void IBM's warranty.
However, technically speaking, it should work just fine on the
``oem_setup_env`` environment level.



Getting started
===============

This section describes the things you need to check, download and setup
before you can continue.



Check your current sudo setup
-----------------------------

If you are interested in this article its very likely that you already
have sudo installed on you AIX LPAR and perhaps you inadvertently, or
luckly, might already be running a LDAP ready sudo binary. If you don't
have sudo installed on your LPAR yet, you should disregard the following
check.

Before you go any further, removing and installing a different sudo
packages, check if your current sudo binary was compiled with LDAP
support. Unless you got your sudo package from Michale's repository its
most likely your sudo won't have LDAP support, however, just to be sure,
check it by running the following command as root:

::

   sudo -V | grep -i ldap

| If your output does not present the highlighted information bellow,
  your sudo is not LDAP ready:
| Configure options: --prefix=/opt/freeware --sbindir=/opt/freeware/sbin
  --libdir=/opt/freeware/lib --mandir=/opt/freeware/man
  --with-logging=syslog --with-logfac=auth --without-pam
  --with-editor=/usr/bin/vi --with-env-editor --with-ignore-dot
  --with-aixauth

**--with-ldap** --with-passprompt=[sudo] password for %p:
--with-tty-tickets

| **``ldap.conf``\ ````\ ``path:``\ ````\ ``/etc/ldap.conf``**
| **``ldap.secret``\ ````\ ``path:``\ ````\ ``/etc/ldap.secret``**



Create directories
------------------

| Create one temporary directory to hold the rpm packages you are going
  to download. I'll use ``/tmp/sudo`` for such purpose.
| Create one directory to hold your ca certificate. I'll use
  ``/etc/ipa`` for such purpose.



Download packages
-----------------

Head to `Michael's website <http://www.perzl.org/aix/index.php>`__ and
download the following RPM packages on their latest versions:

-  `sudo <http://www.perzl.org/aix/index.php?n=Main.Sudo>`__
-  `openldap <http://www.perzl.org/aix/index.php?n=Main.Openldap>`__
-  `openssl <http://www.perzl.org/aix/index.php?n=Main.Openssl>`__
-  `zlib <http://www.perzl.org/aix/index.php?n=Main.Zlib>`__

| 
| Head to the `AIX Toolbox for Linux Applications
  website <http://www-03.ibm.com/systems/power/software/aix/linux/toolbox/alpha.html#G>`__
  and download the ``gettext`` RPM on its latest version. No IBM login
  is required to download packages from the AIX Toolbox.

The reason you should download the gettext RPM package from the AIX
Toolbox for Linux Applications website and not from Michael's website is
because whenever the RPM binary, which is provided by the ``rpm.rte``
fileset, gets updated on AIX (by an TL update for instance) the
``gettext`` libs get overwritten by the fileset, thus breaking rpm on
your LPAR. Michael provides a much better explanation on this issue on
his `FAQ
page <http://www.perzl.org/aix/index.php?n=FAQs.FAQs#toolbox-compatibility-issue>`__.

Once all RPM packages are downloaded, upload them all to ``/tmp/sudo``
on your AIX LPAR.



Packages Installation
=====================

It should go without saying that if you already have sudo installed on
your AIX LPAR you should remove it *before* installing the RPM packages
you have just downloaded.

Go to the ``/tmp/sudo`` directory and test installing the RPM packages
with the following commands:

::

   cd /tmp/sudo
   rpm -ivh *.rpm --test

In accordance with the `POSIX <http://en.wikipedia.org/wiki/POSIX>`__
spirit, if no output is given by the RPM command, your test went well
and you may proceed with installation using the following command:

::

   rpm -ivh *.rpm

Configurations
==============

Now that the packages have been installed some configuration is required
to make it all work.



Name resolution order configuration
-----------------------------------

When Sudo is compiled with LDAP support it consults the Name Service
Switch file to specify the sudoers search order. Sudo looks for a line
beginning with ``sudoers`` and uses this to determine the search order.
Typically this configuration is hosted on the ``/etc/nsswitch.conf``
configuration file on Linux and other Operating Systems that
support/rely on that file.

| On the AIX Operating System, however, the ``/etc/netsvc.conf``
  configuration file is used to specify the ordering of name resolution
  for several commands. In a way it has a similar function to the
  ``/etc/nsswitch.conf`` configuration file on the Linux Operating
  System, although their syntaxes look nothing alike.
| If you check `IBM's netsvc
  documentation <http://www-01.ibm.com/support/knowledgecenter/ssw_aix_71/com.ibm.aix.files/netsvc.conf.htm>`__
  you will see that nowhere it mentions ``sudoers`` as a valid syntax
  option. The short and sweet answer to that is because its not a valid
  syntax entry.

In reality what happens is that the fine people from the Sudo project
decided that, since AIX provides a functionality similar to
``/etc/nsswitch.conf`` through the ``/etc/netsvc.conf`` configuration
file, it would be best to "teach" sudo to use ``/etc/netsvc.conf``
instead of having to implement ``/etc/nsswitch.conf`` on AIX. A very
smart move I might add! `More on this
here. <http://www.sudo.ws/sudoers.ldap.man.html>`__

In order tell sudo on your AIX LPAR to request its rules on LDAP if none
are found on the ``/etc/sudoers`` file, is by adding the following line
on the ``/etc/netsvc.conf`` configuration file:

::

   sudoers = files, ldap

The idea behind doing this is that, if you want to have server specific
sudo rules that bypass the rules on the IPA server, you can do so by
adding them to ``/etc/sudoers`` as you normally would. This sort of
surpasses the lack of HBAC support for AIX for this matter.



OpenLdap configuration
----------------------

As you may see by running ``sudo -V | grep ldap``, sudo expect to find
its ldap configuration on the ``/etc/ldap.conf`` configuration file.
This is not an AIX native configuration file and therefore you'll have
to create it.

| Create a file called ``/etc/ldap.conf`` and add the following content
  to it:

::

   tls_cacert /etc/ipa/ca.crt
   uri ldap://youripaserver.domain.com
   binddn uid=sudo,cn=sysaccounts,cn=etc,dc=domain,dc=com
   bindpw yourclientpassword
   sudoers_base ou=sudoers,dc=domain,dc=com

Make sure to substitute *"youripaserver.domain.com"*, *"domain"*,
*"com"* and *"yourclientpassword"* with the settings you have on your
IPA environment.

This configuration file has nothing to do with the
``/etc/security/ldap/ldap.cfg`` file you use to configure AIX’s LDAP,
this is OpenLdap’s config for sudo and so its only used by sudo. Don’t
worry, this won’t conflict with AIX’s LDAP functionality.



CA Certificate
--------------

Upload your CA certificate to the directory you created called
``/etc/ipa``. Make sure to permission the directory 755 and the
``ca.crt`` file 644.

If you have the proper firewall rules in place and the wget package
installed on you AIX LPAR you can get this CA certificate by running the
following command:

::

   wget -O /etc/ipa/ca.crt http://youripaserver.domain.com/ipa/config/ca.crt



All done
========

If everything goes as planned your sudo will be working with IPA on your
AIX LPAR. You may test this by logging into your AIX LPAR with an LDAP
user and running the following command:

::

   sudo -l

If it brings the sudo rules you set up on your IPA server, like the
example bellow, you are all set.

::

   [thisuser@thisserver]$ sudo -l
   User this user may run the following commands on this server:
       (root) !/bin/cant_run_this_command
       (root) NOPASSWD: /bin/this_command_is_fine
   [thisuser@thisserver]$

One last thing: Before shouting
`Eureka <http://en.wikipedia.org/wiki/Archimedes#Archimedes.27_principle>`__,
make sure that the sudo rules being presented by the ``sudo -l`` command
are being retrieved from the IPA server and not from your local
``/etc/sudoers`` file as we set the resolution order for sudo on
``/etc/netsvc.conf`` to check for local sudo rules before checking for
LDAP sudo rules.



About the author
================

If you have any questions fell free to contact me,
luiz[dot]vianna[at]tivit[dot]com[dot]br on the free-ipa user mailling
list at freeipa-users[at]redhat[dot]com. Removed the obvious signs to
avoid unnecessary spam. ;)

`Category:How to <Category:How_to>`__