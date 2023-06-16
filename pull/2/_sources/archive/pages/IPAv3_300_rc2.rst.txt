The FreeIPA team is proud to announce version FreeIPA v3.0.0 rc 2.

It can be downloaded from http://www.freeipa.org/page/Downloads.

Builds are not yet available but we plan builds for Fedora 18 and
rawhide. The tarball is available on the Downloads page.

For additional information see the AD Trust design page
http://freeipa.org/page/IPAv3_AD_trust and the AD Trust testing page
http://freeipa.org/page/IPAv3_testing_AD_trust.

.. _highlights_since_3.0.0_rc_1:

Highlights since 3.0.0 rc 1
---------------------------

-  Python changes to work with python-ldap 2.3.
-  Add missing indices for automount and principal aliases which will
   improve performance.
-  Provide a new Firefox extension for configuring the browser. Firefox
   15 deprecated the interface we used in the past to set the Kerberos
   negotiation directives. This new extension will be used on Firefox 15
   and beyond, and the older interface for older browsers.
-  Man page improvements
-  A SID can be created as the last step of ipa-adtrust-install.
-  Create a default fallback group for AD trust users.

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

.. _detailed_changelog:

Detailed changelog
------------------

Alexander Bokovoy (3):

-  Make sure external group members are listed for the external group
-  Change the way SID comparison is done for belonging to trusted domain
-  Support python-ldap 2.3 way of making LDAP control

Martin Kosek (9):

-  Use custom zonemgr for reverse zones
-  Validate SELinux users in config-mod
-  Improve StrEnum validation error message
-  Add support for unified samba packages
-  Improve DN usage in ipa-client-install
-  Index ipakrbprincipalalias and ipaautomountkey attributes
-  Do not produce unindexed search on every DEL command
-  Only use service PAC type as an override
-  Fill ipakrbprincipalalias on upgrades

Petr Viktorin (4):

-  Always handle NotFound error in dnsrecord-mod
-  Don't use bare except: clauses in ipa-client-install
-  Fix NS records in installation
-  Wait for secure Dogtag ports when starting the pki services

Petr Vobornik (5):

-  Kerberos authentication extension
-  Kerberos authentication extension makefiles
-  Build and installation of Kerberos authentication extension
-  Configuration pages changed to use new FF extension
-  Removal of delegation-uris instruction from browser config

Rob Crittenden (3):

-  Fix python syntax in ipa-client-automount
-  Clear kernel keyring in client installer, save dbdir on new
   connections
-  Become IPA v3 RC 2 (3.0.0.rc2)

Sumit Bose (12):

-  Add man page paragraph about running ipa-adtrust-install multiple
   times
-  Enhance description of --no-msdcs in man page
-  Add --rid-base and --secondary-rid-base to ipa-adtrust-install man
   page
-  ipa-adtrust-install: remove wrong check for dm_password
-  ipa-adtrust-install: Add fallback group
-  ipa-adtrust-install: replace print with self.print_msg
-  ipasam: add fallback primary group
-  Add SIDs for existing users and groups at the end of
   ipa-adtrust-install
-  Avoid ldapmodify error messages during ipa-adtrust-install
-  ipa-adtrust-install: print list of needed SRV records
-  Add new ipaIDobject to DNA plugin configuraton
-  ipasam: generate proper SID for trusted domain object

Tomas Babej (2):

-  Improve user addition to default group in user-add
-  Restrict admins group modifications
