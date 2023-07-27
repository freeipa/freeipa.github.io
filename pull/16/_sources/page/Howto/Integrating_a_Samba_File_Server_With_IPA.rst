Integrating_a_Samba_File_Server_With_IPA
========================================

`Provided by Loris Santamaria on the freeipa-users@redhat.com
list. <https://www.redhat.com/archives/freeipa-users/2014-October/msg00458.html>`__

Samba is a popular choice for a CIFS file server in Linux and Windows
deployments, and thanks to SSSD v1.12.2+ now it is easier than ever to
integrate a Samba file server in an IPA domain, with the usual goodies
expected from IPA, such as Single Sign On and support for trusted Active
Directory users.

**NOTE**: Only Kerberos authentication will work when accessing Samba
shares using this method. This means that Windows clients not joined to
Active Directory forest trusted by IPA would not be able to access the
shares. This is related to SSSD not yet being able to handle NTLMSSP
authentication.

**NOTE**: When a Windows client accesses shares, Windows UI will need to
be able to resolve SIDs in access control lists. Inability to do so will
affect user experience and the way how applications are expected to work
with the share. `A set of experiments in
2017 <https://talks.vda.li/talks/2017/SambaXP/freeipa_gc.pdf>`__ have
demonstrated that Microsoft does not test various fall backs around this
behavior and only consider the path used by Windows UI to communicate
with a Global Catalog service. It is also a 'client-specific' behavior
and thus is not subject of a protocol interoperability or being
documented anywhere. While for some applications/use cases it may work,
it will not work for many others, thus we cannot really qualify it as a
supported solution from FreeIPA side.

Requirements:
-------------

-  An IPA v3.3+ domain
-  A CentOS or RHEL 7 server, which will be configured as a Samba file
   server (Fedora 21 and RHEL7.1 are preferred)
-  Optionally, one trusted AD forest

**NOTE**: On the IPA masters run ``ipa-adtrust-install`` to configure
IPA masters to handle Samba-specific object classes and attributes.
ipa-adtrust-install is part of ``freeipa-server-trust-ad`` package in
Fedora (``ipa-server-trust-ad`` in RHEL 7 or CentOS).

**''IMPORTANT NOTE**'': On the samba file server it is necessary to
install **sssd v1.12.2+**, available in **RHEL7.1/CentOS7.1** or
**Fedora 21**. The packages sssd-libwbclient and libwbclient(from samba)
use alternatives to switch between these libraries. Packaging can be
different on other distributions and thus it needn't work even with
sssd-libwbclient v1.12+.

1) Install required packages packages:

``yum -y install ipa-client sssd-libwbclient samba samba-client``

2) join file server to the ipa realm:

``ipa-client-install --mkhomedir``

**NOTE**: This step may fail shortly after creating the keytab and
configuring sssd, caused by the version mismatch between ipa server
(3.3) and client (4.1). If failure happens, one can complete the
configuration manually:

``authconfig --enablesssdauth --enablemkhomedir --update`` (on the samba
file server)

``ipa dnsrecord-add my.realm sambatest --a-rec=x.y.w.z`` (on ipa server)

3) On the ipa server create the cifs principal for samba:

``ipa service-add cifs/sambatest.my.realm``

4) Install keytab on the samba file server:

``ipa-getkeytab -s ipaserver.my.realm -p cifs/sambatest.my.realm -k /etc/samba/samba.keytab``

5) Edit ``/etc/samba/smb.conf`` on the samba file server:

::

   [global]
           workgroup = MY
           realm = MY.REALM
           dedicated keytab file = FILE:/etc/samba/samba.keytab
           kerberos method = dedicated keytab
           log file = /var/log/samba/log.%m
           security = ads

   [homes]
           browsable = no
           writable = yes

   [shared]
           path = /home/shared
           writable = yes
           browsable=yes
           write list = @admins

6) To enable samba ``/home`` sharing you should turn the proper selinux
boolean:

``setsebool -P samba_enable_home_dirs on``

7) restart samba

``systemctl restart smb.service``

Testing:
--------

On another linux member of the IPA domain it is possible to connect to
the samba shares using ``smbclient -k``:

``kinit user@MY.REALM``

smbclient -k -L sambatest.my.realm

smbclient -k //sambatest.my.realm/shared

On a Microsoft Windows machine member of a trusted AD domain it is
possible to connect to the samba shares by typing in the windows
explorer location bar:

``\\sambatest.my.realm``

Also, if the AD user is an (indirect) member of the IPA admins group,
thanks to the trust relationship and with the above sample ``smb.conf``
he may have write access to the ``\shared`` folder.