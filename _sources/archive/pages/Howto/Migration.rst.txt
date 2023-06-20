Migration
=========

There are several use cases where administrators may choose to migrate
either to FreeIPA, either on the same platform or OS or on different.
This page shows several procedures for different use cases:



Migration from different identity management solution
-----------------------------------------------------



Migrating from NIS to FreeIPA
----------------------------------------------------------------------------------------------

See `related RHEL Guide for detailed
steps <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Linux_Domain_Identity_Authentication_and_Policy_Guide/migrating-from-nis.html>`__



Migrating from LDAP to FreeIPA
----------------------------------------------------------------------------------------------

See `related RHEL Guide
chapter <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Linux_Domain_Identity_Authentication_and_Policy_Guide/Migrating_from_a_Directory_Server_to_IPA.html>`__
or read `how GNOME project
migrated <https://www.dragonsreach.it/2014/10/12/the-gnome-infrastructures-freeipa-move-behind-the-scenes/>`__
from OpenLDAP to FreeIPA.



Migrating from other FreeIPA to FreeIPA
----------------------------------------------------------------------------------------------

If you want to start a new FreeIPA deployment and you cannot simply
create FreeIPA replica as for example *realm* is now different, new
FreeIPA installation has to be made and data migrated.

Users and groups can be migrated using the ``migrate-ds`` command, just
like with any other LDAP based identity management server. You just need
to make sure that FreeIPA Kerberos related attributes are not migrated
as they need to be generated again by the new FreeIPA server and it's
new `Kerberos <Kerberos>`__ settings or keys. The command doesn't
migrate user private groups. Following command is suggested:

::

   $ echo Secret123 | ipa migrate-ds --bind-dn="cn=Directory Manager" --user-container=cn=users,cn=accounts --group-container=cn=groups,cn=accounts --group-objectclass=posixgroup --user-ignore-attribute={krbPrincipalName,krbextradata,krblastfailedauth,krblastpwdchange,krblastsuccessfulauth,krbloginfailedcount,krbpasswordexpiration,krbticketflags,krbpwdpolicyreference,mepManagedEntry} --user-ignore-objectclass=mepOriginEntry --with-compat ldap://migrated.freeipa.server.test

Until `#3656 <https://fedorahosted.org/freeipa/ticket/3656>`__ is
implemented, other objects (SUDO, HBAC, DNS, ...) have to be migrated
manually, by exporting the LDIF from old FreeIPA instance, selecting the
records to be migrated, updating the attributes in batch (e.g. new
realm) and adding the cleaned LDIF to new FreeIPA.



Migrating existing FreeIPA deployment
-------------------------------------



Upgrading to new FreeIPA release
----------------------------------------------------------------------------------------------

See `Upgrade <Upgrade>`__ page.



General procedure
^^^^^^^^^^^^^^^^^

#. Before you begin, make sure that you have **sufficient redundancy**
   in your FreeIPA deployment (at least **2 running replicas**). If the
   upgrade procedure goes wrong in any way, other FreeIPA server can
   keep the functionality until the upgrade process is successfully
   finished. If there is just one FreeIPA server, consider `preparing a
   new
   replica <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Linux_Domain_Identity_Authentication_and_Policy_Guide/Setting_up_IPA_Replicas.html>`__
   used for the upgrade.
#. When ready, simply upgrade the underlying operating system and
   FreeIPA packages on chosen replica. After the upgrade, it may be
   useful to see ``/var/log/ipaupgrade.log`` and check for errors..
#. When the upgrade is finished, it is recommended to check the upgraded
   services to see that upgrade procedure was successful. This includes
   testing that:

   -  FreeIPA users are still recognized by the operating system

      ``$ id $USER``

   -  Kerberos authentication still works and passwords can be changed

      ``$ kinit $USER``

   -  ipa command works

      ``$ ipa user-find``

   -  Web UI works
   -  `DNS <DNS>`__ service (if provided by the server) works

      ``$ host $(hostname)``

   -  `CA <PKI>`__ services (if appropriate) works

      ``$ ipa cert-find``
      ``$ ipa cert-request $(CSR_FILENAME)``

#. After the FreeIPA upgrade is confirmed to be successful, remaining
   FreeIPA server may be upgraded using the same procedure, one by one.



Migrating to different platform or OS
----------------------------------------------------------------------------------------------

Sometimes, FreeIPA server package update within one operating system may
not be an option. In that case, the following general procedure can be
applied:

#. At the start of the procedure, there is a number of FreeIPA servers
   that are about to be migrated to different platform or OS
#. On one of your servers, create a replica file
   (``ipa-replica-prepare``) and copy the replica file to the new
   server/VM which will host the FreeIPA server
#. Install the, preferably most up-to-date, FreeIPA server on the target
   system (``ipa-replica-install``). It should have all the services as
   the original server had, i.e.

   -  if original server had `CA <PKI>`__ installed (it probably did),
      add ``--setup-ca`` option to ``ipa-replica-install``
   -  Note that the FreeIPA master `CA <PKI>`__ server (this is by
      default the first installed FreeIPA server) is being migrated, you
      need to `promote the new CA replica to the FreeIPA master CA
      server <Howto/Promote_CA_to_Renewal_and_CRL_Master>`__
   -  if original server had `DNS <DNS>`__ installed , also add
      ``--setup-dns`` option to ``ipa-replica-install``

      The new server should now have all the capability of the migrated
      servers. Administrator may want to run at least a basic test to
      test all major functions of the migrated FreeIPA server, namely
      testing that:

   -  FreeIPA users are still recognized by the operating system

      ``$ id $USER``

   -  Kerberos authentication still works and passwords can be changed

      ``$ kinit $USER``

   -  ipa command works

      ``$ ipa user-find``

   -  Web UI works
   -  `DNS <DNS>`__ service (if provided by the server) works

      ``$ host $(hostname)``

   -  `CA <PKI>`__ services (if appropriate) works

      ``$ ipa cert-find``
      ``$ ipa cert-request $(CSR_FILENAME)``

#. If the FreeIPA server is configured to provide `DNS <DNS>`__ service,
   FreeIPA domain SRV records should be already updated and FreeIPA
   clients will also use the migrated FreeIPA server for their function.
   When other `DNS <DNS>`__ service is used, SRV records need to be
   either updated manually, if used. If clients are using fixed list of
   servers, administrator would need to update these lists in
   ``/etc/sssd/sssd.conf`` and ``/etc/krb5.conf`` and other
   configuration files that were manually configured.
#. If the installation was successful and the actual migration is about
   to start, administrator may want to spin off more replica on the new
   platform to:

   -  Keep redundancy in case of failure of one of the migrated server
   -  Split the load when the migrated servers go live

#. When the FreeIPA servers on the migrated platform are ready, old
   FreeIPA servers can be stopped, one by one, so that clients will only
   use the migrated FreeIPA servers:

      ``# ipactl stop``
      This step is important, this will prevent loosing data in case the
      new server misses some functionality and will let you start the
      server again in such case

#. When administrator verifies that clients keep functioning properly,
   old FreeIPA server may be removed:

   #. Log in to one of the migrated FreeIPA servers
   #. List all servers in the realm:

         ``ipa-replica-manage list``

   #. Identity server on the olf platform and start removing them, one
      by one:

         ``ipa-replica-manage del old.ipa.server.fqdn``

      This procedure will also remove these servers from FreeIPA
      `DNS <DNS>`__ SRV records, if used.

#. Old FreeIPA servers can be now uninstalled with
   ``ipa-server-install --uninstall``



Migrating Identity Management in RHEL/CentOS
--------------------------------------------

If you are using FreeIPA/Identity Management in RHEL or CentOS, please
refer to downstream guide for migration process:

-  `Migrating the IdM Server to Red Hat Enterprise Linux
   7 <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Linux_Domain_Identity_Authentication_and_Policy_Guide/upgrading.html#migrating-ipa-proc>`__