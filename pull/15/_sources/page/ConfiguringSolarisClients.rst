ConfiguringSolarisClients
=========================

Back to `FreeIPAv1:Client Configuration
Guide <FreeIPAv1:Client_Configuration_Guide>`__

Introduction
============

This document describes the procedures required to configure various
Solaris operating systems as an IPA client.

Installation
============

Download and install nss-ldap packages from here.

   | ``Solaris 8 packages: ``\ ```http://freeipa.org/downloads/solaris/nss_ldap/8/`` <http://freeipa.org/downloads/solaris/nss_ldap/8/>`__\ `` ``
   | `` Solaris 9 packages: ``\ ```http://freeipa.org/downloads/solaris/nss_ldap/9/`` <http://freeipa.org/downloads/solaris/nss_ldap/9/>`__\ `` ``
   | `` Solaris 10 packages: ``\ ```http://freeipa.org/downloads/solaris/nss_ldap/10/`` <http://freeipa.org/downloads/solaris/nss_ldap/10/>`__\ `` ``
   | `` ``
   | `` For example, pkgadd -d RHATnss-ldap-253-12.i386.pkg``

Configuration
=============



Configuring Solaris 10 as an IPA Client
---------------------------------------



PAM/LDAP/KRB5 configuration
----------------------------------------------------------------------------------------------

/etc/hosts should contain the fully-qualified name of the IPA Solaris
client

::

   | ``10.14.1.48      ipasolaris.example.com       ipasolaris loghost ``

--------------

/etc/resolv.conf should be configured to point to the correct DNS server
that can resolve at least the IPA Solaris client and the ipa server
names.

::

   | ``search example.com ``
   | `` nameserver bindserver.example.com ``

--------------

/etc/nsswitch.conf should be configured to do password and group look up
via LDAP

::

   | ``passwd:     files ldap[NOTFOUND=return] ``
   | `` group:      files ldap[NOTFOUND=return] ``

--------------

/etc/pam.conf should be configured appropriately to use pam kerberos
first.

These lines show how to do pam kerberos authentication for console
login.

::

   | ``login   auth requisite          pam_authtok_get.so.1 ``
   | `` login   auth sufficient         pam_krb5.so.1 ``
   | `` login   auth required           pam_dhkeys.so.1 ``
   | `` login   auth required           pam_unix_cred.so.1 ``
   | `` login   auth required           pam_unix_auth.so.1 use_first_pass ``
   | `` login   auth required           pam_dial_auth.so.1 ``

--------------

/etc/ldap.conf should be configured as shown below

::

   | ``ldap_version 3 ``
   | `` base dc=example,dc=com ``
   | `` nss_base_passwd dc=example,dc=com?sub ``
   | `` nss_base_group dc=example,dc=com?sub ``
   | `` nss_schema rfc2307bis ``
   | `` nss_map_objectclass shadowAccount posixAccount ``
   | `` nss_map_attribute uniqueMember member ``
   | `` nss_initgroups_ignoreusers root,dirsrv ``
   | `` nss_reconnect_maxsleeptime 8 ``
   | `` nss_reconnect_sleeptime 1 ``
   | `` bind_timelimit 5 ``
   | `` timelimit 15 ``
   | `` nss_srv_domain example.com ``
   | `` uri ``\ ```ldap://ipaserver.example.com`` <ldap://ipaserver.example.com>`__\ `` ``

--------------

/etc/krb5/krb5.conf should be configured as follows for kerberos clients
to get kerberos tickets..

::

   | [libdefaults]
   | default_realm = EXAMPLE.COM
   | [realms]
   | EXAMPLE.COM = {
   | kdc = ipaserver.example.com:88
   | admin_server = ipaserver.example.com:749
   | }
   | [domain_realm]
   | .example.com = EXAMPLE.COM
   | example.com = EXAMPLE.COM
   | [logging]
   | default = FILE:/var/krb5/kdc.log
   | kdc = FILE:/var/krb5/kdc.log
   | kdc_rotate = {
   | period = 1d
   | versions = 10
   | }
   | [appdefaults]
   | kinit = {
   | renewable = true
   | forwardable= true
   | }

--------------

/etc/krb5/krb5.keytab - On the IPA server, add a service principal for
the Solaris client machine and generate a keytab file. Place this keytab
on the Solaris machine as /etc/krb5/krb5.keytab

::

   | `` # ipa-addservice host/solarisipaclient.example.com ``
   | ``  # ipa-getkeytab -s ipaserver.example.com -p host/solarisipaclient.example.com -k /tmp/krb5.keytab -e des-cbc-crc ``

--------------

   NOTE: Perform the above mentioned configuration and then reboot the
   Solaris machine so that all the configuration changes are picked up.

--------------



NFS v4 Configuration (only Solaris 10)
----------------------------------------------------------------------------------------------

1. On the IPA server, Obtain a Kerberos ticket for the **admin** user.

::

    # kinit admin

2. On the IPA server, Add an NFS service principal for the client.

   ::

      # ipa-addservice nfs/ipaclient.example.com
      # ipa-getkeytab -s ipaserver.example.com -p nfs/ipaclient.example.com -k /tmp/krb5.keytab -e des-cbc-crc

..

   |Note.png|\ **Note:**

      The Linux NFS implementation still has limited encryption type
      support. You may need to use the **-e des-cbc-crc** to the
      **ipa-getkeytab** command for any **nfs/<FQDN>** service keytab
      you want to set up, both on server and on all clients. This will
      instruct the KDC to generate only DES keys.

3. Copy the ``/tmp/krb5.keytab`` to the Solaris 10 machine, and then
import the contents into the main host keytab using the **ktutil**
utility.

::

   # ktutil
   ktutil: read_kt /tmp/krb5.keytab
   ktutil: write_kt /etc/krb5/krb5.keytab
   ktutil: q

At this point your IPA client should be fully configured to mount NFS
shares using your Kerberos credentials.



Configuring Solaris 9 as an IPA Client
--------------------------------------

Follow Solaris 10 configuration instructions above. Only noticeable
change is in /etc/pam.conf file

::

   | ``login   auth requisite          pam_authtok_get.so.1 ``
   | `` login   auth  sufficient        pam_krb5.so.1 use_first_pass ``
   | `` login   auth  sufficient        pam_unix.so.1 use_first_pass ``
   | `` login   auth required           pam_dhkeys.so.1 ``
   | `` login   auth required           pam_unix_auth.so.1 ``
   | `` login   auth required           pam_dial_auth.so.1 ``



Configuring Solaris 8 as an IPA Client
--------------------------------------

Follow Solaris 10 configuration instructions above. Only noticeable
change is in /etc/pam.conf file

::

   | ``login   auth  sufficient        /usr/lib/security/pam_krb5.so ``
   | `` login   auth required   /usr/lib/security/pam_unix.so use_first_pass ``
   | `` login   auth required   /usr/lib/security/$ISA/pam_dial_auth.so.1 ``



Testing the configuration
=========================

When the Solaris machine is configured per the above instructions, the
following tests should work.

kinit
-----

``Get a Kerberos ticket for an IPA user``

| `` kinit ipauser ( provide password when prompted for )``
| `` klist ( to verify )``

getent
------

::

   | ``Perform the following commands to make sure that getent in Solaris``
   | ``works with IPA.``
   | ``getent passwd admin``
   | ``getent group ipausers``



console login
-------------

::

   | ``At the console of the solaris machine, provide an IPA user name``
   | ``and their Kerberos password to login. ``

ssh
---

``Goto the Solaris machine, get a Kerberos ticket and ssh to the IPA server.``

| ``kinit ipauser@EXAMPLE.COM``
| ``ssh ipauser@ipaserver.example.com``



NFS v4
------

You can use the following command to test the configuration:

::

    # mount -F nfs -o vers=4 -o sec=krb5 ipaserver.example.com:/ /data

Troubleshooting
----------------------------------------------------------------------------------------------

1. If the **mount** command hangs and you see this error:

::

   rpc.svcgssd[3366]: ERROR: GSS-API: error in handle_nullreq: 
   gss_accept_sec_context(): Unspecified GSS failure.  
   Minor code may provide more information - Unknown code krb5 230

Try the following:

-  Destroy the Kerberos cache

   ``# rm -f /tmp/krb*``

-  Obtain a new keytab for the nfs service using **-e des-cbc-crc** for
   the IPA client.
-  Obtain a new keytab for the nfs server principal with **-e
   des-cbc-crc** for the IPA server.

.. |Note.png| image:: Note.png