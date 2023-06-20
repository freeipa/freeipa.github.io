IPAv3_300_beta2
===============

\__NOTOC_\_ The FreeIPA team is proud to announce version FreeIPA v3.0.0
beta 2.

It can be downloaded from http://www.freeipa.org/page/Downloads.

A build is available in the Fedora 18 and rawhide repositories or for
Fedora 17 via the freeipa-devel repo on www.freeipa.org:
http://freeipa.org/downloads/freeipa-devel.repo . To install in Fedora
17 and 18 the updates-testing repository needs to be enabled as well.

For additional information see the AD Trust design page
http://freeipa.org/page/IPAv3_AD_trust and the AD Trust testing page
http://freeipa.org/page/IPAv3_testing_AD_trust.



Highlights since 3.0.0 beta 1
-----------------------------

-  NTLM password hash is generated for existing users on first use of
   IPA cross-realm environment based on their Kerberos keys without
   requiring a password change.
-  Secure identifiers compatible with Active Directory are generated
   automatically for existing users upon set up of IPA cross-realm
   environment.
-  Use certmonger to renew CA subsystem certificates
-  Support for DNS zone transfers to non-IPA slaves
-  Internal change to LDAP Distinguished Name handling to be more robust
-  Better support for Internet Explorer 9 in the UI
-  Allow multiple servers on client install command-line and configuring
   without DNS discovery.
-  Translation updates

Upgrading
---------

An IPA server can be upgraded simply by installing updated rpms. The
server does not need to be shut down in advance.

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
keys (using host-mod).

Feedback
--------

Please provide comments, bugs and other feedback via the freeipa-devel
mailing list: http://www.redhat.com/mailman/listinfo/freeipa-devel



Detailed changelog
------------------

Alexander Bokovoy (11):

-  ipasam: improve SASL bind callback
-  Use smb.conf 'dedicated keytab file' parameter instead of hard-coded
   value
-  reduce redundant checks in ldapsam_search_users() to a single
   statement
-  ipalib/plugins/trust.py: ValidationError takes 'error' named
   argument, not 'reason'
-  Handle various forms of admin accounts when establishing trusts
-  Follow change in samba4 beta4 for sid_check_is_domain to
   sid_check_is_our_sam
-  Rework task naming in LDAP updates to avoid conflicting names in
   certain cases
-  When ipaNTHash is missing, ask IPA to generate it from kerberos keys
-  Ensure ipa-adtrust-install is run with Kerberos ticket for admin user
-  Handle exceptions when establishing trusts
-  Add internationalization to DCE RPC code

David Spångberg (1):

-  Indirect roles in WebUI

Gowrishankar Rajaiyan (1):

-  Adding exit status 3 & 4 to ipa-client-install man page

Jan Cholasta (2):

-  Add --{set,add,del}attr options to commands which are missing them.
-  Raise Base64DecodeError instead of ConversionError when base64
   decoding fails in Bytes parameters.

John Dennis (2):

-  Use DN objects instead of strings
-  Installation fails when CN is set in certificate subject base

Martin Kosek (12):

-  Do not change LDAPObject objectclass list
-  Add automount map/key update permissions
-  Fix ipa-managed-entries man page typo
-  Improve address family handling in sockets
-  Enable SOA serial autoincrement
-  Add range-mod command
-  Warn user if an ID range with incorrect size was created
-  Print ipa-ldap-updater errors during RPM upgrade
-  Enforce CNAME constrains for DNS commands
-  Avoid redundant info message during RPM update
-  Bump bind-dyndb-ldap version for F18
-  Fix winsync agreements creation

Petr Viktorin (7):

-  Fix batch command error reporting
-  Fix wrong option name in ipa-managed-entries man page
-  Fix updating minimum_connections in ipa-upgradeconfig
-  Framework for admin/install tools, with ipa-ldap-updater
-  Arrange stripping .po files
-  Update translations
-  Create /etc/sysconfig/network if it doesn't exist

Petr Vobornik (31):

-  Moved configuration to last position in navigation
-  Display loginas information only after login
-  Password policy measurement units.
-  Web UI: kerberos ticket policy measurement units
-  Add and remove dns per-domain permission in Web UI
-  Differentiation of widget type and text_widget input type
-  Fixed display of attributes_widget in IE9
-  Bigger textarea for permission type=subtree
-  IDs and names for dialogs
-  Fix autoscroll to top in tables in IE
-  Fixed: Unable to select option in combobox in IE and Chrome
-  Fixed: Unable to select option in combobox in IE and Chrome
-  Fixed: combobox stacking in service adder dialog
-  PAC Type options for services in Web UI
-  Update to jquery.1.7.2.min
-  Update to jquery-ui-1.8.21.custom
-  Fix for incorrect event handler definition
-  Removal of unnecessary overrides of jquery-ui styles
-  Unified buttons
-  Web UI tests fix
-  Fixed incorrect use of jQuery.attr for setting disabled attribute
-  Replace use of attr with prop for booleans
-  Add external group
-  Make group external
-  Make group posix
-  Display group type
-  Attribute facet
-  Group external member facet
-  Read-only external facet for non-external groups
-  Handle case when trusted domain user access the Web UI
-  Disable caching of Web UI login_kerberos request
-  Update other facets on delete from search page

Rob Crittenden (12):

-  Centralize timeout for waiting for servers to start.
-  Make client server option multi-valued, allow disabling DNS discovery
-  Don't hardcode serial_autoincrement to True.
-  Support per-principal sessions and handle session update failures
-  Default to no when trying trying to install a replica on wrong
   server.
-  Fix validator for SELinux user map settings in config plugin.
-  Use certmonger to renew CA subsystem certificates
-  Add per-service option to store the types of PAC it supports
-  Convert PKCS#11 subject to string before passing to ipapython.DN
-  Use DN object for Directory Manager in ipa-replica-manage connect
   command
-  Raise proper exception when given a bad DN attribute.
-  Validate default user in ordered list when using setattr, require MLS

Simo Sorce (14):

-  Fix wrong check after allocation.
-  Fix safety checks to prevent orphaning replicas
-  Fix detection of deleted masters
-  Add libtalloc-devel as spec file BuildRequire
-  Add all external samba libraries to BuildRequires
-  Do not check for DNA magic values
-  Move code into common krb5 utils
-  Improve loops around slapi mods
-  Add special modify op to regen ipaNTHash
-  Move mspac structure to be a private pointer
-  Load list of trusted domain on connecting to ldap
-  Properly name function to add ipa external groups
-  Split out manipulation of logon_info blob
-  Add PAC filtering

Sumit Bose (4):

-  Allow silent build if available
-  ipasam: fixes for clang warnings
-  ipasam: replace testing code
-  Fix typo

Tomas Babej (5):

-  Adds check for ipa-join.
-  Permissions of replica files changed to 0600.
-  Handle SSSD restart crash more gently.
-  Corrects help description of selinuxusermap.
-  Improves exception handling in ipa-replica-prepare.