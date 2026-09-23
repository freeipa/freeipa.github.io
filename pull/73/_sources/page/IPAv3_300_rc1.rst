IPAv3_300_rc1
=============

The FreeIPA team is proud to announce version FreeIPA v3.0.0 rc 1.

It can be downloaded from http://www.freeipa.org/page/Downloads.

A build is available in the Fedora 18 and rawhide repositories or for
Fedora 17 via the freeipa-devel repo on www.freeipa.org:
http://freeipa.org/downloads/freeipa-devel.repo . To install in Fedora
17 the updates repo repository needs to be enabled as well. For Fedora
17 you will also need libldb-1.1.12-1 installed for sssd to work. There
are no dependencies on this package.

For additional information see the AD Trust design page
http://freeipa.org/page/IPAv3_AD_trust and the AD Trust testing page
http://freeipa.org/page/IPAv3_testing_AD_trust.



Highlights since 3.0.0 beta 3
-----------------------------

-  Support for the Dogtag CA version 10
-  Verification when setting up AD trust
-  New ipa-client-install option to disable OpenSSH client
   configuration.
-  Expand Referential Integrity checks on hosts, SUDO and HBAC rule
   referential attributes
-  Run the CLEANALLRUV task when deleting a replication agreement to
   remove replication meta-data about removed master. See the
   ipa-replica-manage man page for the list of new commands related to
   CLEANALLRUV command.
-  Try to prevent orphaning other servers when deleting a master.

Upgrading
---------

An IPA server can be upgraded simply by installing updated rpms. The
server does not need to be shut down in advance.

Please note, that the referential integrity extension requires an
extended set of indexes to be configured. RPM update for an IPA server
with a excessive number of hosts, SUDO or HBAC entries may require
several minutes to finish.

If you have multiple servers you may upgrade them one at a time. It is
expected that all servers will be upgraded in a relatively short period
(days or weeks not months). They should be able to co-exist peacefully
but new features will not be available on old servers and enrolling a
new client against an old server will result in the SSH keys not being
uploaded.

Downgrading a server once upgraded is not supported.

Upgrading from 2.2.0 should work but has not been fully tested. Proceed
with caution.

An enrolled client does not need the new packages installed unless you
want to re-enroll it. SSH keys for already installed clients are not
uploaded, you will have to re-enroll the client or manually upload the
keys.

Feedback
--------

Please provide comments, bugs and other feedback via the freeipa-devel
mailing list: http://www.redhat.com/mailman/listinfo/freeipa-devel



Detailed changelog
------------------

Ade Lee (1):

-  Modifications to install scripts for dogtag 10

Alexander Bokovoy (5):

-  Add verification of the AD trust
-  validate SID for trusted domain when adding/modifying ID range
-  Fix error messages and use proper ImportError for dcerpc import
-  Add documentation for 'ipa trust' set of commands
-  Document use of external group membership

Jan Cholasta (3):

-  Add the SSH service to SSSD config file before trying to activate it.
-  Add --no-ssh option to ipa-client-install to disable OpenSSH client
   configuration.
-  SSHPublicKey.fingerprint_dns_sha1 should return unicode value.

Martin Kosek (8):

-  Fix addattr internal error
-  Add attributeTypes to safe schema updater
-  Amend memberAllowCmd and memberDenyCmd attribute types
-  Run index task in ldap updater only when needed
-  Expand Referential Integrity checks
-  Properly convert DN in ipa-client-install
-  Use default reverse zone consistently
-  Fix idrange plugin help

Petr Viktorin (7):

-  ipa-client-install: Obtain host TGT from one specific KDC
-  Fix server installation
-  Use temporary key cache for host key in server installation
-  Update the pot file (translation source)
-  Use Dogtag 10 only when it is available
-  Only stop the main DS instance when upgrading it
-  Use correct Dogtag port in ipaserver.install.certs

Petr Vobornik (4):

-  Prevent opening of multiple dirty dialogs on navigation
-  JSON serialization of long type
-  Show trust status in add success notification
-  Fix integer validation when boundary value is empty string

Rob Crittenden (3):

-  Set SELinux default context to unconfined_u:s0-s0:c0.c1023
-  Run the CLEANALLRUV task when deleting a replication agreement.
-  When deleting a master, try to prevent orphaning other servers.

Sumit Bose (3):

-  ipasam: Fixes build with samba4 rc1
-  Set master_kdc and dns_lookup_kdc to true
-  Update krb5.conf during ipa-adtrust-install

Tomas Babej (2):

-  Make sure selinuxusemap behaves consistently to HBAC rule
-  Improves sssd.conf handling during ipa-client uninstall

Yuri Chornoivan (1):

-  Fix various typos.