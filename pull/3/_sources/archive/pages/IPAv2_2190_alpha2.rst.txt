\__NOTOC_\_ The FreeIPA team is proud to announce version 2.1.90 alpha
2. This will eventually become FreeIPA v2.2.0.

It can be downloaded from http://www.freeipa.org/Downloads or from our
development repo (http://freeipa.org/downloads/freeipa-devel.repo).
Fedora 15, 16 and 17 builds are available.

For Fedora 17 users the the required version of 389-ds-base has not been
pushed to updates-testing yet. You can retrieve it manually from
http://koji.fedoraproject.org/koji/buildinfo?buildID=299543 or download
the packages with:

``# koji download-build 299543``

Alpha 1 was an unannounced release that formed the basis of the first
Fedora 17 package. It was not well-tested, particularly for upgrades,
which is why it wasn't announced at the time. It was released to meet
Fedora 17 package deadlines.

.. _highlights_in_2.1.90_alpha_2:

Highlights in 2.1.90 alpha 2
----------------------------

-  A new KDC LDAP backend, ipa-kdb. This simplifies our set up code and
   will is a big piece of future MS PAC support. It also removes the
   need for the separate ipa_kpasswd daemon, kadmind is used instead.
-  Support for storing SSH user and host public keys.
-  SELinux user map rules. These let you set the SELinux context for
   users in an HBAC rule.
-  Improved DNS UI and command-line with vastly improved argument
   handling.
-  UI screens for Automember were added.
-  Session support in the Web UI. This removes the need to do Kerberos
   negotiation with every request significantly improving Web UI
   performance.
-  Support for S4U2Proxy. This is a Kerberos feature which allows a
   delegated service (HTTP in our case) to request a ticket (ldap) on a
   user's behalf. We no longer require the TGT to be delegated to the
   server. A forwardable TGT is still required.
-  Improved command-line performance. It is approximately 50% faster.
-  MAC address has been added to hosts.

Upgrading
---------

We tested upgrades from 2.1.4 successfully but this is alpha code. We do
not recommend upgrading a production server.

Installing updated rpms is all that is required to upgrade from 2.1.4.

It is unlikely that downgrading to a previous release once 2.1.90 is
installed will work.

Feedback
--------

Please provide comments, bugs and other feedback via the freeipa-devel
mailing list: http://www.redhat.com/mailman/listinfo/freeipa-devel

.. _detailed_changelog_since_2.1.4:

Detailed Changelog since 2.1.4
------------------------------

Adam Young (4):

-  remove enrolled column
-  Add priority to pwpolicy list
-  Remove delegation from browser config
-  ignore generated services file.

Alexander Bokovoy (14):

-  Re-enable web password migration on Fedora 16 after SE Linux policy
   restrictions
-  Check for Python.h during build of py_default_encoding extension
-  Add configure check for libintl.h
-  Create directories for client install
-  Add "Extending FreeIPA" developer guide
-  Small fix to the guide CSS: enable vertical scroll bar
-  Rename included snippets to avoid problems with pylint
-  Fix dependency for samba4-devel package
-  Check through all LDAP servers in the domain during IPA discovery
-  Validate sudo RunAsUser/RunAsGroup arguments
-  Allow hbactest to work with HBAC rules exceeding default IPA limits
-  Add management of inifiles to allow manipulation of systemd units
-  Handle upgrade issues with systemd in Fedora 16 and above
-  Adopt to python-ldap 2.4.6 by removing unused references which are
   not available in python-ldap anymore

Endi S. Dewata (60):

-  Updated DNS zone details page.
-  Replaced description text fields with text areas.
-  Use editable combobox for service type.
-  Added confirmation when adding multiple entries.
-  Added selectable labels for radio buttons.
-  Fixed dependency problem in UI test.
-  Fixed inconsistent required/optional attributes.
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
-  Fixed problem changing page in association facet.
-  Updated sample data.
-  Added paging on search facet.
-  Refactored permission target section.
-  Removed develop.js.
-  Added commands into metadata.
-  Removed HBAC rule type.
-  Removed HBAC deny rule warning.
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

JR Aquino (1):

-  Replication: Adjust replica installation to omit processing memberof
   computations

Jan Cholasta (15):

-  Finalize plugin initialization on demand.
-  Don't leak passwords through kdb5_ldap_util command line arguments.
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

John Dennis (10):

-  If "make rpms" fails so will the next make
-  Remove old RPMROOT contents before it is used for rpmbuild
-  update i18n pot file for branch ipa-2-1
-  Add log manager module
-  modify codebase to utilize IPALogManager, obsoletes logging
-  IPAdmin undefined anonymous parameter lists
-  subclass SimpleLDAPObject
-  Restore default log level in server to INFO
-  Add ipa_memcached service
-  add session manager and cache krb auth

Marko Myllynen (1):

-  include <stdint.h> for uintptr_t

Martin Kosek (52):

-  Add connection failure recovery to IPAdmin
-  Make sure that install tools log
-  Add --zonemgr/--admin-mail validator
-  Create pkey-only option for find commands
-  Allow custom server backend encoding
-  Fix DNS zone --allow-dynupdate option behavior
-  Improve DNS record data validation
-  Polish ipa config help
-  Hosts file not updated when IP is passed as option
-  Fix API.txt
-  Fix LDAP object parameter encoding
-  Remove redundant information from API.txt
-  Fix coverity issues in client CLI tools
-  Make ipa-server-install clean after itself
-  Add --delattr option to complement --setattr/--addattr
-  Improve zonemgr validator and normalizer
-  Change default DNS zone manager to hostmaster
-  Fix config migration option
-  Ask for user confirmation in ipa-server-install
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

Ondrej Hamada (9):

-  Misleading Keytab field
-  Sort password policy by priority
-  Client install checks for nss_ldap
-  User-add random password support
-  HBAC test optional sourcehost option
-  localhost.localdomain clients refused to join
-  Leave nsds5replicaupdateschedule parameter unset
-  Fix 'no-reverse' option description
-  Memberof attribute control and update

Petr Viktorin (5):

-  Switch --group and --membergroup in example for delegation
-  Fix/add options in ipa-managed-entries man page
-  Honor default home directory and login shell in user_add
-  Clean up i18n strings
-  Internationalization for HBAC and ipalib.output

Petr Voborník (55):

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

Rob Crittenden (54):

-  Use absolute paths when trying to find certmonger request id.
-  Reorder privileges so that memberof for permissions are generated
   properly.
-  Fix some pylint issues found in F-16
-  Fix two typos in role help.
-  Move ONLY_CLIENT in spec so services.py always gets generated in
   %install
-  Remove calls to has_managed_entries()
-  Fix copy/paste error in parameter description.
-  Add Ondrej Hamada to Contributors.txt
-  Don't check for 389-instances.
-  Clarify usage of --posix argument in group plugin.
-  Add plugin framework to LDAP updates.
-  Fix some issues introduced when rebasing update patch
-  Mark some attributes required to match the schema.
-  Add SELinux user mapping framework.
-  Display the value of memberOf ACIs in permission plugin.
-  Set minimum version of 389-ds to 1.2.10-0.5.a5
-  Fix typos in in 60basev3.ldif
-  Remove include for errno.h that was specific to 2.1 branch
-  Remove ipa_get_random_salt() from ipapwd_encoding.c
-  update i18n pot file for branch ipa-2-2
-  Remove buffer log handling.
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

Simo Sorce (77):

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
-  ipa-kdb: Properly set password expiration time.
-  conncheck: Additional check to verify the admin password is ok
-  ipa-kdb: Fix expiration time calculation
-  ipa-kdb: Fix legacy password hashes generation
-  ipa-kdb: Fix memory leak
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
-  Modify random salt creation for interoperability
-  Amend #2038 fix
-  Add missing copyright header
-  ipa-kdb: Support re-signing PAC with different checksum
-  spec: We do not need krb5-server-ldap anymore
-  ipa-kdb: fix free() of uninitialized var
-  ipa-kdb: Remove unused CFLAGS/LIBS from Makefiles
-  ipa-kdb: fix memleaks in ipa_kdb_mspac.c
-  ipa-kdb: Fix copy and paste typo
-  ipa-kdb: enhance deref searches
-  ipa-kdb: Add delgation access control support
-  ipa-kdb: return properly when no PAC is available
-  ipa-kdb: Verify the correct checksum in PAC validation
-  ipa-kdb: Create PAC's KDC checksum with right key
-  Disable MS-PAC handling in 2.2
-  Fix replication setup
-  slapi-plugins: use thread-safe ldap library
-  ipa-kdb: add AS auditing support
-  ipa-kdb: Avoid lookup on modify if possible
-  ipa-kdb: set krblastpwdchange only when keys have been effectively
   changed
