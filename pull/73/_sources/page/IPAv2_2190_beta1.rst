IPAv2_2190_beta1
================

\__NOTOC_\_

The FreeIPA team is proud to announce version 2.1.90 beta 1.This will
eventually become FreeIPA v2.2.0.

It can be downloaded from `Downloads <Downloads>`__ or from our
development repo (http://freeipa.org/downloads/freeipa-devel.repo).
Fedora 16 and 17 builds are available.

Builds for Fedora 15 are no longer being provided. Packages that FreeIPA
requires are not available in Fedora 15.



Highlights in 2.1.90 beta 1
---------------------------

-  Forms-based login. If Kerberos negotiate authentication fails you
   have the option of logging in using a form using username and
   password. Or you can go directly to /ipa/ui/login.html if you do not
   have/cannot get a Kerberos ticket. This is the preferred alternative
   login mechanism over enabling KrbMethodK5Passwd.
-  Logout from the UI
-  Support for SSH known-hosts with sssd 1.8.0. This will create a
   known-hosts file dynamically based on information stored in IPA.
-  DNS forwarders now configurable via IPA
-  Configurable by DNS zone: query policy, transfer policy, forward and
   reverse synchronization and forward policy.
-  More consistent hostname validation
-  Recommendation that the compat plugin be disabled during migration
   (unnecessary overhead)
-  On new installations the default users group, ipausers, is now
   non-POSIX

Upgrading
---------

We tested upgrades from 2.1.4 successfully but this is beta code. We do
not recommend upgrading a production server.

Installing updated rpms is all that is required to upgrade from 2.1.4.

It is unlikely that downgrading to a previous release once 2.1.90 is
installed will work.

Upgrading directly from the alpha may work but is untested.

Feedback
--------

Please provide comments, bugs and other feedback via the freeipa-devel
mailing list: http://www.redhat.com/mailman/listinfo/freeipa-devel



Detailed Changelog since 2.1.90 beta 1
--------------------------------------

Jan Cholasta (1):

-  Configure SSH features of SSSD in ipa-client-install.

John Dennis (8):

-  update translation pot file and PY_EXPLICIT_FILES list
-  update po files
-  created Transifex resource, adjust tx config file to point to it.
-  Tweak the session auth to reflect developer consensus.
-  Implement session activity timeout
-  Implement password based session login
-  Log a message when returning non-success HTTP result

Martin Kosek (21):

-  Ease zonemgr restrictions
-  Update schema for bind-dyndb-ldap
-  Global DNS options
-  Query and transfer ACLs for DNS zones
-  Add DNS conditional forwarding
-  Add API for PTR sync control
-  Add gidnumber minvalue
-  Add reverse DNS record when forward is created
-  Sanitize UDP checks in conncheck
-  Add client hostname requirements to man page
-  Add SSHFP update policy for existing zones
-  Improve dns error message
-  Improve dnsrecord-add interactive mode
-  Improve hostname and domain name validation
-  Improve FQDN handling in DNS and host plugins
-  Improve hostname verification in install tools
-  Fix typos in ipa-replica-manage man page
-  Remove memberPrincipal for deleted replicas
-  Fix encoding for setattr/addattr/delattr
-  Add help for new structured DNS framework
-  Improve dnsrecord interactive help

Ondrej Hamada (3):

-  Validate attributes in permission-add
-  Migration warning when compat enabled
-  ipa-client-install not calling authconfig

Petr Viktorin (6):

-  Make ipausers a non-posix group on new installs
-  Add extra checking function to XMLRPC test framework
-  Add common helper for interactive prompts
-  Make sure the nolog argument to ipautil.run is not a bare string
-  Use stricter semantics when checking IP address for DNS records
-  Use stricter semantics when checking IP address for DNS records
-  Use reboot from /sbin

Petr Voborník (18):

-  Fixed content type check in login_password
-  Improved usability of login dialog
-  Removed CSV creation from UI
-  Fixed problem when attributes_widget was displaying empty option
-  Added missing configuration options
-  Static metadata update - new DNS options
-  New checkboxes option: Mutual exclusive
-  DNS Zone UI: added new attributes
-  DNS UI: added A,AAAA create reverse options to adder dialog
-  Fixed displaying of A6 Record
-  New UI for DNS global configuration
-  Multiple fields for one attribute
-  Added attrs to permission when target is group or filter
-  Moved is_empty method from field to IPA object
-  Making validators to return true result if empty
-  Fixed DNS record add handling of 4304 error
-  Added unsupported_validator
-  Fixed redirection in Add and edit in automember hostgroup.
-  Fixed selection of single value in combobox
-  Added logout button
-  Forms based authentication UI

Rob Crittenden (37):

-  Limit the change password permission so it can't change admin
   passwords
-  Don't allow "Modify Group membership" permission to manage admins
-  Add the -v option to sslget to provide more verbose errors
-  Make sure memberof is in replication attribute exclusion list.
-  Don't check for schema uniqueness when comparing in ldapupdate.
-  Add Conflicts on mod_ssl because it interferes with mod_proxy and
   dogtag
-  Don't allow IPA master hosts or important services be deleted.
-  Catch public exceptions when creating the LDAP context in WSGI.
-  Don't consider virtual attributes when validating custom
   objectclasses
-  Add Requires to ipa-client on oddjob-mkhomedir
-  Fix managing winsync replication agreements with ipa-replica-manage
-  Check for duplicate winsync agreement before trying to set one up.
-  Remove unused kpasswd.keytab and ldappwd files if they exist.
-  Make sure 389-ds is running when adding memcache service in upgrade.
-  Don't run restorecon if SELinux is disabled or not present.
-  Limit allowed characters in a netgroup name to alpha, digit, -, \_
   and .
-  Don't call memberof task when re-initializing a replica.
-  Fix bad merge of not calling memberof task when re-initializing a
   replica
-  Add support defaultNamingContext and add --basedn to migrate-ds
-  Fix nested netgroups in NIS.
-  Warn that deleting replica is irreversible, try to detect
   reconnection.
-  Don't set migrated user's GID to that of default users group.
-  Don't delete system users that are added during installation.
-  Only apply validation rules when adding and updating.
-  subclass HTTP_Status from plugable.Plugin, fix not_found tests
-  Make hostnames adhere to new standards in HBAC tests
-  Fix WSGI error handling
-  Add status command to retrieve user lockout status
-  Add support for sudoOrder
-  Make hostnames adhere to new standards in hbactest plugin tests
-  Fix API.txt and VERSION to reflect new sudoOrder option.
-  Add --noac option to ipa-client-install man page
-  Do kinit in client before connecting to backend
-  Only warn if ipa-getkeytab doesn't get all requested enctypes.
-  Fix NSS no_init in the NSSHTTPS class

Simo Sorce (4):

-  ipa-kdb: Fix ACL evaluator
-  policy: add function to check lockout policy
-  ipa-kdb: fix delegation acl check
-  Fix ticket checks when using either s4u2proxy or a delegated krbtgt