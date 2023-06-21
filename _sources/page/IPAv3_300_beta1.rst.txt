IPAv3_300_beta1
===============

The FreeIPA team is proud to announce version FreeIPA v3.0.0 beta 1.

It can be downloaded from http://www.freeipa.org/page/Downloads.

A build is available in Fedora rawhide or for Fedora 17 via the
freeipa-devel repo on www.freeipa.org:
http://freeipa.org/downloads/freeipa-devel.repo

For additional information see the AD Trust design page
http://freeipa.org/page/IPAv3_AD_trust and the AD Trust testing page
http://freeipa.org/page/IPAv3_testing_AD_trust.



Highlights in 3.0.0
-------------------

-  Support for AD Trust
-  Per-domain DNS permissions
-  DNS persistent search enabled by default, new zones are seen
   immediately
-  New DNS resolver library
-  Migration improvements
-  The last administrator cannot be removed
-  Forms-based password reset
-  Redesigned action panels in UI
-  Sessions for command-line users
-  Tool to configure automount client, ipa-client-automount

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
keys.

Feedback
--------

Please provide comments, bugs and other feedback via the freeipa-devel
mailing list: http://www.redhat.com/mailman/listinfo/freeipa-devel



Detailed changelog including 2.2.0
----------------------------------

The development of 3.0 occurred simultaneously with 2.2.0 so there is
some overlap.

Adam Young (10):

-  enable proxy for dogtag
-  split metadata call
-  Make mod_nss renegotiation configuration a public function
-  Execute pki proxy setup when server is upgraded if needed
-  Force the upgrade of pki-setup when upgrading the RPMS
-  Fix dynamic display of UI tabs based on rights
-  remove enrolled column
-  Add priority to pwpolicy list
-  Remove delegation from browser config
-  ignore generated services file.

Alexander Bokovoy (61):

-  Propagate environment when it is required.
-  Incorrect name in examples of ipa help hbactest
-  Unroll groups when testing HBAC rules
-  Convert server install code to platform-independent access to system
   services
-  Convert client-side tools to platform-independent access to system
   services
-  Convert installation tools to platform-independent access to system
   services
-  Cleanup whitespace
-  Introduce platform-specific adaptation for services used by FreeIPA.
-  When external host is specified in HBAC rule, allow its use in
   simulation
-  Unroll StrEnum values when displaying help
-  Configure pam_krb5 on the client only if sssd is not configured
-  Setup and restore ntp configuration on the client side properly
-  Fix 'referenced before assignment' warning
-  Before kinit, try to sync time with the NTP servers of the domain we
   are joining
-  Increase number of 'getent passwd attempts' to 10
-  Force kerberos realm to be a string
-  Include indirect membership and canonicalize hosts during HBAC rules
   testing
-  Refactor backup_and_replace_hostname() into a flexible config
   modification tool
-  Write KRB5REALM to /etc/sysconfig/krb5kdc and make use of common
   backup_config_and_replace_variables() tool
-  Refactor authconfig use in ipa-client-install
-  Document --preserve-sssd option of ipa-client-install
-  Use set class instead of dictview class as set is wider supported
-  hbactest fails while you have svcgroup in hbacrule
-  Add support for systemd environments and use it to support Fedora 16
-  Spin for connection success also when socket is not (yet) available
-  Update spec file to use systemd on Fedora 16 and above
-  Quote multiple workers option
-  Check for Python.h during build of py_default_encoding extension
-  Add configure check for libintl.h
-  Create directories for client install
-  Add "Extending FreeIPA" developer guide
-  Small fix to the guide CSS: enable vertical scroll bar
-  Rename included snippets to avoid problems with pylint
-  Fix dependency for samba4-devel package
-  Merge branch 'master' of git+ssh://git.fedorahosted.org/git/freeipa
-  Check through all LDAP servers in the domain during IPA discovery
-  Validate sudo RunAsUser/RunAsGroup arguments
-  Allow hbactest to work with HBAC rules exceeding default IPA limits
-  Add management of inifiles to allow manipulation of systemd units
-  Handle upgrade issues with systemd in Fedora 16 and above
-  Adopt to python-ldap 2.4.6 by removing unused references which are
   not available in python-ldap anymore
-  When changing multiple booleans with setsebool, pass each of them
   separately.
-  Add separate attribute to store trusted domain SID
-  Use dedicated keytab for Samba
-  Add trust management for Active Directory trusts
-  Use fully qualified PDC name when contacting for extended DN
   information
-  Perform case-insensitive searches for principals on TGS requests
-  Properly handle multiple IP addresses per host when installing trust
   support
-  Restart KDC after installing trust support to allow MS PAC generation
-  Add trust-related ACIs
-  get_fqdn() moved to ipaserver.installutils
-  ipa-sam: update sid_to_id() interface to follow passdb API changes in
   Samba
-  Add python-crypto to build requires for AD server-side code
-  Move AD trust support code to freeipa-server-trust-ad subpackage
-  restart dirsrv as part of ipa-adtrust-install
-  Re-format ipa-adtrust-install final message to be within 80
   characters wide
-  Use correct SID attribute for trusted domains
-  Rename 'ipa trust-add-ad' to 'ipa trust-add --type=ad'
-  Support requests for DOMAIN$ account for trusted domains in ipasam
   module
-  Add error condition handling to the SASL bind callback in ipasam
-  Add support for external group members

Endi S. Dewata (105):

-  Fixed browser configuration pages
-  Hide activation/deactivation link from regular users.
-  Fixed problem selecting value from combobox
-  Fixed inconsistent layout for password reset dialog.
-  Removed 'Hide already enrolled' checkbox.
-  Replaced page dirty dialog title.
-  Updated add and delete association dialog titles.
-  Removed unnecessary HBAC/sudo rule category modification.
-  Fixed command partial failure handling.
-  Fixed default map type in automount map adder dialog.
-  Fixed host OTP status.
-  Fixed host keytab status after setting OTP.
-  Fixed host adder dialog to show default DNS zone.
-  Fixed hard-coded UI messages.
-  Fixed problem adding hostgroup into netgroup.
-  Fixed problem with combobox.
-  Fixed hard-coded UI message in entity.js.
-  Fixed missing permission filter field.
-  Fixed problem with combobox using Sahi
-  Fixed unit test for entity select widget.
-  Fixed layout problem in permission adder dialog.
-  Fixed sudo rule association dialogs.
-  Fixed missing optional field.
-  Fixed labels for run-as users and groups.
-  Fixed problem opening host adder dialog.
-  Removed entitlement menu.
-  Fixed posix group checkbox.
-  Fixed columns in HBAC/sudo rules list pages.
-  Removed HBAC rule type.
-  Fixed missing cancel button in unprovisioning dialog.
-  Fixed problem enabling/disabling DNS zone.
-  Fixed problem enrolling member with the same name.
-  Modified dialog to use sections.
-  Removed undo flags from dialog field specs.
-  Fixed problem on combobox with search limit.
-  Fixed problem displaying special characters.
-  Updated DNS zone details page.
-  Replaced description text fields with text areas.
-  Fixed add/delete arrows position.
-  Fixed duplicate entries in enrollment dialog.
-  Updated color scheme.
-  Fixed tab and dialog widths.
-  Use editable combobox for service type.
-  Disable enroll button if nothing selected.
-  Fixed missing default shell field.
-  I18n clean-up.
-  Disable sudo options Delete button if nothing selected.
-  Added confirmation when adding multiple entries.
-  Added selectable labels for radio buttons.
-  Fixed dependency problem in UI test.
-  Fixed inconsistent required/optional attributes.
-  Removed HBAC deny rule warning.
-  Fixed host Enrolled column.
-  Fixed problem clearing validation error on checkboxes.
-  Fixed "enroll" labels.
-  Merged widget's metadata and param_info.
-  Refactored validation code.
-  Fixed inconsistent image names.
-  Fixed inconsistent details facet validation.
-  Added password field in user adder dialog.
-  Fixed blank krbtpolicy and config pages.
-  Moved facet code into facet.js.
-  Added extensible UI framework.
-  Added current password field.
-  Fixed problem changing page in association facet.
-  Updated sample data.
-  Added paging on search facet.
-  Refactored permission target section.
-  Removed develop.js.
-  Added commands into metadata.
-  Refactored entity object resolution.
-  Fixed ipa.js for sessions.
-  Fixed entity definition in test cases.
-  Added support for radio buttons in table widget.
-  Fixed entity metadata resolution.
-  Refactored facet.load().
-  Added HBAC Test page.
-  Fixed navigation buttons for HBAC Test.
-  Fixed search filter in HBAC Test.
-  Added external fields for HBAC Test.
-  Fixed CSS for HBAC Test
-  Fixed I18n labels for HBAC Test
-  Fixed matched/unmatched checkboxes in HBAC Test
-  Added HBAC Test input validation.
-  Fixed problem loading DNS records.
-  Fixed unmatched checkbox name.
-  Fixed combobox icon position.
-  Fixed combobox search icon position.
-  Reload UI when the user changes.
-  Reload UI on server upgrade.
-  Added account status into user search facet.
-  Added policies into user details page.
-  Load user data and policies in a single batch.
-  Added instructions to generate CSR.
-  Fixed problem removing automount keys and DNS records.
-  Enabled paging on self-service permissions and delegations.
-  Enabled paging on automount keys.
-  Show disabled entries in gray.
-  Fixed inconsistent status labels.
-  Fixed host managed-by adder dialog.
-  Added icons for status column.
-  Hide Add/Delete buttons in self-service mode.
-  Use fixed font when displaying certificate.
-  Show password expiration date.
-  Fixed boot.ldif permission.

JR Aquino (5):

-  Create Tool for Enabling/Disabling Managed Entry Plugins
-  Replication: Adjust replica installation to omit processing memberof
   computations
-  Improve sudorule documentation
-  Create FreeIPA CLI Plugin for the 389 Auto Membership plugin
-  Move Managed Entries into their own container in the replicated
   space.

Jan Cholasta (42):

-  Make sure messagebus is running prior to starting certmonger.
-  Verify that passwords specified through command line options of
   ipa-server-install meet the length requirement.
-  Add option to install without the automatic redirect to the Web UI.
-  Search for users in all the naming contexts present on the directory
   server.
-  Add subscription-manager dependency for RHEL.
-  Verify that the external CA certificate files are correct.
-  Check that install hostname matches the server hostname.
-  Fix client install on IPv6 machines.
-  Fix ipa-replica-prepare always warning the user about not using the
   system hostname.
-  Validate name_from_ip parameter of dnszone.
-  Add a function for formatting network locations of the form host:port
   for use in URLs.
-  Work around pkisilent bugs.
-  Disallow deletion of global password policy.
-  Don't leak passwords through kdb5_ldap_util command line arguments.
-  Remove more redundant configuration values from krb5.conf.
-  Finalize plugin initialization on demand.
-  Parse comma-separated lists of values in all parameter types. This
   can be enabled for a specific parameter by setting the "csv" option
   to True.
-  Fix make-lint crash under certain circumstances.
-  Fix attempted write to attribute of read-only object.
-  Add LDAP schema for SSH public keys.
-  Add LDAP ACIs for SSH public key schema.
-  Add support for SSH public keys to user and host objects.
-  Add API initialization to ipa-client-install.
-  Move the nsupdate functionality to separate function in
   ipa-client-install.
-  Update host SSH public keys on the server during client install.
-  Configure ssh and sshd during ipa-client-install.
-  Base64-decode unicode values in Bytes parameters.
-  Add SSH service to platform-specific services.
-  Move the compat module from ipalib to ipapython.
-  Configure SSH features of SSSD in ipa-client-install.
-  Wait for child process to terminate after receiving SIGINT in
   ipautil.run.
-  Parse zone indices in IPv6 addresses in CheckedIPAddress.
-  Fix uses of O=REALM instead of the configured certificate subject
   base.
-  Fix the procedure for getting default values of command parameters.
-  Change parameters to use only default_from for dynamic default
   values.
-  Check whether the default user group is POSIX when adding new user
   with --noprivate.
-  Check configured maximum user login length on user rename.
-  Fix internal error when renaming user with an empty string.
-  Refactor exc_callback invocation.
-  Set the "KerberosAuthentication" option in sshd_config to "no"
   instead of "yes".
-  Redo boolean value encoding.
-  SSH configuration fixes.

John Dennis (38):

-  DN objects should support the insert method
-  Test DN object non-latin Unicode support
-  convert unittests to use DN objects
-  invalid i18n string in dns.py
-  update LINGUAS file, add missing po files
-  Update all po files
-  compute accurate translation statistics
-  add documentation validation to makeapi tool
-  internationalize help topics
-  internationalize cli help framework
-  improve i18n docstring extraction
-  Fix Spanish po translation file
-  Unable to Download Certificate with Browser
-  Add log manager module
-  modify codebase to utilize IPALogManager, obsoletes logging
-  IPAdmin undefined anonymous parameter lists
-  subclass SimpleLDAPObject
-  Restore default log level in server to INFO
-  If "make rpms" fails so will the next make
-  Remove old RPMROOT contents before it is used for rpmbuild
-  update i18n pot file for branch master
-  Add ipa_memcached service
-  add session manager and cache krb auth
-  Update pot file and list of explicit Python files needing translation
-  pulled new po files from Transifex
-  update translation pot file
-  Tweak the session auth to reflect developer consensus.
-  Implement session activity timeout
-  Implement password based session login
-  Log a message when returning non-success HTTP result
-  Replace broken i18n shell test with Python test
-  improve handling of ds instances during uninstall
-  Use indexed format specifiers in i18n strings
-  text unit test should validate using installed mo file
-  Validate DN & RDN parameters for migrate command
-  don't append basedn to container if it is included
-  Fix name error in hbactest
-  validate i18n strings when running "make lint"

Lars Sjostrom (1):

-  Add disovery domain if client domain is different from server domain

Marko Myllynen (2):

-  include <stdint.h> for uintptr_t
-  Don't remove /tmp when removing temp cert dir

Martin Kosek (171):

-  Add missing attribute labels for sudorule
-  Fix automountkey-mod
-  Fix automountlocation-import conflicts
-  ipa-client-install breaks network configuration
-  Fix sudo help and summaries
-  Let Bind track data changes
-  Improve man pages structure
-  Improve ipa-join man page
-  Fix permissions in installers
-  Fix configure.jar permissions
-  Set bind and bind-dyndb-ldap min nvr
-  Fix pylint false positive in hbactest module
-  ipactl does not stop dirsrv
-  dirsrv is not stopped correctly in the fallback
-  Remove checks for ds-replication plugin
-  Fix /usr/bin/ipa dupled server list
-  Revert "Always require SSL in the Kerberos authorization block."
-  Fix error messages in hbacrule
-  Fix LDAPCreate search failure
-  Fix HBAC tests hostnames
-  ipa-client assumes a single namingcontext
-  migrate process cannot handle multivalued pkey attribute
-  Be more clear about selfsign option
-  Install tools crash when password prompt is interrupted
-  Improve ipa-replica-prepare DNS check
-  Prevent collisions of hostgroup and netgroup
-  Make sure ipa-client-install returns correct error code
-  Improve default user/group object class validation
-  Fix i18n in config plugin
-  Fix dnszone-add name_from_ip server validation
-  Improve handling of GIDs when migrating groups
-  ipa-client-install hangs if the discovered server is unresponsive
-  Optimize member/memberof searches in LDAP
-  Make IPv4 address parsing more strict
-  Check hostname resolution sanity
-  Hostname used by IPA must be a system hostname
-  Check /etc/hosts file in ipa-server-install
-  Fix ipa-client-install -U option alignment
-  Improve hostgroup/netgroup collision checks
-  Fix client krb5 domain mapping and DNS
-  Add --zonemgr/--admin-mail validator
-  Fix ipa-managed-entries password option long form
-  Create pkey-only option for find commands
-  Fix ipa-server-install answer cache
-  Fix ipa-replica-conncheck port labels
-  Allow custom server backend encoding
-  Fix DNS zone --allow-dynupdate option behavior
-  Improve DNS record data validation
-  Polish ipa config help
-  Hosts file not updated when IP is passed as option
-  Fix API.txt
-  Fix LDAP object parameter encoding
-  Remove redundant information from API.txt
-  Fix ipa-managed-entries bind procedure
-  Let PublicError accept Gettext objects
-  Fix coverity issues in client CLI tools
-  Enable automember for upgraded servers
-  Make ipa-server-install clean after itself
-  Add --delattr option to complement --setattr/--addattr
-  Revert "Add DNS service records for Windows"
-  Improve zonemgr validator and normalizer
-  Change default DNS zone manager to hostmaster
-  Fix config migration option
-  Ask for user confirmation in ipa-server-install
-  Add connection failure recovery to IPAdmin
-  Add DNS check to conncheck port probe
-  Refactor dnsrecord processing
-  Fix Parameter csv parsing
-  Improve CLI output for complex commands
-  Create per-type DNS API
-  Fix maxvalue in DNS plugin
-  Fix LDAP add calls in replication module
-  Prevent service restart failures in ipa-replica-install
-  Fix LDAP updates in ipa-replica-install
-  Let replicas install without DNS
-  Restore ACI when aci_mod fails
-  Add missing --pkey-only option for selfservice and delegation
-  Replace float with Decimal
-  Improve host-add error message
-  Fix ipa-server-install for dual NICs
-  Fix selfservice-find crashes
-  Mark optional DNS record parts
-  Fix ldap2 combine_filters for ldap2.MATCH_NONE
-  Add missing managing hosts filtering options
-  Improve netgroup-add error messages
-  Fix TXT record parsing
-  Fix NSEC record conversion
-  Add SRV record target validator
-  Add data field for A6 record
-  Improve dnszone-add error message
-  Improve migration help
-  Fix raw format for ACI commands
-  Improve password change error message
-  Remove debug messages
-  Add argument help to CLI
-  Return proper DN in netgroup-add
-  Remove unused options from ipa-managed-entries
-  Add Petr Viktorín to Contributors.txt
-  Ease zonemgr restrictions
-  Update schema for bind-dyndb-ldap
-  Global DNS options
-  Query and transfer ACLs for DNS zones
-  Add DNS conditional forwarding
-  Add API for PTR sync control
-  Add gidnumber minvalue
-  Add reverse DNS record when forward is created
-  Sanitize UDP checks in conncheck
-  Add client hostname requirements to man
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
-  Ignore case in yes/no prompts
-  Refresh resolvers after DNS install
-  Fix migration plugin compat check
-  Fix ipa-replica-manage TLS connection error
-  Treat UPGs correctly in winsync replication
-  Allow port numbers for idnsForwarders
-  Add missing global options in dnsconfig
-  Fix precallback validators in DNS plugin
-  Harden raw record processing in DNS plugin
-  Fix LDAP effective rights control with python-ldap 2.4.x
-  Avoid deleting DNS zone when a context is reused
-  Fix default SOA serial format
-  Amend permissions for new DNS attributes
-  Improve user awareness about dnsconfig
-  Fix dnsrecord-del interactive mode
-  Tolerate UDP port failures in conncheck
-  Improve automount indirect map error message
-  Forbid public access to DNS tree
-  Configure SELinux for httpd during upgrades
-  Fix installation when server hostname is not in a default domain
-  Return correct record name in DNS plugin
-  Fix dnsrecord_add interactive mode
-  Fix DNS and permissions unit tests
-  Raise proper exception when LDAP limits are exceeded
-  Do not fail migration because of duplicate groups
-  Fix help of --hostname option in ipa-client-install
-  Sort password policies properly with --pkey-only
-  Improve error message in zonemgr validator
-  Make ipa 2.2 client capable of joining an older server
-  Fix python Requires in Fedora 17 build
-  Remove ipa-server-install LDAP update errors
-  Remove LDAP limits from DNS service
-  Replace DNS client based on acutil with python-dns
-  Fix default_server configuration in ipapython.config
-  Reset krbtpolicy when a unit test is finished
-  Add rename option for DNS records
-  permission-find missed some results with --pkey-only option
-  Allow relative DNS name in NS validator
-  Fill new DNS zone update policy by default
-  Improve migration NotFound error
-  Fix dnszone-mod --forwader option help string
-  Add sysupgrade state file
-  Enable persistent search by default
-  Enable psearch on upgrades
-  Only set sebools when necessary
-  Password change capability for form-based auth
-  Remove trust work unit test failures
-  Decimal parameter conversion and normalization
-  Remove ipaNTHash from global allow ACI
-  Add missing libsss_idmap Requires on freeipa-server-trust-ad
-  Per-domain DNS record permissions
-  Create default range entry after upgrade

Nalin Dahyabhai (5):

-  list users from nested groups, too
-  note that PKCS#12 files also contain private keys, and that the
   "pkinit" options refer to the KDC's credentials
-  index the fqdn and macAddress attributes for the sake of the compat
   plugin
-  create a "cn=computers" compat area populated with ieee802Device
   entries corresponding to computers with fqdn and macAddress
   attributes
-  add a pair of ethers maps for computers with hardware addresses on
   file

Ondrej Hamada (26):

-  Misleading Keytab field
-  Client install root privileges check
-  Sort password policy by priority
-  Client install checks for nss_ldap
-  User-add random password support
-  HBAC test optional sourcehost option
-  localhost.localdomain clients refused to join
-  Leave nsds5replicaupdateschedule parameter unset
-  Fix 'no-reverse' option description
-  Memberof attribute control and update
-  Validate attributes in permission-add
-  Migration warning when compat enabled
-  ipa-client-install not calling authconfig
-  More exception handlers in ipa-client-install
-  Search allowed attributes in superior objectclasses
-  Typos in FreeIPA messages
-  Netgroup nisdomain and hosts validation
-  Confusing default user groups
-  Unable to rename permission object
-  Fix empty external member processing
-  Allow one letter net/hostgroups names
-  permission-mod prompts for all parameters
-  ipa-server-install reword message
-  Always set ipa_hostname for sssd.conf
-  Case sensitive renaming of objects
-  Change random passwords behaviour

Petr Viktorin (60):

-  Switch --group and --membergroup in example for delegation
-  Fix/add options in ipa-managed-entries man page
-  Honor default home directory and login shell in user_add
-  Clean up i18n strings
-  Internationalization for HBAC and ipalib.output
-  Make ipausers a non-posix group on new installs
-  Add extra checking function to XMLRPC test framework
-  Add common helper for interactive prompts
-  Make sure the nolog argument to ipautil.run is not a bare string
-  Use stricter semantics when checking IP address for DNS records
-  Use reboot from /sbin
-  Allow removing sudo commands with special characters from command
   groups
-  Enforce that required attributes can't be set to None in CRUD Update
-  Mark most config options as required
-  Don't crash when searching with empty relationship options
-  Remove ipausers' gidnumber from tests
-  Use nose tools to check for exceptions
-  Only split CSV in the client, quote instead of escaping
-  Add missing BuildRequires
-  Use valid argument names in tests
-  Add CLI parsing tests
-  Allow multi-line CSV parameters
-  Move test skipping to class setup
-  Fix little test errors
-  Test the batch plugin
-  Defer conversion and validation until after --{add,del,set}attr are
   handled
-  Limit permission and selfservice names to alphanumerics, -, \_, space
-  Convert --setattr values for attributes marked no_update
-  Fix expected error messages in tests
-  Remove pattern_errmsg from API.txt
-  Pass make-test arguments through to Nose
-  Document the 'nonempty' flag
-  Additional tests for pwpolicy
-  Update hostname validator error messages in tests
-  Do not use extra command options in the automount plugin
-  Do not crash on empty reverse member options
-  Do not crash on empty --setattr, --getattr, --addattr
-  Don't fail when adding default objectclasses using config-mod
-  Remove duplicate and unused utility code
-  Validate externalhost (when added by --addattr/--setattr)
-  Do not use extra command options in ACI, permission, selfservice
-  Check for empty/single value parameters before calling callbacks
-  Disallow '<' and non-ASCII characters in the DM password
-  Fix the pwpolicy_find post_callback
-  Disallow setattr on no_update/no_create params
-  Provide a better error message when deleting nonexistent attributes
-  Move install script error handling to a common function
-  Add more automount tests
-  Add samba4-python to BuildRequires
-  Prevent deletion of the last admin
-  Only allow root to run update plugins
-  Clean keytabs before installing new keys into them
-  Fix update plugin order
-  Rework the CallbackInterface
-  Improve ipa-client-install debug output
-  Improve autodiscovery logging
-  Fail on unknown Command options
-  Typo fixes
-  Improve output validation
-  Explicitly filter options that permission-{add,mod} passes to
   aci-{add,mod}

Petr Vobornik (158):

-  error dialog for batch command
-  Uncheck checkboxes in association after deletion
-  Show error in adding associations
-  Validation of details facet before update
   https://fedorahosted.org/freeipa/ticket/1676 The ticket is a
   duplicate of server error, but it revealed few UI errors.
-  Modify serial associator to use batch
-  Modifying sudo options refreshes the whole page
-  Enable update and reset button only if dirty
-  Attributes table not scrollable
-  Fixed: JavaScript type error in entitlement page
-  Fixed inconsistency in enabling delete buttons
-  Code cleanup: widget creation
-  Fixed: Column header for attributes table should be full width
-  Fixed: Enrolment dialog offers to add entity to reflexive
   association.
-  Fixed: Some widgets do not have space for validation error message
-  Disables gid field if not posix group in group adder dialog
-  Fixed links to images in config and migration pages
-  Split Web UI initialization to several smaller calls #2
-  Split Web UI initialization to several smaller calls
-  Added missing fields to password policy page
-  Fixed: Unable to add external user for RunAs User for Sudo rules
-  Circular entity dependency
-  Fixed: Duplicate CSS definitions
-  Fixing infinite loop in UI navigation unit test.
-  Minor visual enhancement of required indicator
-  Page is cleared before it is visible
-  Field for DNS SOA class changed to combobox with options
-  Extending facet's mechanism of gathering changes
-  Added cross browser support of Array.indexOf method
-  Splitting widget into widget and field
-  Splitting basic widgets into visual widgets and fields
-  Improved fields dirty status detection logic
-  Builders and collections for fields and widgets
-  Removing sections as special type of object
-  Added possibility to define facet/dialog specific policies
-  Modifying users to work with new concept
-  Modifying hosts to work with new concept
-  Modifying dns to work with new concept
-  Modifying services to work with new concept
-  Separation of writable update from field load method
-  Modifying ACI to work with new concept
-  Modifying groups to work with new concept
-  Code cleanup of HBAC, Sudo rules
-  Changing definition of basic fields in section from factory to type
-  Modifying automount to work with new concept
-  Fixed unit tests after widget refactoring
-  Removed usage of bitwise assignment operators in logical operations
-  Search facets show translated boolean values
-  Better displaying of long names in tables and facet headers
-  Additional better displaying of long names
-  Reordered facets in ACI
-  Association facets are read only in self service
-  Added facet tabs coloring
-  Fixed displaying of external records in rule association widgets
-  Distinguishing of external values in association tables
-  Better table column width computing
-  Fixed labels in Sudo, HBAC rules
-  Parsing of IPv4 and IPv6 addresses
-  Added support of custom field validators
-  Added validation logic to multivalued text field
-  Added client-side validation of A and AAAA DNS records
-  Fixed IPv6 validation special case: single colon
-  Added support for memberof attribute in permission
-  Added IP address validator to Host and DNS record adder dialog
-  Fixed entity link disabling
-  Fixed content type check in login_password
-  Improved usability of login dialog
-  Removed CSV creation from UI
-  Fixed mask validation in network_validator
-  Fixed checkbox value in table without pkey
-  Certificate serial number in hex format - ui testing data
-  Fixed evaluating checkbox dirty status
-  Better hbactest validation message
-  Content is no more overwritten by error message
-  Show_content on refresh success
-  Fixed rpm build warning - extension.js listed twice
-  Add support of new options in dnsconfig
-  DNS forwarder validator
-  Added mac address to host page
-  Facet expiration flag
-  Inter-facet expiration
-  Reworked netgroup Web UI to allow setting user/host category
-  Fixed: permission attrs table didn't update its available options on
   load
-  Added attrs field to permission for target=subtree
-  DNS forward policy: checkboxes changed to radio buttons
-  Removed mutex option from checkboxes
-  Removal of memberofindirect_permissons from privileges
-  User is notified that password needs to be reset in forms-based login
-  Added permission field to delegation
-  Paging disable for password policies
-  General builder support
-  Action lists
-  Control buttons
-  Redefined details control buttons
-  Redefined search control buttons
-  Hide search facet add/delete buttons in self-service
-  Batch action for search page control buttons
-  General details facet actions
-  Consistent change of entry status.
-  Instructions to generate cert use certutil instead of openssl
-  Host page fixed to work with disabled DNS support
-  Improved calculation of max pkey length in facet header
-  Correction of nested search facets tab labels
-  Refactored action list and control buttons to use shared list of
   actions
-  Refactored entities to use changed actions concept
-  Action panel
-  User password widget modified.
-  Action panel for user
-  Added missing i18n in action list and action panel
-  Add shadow to dialog
-  Enable reset password action according to attribute perrmission
-  Added cancel button to service unprovision dialog
-  Removal of illegal options in JSON-RPC calls
-  Added links to netgroup member tables
-  Text widget's dirty state is changed on various input methods
-  Change json serialization to serialize useful data
-  Removal of illegal options in association dialog
-  Update of serverconfig ipaconfigstring options
-  Action panel for host enrollment
-  Action panel for service provisioning
-  Separate reset password page
-  Added password reset capabilities to unauthorized dialog
-  Set network.http.sendRefererHeader to 2 on browser config
-  Custom Web UI error message for IPA error 911
-  Trust Web UI
-  Same password validator
-  Action panel for certificates
-  Web UI password is going to expire in n days notification
-  Refactored associatin facet to use facet buttons with actions
-  Continuation of removing of not supported command options from Web UI
-  UI for SELinux user mapping
-  Added refresh button for UI
-  Modifying DNS UI to benefit from new DNS API
-  Added paging to DNS record search facet
-  Navigation and redirection to various facets
-  Automember UI
-  Automember UI - default groups
-  Automember UI - Fixed I18n labels
-  Removed question marks from field labels
-  UI support for ssh keys
-  Redirection to PTR records from A,AAAA records
-  Fixed problem when attributes_widget was displaying empty option
-  Added missing configuration options
-  Static metadata update - new DNS options
-  New checkboxes option: Mutual exclusive
-  DNS Zone UI: added new attributes
-  DNS UI: added A,AAAA create reverse options to adder dialog
-  Fixed displaying of A6 Record
-  New UI for DNS global configuration
-  Moved is_empty method from field to IPA object
-  Making validators to return true result if empty
-  Fixed DNS record add handling of 4304 error
-  Added unsupported_validator
-  Fixed redirection in Add and edit in automember hostgroup.
-  Fixed selection of single value in combobox
-  Multiple fields for one attribute
-  Added attrs to permission when target is group or filter
-  Added logout button
-  Forms based authentication UI

Rob Crittenden (191):

-  Add information on setting api.env.host in the ipactl.8 man page
-  Log each command in a batch separately.
-  Do batch logging on successful commands too, not just failures.
-  Fix wording in examples of delegation plugin.
-  Suppress 389-ds debug output when starting services
-  Fix thread deadlock by using pthreads library instead of NSPR.
-  Change the way has_keytab is determined, also check for password.
-  Add additional pam ftp services to HBAC, and a ftp HBAC service group
-  Add label for HBAC services to show as members
-  Add option to only prompt once for passwords, use in entitle_register
-  Retrieve password/keytab state when modifying a host.
-  Disable reverse lookups in ipa-join and ipa-getkeytab
-  Remove more 389-ds files/directories on uninstallation.
-  Remove 389-ds upgrade state during uninstall
-  Set min nvr of pki-ca to 9.0.12 for fix in BZ 700505
-  Add common is_installed() fn, better uninstall logging, check for
   errors.
-  Add external source hosts to HBAC.
-  Roll back changes if client installation fails.
-  Add netgroup as possible memberOf for hostgroups
-  Sort lists so order is predictable and tests pass as expected.
-  Suppress managed netgroups from showing as memberof hostgroups.
-  Use the IPA server cert profile in the installer.
-  Set min nvr of 389-ds-base to 1.2.9.7-1 for BZ 728605
-  Don't allow a OTP to be set on an enrolled host
-  Remove normalizer that made role, privilege and permission names
   lower-case
-  Improved handling for ipa-pki-proxy.conf
-  The precendence on the modrdn plugin was set in the wrong location.
-  Update ipa-ldap-updater man page saying it is not an end-user utility
-  Skip the cert validator if the csr we are passed in is a valid
   filename
-  Change the Requires for the server and server-selinux for proper
   order
-  Suppress managed netgroups as indirect members of hosts.
-  The return value of restorecon is not reliable, ignore it.
-  Normalize uid in user principal to lower-case and do validation
-  Shut down duplicated file handle when HTTP response code is not 200.
-  Don't log one-time password in logs when configuring client.
-  Always require SSL in the Kerberos authorization block.
-  Include failed service and service groups in hbac rule management
-  Add regular expression pattern to host names.
-  Detect CA installation type in ipa-replica-prepare and
   ipa-ca-install.
-  Require current password when using passwd to change your own
   password.
-  Migration: don't assume there is only one naming context, add
   logging.
-  When calculating indirect membership don't test nesting on users and
   hosts.
-  Fix DNS permissions and membership in privileges
-  Fix upgrades of selfsign server
-  Make ipa-join work against an LDAP server that disallows anon binds
-  Fix has_upg() to work with relocated managed entries configuration.
-  Work around limits not being updatable in 389-ds.
-  Save the value of hostname even if it doesn't appear in
   /etc/sysconfig/network
-  Add explicit instructions to ipa-replica-manage for winsync
   replication
-  Set min nvr of 389-ds-base to 1.2.10-0.4.a4 for limits fixes (740942,
   742324)
-  Handle an empty value in a name/value pair in
   config_replace_variables()
-  Update all LDAP configuration files that we can.
-  If our domain is already configured in sssd.conf start with a new
   config.
-  Fix typo in invalid PTR record error message
-  Fix problems in help system
-  Fix nis netgroup config entry so users appear in netgroup triple.
-  Don't allow default objectclass list to be empty.
-  Remove calls to has_managed_entries()
-  Fix copy/paste error in parameter description.
-  Add Ondrej Hamada to Contributors.txt
-  Don't check for 389-instances.
-  Clarify usage of --posix argument in group plugin.
-  Add plugin framework to LDAP updates.
-  Fix some issues introduced when rebasing update patch
-  Remove extraneous trailing single quote in nis.uldif
-  Mark some attributes required to match the schema.
-  Use absolute paths when trying to find certmonger request id.
-  Reorder privileges so that memberof for permissions are generated
   properly
-  Add SELinux user mapping framework.
-  Require an HTTP Referer header in the server. Send one in ipa tools.
-  Display the value of memberOf ACIs in permission plugin.
-  Fix two typos in role help.
-  Configure s4u2proxy during installation.
-  Document the ping plugin.
-  Catch exception when trying to list missing managed entries
   definitions
-  Fix some typos in automember help and paramters.
-  Add labels so HBAC and Sudo rules show under hosts/hostgroups.
-  Use correct template variable for hosts, FQDN.
-  In sudo when the category is all do not allow members, and vice
   versa.
-  Update and package ipa-upgradeconfig man page.
-  Fix deletion of HBAC Rules when there are SELinux user maps defined
-  Add support for storing MAC address in host entries.
-  Don't try to bind on TLS failure
-  Check for the existence of a replication agreement before deleting
   it.
-  %ghost the UI files that we install/create on the fly
-  Make submount automount maps work.
-  Require minimum SSF 56, confidentially. Also ensure minssf <= maxssf.
-  Consolidate external member code into two functions in baseldap.py
-  Make ipaconfigstring modifiable by users.
-  Don't use sets when calculating the modlist so order is preserved.
-  Add update files for SELinuxUserMap
-  Add update file for new schema in v2.2/3.0
-  Stop and uninstall ipa_kpasswd on upgrade, fix dbmodules in krb5.conf
-  Don't set delegation flag in client, we're using S4U2Proxy now
-  Update S4U2proxy delegation list when creating replicas
-  Correct update syntax in 30-s4u2proxy.update
-  Remove Apache ccache on upgrade.
-  Add S4U2Proxy delegation permissions on upgrades
-  Disable false pylint error in freeipa-systemd-upgrade
-  Enable ipa_memcached when upgrading
-  Configure ipa_memcached when a replica is installed.
-  Use FQDN in place of FQHN for consistency in sub_dict.
-  Set min for 389-ds-base to 1.2.10.1-1 to fix install segfault, schema
   replication.
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
-  Set minimum version of selinux-policy to pick up memcached fix
-  Fix nsslapd-anonlimitsdn dn in cn=config
-  Set SELinux boolean httpd_manage_ipa so ipa_memcached will work.
-  Don't set dbdir in the connection until after the connection is
   created.
-  Display serial number as HEX (DECIMAL) when showing certificates.
-  Add subject key identifier to the dogtag server cert profile.
-  Configure a basic ldap.conf for OpenLDAP in /etc/openldap/ldap.conf
-  Import the ipaserver plugins based on context, not env.in_server.
-  Don't allow hosts and services of IPA masters to be disabled.
-  Use a consistent parameter name in errors, defaulting to cli_name.
-  No longer shell escape the DM password when calling pkisilent.
-  Fix test failure testing rename with an invalid hostname.
-  Fix attributes that contain DNs when migrating.
-  Normalize the primary key value to lowercase during migration.
-  Fix unit tests to work with new comma-support, validation
   requirements
-  Set minimum version of 389-ds-base to 1.2.10.4-2 to fix upgrade issue
-  Set nsslapd-minssf-exclude-rootdse to on so the DSE is always
   available.
-  Add requires on python-krbV to client subpackage
-  Fix failure count interval attribute name in query for password
   policy.
-  Handle updating replication agreements that lack
   nsDS5ReplicatedAttributeList
-  Don't create private groups for migrated users, check for valid
   gidnumber
-  Add updated Output format for batch to API.txt
-  Make revocation_reason required when revoking a certificate.
-  Add missing comma to list of services that cannot be disabled.
-  Return consistent value when hostcat and usercat is all.
-  Dereference pointer when comparing password history in qsort compare.
-  Configure certmonger to execute restart scripts on renewal.
-  Remove the running state when uninstalling DS instances.
-  Return consistent expiration message for forms-based login
-  Use mixed-case for Read DNS Entries permission
-  Update docs for user-status, always show disabled, time for each
   server.
-  Revert "Search allowed attributes in superior objectclasses"
-  Revert "Validate attributes in permission-add"
-  Return LDAP_SUCCESS on mods on a referral entry.
-  Fix overlapping cn param/option issue, pass cn as aciname in find
-  Implement permission/aci find by subtree
-  Include more information when IP address is not local during
   installation.
-  Validate on the user-provided domain name in the installer.
-  During replication installation see if an agreement already exists.
-  Check for locked-out user before incrementing lastfail.
-  Retry retrieving ldap principals when setting up replication.
-  Normalize uid to lower case in winsync.
-  Enforce sizelimit in permission-find, post_callback returns truncated
-  If SELinux is enabled ensure we also have restorecon.
-  Store session cookie in ccache for cli users
-  Add flag to ipa-client-install to managed order of ipa_server in sssd
-  Increase LimitRequestFieldSize in Apache config to support a 64KiB
   PAC
-  Add logging to ipa-upgradeconfig
-  Configure automount using autofs or sssd.
-  Defer adding ipa-cifs-delegation-targets until the Updates phase.
-  Add missing option to range_add in API.txt
-  Fix compatibility with Fedora 18.
-  Become IPA v3 beta 1 (3.0.0.pre1)

Simo Sorce (104):

-  Set VERSION to 2.99.0 on the 3.0 development branch
-  Fix build warnings
-  ipa-pwd_extop: use endian.h instead of nih function
-  krbinstance: use helper function to get realm suffix
-  ipa-pwd-extop: Remove unused variables and code to set them
-  ipa-pwd-extop: do not append mkvno to krbExtraData
-  ipa-pwd-extop: Use the proper mkvno number in keys
-  ipa-pwd-extop: re-indent code using old style
-  ipa-pwd-extop: Use common krb5 structs from kdb.h
-  ipa-pwd-extop: Move encryption of keys in common
-  ipa-pwd-extop: Move encoding in common too
-  ipa-pwd-extop: make encsalt parsing function common
-  ipa-kdb: Initial plugin skeleton
-  ipa-kdb: add exports file
-  ipa-kdb: initialize module functions
-  ipa-kdb: implement get_time function
-  ipa-kdb: add common utility ldap wrapper functions
-  ipa-kdb: functions to get principal
-  ipa-kdb: add function to free principals
-  ipa-kdb: add functions to delete principals
-  ipa-kdb: add function to iterate over principals
-  ipa-kdb: add functions to change principals
-  ipa-kdb: Get/Store Master Key directly from LDAP
-  ipa-kdb: implement function to retrieve password policies
-  ipa-kdb: implement change_pwd function
-  util: add password policy manipulation functions
-  ipa-pwd-extop: Use common password policy code
-  ipa-kdb: add password policy support
-  ipa-pwd-extop: Allow kadmin to set krb keys
-  ipa-kdb: Change install to use the new ipa-kdb kdc backend
-  install: Remove uid=kdc user
-  ipa-kdb: Be flexible
-  install: Use proper case for boolean values
-  daemons: Remove ipa_kpasswd
-  schema: Split ipadns definitions from basev2 ones
-  v3-schema: Add new ipaExternalGroup objectclass
-  install: We do not need a ldap password anymore
-  install: We do not need a kpasswd keytab anymore
-  conncheck: Fix List of ports to check
-  ipa-kdb: Properly set password expiration time.
-  schema: Add new attributes and objectclasses for AD Trusts
-  conncheck: Additional check to verify the admin password is ok
-  ipa-pwd-extop: Fix segfault in password change.
-  ipa-pwd-extop: Enforce old password checks
-  ipa-kdb: Fix expiration time calculation
-  ipa-client-install: Fix joining when LDAP access is restricted
-  replica-prepare: anonymous binds may be disallowed
-  ipa-kdb: Fix legacy password hashes generation
-  updates: Change default limits on ldap searches
-  ipa-kdb: Fix memory leak
-  Modify random salt creation for interoperability
-  Amend #2038 fix
-  Fix CID 10742: Unchecked return value
-  Fix CID 10743: Unchecked return value
-  Fix CID 10745: Unchecked return value
-  Fix CID 11019: Resource leak
-  Fix CID 11020: Resource leak
-  Fix CID 11021: Resource leak
-  Fix CID 11022: Resource leak
-  Fix CID 11023: Resource leak
-  Fix CID 11024: Resource leak
-  Fix CID 11025: Resource leak
-  Fix CID 11026: Resource leak
-  Fix CID 11027: Wrong sizeof argument
-  Add support for generating PAC for AS requests for user principals
-  MS-PAC: Add support for verifying PAC in TGS requests
-  Add missing copyright header
-  Add NT domain GUID attribute.
-  Create skeleton CLDAP server as a DS plugin
-  ipa-cldap: Implement worker thread.
-  ipa-cldap: Decode CLDAP request.
-  ipa-cldap: Create netlogon blob
-  ipa-cldap: send cldap reply
-  ipa-kdb: Support re-signing PAC with different checksum
-  spec: We do not need krb5-server-ldap anymore
-  ipa-kdb: fix free() of uninitialized var
-  ipa-kdb: Remove unused CFLAGS/LIBS from Makefiles
-  ipa-kdb: fix memleaks in ipa_kdb_mspac.c
-  ipa-kdb: Fix copy and paste typo
-  ipa-kdb: Delegation ACL schema
-  ipa-kdb: enhance deref searches
-  ipa-kdb: Add delgation access control support
-  ipa-kdb: return properly when no PAC is available
-  ipa-cldap: Support clients asking for default domain
-  ipa-kdb: Verify the correct checksum in PAC validation
-  ipa-kdb: Create PAC's KDC checksum with right key
-  Fix replication setup
-  slapi-plugins: use thread-safe ldap library
-  ipa-kdb: add AS auditing support
-  ipa-kdb: Avoid lookup on modify if possible
-  ipa-kdb: set krblastpwdchange only when keys have been effectively
   changed
-  Remove compat defines
-  Require krb5 1.10
-  ipa-kdb: Fix ACL evaluator
-  policy: add function to check lockout policy
-  ipa-kdb: fix delegation acl check
-  Fix ticket checks when using either s4u2proxy or a delegated krbtgt
-  Fix memleak and silence Coverity defects
-  Fix MS-PAC checks when using s4u2proxy
-  Fix theoretical leak discovered by coverity
-  Fix migration code password setting.
-  Fix setting domain_sid
-  ipa-kdb: Add MS-PAC on constrained delegation.
-  Add support for disabling KDC writes

Sumit Bose (32):

-  Call standard_logging_setup() before any logging is done
-  Add ipa-adtrust-install utility
-  Fix ACIs in ipa-adtrust-install
-  Update samba LDAP schema
-  Fix typo in v3 base schema
-  Add admin SIDs
-  ipa-pwd-extop: allow password change on all connections with SSF>1
-  Add DNS service records for Windows
-  Add DNS service records for Windows
-  Move our own domain info into cn=etc
-  Add trust objectclass and attributes to v3 schema
-  Use new objectclasses and attributes for trust
-  Fix some pylint warnings
-  Add ipasam samba passdb backend
-  activate CLDAP
-  Make pwd-extop aware of new ipaNTHash attribute
-  Add a second module init call for newer samba versions
-  Use exop instead of kadmin.local
-  ipasam: remove unused struct elements
-  Move some krb5 keys related functions from ipa-client to util
-  Add sidgen postop and task
-  Filter groups in the PAC
-  Add configure check for C Unit-Test framework check
-  Add external domain extop DS plugin
-  Use lower case names in LDAP to meet freeIPA convention
-  Extend LDAP schema
-  Add objects for initial ID range
-  Set RID bases for local domain during ipa-adtrust-install
-  Add CLI for ID ranges
-  Add range check preop plugin
-  Use DN objects instead of strings in adtrustinstance
-  Set samba_portmapper SELinux boolean during ipa-adtrust-install

Yuri Chornoivan (1):

-  Fix typos