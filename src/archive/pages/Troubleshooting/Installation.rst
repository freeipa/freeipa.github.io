Installation
============

This page contains troubleshooting advice for FreeIPA server
installation. For trouble shooting other issues, refer to the index at
`Troubleshooting <Troubleshooting>`__.



Server Installation
===================

When installation crashes, check installation log in
``/var/log/ipaserver-install.log``.

If the installation crashed on installing `PKI <PKI>`__ server (Dogtag),
check it's logs as well. The most useful logs are the following:

-  ``/var/log/pki/pki-ca-spawn.$TIME_OF_INSTALLATION.log``
-  ``/var/log/pki/pki-tomcat/catalina.out``
-  ``/var/log/pki/pki-tomcat/ca/system``
-  ``/var/log/pki/pki-tomcat/ca/debug``



pki-selinux policy not loaded properly
======================================

If you see in ``ipaserver-install.log`` line:
``/usr/bin/runcon: invalid context: unconfined_u:system_r:pki_ca_script_t:s0:``
``Invalid argument"`` Then the culprit might be that ``pki-selinux``
failed to load its policy. The best thing to do is to force re-install
``pki-selinux`` (and check for any errors in the /var/log/messages file
or journal).



Replica Installation
====================

When installation crashes, check installation log in
``/var/log/ipareplica-install.log``. When CA is being installed on a
replica, check the aforementioned `PKI <PKI>`__ logs as well.



Migrating from RHEL 6/CentOS 6
------------------------------

-  Installation of certificate server fails with:

      ::

         Clone does not have all the required certificates

      It indicates `bug
      1322059 <https://bugzilla.redhat.com/show_bug.cgi?id=1322059>`__.
      Issue is that RHEL6, while creating replica file, uses
      certificates from a file which was created during server
      installation and potentially contains expired certificates instead
      of fetching the certs from database where they are valid. It is
      fixed on FreeIPA 3.2+. Recovery is to update the file with valid
      certs and then run ``ipa-replica-prepare`` again and try replica
      installation again:

      #. create a /root/dbpass file containing the 'internal' (not
         'internaldb') password from /etc/pki-ca/password

      #. create a /root/dmpass file containing the DM password

      #. run PKCS12Export:

      #. ::

            #PKCS12Export -debug -d /var/lib/pki-ca/alias -p /root/dbpass -w /root/dmpass -o /root/cacert.p12

-  Installation of certificate server fails with:

      ::

         Error while updating security domain: java.io.IOException: 2

      It indicates `bug
      1256039 <https://bugzilla.redhat.com/show_bug.cgi?id=1256039>`__.
      The issue is that certificate server users on master server has
      invalid, usually expired cert, in its database entry even though
      all certs tracked by certmonger are valid. Recovery is to update
      the database entry with correct certificate. Run following script
      or use manual method is described at `freeipa users list
      mail <https://www.redhat.com/archives/freeipa-users/2016-April/msg00143.html>`__.
      ::

         # /usr/share/pki/scripts/restore-subsystem-user.py -v



Replica Installation fails with Invalid Credentials
---------------------------------------------------

-  Installation of the replica fails with:

| ``  [27/40]: setting up initial replication``
| ``Starting replication, please wait until this has completed.``
| ``Update in progress, 15 seconds elapsed``
| ```1`` <ldap://master.example.com:389>`__\ `` reports: Update failed! Status: [49  - LDAP error: Invalid credentials]``
| ``  [error] RuntimeError: Failed to start replication``
| ``Your system may be partly configured.``
| ``Run /usr/sbin/ipa-server-install --uninstall to clean up.``
| ``ipa.ipapython.install.cli.install_tool(CompatServerReplicaInstall): ERROR    Failed to start replication``
| ``ipa.ipapython.install.cli.install_tool(CompatServerReplicaInstall): ERROR    The ipa-replica-install command failed. See /var/log/ipareplica-install.log for more information``

This can happen when the ipa-replica-install command is called with
--no-ntp and the clocks of the master and the replica are not in sync.
Once they are synchronized (either manually or with NTP or chrony),
ipa-replica-install should succeed



Client Installation
===================

When installation does not work as expected, check installation log in
``/var/log/ipaclient-install.log``. You can run installation in verbose
mode if you run ``ipa-client-install`` with ``--debug`` option. (Log
files always contain debug information, so you do not need to re-run
installation with ``--debug`` option.)



Installation breaks on decoding/downloading CA certificate
----------------------------------------------------------

-  \`ipa-client-install\` may crash with error like

      ``ipalib.errors.LDAPError: failed to decode certificate: (SEC_ERROR_INVALID_ARGS) security library: invalid arguments.``

-  This may mean that `PKI <PKI>`__ CA Certificate stored in LDAP was
   not properly imported during upgrade in some of the older versions
-  Verify that the CA certificate is stored correctly

      ``$ ldapsearch -h your.ipa.server.fqdn -x -b "cn=CAcert,cn=ipa,cn=etc,dc=example,dc=test"``

-  The ``cACertificate;binary`` should contain the encoded certificate,
   typically starting with ``MII`` characters
-  If the certificate is missing, go to any FreeIPA master to let
   updater regenerate it:

      ``# kinit admin``
      ``# ldapdelete -Y GSSAPI "cn=CAcert,cn=ipa,cn=etc,dc=example,dc=test"``
      ``# ipa-ldap-updater --upgrade``



Failed to update DNS records
----------------------------

When client cannot update the DNS record in FreeIPA managed DNS zone:

-  Make sure that the respective FreeIPA DNS zone has *Dynamic Updates*
   option enabled:

``$ ipa dnszone-mod zone.name.example. --dynamic-update=TRUE``

-  Make sure that the FreeIPA server with DNS service has port 53 opened
   for **both UDP and TCP** (`related user
   case <https://www.redhat.com/archives/freeipa-users/2015-March/msg00693.html>`__)



Installation breaks on Joining realm
------------------------------------

ipa-client-install may fail with the following error:

| `` Joining realm failed: Failed to add key to the keytab``
| `` child exited with 11``
| `` ``
| `` Installation failed. Rolling back changes.``

This failure may be caused by an empty /etc/krb5.keytab. In this case,
simply delete the file and restart the installation.