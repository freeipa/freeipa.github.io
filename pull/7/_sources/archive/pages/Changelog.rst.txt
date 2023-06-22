\__NOTOC_\_

This isn't every single check-in between versions but will hopefully
will provide the highlights of the changes.

.. _version_3.3.3_11012013:

Version 3.3.3 (11/01/2013)
==========================

.. _ana_krivokapic_4:

Ana Krivokapic (4):
^^^^^^^^^^^^^^^^^^^

-  Add ipa-advise plugins for nss-pam-ldapd legacy clients
-  Do not roll back failed client installation on server
-  Make sure nsds5ReplicaStripAttrs is set on agreements
-  Add test for external CA installation

.. _jakub_hrozek_1:

Jakub Hrozek (1):
^^^^^^^^^^^^^^^^^

-  trusts: combine filters with AND to make sure only the intended
   domain matches

.. _jan_cholasta_1:

Jan Cholasta (1):
^^^^^^^^^^^^^^^^^

-  Track DS certificate with certmonger on replicas.

.. _martin_kosek_14:

Martin Kosek (14):
^^^^^^^^^^^^^^^^^^

-  Do not allow '%' in DM password
-  Remove --no-serial-autoincrement
-  PKI installation on replica failing due to missing proxy conf
-  Use consistent realm name in cainstance and dsinstance
-  Winsync re-initialize should not run memberOf fixup task
-  Installer should always wait until CA starts up
-  Administrative password change does not respect password policy
-  Do not add kadmin/changepw ACIs on new installs
-  Make set_directive and get_directive more strict
-  Remove mod_ssl conflict
-  Add nsswitch.conf to FILES section of ipa-client-install man page
-  Remove ipa-pwd-extop and ipa-enrollment duplicate error strings
-  Remove deprecated AllowLMhash config
-  Become IPA 3.3.3

.. _petr_viktorin_6:

Petr Viktorin (6):
^^^^^^^^^^^^^^^^^^

-  test_caless.TestCertInstall: Fix 'test_no_ds_password' test case
-  Use new CLI options in certinstall tests
-  test_simple_replication: Fix waiting for replication
-  freeipa.spec: Fix changelog dates
-  Tests: mkdir_recursive: Don't fail when top-level directory doesn't
   exist
-  beakerlib plugin: Don't try to submit logs if they are missing

.. _petr_vobornik_1:

Petr Vobornik (1):
^^^^^^^^^^^^^^^^^^

-  Fix password expiration notification

.. _sumit_bose_3:

Sumit Bose (3):
^^^^^^^^^^^^^^^

-  Use the right attribute with ipapwd_entry_checks for MagicRegen
-  Remove AllowLMhash from the allowed IPA config strings
-  Remove generation and handling of LM hashes

.. _tomas_babej_23:

Tomas Babej (23):
^^^^^^^^^^^^^^^^^

-  trusts: Do not create ranges for subdomains in case of POSIX trust
-  ipa-upgradeconfig: Remove backed up smb.conf
-  ipa-adtrust-install: Add warning that we will break existing samba
   configuration
-  adtrustinstance: Properly handle uninstall of AD trust instance
-  adtrustinstance: Move attribute definitions from setup to init method
-  ipatests: Extend the order plugin to properly handle inheritance
-  Get the created range type in case of re-establishing trust
-  ipatests: Add Active Directory support to configuration
-  ipatests: Extend domain object with 'ad' role support and WinHosts
-  ipatests: Extend IntegrationTest with multiple AD domain support
-  ipatests: Create util module for ipatests
-  ipatests: Add WinHost class
-  ipatests: Add AD-integration related tasks
-  ipatests: Add AD integration test case
-  trusts: Fix typo in error message for realm-domain mismatch
-  advice: Add legacy client configuration script using nss-ldap
-  ipatests: Extend clear_sssd_cache to support non-systemd platforms
-  ipatests: Restore SELinux context after restoring files from backup
-  ipatests: Do not use /usr/bin hardcoded paths
-  ipatests: Add support for extra roles referenced by a keyword
-  ipatests: Use command -v instead of which in legacy client advice
-  ipatests: Add integration tests for legacy clients
-  ipatests: test_trust: use domain name instead of realm for user
   lookups

.. _version_3.3.2_10042013:

Version 3.3.2 (10/04/2013)
==========================

.. _alexander_bokovoy_11:

Alexander Bokovoy (11):
^^^^^^^^^^^^^^^^^^^^^^^

-  ipa-sam: do not modify objectclass when trust object already created
-  ipa-sam: do not leak LDAPMessage on ipa-sam initialization
-  ipa-sam: report supported enctypes based on Kerberos realm
   configuration
-  ipaserver/dcerpc.py: populate forest trust information using
   realmdomains
-  trusts: support subdomains in a forest
-  frontend: report arguments errors with better detail
-  ipaserver/dcerpc: remove use of trust account authentication
-  trust: integrate subdomains support into trust-add
-  ipasam: for subdomains pick up defaults for missing values
-  KDC: implement transition check for trusted domains
-  ipa-kdb: Handle parent-child relationship for subdomains

.. _ana_krivokapic_5:

Ana Krivokapic (5):
^^^^^^^^^^^^^^^^^^^

-  Add integration tests for forced client re-enrollment
-  Create DS user and group during ipa-restore
-  Add warning when uninstalling active replica
-  Do not crash if DS is down during server uninstall
-  Follow tmpfiles.d packaging guidelines

.. _jan_cholasta_3:

Jan Cholasta (3):
^^^^^^^^^^^^^^^^^

-  Fix nsslapdPlugin object class after initial replication.
-  Read passwords from stdin when importing PKCS#12 files with pk12util.
-  Allow PKCS#12 files with empty password in install tools.

.. _martin_kosek_5:

Martin Kosek (5):
^^^^^^^^^^^^^^^^^

-  Use FQDN when creating MSDCS SRV records
-  Do not set DNS discovery domain in server mode
-  Require new SSSD to pull required AD subdomain fixes
-  Remove faulty DNS memberOf Task
-  Become IPA 3.3.2

.. _nathaniel_mccallum_1:

Nathaniel McCallum (1):
^^^^^^^^^^^^^^^^^^^^^^^

-  Ensure credentials structure is initialized

.. _petr_spacek_1:

Petr Spacek (1):
^^^^^^^^^^^^^^^^

-  Add timestamps to named debug logs in /var/named/data/named.run

.. _petr_viktorin_15:

Petr Viktorin (15):
^^^^^^^^^^^^^^^^^^^

-  Remove \__all_\_ specifications in ipaclient and ipaserver.install
-  Make make-lint compatible with Pylint 1.0
-  test_integration.host: Move transport-related functionality to a new
   module
-  test_integration: Add OpenSSHTransport, used if paramiko is not
   available
-  ipatests.test_integration.test_caless: Fix mkdir_recursive call
-  ipatests.beakerlib_plugin: Warn instead of failing when some logs are
   missing
-  ipatests.order_plugin: Exclude test generators from the order
-  ipatests.beakerlib_plugin: Add argument of generated tests to test
   captions
-  ipatests.test_cmdline.test_help: Re-raise unexpected exceptions on
   failure
-  Add tests for installing with empty PKCS#12 password
-  Update translations from Transifex
-  ipa-client-install: Use direct RPC instead of api.Command
-  ipa-client-install: Verify RPC connection with a ping
-  Do not fail upgrade if the global anonymous read ACI is not found
-  ipapython.nsslib: Name arguments to NSPRError

.. _petr_vobornik_5:

Petr Vobornik (5):
^^^^^^^^^^^^^^^^^^

-  Fix RUV search scope in ipa-replica-manage
-  Fix redirection on deletion of last dns record entry
-  Allow edit of ipakrbokasdelegate in Web UI when attrlevelrights are
   unknown
-  Fix enablement of automount map type selector
-  ipatests.test_integration.host: Add logging to ldap_connect()

.. _simo_sorce_1:

Simo Sorce (1):
^^^^^^^^^^^^^^^

-  Add Delegation Info to MS-PAC

.. _sumit_bose_1:

Sumit Bose (1):
^^^^^^^^^^^^^^^

-  CLDAP: do not read IPA domain from hostname

.. _tomas_babej_3:

Tomas Babej (3):
^^^^^^^^^^^^^^^^

-  Use getent admin@domain for nss check in ipa-client-install
-  Do not add trust to AD in case of IPA realm-domain mismatch
-  Warn user about realm-domain mismatch in install scripts

.. _version_3.3.1_08292013:

Version 3.3.1 (08/29/2013)
==========================

.. _alexander_bokovoy_1:

Alexander Bokovoy (1):
^^^^^^^^^^^^^^^^^^^^^^

-  Remove systemd upgrader as it is not used anymore

.. _ana_krivokapic_4_1:

Ana Krivokapic (4):
^^^^^^^^^^^^^^^^^^^

-  Handle --subject option in ipa-server-install
-  Fix broken replica installation
-  Add integration tests for Kerberos Flags
-  Fix tests which fail after ipa-adtrust-install

.. _jakub_hrozek_1_1:

Jakub Hrozek (1):
^^^^^^^^^^^^^^^^^

-  EXTDOM: Do not overwrite domain_name for INP_SID

.. _jan_cholasta_12:

Jan Cholasta (12):
^^^^^^^^^^^^^^^^^^

-  Make PKCS#12 handling in ipa-server-certinstall closer to what other
   tools do.
-  Port ipa-server-certinstall to the admintool framework.
-  Remove unused NSSDatabase and CertDB method
   find_root_cert_from_pkcs12.
-  Ignore empty mod error when updating DS SSL config in
   ipa-server-certinstall.
-  Replace only the cert instead of the whole NSS DB in
   ipa-server-certinstall.
-  Untrack old and track new cert with certmonger in
   ipa-server-certinstall.
-  Add --pin option to ipa-server-certinstall.
-  Ask for PKCS#12 password interactively in ipa-server-certinstall.
-  Fix nsSaslMapping object class before configuring SASL mappings.
-  Add --dirman-password option to ipa-server-certinstall.
-  Fix ipa-server-certinstall usage string.
-  Fix service-disable in CA-less install.

.. _martin_kosek_3:

Martin Kosek (3):
^^^^^^^^^^^^^^^^^

-  Prevent \*.pyo and \*.pyc multilib problems
-  Remove rpmlint warnings in spec file
-  Fix selected minor issues in the spec file and license

.. _nathaniel_mccallum_1_1:

Nathaniel McCallum (1):
^^^^^^^^^^^^^^^^^^^^^^^

-  Bypass ipa-replica-conncheck ssh tests when ssh is not installed

.. _petr_viktorin_4:

Petr Viktorin (4):
^^^^^^^^^^^^^^^^^^

-  Allow freeipa-tests to work with older paramiko versions
-  Add missing license header to ipa-test-config
-  Add CA-less install tests
-  Add man pages for testing tools

.. _petr_vobornik_7:

Petr Vobornik (7):
^^^^^^^^^^^^^^^^^^

-  Removal of deprecated selenium tests
-  Add base-id, range-size and range-type options to trust-add dialog
-  Hide 'New Certificate' action on CA-less install
-  Web UI integration tests: CA-less
-  Web UI Integration tests: Kerberos Flags
-  Web UI integration tests: ID range types
-  Update idrange search facet after trust creation

.. _rob_crittenden_1:

Rob Crittenden (1):
^^^^^^^^^^^^^^^^^^^

-  Re-order NULL check in ipa_lockout.

.. _simo_sorce_3:

Simo Sorce (3):
^^^^^^^^^^^^^^^

-  pwd-plugin: Fix ignored return error
-  kdb-mspac: Fix out of bounds memset
-  kdb-princ: Fix memory leak

.. _sumit_bose_1_1:

Sumit Bose (1):
^^^^^^^^^^^^^^^

-  CLDAP: make sure an empty reply is returned on any error

.. _tomas_babej_6:

Tomas Babej (6):
^^^^^^^^^^^^^^^^

-  Remove support for IPA deployments with no persistent search
-  Remove redundant shebangs
-  Perform dirsrv tuning at platform level
-  Make CS.cfg edits with CA instance stopped
-  Fix incorrect error message occurence when re-adding the trust
-  Log proper error message when defaultNamingContext not found

.. _version_3.3.0_08082013:

Version 3.3.0 (08/08/2013)
==========================

.. _alexander_bokovoy_9:

Alexander Bokovoy (9):
^^^^^^^^^^^^^^^^^^^^^^

-  Fix cldap parser to work with a single equality filter (NtVer=...)
-  Make sure domain_name is also set when processing INP_NAME requests
-  Fix extdom plugin to provide unqualified name in response as sssd
   expects
-  Generate syntethic MS-PAC for all services running on IPA master
-  ipa-adtrust-install: configure compatibility tree to serve trusted
   domain users
-  ipa-kdb: cache KDC hostname on startup
-  ipa-kdb: reinit mspac on HTTP TGT acquisition to aid trust-add case
-  ipaserver/dcerpc: attempt to resolve SIDs through SSSD first
-  Rename slapi-nis configuration variable

.. _ana_krivokapic_26:

Ana Krivokapic (26):
^^^^^^^^^^^^^^^^^^^^

-  Prompt for nameserver IP address in dnszone-add
-  Do not display success message on failure in web UI
-  Ignore files generated by build
-  Deprecate options --dom-sid and --dom-name in idrange-mod
-  Prevent error when running IPA commands with su/sudo
-  Fix displaying of success message
-  Fix location of service.crt in .gitignore
-  Improve handling of options in ipa-client-install
-  Fail when adding a trust with a different range
-  Do not display traceback to user
-  Require rid-base and secondary-rid-base in idrange-add after
   ipa-adtrust-install
-  Fix bug in adtrustinstance
-  Use correct DS instance in ipactl status
-  Avoid systemd service deadlock during shutdown
-  Make sure replication works after DM password is changed
-  Use --ignore-dependencies only when necessary
-  Properly handle non-existent cert files
-  Add 'ipa_server_mode' option to SSSD configuration
-  Bump version of sssd in spec file
-  Use admin@REALM when testing if SSSD is ready
-  Fix internal error in idrange-add
-  Honor 'enabled' option for widgets.
-  Expose ipaRangeType in Web UI
-  Add ipa-advise plugins for legacy clients
-  Enable running API commands in ipa-advise plugins
-  Add new command compat-is-enabled

.. _diane_trout_1:

Diane Trout (1):
^^^^^^^^^^^^^^^^

-  Fix log format not a string literal.

.. _jakub_hrozek_3:

Jakub Hrozek (3):
^^^^^^^^^^^^^^^^^

-  Remove unused variable
-  IPA KDB MS-PAC: return ENOMEM if allocation fails
-  IPA KDB MS-PAC: remove unused variable

.. _jan_cholasta_21:

Jan Cholasta (21):
^^^^^^^^^^^^^^^^^^

-  Use the correct PKCS#12 file for HTTP server.
-  Remove stray error condition in ipa-server-install.
-  Handle exceptions gracefully when verifying PKCS#12 files.
-  Skip empty lines when parsing pk12util output.
-  Do not allow installing CA replicas in CA-less setup.
-  Do not track DS certificate in CA-less setup.
-  Fix CA-less check in ipa-replica-install and ipa-ca-install.
-  Do not skip SSSD known hosts in ipa-client-install --ssh-trust-dns.
-  Enable SASL mapping fallback.
-  Skip cert issuer validation in service and host commands in CA-less
   install.
-  Check trust chain length in CA-less install.
-  Use LDAP search instead of \*group_show to check if a group exists.
-  Use LDAP search instead of \*group_show to check for a group
   objectclass.
-  Use LDAP modify operation directly to add/remove group members.
-  Add missing substring indices for attributes managed by the referint
   plugin.
-  Add missing equality index for ipaUniqueId.
-  Run gpg-agent explicitly when encrypting/decrypting files.
-  Add new hidden command option to suppress processing of membership
   attributes.
-  Ask for PKCS#12 password interactively in ipa-server-install.
-  Ask for PKCS#12 password interactively in ipa-replica-prepare.
-  Print newline after receiving EOF in installutils.read_password.

.. _lukas_slebodnik_4:

Lukas Slebodnik (4):
^^^^^^^^^^^^^^^^^^^^

-  Use pkg-config to detect cmocka
-  Use right function prototype for thread function
-  Remove unused variable
-  Remove unused variable

.. _martin_kosek_17:

Martin Kosek (17):
^^^^^^^^^^^^^^^^^^

-  Set KRB5CCNAME so that dirsrv can work with newer krb5-server
-  Handle DIR type CCACHEs in test_cmdline properly
-  Avoid exporting KRB5_KTNAME in dirsrv env
-  Remove redundant u'' character
-  Drop SELinux subpackage
-  Drop redundant directory /var/cache/ipa/sessions
-  Remove entitlement support
-  Run server upgrade and restart in posttrans
-  Require new selinux-policy replacing old server-selinux subpackage
-  Bump minimum SSSD version
-  Become 3.3.0 Beta 1
-  Free NSS objects in --external-ca scenario
-  Use valid LDAP search base in migration plugin
-  Increase default SASL buffer size
-  Become 3.3.0 Beta 2
-  Add requires for slapi-nis and SSSD
-  Become 3.3.0

.. _nathaniel_mccallum_10:

Nathaniel McCallum (10):
^^^^^^^^^^^^^^^^^^^^^^^^

-  Add ipaUserAuthType and ipaUserAuthTypeClass
-  Add IPA OTP schema and ACLs
-  ipa-kdb: Add OTP support
-  Add the krb5/FreeIPA RADIUS companion daemon
-  Remove unnecessary prefixes from ipa-pwd-extop files
-  Add OTP support to ipa-pwd-extop
-  Fix client install exception if /etc/ssh is missing
-  Permit reads to ipatokenRadiusProxyUser objects
-  Fix for small syntax error in OTP schema
-  Use libunistring ulc_casecmp() on unicode strings

.. _petr_spacek_1_1:

Petr Spacek (1):
^^^^^^^^^^^^^^^^

-  ipa-client-install: Add 'debug' and 'show' statements to nsupdate
   commands

.. _petr_viktorin_33:

Petr Viktorin (33):
^^^^^^^^^^^^^^^^^^^

-  Remove leading zero from IPA_NUM_VERSION
-  Relax getkeytab test to allow additional messages on stderr
-  Remove code to install Dogtag 9
-  Flush stream after writing service messages
-  Make an ipa-tests package
-  Add ipa-run-tests command
-  Add Nose plugin for BeakerLib integration
-  Add a plugin for test ordering
-  Add a framework for integration test configuration
-  Add a framework for integration testing
-  Introduce a class for remote commands
-  Collect logs from tests
-  Show logs in failed tests
-  tests: Allow public keys for authentication to the remote machines
-  tests: Configure/unconfigure remote hosts
-  Host class improvements
-  Use dosctrings in BeakerLib phase descriptions
-  Make BeakerLib logging less verbose
-  BeakerLib plugin: Log http links in test docstrings
-  Integration test config: Make it possible to specify host IP
-  ipa-client: Use "ipa" as the package name for i18n
-  Move BeakerLibProcess out of BeakerLibPlugin
-  test_integration: Add log collection to Host
-  test_integration: Set up CA on replicas by default
-  Add more test tasks
-  Add install_topo to test tasks
-  Add the ipa-test-task tool
-  Add tar and xz dependencies to the freeipa-tests package
-  Correct default value of LDAPClient.get_entries scope argument
-  test_simple_replication: Wait for replication to finish before
   checking
-  Add the new no_member option to CLI tests
-  Update translations
-  Fix installutils.get_password without a TTY

.. _petr_vobornik_24:

Petr Vobornik (24):
^^^^^^^^^^^^^^^^^^^

-  Fix: HBAC Test tab is missing
-  Move spec modifications from facet factories to pre_ops
-  Unite and move facet pre_ops to related modules
-  Web UI: move ./_base/metadata_provider.js to ./metadata.js
-  Regression fix: missing control buttons in nested search facets
-  Make ssbrowser.html work in IE 10
-  Fix regression: missing facet tab group labels
-  Regression fix: rule table with ext. member support doesn't offer any
   items
-  Fix default value selection in radio widget
-  Do not redirect to https in /ipa/ui on non-HTML files
-  Create Firefox configuration extension on CA-less install
-  Disable checkboxes and radios for readonly attributes
-  Better automated test support
-  Fix container element in adder dialogs
-  Upstream Web UI tests
-  Web UI search optimization
-  Break long words in notification area
-  Remove word 'field' from GECOS param label
-  Web UI integration tests: Add trust tests
-  Web UI integration tests: Add ui_driver method descriptions
-  Web UI integration tests: Verify data after add and mod
-  Web UI integration tests: Compute range sizes to avoid overlaps
-  Web UI integration tests: PEP8 fixes
-  Web UI integration tests: Code quality fixes

.. _rob_crittenden_4:

Rob Crittenden (4):
^^^^^^^^^^^^^^^^^^^

-  Bump version for development branch to 3.2.99
-  Return the correct Content-type on negotiated XML-RPC requests.
-  Add Camellia ciphers to allowed list.
-  Hide sensitive attributes in LDAP updater logging and output

.. _simo_sorce_2:

Simo Sorce (2):
^^^^^^^^^^^^^^^

-  CLDAP: Fix domain handling in netlogon requests
-  CLDAP: Return empty reply on non-fatal errors

.. _sumit_bose_5:

Sumit Bose (5):
^^^^^^^^^^^^^^^

-  Fix format string typo
-  Fix type of printf argument
-  Add PAC to master host TGTs
-  extdom: replace winbind calls with POSIX/SSSD calls
-  Remove winbind client configure check

.. _tomas_babej_32:

Tomas Babej (32):
^^^^^^^^^^^^^^^^^

-  Remove redundancy from hbactest help text
-  Do not translate trust type and direction with --raw in trust_show
   and trust-find
-  Support multiple local domain ranges with RID base set
-  Do not allow removal of ID range of an active trust
-  Use private ccache in ipa install tools
-  Remove redundant check for env.interactive
-  Add prompt_param method to avoid code duplication
-  Incorporate interactive prompts in idrange-add
-  Do not check userPassword with 7-bit plugin
-  Manage ipa-otpd.socket by IPA
-  Add ipaRangeType attribute to LDAP Schema
-  Add update plugin to fill in ipaRangeType attribute
-  Extend idrange commands to support new range origin types
-  PEP8 fixes in idrange.py
-  Remove hardcoded values from idrange plugin tests
-  Return ipaRangeType as a list in idrange commands
-  Do not redirect ipa/crl to HTTPS
-  Add --range-type option that forces range type of the trusted domain
-  Add libsss_nss_idmap-devel to BuildRequires
-  Change group ownership of CRL publish directory
-  Provide ipa-advise tool
-  Use AD LDAP probing to create trusted domain ID range
-  Move requirement for keyutils to freeipa-python package
-  Change shebang to absolute path in ipa-client-automount
-  Skip referrals when converting LDAP result to LDAPEntry
-  Refactor the interactive prompt logic in idrange_add
-  Limit pwpolicy maxlife to 20000 days
-  Use case-insensitive dict for trusted domain info
-  Improve help entry for ipa host
-  Remove overlapping use-cases of the same result variable
-  Add a word wrapping for comment log messages to AdviceLogger
-  Wrap lines in the list of available advices

.. _version_3.3.0_beta_1_07242013:

Version 3.3.0 Beta 1 (07/24/2013)
=================================

.. _alexander_bokovoy_8:

Alexander Bokovoy (8):
^^^^^^^^^^^^^^^^^^^^^^

-  Fix cldap parser to work with a single equality filter (NtVer=...)
-  Make sure domain_name is also set when processing INP_NAME requests
-  Fix extdom plugin to provide unqualified name in response as sssd
   expects
-  Generate syntethic MS-PAC for all services running on IPA master
-  ipa-adtrust-install: configure compatibility tree to serve trusted
   domain users
-  ipa-kdb: cache KDC hostname on startup
-  ipa-kdb: reinit mspac on HTTP TGT acquisition to aid trust-add case
-  ipaserver/dcerpc: attempt to resolve SIDs through SSSD first

.. _ana_krivokapic_21:

Ana Krivokapic (21):
^^^^^^^^^^^^^^^^^^^^

-  Prompt for nameserver IP address in dnszone-add
-  Do not display success message on failure in web UI
-  Ignore files generated by build
-  Deprecate options --dom-sid and --dom-name in idrange-mod
-  Prevent error when running IPA commands with su/sudo
-  Fix displaying of success message
-  Fix location of service.crt in .gitignore
-  Improve handling of options in ipa-client-install
-  Fail when adding a trust with a different range
-  Do not display traceback to user
-  Require rid-base and secondary-rid-base in idrange-add after
   ipa-adtrust-install
-  Fix bug in adtrustinstance
-  Use correct DS instance in ipactl status
-  Avoid systemd service deadlock during shutdown
-  Make sure replication works after DM password is changed
-  Use --ignore-dependencies only when necessary
-  Properly handle non-existent cert files
-  Add 'ipa_server_mode' option to SSSD configuration
-  Bump version of sssd in spec file
-  Use admin@REALM when testing if SSSD is ready
-  Fix internal error in idrange-add

.. _diane_trout_1_1:

Diane Trout (1):
^^^^^^^^^^^^^^^^

-  Fix log format not a string literal.

.. _jakub_hrozek_3_1:

Jakub Hrozek (3):
^^^^^^^^^^^^^^^^^

-  Remove unused variable
-  IPA KDB MS-PAC: return ENOMEM if allocation fails
-  IPA KDB MS-PAC: remove unused variable

.. _jan_cholasta_21_1:

Jan Cholasta (21):
^^^^^^^^^^^^^^^^^^

-  Use the correct PKCS#12 file for HTTP server.
-  Remove stray error condition in ipa-server-install.
-  Handle exceptions gracefully when verifying PKCS#12 files.
-  Skip empty lines when parsing pk12util output.
-  Do not allow installing CA replicas in CA-less setup.
-  Do not track DS certificate in CA-less setup.
-  Fix CA-less check in ipa-replica-install and ipa-ca-install.
-  Do not skip SSSD known hosts in ipa-client-install --ssh-trust-dns.
-  Enable SASL mapping fallback.
-  Skip cert issuer validation in service and host commands in CA-less
   install.
-  Check trust chain length in CA-less install.
-  Use LDAP search instead of \*group_show to check if a group exists.
-  Use LDAP search instead of \*group_show to check for a group
   objectclass.
-  Use LDAP modify operation directly to add/remove group members.
-  Add missing substring indices for attributes managed by the referint
   plugin.
-  Add missing equality index for ipaUniqueId.
-  Run gpg-agent explicitly when encrypting/decrypting files.
-  Add new hidden command option to suppress processing of membership
   attributes.
-  Ask for PKCS#12 password interactively in ipa-server-install.
-  Ask for PKCS#12 password interactively in ipa-replica-prepare.
-  Print newline after receiving EOF in installutils.read_password.

.. _lukas_slebodnik_1:

Lukas Slebodnik (1):
^^^^^^^^^^^^^^^^^^^^

-  Use pkg-config to detect cmocka

.. _martin_kosek_11:

Martin Kosek (11):
^^^^^^^^^^^^^^^^^^

-  Set KRB5CCNAME so that dirsrv can work with newer krb5-server
-  Handle DIR type CCACHEs in test_cmdline properly
-  Avoid exporting KRB5_KTNAME in dirsrv env
-  Remove redundant u'' character
-  Drop SELinux subpackage
-  Drop redundant directory /var/cache/ipa/sessions
-  Remove entitlement support
-  Run server upgrade and restart in posttrans
-  Require new selinux-policy replacing old server-selinux subpackage
-  Bump minimum SSSD version
-  Become 3.3.0 Beta 1

.. _nathaniel_mccallum_10_1:

Nathaniel McCallum (10):
^^^^^^^^^^^^^^^^^^^^^^^^

-  Add ipaUserAuthType and ipaUserAuthTypeClass
-  Add IPA OTP schema and ACLs
-  ipa-kdb: Add OTP support
-  Add the krb5/FreeIPA RADIUS companion daemon
-  Remove unnecessary prefixes from ipa-pwd-extop files
-  Add OTP support to ipa-pwd-extop
-  Fix client install exception if /etc/ssh is missing
-  Permit reads to ipatokenRadiusProxyUser objects
-  Fix for small syntax error in OTP schema
-  Use libunistring ulc_casecmp() on unicode strings

.. _petr_spacek_1_2:

Petr Spacek (1):
^^^^^^^^^^^^^^^^

-  ipa-client-install: Add 'debug' and 'show' statements to nsupdate
   commands

.. _petr_viktorin_21:

Petr Viktorin (21):
^^^^^^^^^^^^^^^^^^^

-  Remove leading zero from IPA_NUM_VERSION
-  Relax getkeytab test to allow additional messages on stderr
-  Remove code to install Dogtag 9
-  Flush stream after writing service messages
-  Make an ipa-tests package
-  Add ipa-run-tests command
-  Add Nose plugin for BeakerLib integration
-  Add a plugin for test ordering
-  Add a framework for integration test configuration
-  Add a framework for integration testing
-  Introduce a class for remote commands
-  Collect logs from tests
-  Show logs in failed tests
-  tests: Allow public keys for authentication to the remote machines
-  tests: Configure/unconfigure remote hosts
-  Host class improvements
-  Use dosctrings in BeakerLib phase descriptions
-  Make BeakerLib logging less verbose
-  BeakerLib plugin: Log http links in test docstrings
-  Integration test config: Make it possible to specify host IP
-  ipa-client: Use "ipa" as the package name for i18n

.. _petr_vobornik_18:

Petr Vobornik (18):
^^^^^^^^^^^^^^^^^^^

-  Fix: HBAC Test tab is missing
-  Move spec modifications from facet factories to pre_ops
-  Unite and move facet pre_ops to related modules
-  Web UI: move ./_base/metadata_provider.js to ./metadata.js
-  Regression fix: missing control buttons in nested search facets
-  Make ssbrowser.html work in IE 10
-  Fix regression: missing facet tab group labels
-  Regression fix: rule table with ext. member support doesn't offer any
   items
-  Fix default value selection in radio widget
-  Do not redirect to https in /ipa/ui on non-HTML files
-  Create Firefox configuration extension on CA-less install
-  Disable checkboxes and radios for readonly attributes
-  Better automated test support
-  Fix container element in adder dialogs
-  Upstream Web UI tests
-  Web UI search optimization
-  Break long words in notification area
-  Remove word 'field' from GECOS param label

.. _rob_crittenden_4_1:

Rob Crittenden (4):
^^^^^^^^^^^^^^^^^^^

-  Bump version for development branch to 3.2.99
-  Return the correct Content-type on negotiated XML-RPC requests.
-  Add Camellia ciphers to allowed list.
-  Hide sensitive attributes in LDAP updater logging and output

.. _simo_sorce_2_1:

Simo Sorce (2):
^^^^^^^^^^^^^^^

-  CLDAP: Fix domain handling in netlogon requests
-  CLDAP: Return empty reply on non-fatal errors

.. _sumit_bose_5_1:

Sumit Bose (5):
^^^^^^^^^^^^^^^

-  Fix format string typo
-  Fix type of printf argument
-  Add PAC to master host TGTs
-  extdom: replace winbind calls with POSIX/SSSD calls
-  Remove winbind client configure check

.. _tomas_babej_22:

Tomas Babej (22):
^^^^^^^^^^^^^^^^^

-  Remove redundancy from hbactest help text
-  Do not translate trust type and direction with --raw in trust_show
   and trust-find
-  Support multiple local domain ranges with RID base set
-  Do not allow removal of ID range of an active trust
-  Use private ccache in ipa install tools
-  Remove redundant check for env.interactive
-  Add prompt_param method to avoid code duplication
-  Incorporate interactive prompts in idrange-add
-  Do not check userPassword with 7-bit plugin
-  Manage ipa-otpd.socket by IPA
-  Add ipaRangeType attribute to LDAP Schema
-  Add update plugin to fill in ipaRangeType attribute
-  Extend idrange commands to support new range origin types
-  PEP8 fixes in idrange.py
-  Remove hardcoded values from idrange plugin tests
-  Return ipaRangeType as a list in idrange commands
-  Do not redirect ipa/crl to HTTPS
-  Add --range-type option that forces range type of the trusted domain
-  Add libsss_nss_idmap-devel to BuildRequires
-  Change group ownership of CRL publish directory
-  Provide ipa-advise tool
-  Use AD LDAP probing to create trusted domain ID range

.. _version_3.2.2_07172013:

Version 3.2.2 (07/17/2013)
==========================

.. _ana_krivokapic_8:

Ana Krivokapic (8):
^^^^^^^^^^^^^^^^^^^

-  Fix displaying of success message
-  Improve handling of options in ipa-client-install
-  Do not display traceback to user
-  Fix bug in adtrustinstance
-  Use correct DS instance in ipactl status
-  Avoid systemd service deadlock during shutdown
-  Make sure replication works after DM password is changed
-  Use --ignore-dependencies only when necessary

.. _jan_cholasta_16:

Jan Cholasta (16):
^^^^^^^^^^^^^^^^^^

-  Use the correct PKCS#12 file for HTTP server.
-  Remove stray error condition in ipa-server-install.
-  Handle exceptions gracefully when verifying PKCS#12 files.
-  Skip empty lines when parsing pk12util output.
-  Do not allow installing CA replicas in CA-less setup.
-  Do not track DS certificate in CA-less setup.
-  Fix CA-less check in ipa-replica-install and ipa-ca-install.
-  Do not skip SSSD known hosts in ipa-client-install --ssh-trust-dns.
-  Skip cert issuer validation in service and host commands in CA-less
   install.
-  Check trust chain length in CA-less install.
-  Use LDAP search instead of \*group_show to check if a group exists.
-  Use LDAP search instead of \*group_show to check for a group
   objectclass.
-  Use LDAP modify operation directly to add/remove group members.
-  Add missing substring indices for attributes managed by the referint
   plugin.
-  Add missing equality index for ipaUniqueId.
-  Run gpg-agent explicitly when encrypting/decrypting files.

.. _lukas_slebodnik_1_1:

Lukas Slebodnik (1):
^^^^^^^^^^^^^^^^^^^^

-  Use pkg-config to detect cmocka

.. _martin_kosek_7:

Martin Kosek (7):
^^^^^^^^^^^^^^^^^

-  Remove entitlement support
-  Enable SASL mapping fallback.
-  Drop SELinux subpackage
-  Drop redundant directory /var/cache/ipa/sessions
-  Run server upgrade and restart in posttrans
-  Require new selinux-policy replacing old server-selinux subpackage
-  Become 3.2.2

.. _nathaniel_mccallum_3:

Nathaniel McCallum (3):
^^^^^^^^^^^^^^^^^^^^^^^

-  Fix client install exception if /etc/ssh is missing
-  Permit reads to ipatokenRadiusProxyUser objects
-  Fix for small syntax error in OTP schema

.. _petr_vobornik_5_1:

Petr Vobornik (5):
^^^^^^^^^^^^^^^^^^

-  Regression fix: rule table with ext. member support doesn't offer any
   items
-  Fix default value selection in radio widget
-  Do not redirect to https in /ipa/ui on non-HTML files
-  Create Firefox configuration extension on CA-less install
-  Disable checkboxes and radios for readonly attributes

.. _rob_crittenden_1_1:

Rob Crittenden (1):
^^^^^^^^^^^^^^^^^^^

-  Return the correct Content-type on negotiated XML-RPC requests.

.. _sumit_bose_1_2:

Sumit Bose (1):
^^^^^^^^^^^^^^^

-  Fix type of printf argument

.. _tomas_babej_2:

Tomas Babej (2):
^^^^^^^^^^^^^^^^

-  Do not redirect ipa/crl to HTTPS
-  Change group ownership of CRL publish directory

.. _version_3.2.1_06072013:

Version 3.2.1 (06/07/2013)
==========================

.. _alexander_bokovoy_1_1:

Alexander Bokovoy (1):
^^^^^^^^^^^^^^^^^^^^^^

-  Fix cldap parser to work with a single equality filter (NtVer=...)

.. _ana_krivokapic_3:

Ana Krivokapic (3):
^^^^^^^^^^^^^^^^^^^

-  Prompt for nameserver IP address in dnszone-add
-  Do not display success message on failure in web UI
-  Prevent error when running IPA commands with su/sudo

.. _diane_trout_1_2:

Diane Trout (1):
^^^^^^^^^^^^^^^^

-  Fix log format not a string literal.

.. _martin_kosek_4:

Martin Kosek (4):
^^^^^^^^^^^^^^^^^

-  Set KRB5CCNAME so that dirsrv can work with newer krb5-server
-  Avoid exporting KRB5_KTNAME in dirsrv env
-  Remove redundant u'' character
-  Become 3.2.1

.. _nathaniel_mccallum_6:

Nathaniel McCallum (6):
^^^^^^^^^^^^^^^^^^^^^^^

-  Add ipaUserAuthType and ipaUserAuthTypeClass
-  Add IPA OTP schema and ACLs
-  ipa-kdb: Add OTP support
-  Add the krb5/FreeIPA RADIUS companion daemon
-  Remove unnecessary prefixes from ipa-pwd-extop files
-  Add OTP support to ipa-pwd-extop

.. _petr_spacek_1_3:

Petr Spacek (1):
^^^^^^^^^^^^^^^^

-  ipa-client-install: Add 'debug' and 'show' statements to nsupdate
   commands

.. _petr_viktorin_1:

Petr Viktorin (1):
^^^^^^^^^^^^^^^^^^

-  Remove leading zero from IPA_NUM_VERSION

.. _petr_vobornik_7_1:

Petr Vobornik (7):
^^^^^^^^^^^^^^^^^^

-  Fix: HBAC Test tab is missing
-  Move spec modifications from facet factories to pre_ops
-  Unite and move facet pre_ops to related modules
-  Web UI: move ./_base/metadata_provider.js to ./metadata.js
-  Regression fix: missing control buttons in nested search facets
-  Make ssbrowser.html work in IE 10
-  Fix regression: missing facet tab group labels

.. _simo_sorce_2_2:

Simo Sorce (2):
^^^^^^^^^^^^^^^

-  CLDAP: Fix domain handling in netlogon requests
-  CLDAP: Return empty reply on non-fatal errors

.. _sumit_bose_1_3:

Sumit Bose (1):
^^^^^^^^^^^^^^^

-  Fix format string typo

.. _tomas_babej_9:

Tomas Babej (9):
^^^^^^^^^^^^^^^^

-  Remove redundancy from hbactest help text
-  Support multiple local domain ranges with RID base set
-  Do not allow removal of ID range of an active trust
-  Use private ccache in ipa install tools
-  Remove redundant check for env.interactive
-  Add prompt_param method to avoid code duplication
-  Incorporate interactive prompts in idrange-add
-  Do not check userPassword with 7-bit plugin
-  Manage ipa-otpd.socket by IPA

.. _version_3.2.0_05102013:

Version 3.2.0 (05/10/2013)
==========================

.. _alexander_bokovoy_9_1:

Alexander Bokovoy (9):
^^^^^^^^^^^^^^^^^^^^^^

-  Update plugin to upload CA certificate to LDAP
-  ipasam: use base scope when fetching domain information about own
   domain
-  ipaserver/dcerpc: enforce search_s without schema checks for GC
   searching
-  ipa-replica-manage: migrate to single_value after LDAPEntry updates
-  Process exceptions when talking to Dogtag
-  ipasam: add enumeration of UPN suffixes based on the realm domains
-  Enhance ipa-adtrust-install for domains with multiple IPA server
-  spec: detect Kerberos DAL driver ABI change from installed krb5-devel
-  Resolve SIDs in Web UI

.. _ana_krivokapic_24:

Ana Krivokapic (24):
^^^^^^^^^^^^^^^^^^^^

-  Raise ValidationError for incorrect subtree option.
-  Add crond as a default HBAC service
-  Take into consideration services when deleting replicas
-  Add list of domains associated to our realm to cn=etc
-  Improve error messages for external group members
-  Remove check for alphabetic only characters from domain name
   validation
-  Fix internal error for ipa show-mappings
-  Realm Domains page
-  Use default NETBIOS name in unattended ipa-adtrust-install
-  Add mkhomedir option to ipa-server-install and ipa-replica-install
-  Remove CA cert on client uninstall
-  Fix output for some CLI commands
-  Add missing summary message to dnszone_del
-  Remove HBAC source hosts from web UI
-  Remove any reference to HBAC source hosts from help
-  Deprecate HBAC source hosts from CLI
-  Integrate realmdomains with IPA DNS
-  Improve help text for HBAC service groups
-  Do not sort dictionaries in assert_deepequal utility function
-  Handle missing /etc/ipa in ipa-client-install
-  Fix the spec file
-  Do not display an interactive mode message in unattended mode
-  Add missing permissions to Host Administrators privilege
-  Always stop dirsrv in 'ipactl stop'

.. _brian_cook_1:

Brian Cook (1):
^^^^^^^^^^^^^^^

-  Add DNS Setup Prompt to Install

.. _jr_aquino_1:

JR Aquino (1):
^^^^^^^^^^^^^^

-  Allow PKI-CA Replica Installs when CRL exceeds default maxber value

.. _jakub_hrozek_1_2:

Jakub Hrozek (1):
^^^^^^^^^^^^^^^^^

-  Allow ipa-replica-conncheck and ipa-adtrust-install to read krb5
   includedir

.. _jan_cholasta_33:

Jan Cholasta (33):
^^^^^^^^^^^^^^^^^^

-  Pylint cleanup.
-  Drop ipapython.compat.
-  Add support for RFC 6594 SSHFP DNS records.
-  Raise ValidationError on invalid CSV values.
-  Run interactive_prompt callbacks after CSV values are split.
-  Add custom mapping object for LDAP entry data.
-  Add make_entry factory method to LDAPConnection.
-  Remove the Entity class.
-  Remove the Entry class.
-  Use the dn attribute of LDAPEntry to set/get DNs of entries.
-  Preserve case of attribute names in LDAPEntry.
-  Aggregate IPASimpleLDAPObject in LDAPEntry.
-  Support attributes with multiple names in LDAPEntry.
-  Use full DNs in plugin code.
-  Remove DN normalization from the baseldap plugin.
-  Remove support for DN normalization from LDAPClient.
-  Fix remove while iterating in suppress_netgroup_memberof.
-  Remove disabled entries from sudoers compat tree.
-  Fix internal error in output_for_cli method of
   sudorule_{enable,disable}.
-  Do not fail if schema cannot be retrieved from LDAP server.
-  Allow disabling LDAP schema retrieval in LDAPClient and IPAdmin.
-  Allow disabling attribute decoding in LDAPClient and IPAdmin.
-  Disable schema retrieval and attribute decoding when talking to AD
   GC.
-  Add Kerberos ticket flags management to service and host plugins.
-  Do actually stop pki_cad in stop_pkicad instead of starting it.
-  Use only one URL for OCSP and CRL in IPA certificate profile.
-  Use A/AAAA records instead of CNAME records in ipa-ca.
-  Delete DNS records in ipa-ca on ipa-csreplica-manage del.
-  Use correct zone when removing DNS records of a master.
-  Add DNS records for existing masters when installing DNS for the
   first time.
-  Add ipa-ca records for existing CA masters when installing DNS for
   the first time.
-  Add support for OpenSSH 6.2.
-  Fix normalization of FQDNs in DNS installer code.

.. _john_dennis_2:

John Dennis (2):
^^^^^^^^^^^^^^^^

-  Cookie Expires date should be locale insensitive
-  Use secure method to acquire IPA CA certificate

.. _lynn_root_3:

Lynn Root (3):
^^^^^^^^^^^^^^

-  Added the ability to do Beta versioning
-  Fixed the catch of the hostname option during ipa-server-install
-  Raise ValidationError when CSR does not have a subject hostname

.. _martin_kosek_65:

Martin Kosek (65):
^^^^^^^^^^^^^^^^^^

-  Add Lynn Root to Contributors.txt
-  Enable SSSD on client install
-  Fix delegation-find command --group handling
-  Do not crash when Kerberos SRV record is not found
-  permission-find no longer crashes with --targetgroup
-  Avoid CRL migration error message
-  Sort LDAP updates properly
-  Upgrade process should not crash on named restart
-  Installer should not connect to 127.0.0.1
-  Fix migration for openldap DS
-  Remove unused krbV imports
-  Use fully qualified CCACHE names
-  Fix permission_find test error
-  Add trusconfig-show and trustconfig-mod commands
-  ipa-kdb: add sentinel for LDAPDerefSpec allocation
-  ipa-kdb: avoid ENOMEM when all SIDs are filtered out
-  ipa-kdb: reinitialize LDAP configuration for known realms
-  Add SID blacklist attributes
-  ipa-kdb: read SID blacklist from LDAP
-  ipa-sam: Fill SID blacklist when trust is added
-  ipa-adtrust-install should ask for SID generation
-  Test NetBIOS name clash before creating a trust
-  Generalize AD GC search
-  Do not hide SID resolver error in group-add-member
-  Add support for AD users to hbactest command
-  Fix hbachelp examples formatting
-  ipa-kdb: remove memory leaks
-  ipa-kdb: fix retry logic in ipadb_deref_search
-  Add autodiscovery section in ipa-client-install man pages
-  Avoid internal error when user is not Trust admin
-  Use fixed test domain in realmdomains test
-  Bump FreeIPA version for development branch
-  Remove ORDERING for IA5 attributeTypes
-  Fix includedir directive in krb5.conf template
-  Use new 389-ds-base cleartext password API
-  Do not hide idrange-add errors when adding trust
-  Preserve order of servers in ipa-client-install
-  Avoid multiple client discovery with fixed server list
-  Update named.conf parser
-  Use tkey-gssapi-keytab in named.conf
-  Do not force named connections on upgrades
-  ipa-client discovery with anonymous access off
-  Use temporary CCACHE in ipa-client-install
-  Improve client install LDAP cert retrieval fallback
-  Configure ipa_dns DS plugin on install and upgrade
-  Fix structured DNS record output
-  Bump selinux-policy requires
-  Clean spec file for Fedora 19
-  Remove build warnings
-  Remove syslog.target from ipa.server
-  Put pid-file to named.conf
-  Update mod_wsgi socket directory
-  Normalize RA agent certificate
-  Require 389-base-base 1.3.0.5
-  Change CNAME and DNAME attributes to single valued
-  Improve CNAME record validation
-  Improve DNAME record validation
-  Become 3.2.0 Prerelease 1
-  Fix trustconfig-mod primary group error
-  Require new samba and krb5
-  Add userClass attribute for hosts
-  Update pki proxy configuration
-  Do not add ipa-ca records on CA-less installs
-  Fix ipa-ca DNS name creation
-  Fix SASL_NOCANON behavior for LDAPI

.. _nathaniel_mccallum_1_2:

Nathaniel McCallum (1):
^^^^^^^^^^^^^^^^^^^^^^^

-  Ignore log files from automake tests

.. _petr_spacek_1_4:

Petr Spacek (1):
^^^^^^^^^^^^^^^^

-  Add 389 DS plugin for special idnsSOASerial attribute handling

.. _petr_viktorin_113:

Petr Viktorin (113):
^^^^^^^^^^^^^^^^^^^^

-  Sort Options and Outputs in API.txt
-  Add the CA cert to LDAP after the CA install
-  Better logging for AdminTool and ipa-ldap-updater
-  Port ipa-replica-prepare to the admintool framework
-  Make ipapython.dogtag log requests at debug level, not info
-  Don't add another nsDS5ReplicaId on updates if one already exists
-  Improve \`ipa --help\` output
-  Print help to stderr on error
-  Store the OptionParser in the API, use it to print unified help
   messages
-  Simplify \`ipa help topics\` output
-  Add command summary to \`ipa COMMAND --help\` output
-  Mention \`ipa COMMAND --help\` as the preferred way to get command
   help
-  Parse command arguments before creating a context
-  Add tests for the help command & --help options
-  In topic help text, mention how to get help for commands
-  Check SSH connection in ipa-replica-conncheck
-  Use ipauniqueid for the RDN of sudo commands
-  Prevent a sudo command from being deleted if it is a member of a sudo
   rule
-  Update sudocmd ACIs to use targetfilter
-  Add the version option to all Commands
-  Add ipalib.messages
-  Add client capabilities, enable messages
-  Rename the "messages" Output of the i18n_messages command to "texts"
-  Fix permission validation and normalization in aci.py
-  Remove csv_separator and csv_skipspace Param arguments
-  Drop support for CSV in the CLI client
-  Update argument docs to reflect dropped CSV support
-  Update plugin docstrings (topic help) to reflect dropped CSV support
-  cli: Do interactive prompting after a context is created
-  Remove some unused imports
-  Remove unused methods from Entry, Entity, and IPAdmin
-  Derive Entity class from Entry, and move it to ldapupdate
-  Use explicit loggers in ldap2 code
-  Move LDAPEntry to ipaserver.ipaldap and derive Entry from it
-  Remove connection-creating code from ShemaCache
-  Move the decision to force schema updates out of IPASimpleLDAPObject
-  Move SchemaCache and IPASimpleLDAPObject to ipaserver.ipaldap
-  Start LDAPConnection, a common base for ldap2 and IPAdmin
-  Make IPAdmin not inherit from IPASimpleLDAPObject
-  Move schema-related methods to LDAPConnection
-  Move DN handling methods to LDAPConnection
-  Move filter making methods to LDAPConnection
-  Move entry finding methods to LDAPConnection
-  Remove unused proxydn functionality from IPAdmin
-  Move entry add, update, remove, rename to LDAPConnection
-  Implement some of IPAdmin's legacy methods in terms of LDAPConnection
   methods
-  Replace setValue by keyword arguments when creating entries
-  Use update_entry with a single entry in adtrustinstance
-  Replace entry.getValues() by entry.get()
-  Replace entry.setValue/setValues by item assignment
-  Replace add_s and delete_s by their newer equivalents
-  Change {add,update,delete}_entry to take LDAPEntries
-  Remove unused imports from ipaserver/install
-  Remove unused bindcert and bindkey arguments to IPAdmin
-  Turn the LDAPError handler into a context manager
-  Remove dbdir, binddn, bindpwd from IPAdmin
-  Remove IPAdmin.updateEntry calls from fix_replica_agreements
-  Remove IPAdmin.get_dns_sorted_by_length
-  Replace IPAdmin.checkTask by replication.wait_for_task
-  Introduce LDAPEntry.single_value for getting single-valued attributes
-  Remove special-casing for missing and single-valued attributes in
   LDAPUpdate._entry_to_entity
-  Replace entry.getValue by entry.single_value
-  Replace getList by a get_entries method
-  Remove toTupleList and attrList from LDAPEntry
-  Rename LDAPConnection to LDAPClient
-  Replace addEntry with add_entry
-  Replace deleteEntry with delete_entry
-  Fix typo and traceback suppression in replication.py
-  replace getEntry with get_entry (or get_entries if scope !=
   SCOPE_BASE)
-  Inline inactivateEntry in its only caller
-  Inline waitForEntry in its only caller
-  Proxy LDAP methods explicitly rather than using \__getattr_\_
-  Remove search_s and search_ext_s from IPAdmin
-  Replace IPAdmin.start_tls_s by an \__init_\_ argument
-  Remove IPAdmin.sasl_interactive_bind_s
-  Remove IPAdmin.simple_bind_s
-  Remove IPAdmin.unbind_s(), keep unbind()
-  Use ldap instead of \_ldap in ipaldap
-  Do not use global variables in migration.py
-  Use IPAdmin rather than raw python-ldap in migration.bind
-  Use IPAdmin rather than raw python-ldap in ipactl
-  Remove some uses of raw python-ldap
-  Improve LDAPEntry tests
-  Fix installing server with external CA
-  Change DNA magic value to -1 to make UID 999 usable
-  Move ipaldap to ipapython
-  Remove ipaserver/ipaldap.py
-  Use IPAdmin rather than raw python-ldap in ipa-client-install
-  Use IPAdmin rather than raw python-ldap in migration.py and
   ipadiscovery.py
-  Remove unneeded python-ldap imports
-  Don't download the schema in ipadiscovery
-  ipa-server-install: Make temporary pin files available for the whole
   installation
-  ipa-server-install: Remove the --selfsign option
-  Remove unused ipapython.certdb.CertDB class
-  ipaserver.install.certs: Introduce NSSDatabase as a more generic
   certutil wrapper
-  Trust CAs from PKCS#12 files even if they don't have Friendly Names
-  dsinstance, httpinstance: Don't hardcode 'Server-Cert'
-  Support installing with custom SSL certs, without a CA
-  Load the CA cert into server NSS databases
-  Do not call cert-\* commands in host plugin if a RA is not available
-  ipa-client-install: Do not request host certificate if server is
   CA-less
-  Display full command documentation in online help
-  Remove 'cn' attribute from idnsRecord and idnsZone objectClasses
-  ipa-server-install: correct help text for --external_{cert,ca}_file
-  Update translations from Transifex
-  Uninstall selfsign CA on upgrade
-  Remove obsolete self-sign references from man pages, docstrings,
   comments
-  Drop --selfsign server functionality
-  Use two digits for each part of NUM_VERSION
-  Fix syntax of the dc attributeType
-  Fix syntax errors in schema files
-  Only require libsss_nss_idmap-python in Fedora 19+
-  Update translations from Transifex

.. _petr_vobornik_181:

Petr Vobornik (181):
^^^^^^^^^^^^^^^^^^^^

-  Make confirm_dialog a base class of revoke and restore certificate
   dialogs
-  Make confirm_dialog a base class for deleter dialog
-  Make confirm_dialog a base class for message_dialog
-  Confirm mixin
-  Confirm adder dialog by enter
-  Confirm error dialog by enter
-  Focus last dialog when some is closed
-  Confirm association dialogs by enter
-  Standardize login password reset, user reset password and host set
   OTP dialogs
-  Focus first input element after 'Add and Add another'
-  Enable mod_deflate
-  Use Uglify.js for JS optimization
-  Dojo Builder
-  Config files for builder of FreeIPA UI layer
-  Minimal Dojo layer
-  Web UI development environment directory structure and configuration
-  Web UI Sync development utility
-  Move of Web UI non AMD dep. libs to libs subdirectory
-  Move of core Web UI files to AMD directory
-  Update JavaScript Lint configuration file
-  AMD config file
-  Change Web UI sources to simple AMD modules
-  Updated makefiles to build FreeIPA Web UI layer
-  Change tests to use AMD loader
-  Fix BuildRequires: rhino replaced with java-1.7.0-openjdk
-  Develop.js extended
-  Allow to specify modules for which builder doesn't raise dependency
   error
-  Web UI build profile updated
-  Combobox keyboard support
-  Fix dirty state update of editable combobox
-  Fix handling of no_update flag in Web UI
-  Web UI: configurable SID blacklists
-  Web UI:Certificate pages
-  Web UI:Choose different search option for cert-find
-  Fixed Web UI build error caused by rhino changes in F19
-  Nestable checkbox/radio widget
-  Added Web UI support for service PAC type option: NONE
-  Web UI: Disable cert functionality if a CA is not available
-  Add ipakrbokasdelegate option to service and host Web UI pages
-  Run permission target switch action only for visible widgets
-  Filter groups by type (POSIX, non-POSIX, external)
-  Global trust config page
-  Don't show trusts pages when trust is not configured
-  Fix regression in group type selection in group adder dialog
-  Fix: Certificate status is not visible in Service and Host page
-  jsl update
-  Update of Dojo build
-  Basic implementation of registers
-  i18n - internationalized text provider
-  Phases - application lifecycle
-  Config.js
-  Menu and application controller refactoring
-  Removed old navigation code
-  Remove IPA.nav usage, obsolete entity.get_primary_key
-  Fix nested facet search
-  Remove IPA.current_entity usage
-  Set pkeys to add,remove dialog
-  File dependencies added to Web UI Makefile
-  Add menu memory
-  Rename path array from hash to path in hash generation
-  Fix selection of menu in automember
-  Fix facet needs_update behavior
-  Removed incorrect success message when adding of external member
   failed
-  Removed entity.get_primary from association facet
-  get_primary_key function usages removed
-  DNS menu fixed
-  Certificates, Realm domains added to navigation
-  Remove old navigation code in certificates
-  Fix needs_update on object change
-  Don't expect key for singleton objects (dnsconfig, config,
   realmdomains)
-  Raise only one "set" event on facet.state.set
-  Fix dirty dialog behavior
-  Add handling of runtime and shutdown phase. App-init renamed to init.
-  Fix unit tests
-  Web UI plugin loader
-  Fix hbactest styles
-  Menu proxy
-  Proper removal of dns menu item when dns is not installed
-  Fixed errors in DNS pages
-  Fix in state change handling and reporting
-  Fix tab switching for nested entities
-  Fix add/deletion of automember rule - caused by not setting facet for
   entity adder dialog
-  Use dojo/on instead of dojo/topic for facet-xxx events'
-  Rename alternation phase to customization
-  Replace id usage in App widget by class
-  Add phase on exact position
-  Metadata and text providers
-  Limit Provider reporting
-  Use text.get for transforming values supplied by spec
-  Replace IPA.get_message with text.get
-  Replace IPA.messages with @i18n definition in spec objects
-  Replace IPA.messages with @i18n definition for label specs
-  Replace IPA.messages with @i18n definition for add_title specs
-  Replace IPA.messages with @i18n definition for remove_title specs
-  Replace IPA.messages with @i18n definition for message specs
-  Replace IPA.messages with @i18n definition for title specs
-  Use text.get in IPA.notify_success
-  Replace remaining IPA.messages with text.get calls
-  Fix facet section labels
-  Remove invalid label definition from cert search facet
-  Replace IPA.get_message with text.get
-  Remove text.get usage from spec
-  Add pre and post build operations
-  Spec modification by diff object
-  Builder: added pre_ops and post_ops
-  Modularize group.js
-  Modularize details.js
-  Builder: factory,ctor overrides, mass build
-  Replace old builder by new implementation
-  Rename build constructor to ctor
-  Spec utils
-  Basic build tests
-  Rename factory to $factory in spec objects
-  Builder: return null if no spec supplied
-  Builder: fix overrides names - add $
-  Builder: fix infinite loop when using spec with circular dependency
-  Rename factory to $factory in spec objects modifications
-  Builder: return object when it's already built
-  Use IPA.object() as a base factory for framework objects
-  Handle built object in spec
-  Report phase errors
-  Builder: allow to use custom factory/ctor when using type
-  Fix construct registry map reference
-  Replace IPA.facet_builder with facets.builder
-  Builder: do not break on expected errors
-  Builder: remove item from singleton registry
-  Builder: fix inner array and obj references
-  Use entities module for entity registration, build and holding
-  Builder: add set method to Singleton_registry
-  Builder: build type without prior registration
-  Phases: warn when adding task for nonexistent phase
-  Builder: create Construct_registry by default in builder
-  Builder: global builder and registry
-  Replace IPA.widget_factories and IPA_field_factories with registry
-  Builder: allow string spec as spec property instead of type
-  Replace build logic in widget and field builder by new builder
-  Registry and builder for formatters
-  Builder: return null if no spec supplied - fix
-  Replace formatter creation with definition in specs
-  Builder and registry for validators
-  Change widget.build_child interface to the builder's
-  Builder and registry for actions
-  Replace usage of action factories with types
-  Fix incorrect type -> $type conversion
-  Make facet and entity policies declarative
-  Make summary conditions declarative
-  Allow metadata provider format for field metadata declaration
-  Replace IPA.get_entity_param calls in specs with provider strings
-  Replace IPA.get_command_option calls in specs with provider strings
-  Replace IPA.get_command_arg calls in specs with provider strings
-  Builders: allow pre_ops and post_ops in build overrides
-  Use builder for entity dialogs
-  Builder: allow registration without factory or ctor
-  Fix hbactest after rebase
-  Fix trustconfig after rebase
-  Entity registry and builder which allow definition by spec
-  Entity: allow definition of facet_groups in entity specs
-  Builder: handle expected errors in post_ops
-  Entity build: test for enabled in post_op
-  Convert definitions of entities to spec objects
-  Replace IPA.metadata.objects... with declarative definitions
-  Remove cert menu item when disabled
-  Don't automatically refresh facet after action success
-  Move spec creations of sudorule, hbacrule, netgroup and
   selinuxusermap details facet from their factories
-  Removal of IPA.metadata usages
-  Add widget updated event
-  Fix rule table add/delete button enablement
-  Replace ./facets with reg.facet
-  Remove entities.js, facets.js
-  Generate plugin index dynamically
-  Switch customization and registration phase
-  Do not offer already added members in association dialogs when
   different casing
-  Builder: fix join of pre_ops and post_ops arrays
-  Fix: make association facets in selfservice readonly
-  Builder: Singleton_registry: return null when construction spec not
   available
-  Navigation: handle invalid routes
-  Fix trustconfig specification
-  Fix WebUI crash when server installed as CA-less
-  Fix crash on ssh key add
-  Fix crash on host deleletion
-  Enable standalone facets in menu.add_item

.. _rob_crittenden_29:

Rob Crittenden (29):
^^^^^^^^^^^^^^^^^^^^

-  Convert uniqueMember members into DN objects.
-  Add ====Ana Krivokapic to Contributors.txt
-  Do SSL CA verification and hostname validation.
-  Don't initialize NSS if we don't have to, clean up unused cert refs
-  Update anonymous access ACI to protect secret attributes.
-  Make certmonger a (pre) requires on server, restart it before
   upgrading
-  Use new certmonger locking to prevent NSS database corruption.
-  Improve migration performance
-  Add LDAP server fallback to client installer
-  Prevent a crash when no entries are successfully migrated.
-  Implement the cert-find command for the dogtag CA backend.
-  Add missing v3 schema on upgrades, fix typo in schema.
-  Don't base64-encode the CA cert when uploading it during an upgrade.
-  Extend ipa-replica-manage to be able to manage DNA ranges.
-  Improve some error handling in ipa-replica-manage
-  Fix lockout of LDAP bind.
-  Fix two failing tests due to missing krb ticket flags
-  Full system backup and restore
-  Apply LDAP update files in blocks of 10, as originally designed.
-  Revert "Fix permission_find test error"
-  Become 3.2.0 Beta 1
-  Handle socket.gethostbyaddr() exceptions when verifying hostnames.
-  Require version of NSS that properly parses base64-encoded certs
-  Drop uniqueMember mapping with nss-pam-ldapd.
-  Add Nathaniel McCallum to Contributors.txt
-  Handle a 501 in cert-find from dogtag as a "not supported"
-  Specify the location for the agent PKCS#12 file so we don't have to
   move it.
-  Set KRB5CCNAME so httpd s4u2proxy can with with newer krb5-server
-  Become 3.2.0

.. _simo_sorce_2_3:

Simo Sorce (2):
^^^^^^^^^^^^^^^

-  Log info on failure to connect
-  Upload CA cert in the directory on install

.. _sumit_bose_21:

Sumit Bose (21):
^^^^^^^^^^^^^^^^

-  ipa-kdb: remove unused variable
-  ipa-kdb: Uninitialized scalar variable in ipadb_reinit_mspac()
-  ipa-sam: Array compared against 0 in ipasam_set_trusted_domain()
-  ipa-kdb: Dereference after null check in ipa_kdb_mspac.c
-  ipa-lockout: Wrong sizeof argument in ipa_lockout.c
-  ipa-extdom: Double-free in ipa_extdom_common.c
-  ipa-pwd: Unchecked return value ipapwd_chpwop()
-  Revert "MS-PAC: Special case NFS services"
-  Add NFS specific default for authorization data type
-  ipa-kdb: Read global defaul ipaKrbAuthzData
-  ipa-kdb: Read ipaKrbAuthzData with other principal data
-  ipa-kdb: add PAC only if requested
-  Add unit test for get_authz_data_types()
-  Mention PAC issue with NFS in service plugin doc
-  Allow 'nfs:NONE' in global configuration
-  Add support for cmocka C-Unit Test framework
-  ipa-pwd-extop: do not use dn until it is really set
-  Do not lookup up the domain too early if only the SID is known
-  Do not store SID string in a local buffer
-  Allow ID-to-SID mappings in the extdom plugin
-  ipa-kdb: Free talloc autofree context when module is closed

.. _timo_aaltonen_1:

Timo Aaltonen (1):
^^^^^^^^^^^^^^^^^^

-  convert the base platform modules into packages

.. _tomas_babej_27:

Tomas Babej (27):
^^^^^^^^^^^^^^^^^

-  Relax restriction for leading/trailing whitespaces in \*-find
   commands
-  Forbid overlapping rid ranges for the same id range
-  Fix a typo in ipa-adtrust-install help
-  Prevent integer overflow when setting krbPasswordExpiration
-  Add option to specify SID using domain name to idrange-add/mod
-  Prevent changing protected group's name using --setattr
-  Use default.conf as flag of IPA client being installed
-  Make sure appropriate exit status is returned in make-test
-  Make options checks in idrange-add/mod consistent
-  Add trusted domain range objectclass when using idrange-mod
-  Perform secondary rid range overlap check for local ranges only
-  Add support for re-enrolling hosts using keytab
-  Make sure uninstall script prompts for reboot as last
-  Remove implicit Str to DN conversion using \*-attr
-  Enforce exact SID match when adding or modifying a ID range
-  Allow host re-enrollment using delegation
-  Add logging to join command
-  Properly handle ipa-replica-install when its zone is not managed by
   IPA
-  Add nfs:NONE to default PAC types only when needed
-  Update only selected attributes for winsync agreement
-  Add hint message about --force-join option when enrollment fails
-  Avoid removing sss from nssswitch.conf during client uninstall
-  Allow underscore in record targets
-  Make gecos field editable in Web UI
-  Preserve already configured options in openldap conf
-  Enforce host existence only where needed in ipa-replica-manage
-  Handle connection timeout in ipa-replica-manage

.. _version_3.2.0_beta_1_04162013:

Version 3.2.0 Beta 1 (04/16/2013)
=================================

.. _alexander_bokovoy_1_2:

Alexander Bokovoy (1):
^^^^^^^^^^^^^^^^^^^^^^

-  spec: detect Kerberos DAL driver ABI change from installed krb5-devel

.. _ana_krivokapic_7:

Ana Krivokapic (7):
^^^^^^^^^^^^^^^^^^^

-  Remove CA cert on client uninstall
-  Fix output for some CLI commands
-  Add missing summary message to dnszone_del
-  Remove HBAC source hosts from web UI
-  Remove any reference to HBAC source hosts from help
-  Deprecate HBAC source hosts from CLI
-  Integrate realmdomains with IPA DNS

.. _jan_cholasta_4:

Jan Cholasta (4):
^^^^^^^^^^^^^^^^^

-  Do actually stop pki_cad in stop_pkicad instead of starting it.
-  Use only one URL for OCSP and CRL in IPA certificate profile.
-  Use A/AAAA records instead of CNAME records in ipa-ca.
-  Delete DNS records in ipa-ca on ipa-csreplica-manage del.

.. _martin_kosek_2:

Martin Kosek (2):
^^^^^^^^^^^^^^^^^

-  Fix trustconfig-mod primary group error
-  Require new samba and krb5

.. _petr_viktorin_7:

Petr Viktorin (7):
^^^^^^^^^^^^^^^^^^

-  Display full command documentation in online help
-  Remove 'cn' attribute from idnsRecord and idnsZone objectClasses
-  ipa-server-install: correct help text for --external_{cert,ca}_file
-  Update translations from Transifex
-  Uninstall selfsign CA on upgrade
-  Remove obsolete self-sign references from man pages, docstrings,
   comments
-  Drop --selfsign server functionality

.. _petr_vobornik_6:

Petr Vobornik (6):
^^^^^^^^^^^^^^^^^^

-  Add ipakrbokasdelegate option to service and host Web UI pages
-  Run permission target switch action only for visible widgets
-  Filter groups by type (POSIX, non-POSIX, external)
-  Global trust config page
-  Don't show trusts pages when trust is not configured
-  Fix regression in group type selection in group adder dialog

.. _rob_crittenden_5:

Rob Crittenden (5):
^^^^^^^^^^^^^^^^^^^

-  Fix two failing tests due to missing krb ticket flags
-  Full system backup and restore
-  Apply LDAP update files in blocks of 10, as originally designed.
-  Revert "Fix permission_find test error"
-  Become 3.2.0 Beta 1

.. _tomas_babej_2_1:

Tomas Babej (2):
^^^^^^^^^^^^^^^^

-  Add nfs:NONE to default PAC types only when needed
-  Update only selected attributes for winsync agreement

.. _version_3.2.0_prerelease_1_04022013:

Version 3.2.0 Prerelease 1 (04/02/2013)
=======================================

.. _alexander_bokovoy_7:

Alexander Bokovoy (7):
^^^^^^^^^^^^^^^^^^^^^^

-  Update plugin to upload CA certificate to LDAP
-  ipasam: use base scope when fetching domain information about own
   domain
-  ipaserver/dcerpc: enforce search_s without schema checks for GC
   searching
-  ipa-replica-manage: migrate to single_value after LDAPEntry updates
-  Process exceptions when talking to Dogtag
-  ipasam: add enumeration of UPN suffixes based on the realm domains
-  Enhance ipa-adtrust-install for domains with multiple IPA server

.. _ana_krivokapic_10:

Ana Krivokapic (10):
^^^^^^^^^^^^^^^^^^^^

-  Raise ValidationError for incorrect subtree option.
-  Add crond as a default HBAC service
-  Take into consideration services when deleting replicas
-  Add list of domains associated to our realm to cn=etc
-  Improve error messages for external group members
-  Remove check for alphabetic only characters from domain name
   validation
-  Fix internal error for ipa show-mappings
-  Realm Domains page
-  Use default NETBIOS name in unattended ipa-adtrust-install
-  Add mkhomedir option to ipa-server-install and ipa-replica-install

.. _brian_cook_1_1:

Brian Cook (1):
^^^^^^^^^^^^^^^

-  Add DNS Setup Prompt to Install

.. _jr_aquino_1_1:

JR Aquino (1):
^^^^^^^^^^^^^^

-  Allow PKI-CA Replica Installs when CRL exceeds default maxber value

.. _jakub_hrozek_1_3:

Jakub Hrozek (1):
^^^^^^^^^^^^^^^^^

-  Allow ipa-replica-conncheck and ipa-adtrust-install to read krb5
   includedir

.. _jan_cholasta_24:

Jan Cholasta (24):
^^^^^^^^^^^^^^^^^^

-  Pylint cleanup.
-  Drop ipapython.compat.
-  Add support for RFC 6594 SSHFP DNS records.
-  Raise ValidationError on invalid CSV values.
-  Run interactive_prompt callbacks after CSV values are split.
-  Add custom mapping object for LDAP entry data.
-  Add make_entry factory method to LDAPConnection.
-  Remove the Entity class.
-  Remove the Entry class.
-  Use the dn attribute of LDAPEntry to set/get DNs of entries.
-  Preserve case of attribute names in LDAPEntry.
-  Aggregate IPASimpleLDAPObject in LDAPEntry.
-  Support attributes with multiple names in LDAPEntry.
-  Use full DNs in plugin code.
-  Remove DN normalization from the baseldap plugin.
-  Remove support for DN normalization from LDAPClient.
-  Fix remove while iterating in suppress_netgroup_memberof.
-  Remove disabled entries from sudoers compat tree.
-  Fix internal error in output_for_cli method of
   sudorule_{enable,disable}.
-  Do not fail if schema cannot be retrieved from LDAP server.
-  Allow disabling LDAP schema retrieval in LDAPClient and IPAdmin.
-  Allow disabling attribute decoding in LDAPClient and IPAdmin.
-  Disable schema retrieval and attribute decoding when talking to AD
   GC.
-  Add Kerberos ticket flags management to service and host plugins.

.. _john_dennis_2_1:

John Dennis (2):
^^^^^^^^^^^^^^^^

-  Cookie Expires date should be locale insensitive
-  Use secure method to acquire IPA CA certificate

.. _lynn_root_4:

Lynn Root (4):
^^^^^^^^^^^^^^

-  Switch %r specifiers to '%s' in Public errors
-  Added the ability to do Beta versioning
-  Fixed the catch of the hostname option during ipa-server-install
-  Raise ValidationError when CSR does not have a subject hostname

.. _martin_kosek_58:

Martin Kosek (58):
^^^^^^^^^^^^^^^^^^

-  Add Lynn Root to Contributors.txt
-  Enable SSSD on client install
-  Fix delegation-find command --group handling
-  Do not crash when Kerberos SRV record is not found
-  permission-find no longer crashes with --targetgroup
-  Avoid CRL migration error message
-  Sort LDAP updates properly
-  Upgrade process should not crash on named restart
-  Installer should not connect to 127.0.0.1
-  Fix migration for openldap DS
-  Remove unused krbV imports
-  Use fully qualified CCACHE names
-  Fix permission_find test error
-  Add trusconfig-show and trustconfig-mod commands
-  ipa-kdb: add sentinel for LDAPDerefSpec allocation
-  ipa-kdb: avoid ENOMEM when all SIDs are filtered out
-  ipa-kdb: reinitialize LDAP configuration for known realms
-  Add SID blacklist attributes
-  ipa-kdb: read SID blacklist from LDAP
-  ipa-sam: Fill SID blacklist when trust is added
-  ipa-adtrust-install should ask for SID generation
-  Test NetBIOS name clash before creating a trust
-  Generalize AD GC search
-  Do not hide SID resolver error in group-add-member
-  Add support for AD users to hbactest command
-  Fix hbachelp examples formatting
-  ipa-kdb: remove memory leaks
-  ipa-kdb: fix retry logic in ipadb_deref_search
-  Add autodiscovery section in ipa-client-install man pages
-  Avoid internal error when user is not Trust admin
-  Use fixed test domain in realmdomains test
-  Bump FreeIPA version for development branch
-  Remove ORDERING for IA5 attributeTypes
-  Fix includedir directive in krb5.conf template
-  Use new 389-ds-base cleartext password API
-  Do not hide idrange-add errors when adding trust
-  Preserve order of servers in ipa-client-install
-  Avoid multiple client discovery with fixed server list
-  Update named.conf parser
-  Use tkey-gssapi-keytab in named.conf
-  Do not force named connections on upgrades
-  ipa-client discovery with anonymous access off
-  Use temporary CCACHE in ipa-client-install
-  Improve client install LDAP cert retrieval fallback
-  Configure ipa_dns DS plugin on install and upgrade
-  Fix structured DNS record output
-  Bump selinux-policy requires
-  Clean spec file for Fedora 19
-  Remove build warnings
-  Remove syslog.target from ipa.server
-  Put pid-file to named.conf
-  Update mod_wsgi socket directory
-  Normalize RA agent certificate
-  Require 389-base-base 1.3.0.5
-  Change CNAME and DNAME attributes to single valued
-  Improve CNAME record validation
-  Improve DNAME record validation
-  Become 3.2.0 Prerelease 1

.. _petr_spacek_1_5:

Petr Spacek (1):
^^^^^^^^^^^^^^^^

-  Add 389 DS plugin for special idnsSOASerial attribute handling

.. _petr_viktorin_101:

Petr Viktorin (101):
^^^^^^^^^^^^^^^^^^^^

-  Sort Options and Outputs in API.txt
-  Add the CA cert to LDAP after the CA install
-  Better logging for AdminTool and ipa-ldap-updater
-  Port ipa-replica-prepare to the admintool framework
-  Make ipapython.dogtag log requests at debug level, not info
-  Don't add another nsDS5ReplicaId on updates if one already exists
-  Improve \`ipa --help\` output
-  Print help to stderr on error
-  Store the OptionParser in the API, use it to print unified help
   messages
-  Simplify \`ipa help topics\` output
-  Add command summary to \`ipa COMMAND --help\` output
-  Mention \`ipa COMMAND --help\` as the preferred way to get command
   help
-  Parse command arguments before creating a context
-  Add tests for the help command & --help options
-  In topic help text, mention how to get help for commands
-  Check SSH connection in ipa-replica-conncheck
-  Use ipauniqueid for the RDN of sudo commands
-  Prevent a sudo command from being deleted if it is a member of a sudo
   rule
-  Update sudocmd ACIs to use targetfilter
-  Add the version option to all Commands
-  Add ipalib.messages
-  Add client capabilities, enable messages
-  Rename the "messages" Output of the i18n_messages command to "texts"
-  Fix permission validation and normalization in aci.py
-  Remove csv_separator and csv_skipspace Param arguments
-  Drop support for CSV in the CLI client
-  Update argument docs to reflect dropped CSV support
-  Update plugin docstrings (topic help) to reflect dropped CSV support
-  cli: Do interactive prompting after a context is created
-  Remove some unused imports
-  Remove unused methods from Entry, Entity, and IPAdmin
-  Derive Entity class from Entry, and move it to ldapupdate
-  Use explicit loggers in ldap2 code
-  Move LDAPEntry to ipaserver.ipaldap and derive Entry from it
-  Remove connection-creating code from ShemaCache
-  Move the decision to force schema updates out of IPASimpleLDAPObject
-  Move SchemaCache and IPASimpleLDAPObject to ipaserver.ipaldap
-  Start LDAPConnection, a common base for ldap2 and IPAdmin
-  Make IPAdmin not inherit from IPASimpleLDAPObject
-  Move schema-related methods to LDAPConnection
-  Move DN handling methods to LDAPConnection
-  Move filter making methods to LDAPConnection
-  Move entry finding methods to LDAPConnection
-  Remove unused proxydn functionality from IPAdmin
-  Move entry add, update, remove, rename to LDAPConnection
-  Implement some of IPAdmin's legacy methods in terms of LDAPConnection
   methods
-  Replace setValue by keyword arguments when creating entries
-  Use update_entry with a single entry in adtrustinstance
-  Replace entry.getValues() by entry.get()
-  Replace entry.setValue/setValues by item assignment
-  Replace add_s and delete_s by their newer equivalents
-  Change {add,update,delete}_entry to take LDAPEntries
-  Remove unused imports from ipaserver/install
-  Remove unused bindcert and bindkey arguments to IPAdmin
-  Turn the LDAPError handler into a context manager
-  Remove dbdir, binddn, bindpwd from IPAdmin
-  Remove IPAdmin.updateEntry calls from fix_replica_agreements
-  Remove IPAdmin.get_dns_sorted_by_length
-  Replace IPAdmin.checkTask by replication.wait_for_task
-  Introduce LDAPEntry.single_value for getting single-valued attributes
-  Remove special-casing for missing and single-valued attributes in
   LDAPUpdate._entry_to_entity
-  Replace entry.getValue by entry.single_value
-  Replace getList by a get_entries method
-  Remove toTupleList and attrList from LDAPEntry
-  Rename LDAPConnection to LDAPClient
-  Replace addEntry with add_entry
-  Replace deleteEntry with delete_entry
-  Fix typo and traceback suppression in replication.py
-  replace getEntry with get_entry (or get_entries if scope !=
   SCOPE_BASE)
-  Inline inactivateEntry in its only caller
-  Inline waitForEntry in its only caller
-  Proxy LDAP methods explicitly rather than using \__getattr_\_
-  Remove search_s and search_ext_s from IPAdmin
-  Replace IPAdmin.start_tls_s by an \__init_\_ argument
-  Remove IPAdmin.sasl_interactive_bind_s
-  Remove IPAdmin.simple_bind_s
-  Remove IPAdmin.unbind_s(), keep unbind()
-  Use ldap instead of \_ldap in ipaldap
-  Do not use global variables in migration.py
-  Use IPAdmin rather than raw python-ldap in migration.bind
-  Use IPAdmin rather than raw python-ldap in ipactl
-  Remove some uses of raw python-ldap
-  Improve LDAPEntry tests
-  Fix installing server with external CA
-  Change DNA magic value to -1 to make UID 999 usable
-  Move ipaldap to ipapython
-  Remove ipaserver/ipaldap.py
-  Use IPAdmin rather than raw python-ldap in ipa-client-install
-  Use IPAdmin rather than raw python-ldap in migration.py and
   ipadiscovery.py
-  Remove unneeded python-ldap imports
-  Don't download the schema in ipadiscovery
-  ipa-server-install: Make temporary pin files available for the whole
   installation
-  ipa-server-install: Remove the --selfsign option
-  Remove unused ipapython.certdb.CertDB class
-  ipaserver.install.certs: Introduce NSSDatabase as a more generic
   certutil wrapper
-  Trust CAs from PKCS#12 files even if they don't have Friendly Names
-  dsinstance, httpinstance: Don't hardcode 'Server-Cert'
-  Support installing with custom SSL certs, without a CA
-  Load the CA cert into server NSS databases
-  Do not call cert-\* commands in host plugin if a RA is not available
-  ipa-client-install: Do not request host certificate if server is
   CA-less

.. _petr_vobornik_38:

Petr Vobornik (38):
^^^^^^^^^^^^^^^^^^^

-  Make confirm_dialog a base class of revoke and restore certificate
   dialogs
-  Make confirm_dialog a base class for deleter dialog
-  Make confirm_dialog a base class for message_dialog
-  Confirm mixin
-  Confirm adder dialog by enter
-  Confirm error dialog by enter
-  Focus last dialog when some is closed
-  Confirm association dialogs by enter
-  Standardize login password reset, user reset password and host set
   OTP dialogs
-  Focus first input element after 'Add and Add another'
-  Enable mod_deflate
-  Use Uglify.js for JS optimization
-  Dojo Builder
-  Config files for builder of FreeIPA UI layer
-  Minimal Dojo layer
-  Web UI development environment directory structure and configuration
-  Web UI Sync development utility
-  Move of Web UI non AMD dep. libs to libs subdirectory
-  Move of core Web UI files to AMD directory
-  Update JavaScript Lint configuration file
-  AMD config file
-  Change Web UI sources to simple AMD modules
-  Updated makefiles to build FreeIPA Web UI layer
-  Change tests to use AMD loader
-  Fix BuildRequires: rhino replaced with java-1.7.0-openjdk
-  Develop.js extended
-  Allow to specify modules for which builder doesn't raise dependency
   error
-  Web UI build profile updated
-  Combobox keyboard support
-  Fix dirty state update of editable combobox
-  Fix handling of no_update flag in Web UI
-  Web UI: configurable SID blacklists
-  Web UI:Certificate pages
-  Web UI:Choose different search option for cert-find
-  Fixed Web UI build error caused by rhino changes in F19
-  Nestable checkbox/radio widget
-  Added Web UI support for service PAC type option: NONE
-  Web UI: Disable cert functionality if a CA is not available

.. _rob_crittenden_16:

Rob Crittenden (16):
^^^^^^^^^^^^^^^^^^^^

-  Convert uniqueMember members into DN objects.
-  Add Ana Krivokapic to Contributors.txt
-  Do SSL CA verification and hostname validation.
-  Don't initialize NSS if we don't have to, clean up unused cert refs
-  Update anonymous access ACI to protect secret attributes.
-  Make certmonger a (pre) requires on server, restart it before
   upgrading
-  Use new certmonger locking to prevent NSS database corruption.
-  Improve migration performance
-  Add LDAP server fallback to client installer
-  Prevent a crash when no entries are successfully migrated.
-  Implement the cert-find command for the dogtag CA backend.
-  Add missing v3 schema on upgrades, fix typo in schema.
-  Don't base64-encode the CA cert when uploading it during an upgrade.
-  Extend ipa-replica-manage to be able to manage DNA ranges.
-  Improve some error handling in ipa-replica-manage
-  Fix lockout of LDAP bind.

.. _simo_sorce_2_4:

Simo Sorce (2):
^^^^^^^^^^^^^^^

-  Log info on failure to connect
-  Upload CA cert in the directory on install

.. _sumit_bose_17:

Sumit Bose (17):
^^^^^^^^^^^^^^^^

-  ipa-kdb: remove unused variable
-  ipa-kdb: Uninitialized scalar variable in ipadb_reinit_mspac()
-  ipa-sam: Array compared against 0 in ipasam_set_trusted_domain()
-  ipa-kdb: Dereference after null check in ipa_kdb_mspac.c
-  ipa-lockout: Wrong sizeof argument in ipa_lockout.c
-  ipa-extdom: Double-free in ipa_extdom_common.c
-  ipa-pwd: Unchecked return value ipapwd_chpwop()
-  Revert "MS-PAC: Special case NFS services"
-  Add NFS specific default for authorization data type
-  ipa-kdb: Read global defaul ipaKrbAuthzData
-  ipa-kdb: Read ipaKrbAuthzData with other principal data
-  ipa-kdb: add PAC only if requested
-  Add unit test for get_authz_data_types()
-  Mention PAC issue with NFS in service plugin doc
-  Allow 'nfs:NONE' in global configuration
-  Add support for cmocka C-Unit Test framework
-  ipa-pwd-extop: do not use dn until it is really set

.. _timo_aaltonen_1_1:

Timo Aaltonen (1):
^^^^^^^^^^^^^^^^^^

-  convert the base platform modules into packages

.. _tomas_babej_18:

Tomas Babej (18):
^^^^^^^^^^^^^^^^^

-  Relax restriction for leading/trailing whitespaces in \*-find
   commands
-  Forbid overlapping rid ranges for the same id range
-  Fix a typo in ipa-adtrust-install help
-  Prevent integer overflow when setting krbPasswordExpiration
-  Add option to specify SID using domain name to idrange-add/mod
-  Prevent changing protected group's name using --setattr
-  Use default.conf as flag of IPA client being installed
-  Make sure appropriate exit status is returned in make-test
-  Make options checks in idrange-add/mod consistent
-  Add trusted domain range objectclass when using idrange-mod
-  Perform secondary rid range overlap check for local ranges only
-  Add support for re-enrolling hosts using keytab
-  Make sure uninstall script prompts for reboot as last
-  Remove implicit Str to DN conversion using \*-attr
-  Enforce exact SID match when adding or modifying a ID range
-  Allow host re-enrollment using delegation
-  Add logging to join command
-  Properly handle ipa-replica-install when its zone is not managed by
   IPA

.. _sumit_bose_1_4:

Sumit Bose (1):
^^^^^^^^^^^^^^^

-  ipa-kdb: Free talloc autofree context when module is closed

.. _version_3.1.5_06032013:

Version 3.1.5 (06/03/2013)
==========================

====Alexander Bokovoy (1)

-  Fix cldap parser to work with a single equality filter (NtVer=...)

.. _martin_kosek_1:

Martin Kosek (1):
^^^^^^^^^^^^^^^^^

-  Become IPA 3.1.5

.. _petr_viktorin_1_1:

Petr Viktorin (1):
^^^^^^^^^^^^^^^^^^

-  Remove leading zero from IPA_NUM_VERSION

.. _simo_sorce_2_5:

Simo Sorce (2):
^^^^^^^^^^^^^^^

-  CLDAP: Fix domain handling in netlogon requests
-  CLDAP: Return empty reply on non-fatal errors

.. _version_3.1.4_05072013:

Version 3.1.4 (05/07/2013)
==========================

.. _alexander_bokovoy_1_3:

Alexander Bokovoy (1):
^^^^^^^^^^^^^^^^^^^^^^

-  Enhance ipa-adtrust-install for domains with multiple IPA server

.. _ana_krivokapic_8_1:

Ana Krivokapic (8):
^^^^^^^^^^^^^^^^^^^

-  Add mkhomedir option to ipa-server-install and ipa-replica-install
-  Remove CA cert on client uninstall
-  Remove HBAC source hosts from web UI
-  Remove any reference to HBAC source hosts from help
-  Deprecate HBAC source hosts from CLI
-  Handle missing /etc/ipa in ipa-client-install
-  Fix the spec file
-  Add missing permissions to Host Administrators privilege

.. _jan_cholasta_7:

Jan Cholasta (7):
^^^^^^^^^^^^^^^^^

-  Do actually stop pki_cad in stop_pkicad instead of starting it.
-  Use only one URL for OCSP and CRL in IPA certificate profile.
-  Use A/AAAA records instead of CNAME records in ipa-ca.
-  Delete DNS records in ipa-ca on ipa-csreplica-manage del.
-  Do not use new LDAP API in old code.
-  Use correct zone when removing DNS records of a master.
-  Add support for OpenSSH 6.2.

.. _martin_kosek_4_1:

Martin Kosek (4):
^^^^^^^^^^^^^^^^^

-  Require 389-base-base 1.3.0.5
-  Add userClass attribute for hosts
-  Update pki proxy configuration
-  Become IPA 3.1.4

.. _petr_viktorin_2:

Petr Viktorin (2):
^^^^^^^^^^^^^^^^^^

-  Display full command documentation in online help
-  Use two digits for each part of NUM_VERSION

.. _rob_crittenden_3:

Rob Crittenden (3):
^^^^^^^^^^^^^^^^^^^

-  Handle socket.gethostbyaddr() exceptions when verifying hostnames.
-  Drop uniqueMember mapping with nss-pam-ldapd.
-  Specify the location for the agent PKCS#12 file so we don't have to
   move it.

.. _sumit_bose_1_5:

Sumit Bose (1):
^^^^^^^^^^^^^^^

-  ipa-pwd-extop: do not use dn until it is really set

.. _tomas_babej_2_2:

Tomas Babej (2):
^^^^^^^^^^^^^^^^

-  Properly handle ipa-replica-install when its zone is not managed by
   IPA
-  Allow underscore in record targets

.. _version_3.1.3_03262013:

Version 3.1.3 (03/26/2013)
==========================

.. _alexander_bokovoy_2:

Alexander Bokovoy (2):
^^^^^^^^^^^^^^^^^^^^^^

-  ipasam: use base scope when fetching domain information about own
   domain
-  Process exceptions when talking to Dogtag

.. _ana_krivokapic_6:

Ana Krivokapic (6):
^^^^^^^^^^^^^^^^^^^

-  Take into consideration services when deleting replicas
-  Add list of domains associated to our realm to cn=etc
-  Remove check for alphabetic only characters from domain name
   validation
-  Fix internal error for ipa show-mappings
-  Realm Domains page
-  Use default NETBIOS name in unattended ipa-adtrust-install

.. _jakub_hrozek_1_4:

Jakub Hrozek (1):
^^^^^^^^^^^^^^^^^

-  Allow ipa-replica-conncheck and ipa-adtrust-install to read krb5
   includedir

.. _jan_cholasta_6:

Jan Cholasta (6):
^^^^^^^^^^^^^^^^^

-  Pylint cleanup.
-  Raise ValidationError on invalid CSV values.
-  Run interactive_prompt callbacks after CSV values are split.
-  Fix remove while iterating in suppress_netgroup_memberof.
-  Remove disabled entries from sudoers compat tree.
-  Fix internal error in output_for_cli method of
   sudorule_{enable,disable}.

.. _martin_kosek_33:

Martin Kosek (33):
^^^^^^^^^^^^^^^^^^

-  Fix migration for openldap DS
-  Remove unused krbV imports
-  Use fully qualified CCACHE names
-  Fix permission_find test error
-  Add trusconfig-show and trustconfig-mod commands
-  ipa-kdb: add sentinel for LDAPDerefSpec allocation
-  ipa-kdb: avoid ENOMEM when all SIDs are filtered out
-  ipa-kdb: reinitialize LDAP configuration for known realms
-  Add SID blacklist attributes
-  ipa-kdb: read SID blacklist from LDAP
-  ipa-sam: Fill SID blacklist when trust is added
-  ipa-adtrust-install should ask for SID generation
-  Test NetBIOS name clash before creating a trust
-  Generalize AD GC search
-  Do not hide SID resolver error in group-add-member
-  Add support for AD users to hbactest command
-  Fix hbachelp examples formatting
-  ipa-kdb: remove memory leaks
-  ipa-kdb: fix retry logic in ipadb_deref_search
-  Add autodiscovery section in ipa-client-install man pages
-  Avoid internal error when user is not Trust admin
-  Use fixed test domain in realmdomains test
-  Remove ORDERING for IA5 attributeTypes
-  Fix includedir directive in krb5.conf template
-  Preserve order of servers in ipa-client-install
-  Avoid multiple client discovery with fixed server list
-  Fix client discovery crash
-  ipa-client discovery with anonymous access off
-  Use temporary CCACHE in ipa-client-install
-  Improve client install LDAP cert retrieval fallback
-  Configure ipa_dns DS plugin on install and upgrade
-  Bump selinux-policy requires
-  Become 3.1.3

.. _petr_spacek_1_6:

Petr Spacek (1):
^^^^^^^^^^^^^^^^

-  Add 389 DS plugin for special idnsSOASerial attribute handling

.. _petr_viktorin_23:

Petr Viktorin (23):
^^^^^^^^^^^^^^^^^^^

-  Add the CA cert to LDAP after the CA install
-  Port ipa-replica-prepare to the admintool framework
-  Don't add another nsDS5ReplicaId on updates if one already exists
-  Improve \`ipa --help\` output
-  Print help to stderr on error
-  Store the OptionParser in the API, use it to print unified help
   messages
-  Simplify \`ipa help topics\` output
-  Add command summary to \`ipa COMMAND --help\` output
-  Mention \`ipa COMMAND --help\` as the preferred way to get command
   help
-  Parse command arguments before creating a context
-  Add tests for the help command & --help options
-  In topic help text, mention how to get help for commands
-  Check SSH connection in ipa-replica-conncheck
-  Use ipauniqueid for the RDN of sudo commands
-  Prevent a sudo command from being deleted if it is a member of a sudo
   rule
-  Update sudocmd ACIs to use targetfilter
-  Add the version option to all Commands
-  Add ipalib.messages
-  Add client capabilities, enable messages
-  Rename the "messages" Output of the i18n_messages command to "texts"
-  Fix permission validation and normalization in aci.py
-  cli: Do interactive prompting after a context is created
-  Fix installing server with external CA

.. _petr_vobornik_36:

Petr Vobornik (36):
^^^^^^^^^^^^^^^^^^^

-  Make confirm_dialog a base class of revoke and restore certificate
   dialogs
-  Make confirm_dialog a base class for deleter dialog
-  Make confirm_dialog a base class for message_dialog
-  Confirm mixin
-  Confirm adder dialog by enter
-  Confirm error dialog by enter
-  Focus last dialog when some is closed
-  Confirm association dialogs by enter
-  Standardize login password reset, user reset password and host set
   OTP dialogs
-  Focus first input element after 'Add and Add another'
-  Enable mod_deflate
-  Use Uglify.js for JS optimization
-  Dojo Builder
-  Config files for builder of FreeIPA UI layer
-  Minimal Dojo layer
-  Web UI development environment directory structure and configuration
-  Web UI Sync development utility
-  Move of Web UI non AMD dep. libs to libs subdirectory
-  Move of core Web UI files to AMD directory
-  Update JavaScript Lint configuration file
-  AMD config file
-  Change Web UI sources to simple AMD modules
-  Updated makefiles to build FreeIPA Web UI layer
-  Change tests to use AMD loader
-  Fix BuildRequires: rhino replaced with java-1.7.0-openjdk
-  Develop.js extended
-  Allow to specify modules for which builder doesn't raise dependency
   error
-  Web UI build profile updated
-  Combobox keyboard support
-  Fix dirty state update of editable combobox
-  Fix handling of no_update flag in Web UI
-  Web UI: configurable SID blacklists
-  Web UI:Certificate pages
-  Web UI:Choose different search option for cert-find
-  Added Web UI support for service PAC type option: NONE
-  Load extension.js after UI AMD modules.

.. _rob_crittenden_10:

Rob Crittenden (10):
^^^^^^^^^^^^^^^^^^^^

-  Make certmonger a (pre) requires on server, restart it before
   upgrading
-  Use new certmonger locking to prevent NSS database corruption.
-  Better logging for AdminTool and ipa-ldap-updater
-  Improve migration performance
-  Add LDAP server fallback to client installer
-  Prevent a crash when no entries are successfully migrated.
-  Implement the cert-find command for the dogtag CA backend.
-  Add missing v3 schema on upgrades, fix typo in schema.
-  Don't base64-encode the CA cert when uploading it during an upgrade.
-  Improve some error handling in ipa-replica-manage

.. _sumit_bose_7:

Sumit Bose (7):
^^^^^^^^^^^^^^^

-  ipa-kdb: remove unused variable
-  ipa-kdb: Uninitialized scalar variable in ipadb_reinit_mspac()
-  ipa-sam: Array compared against 0 in ipasam_set_trusted_domain()
-  ipa-kdb: Dereference after null check in ipa_kdb_mspac.c
-  ipa-lockout: Wrong sizeof argument in ipa_lockout.c
-  ipa-extdom: Double-free in ipa_extdom_common.c
-  ipa-pwd: Unchecked return value ipapwd_chpwop()

.. _tomas_babej_13:

Tomas Babej (13):
^^^^^^^^^^^^^^^^^

-  Fix a typo in ipa-adtrust-install help
-  Prevent integer overflow when setting krbPasswordExpiration
-  Add option to specify SID using domain name to idrange-add/mod
-  Prevent changing protected group's name using --setattr
-  Use default.conf as flag of IPA client being installed
-  Make sure appropriate exit status is returned in make-test
-  Make options checks in idrange-add/mod consistent
-  Add trusted domain range objectclass when using idrange-mod
-  Perform secondary rid range overlap check for local ranges only
-  Make sure uninstall script prompts for reboot as last
-  Remove implicit Str to DN conversion using \*-attr
-  Enforce exact SID match when adding or modifying a ID range
-  Add logging to join command

.. _sumit_bose_1_6:

Sumit Bose (1):
^^^^^^^^^^^^^^^

-  ipa-kdb: Free talloc autofree context when module is closed

.. _version_3.1.2_01232013:

Version 3.1.2 (01/23/2013)
==========================

.. _alexander_bokovoy_1_4:

Alexander Bokovoy (1):
^^^^^^^^^^^^^^^^^^^^^^

-  Update plugin to upload CA certificate to LDAP

.. _ana_krivokapic_1:

Ana Krivokapic (1):
^^^^^^^^^^^^^^^^^^^

-  Raise ValidationError for incorrect subtree option.

.. _john_dennis_1:

John Dennis (1):
^^^^^^^^^^^^^^^^

-  Use secure method to acquire IPA CA certificate

.. _martin_kosek_5_1:

Martin Kosek (5):
^^^^^^^^^^^^^^^^^

-  permission-find no longer crashes with --targetgroup
-  Avoid CRL migration error message
-  Sort LDAP updates properly
-  Upgrade process should not crash on named restart
-  Installer should not connect to 127.0.0.1

.. _rob_crittenden_5_1:

Rob Crittenden (5):
^^^^^^^^^^^^^^^^^^^

-  Convert uniqueMember members into DN objects.
-  Do SSL CA verification and hostname validation.
-  Don't initialize NSS if we don't have to, clean up unused cert refs
-  Update anonymous access ACI to protect secret attributes.
-  Become IPA 3.1.2

.. _simo_sorce_1_1:

Simo Sorce (1):
^^^^^^^^^^^^^^^

-  Upload CA cert in the directory on install

.. _version_3.1.1_01082013:

Version 3.1.1 (01/08/2013)
==========================

.. _jr_aquino_1_2:

JR Aquino (1):
^^^^^^^^^^^^^^

-  Allow PKI-CA Replica Installs when CRL exceeds default maxber value

.. _john_dennis_1_1:

John Dennis (1):
^^^^^^^^^^^^^^^^

-  Cookie Expires date should be locale insensitive

.. _lynn_root_2:

Lynn Root (2):
^^^^^^^^^^^^^^

-  Fixed the catch of the hostname option during ipa-server-install
-  Raise ValidationError when CSR does not have a subject hostname

.. _martin_kosek_4_2:

Martin Kosek (4):
^^^^^^^^^^^^^^^^^

-  Add Lynn Root to Contributors.txt
-  Enable SSSD on client install
-  Fix delegation-find command --group handling
-  Do not crash when Kerberos SRV record is not found

.. _rob_crittenden_1_2:

Rob Crittenden (1):
^^^^^^^^^^^^^^^^^^^

-  Become IPA 3.1.1

.. _simo_sorce_1_2:

Simo Sorce (1):
^^^^^^^^^^^^^^^

-  Log info on failure to connect

.. _tomas_babej_2_3:

Tomas Babej (2):
^^^^^^^^^^^^^^^^

-  Relax restriction for leading/trailing whitespaces in \*-find
   commands
-  Forbid overlapping rid ranges for the same id range

.. _version_3.0.2_12112012:

Version 3.0.2 (12/11/2012)
==========================

.. _alexander_bokovoy_3:

Alexander Bokovoy (3):
^^^^^^^^^^^^^^^^^^^^^^

-  ipasam: better Kerberos error handling in ipasam
-  trusts: replace use of python-crypto by m2crypto
-  Propagate kinit errors with trust account

.. _jakub_hrozek_4:

Jakub Hrozek (4):
^^^^^^^^^^^^^^^^^

-  Make enabling the autofs service more robust
-  ipachangeconf: allow specifying non-default delimeter for options
-  Specify includedir in krb5.conf on new installs
-  Add the includedir to krb5.conf on upgrades

.. _john_dennis_1_2:

John Dennis (1):
^^^^^^^^^^^^^^^^

-  Compliant client side session cookie behavior

.. _lubomir_rintel_1:

Lubomir Rintel (1):
^^^^^^^^^^^^^^^^^^^

-  Drop unused readline import

.. _martin_kosek_5_2:

Martin Kosek (5):
^^^^^^^^^^^^^^^^^

-  Prepare spec file for Fedora 18
-  Filter suffix in replication management tools
-  Change network configuration file
-  Improve ipa-replica-prepare error message
-  Fix sshd feature check

.. _petr_viktorin_2_1:

Petr Viktorin (2):
^^^^^^^^^^^^^^^^^^

-  Provide explicit user name for Dogtag installation scripts
-  Add Lubomir Rintel to Contributors.txt

.. _petr_vobornik_4:

Petr Vobornik (4):
^^^^^^^^^^^^^^^^^^

-  WebUI: Change of default value of type of new group back to POSIX
-  Editable sshkey, mac address field after upgrade
-  Better licensing information of 3rd party code
-  Better error message for login of users from other realms

.. _rob_crittenden_5_2:

Rob Crittenden (5):
^^^^^^^^^^^^^^^^^^^

-  Honor the kdb options disabling KDC writes in ipa_lockout plugin
-  Only update the list of running services in the installer or ipactl.
-  Set min for selinux-policy to 3.11.1-60
-  Reorder XML-RPC initialization in ipa-join to avoid segfault.
-  Become IPA 3.0.2

.. _simo_sorce_1_3:

Simo Sorce (1):
^^^^^^^^^^^^^^^

-  MS-PAC: Special case NFS services

.. _sumit_bose_3_1:

Sumit Bose (3):
^^^^^^^^^^^^^^^

-  Lookup the user SID in external group as well
-  Restart sssd after authconfig update
-  Do not recommend how to configure DNS in error message

.. _tomas_babej_1:

Tomas Babej (1):
^^^^^^^^^^^^^^^^

-  Add detection for users from trusted/invalid realms

.. _version_3.1.0_12102012:

Version 3.1.0 (12/10/2012)
==========================

.. _ade_lee_1:

Ade Lee (1):
^^^^^^^^^^^^

-  Changes to use a single database for dogtag and IPA

.. _alexander_bokovoy_8_1:

Alexander Bokovoy (8):
^^^^^^^^^^^^^^^^^^^^^^

-  ipa-kdb: Support Windows 2012 Server
-  Remove bogus check for smbpasswd
-  Warn about DNA plugin configuration when working with local ID ranges
-  Resolve external members from trusted domain via Global Catalog
-  Clarify trust-add help regarding multiple runs against the same
   domain
-  ipasam: better Kerberos error handling in ipasam
-  trusts: replace use of python-crypto by m2crypto
-  Propagate kinit errors with trust account

.. _endi_sukma_dewata_1:

Endi Sukma Dewata (1):
^^^^^^^^^^^^^^^^^^^^^^

-  Configuring CA with ConfigParser.

.. _jakub_hrozek_5:

Jakub Hrozek (5):
^^^^^^^^^^^^^^^^^

-  ipa-client-automount: Add the autofs service if it doesn't exist yet
-  Make enabling the autofs service more robust
-  ipachangeconf: allow specifying non-default delimeter for options
-  Specify includedir in krb5.conf on new installs
-  Add the includedir to krb5.conf on upgrades

.. _jan_cholasta_1_1:

Jan Cholasta (1):
^^^^^^^^^^^^^^^^^

-  Reword description of the --passsync option of ipa-replica-manage.

.. _john_dennis_2_2:

John Dennis (2):
^^^^^^^^^^^^^^^^

-  log dogtag errors
-  Compliant client side session cookie behavior

.. _lubomir_rintel_1_1:

Lubomir Rintel (1):
^^^^^^^^^^^^^^^^^^^

-  Drop unused readline import

.. _martin_kosek_18:

Martin Kosek (18):
^^^^^^^^^^^^^^^^^^

-  Update SELinux policy for dogtag10
-  Bump 389-ds-base minimum in our spec file
-  Add OCSP and CRL URIs to certificates
-  Stop and disable conflicting time&date services
-  Create reverse zone in unattended mode
-  Add fallback for httpd restarts on sysV platforms
-  Report ipa-upgradeconfig errors during RPM upgrade
-  Avoid uninstalling dependencies during package lifetime
-  Remove servertrls and clientctrls options from rename_s
-  Use common encoding in modlist generation
-  Process relative nameserver DNS record correctly
-  Do not require resolvable nameserver in DNS install
-  Disable global forwarding per-zone
-  Prepare spec file for Fedora 18
-  Filter suffix in replication management tools
-  Change network configuration file
-  Improve ipa-replica-prepare error message
-  Fix sshd feature check

.. _nikolai_kondrashov_1:

Nikolai Kondrashov (1):
^^^^^^^^^^^^^^^^^^^^^^^

-  Add uninstall command hints to ipa-*-instal

.. _petr_viktorin_12:

Petr Viktorin (12):
^^^^^^^^^^^^^^^^^^^

-  Fix schema replication from old masters
-  Use correct Dogtag configuration in get_pin and get_ca_certchain
-  Update certmap.conf on IPA upgrades
-  Properly stop tracking certificates on uninstall
-  Provide 'protocol' argument to IPAdmin
-  Make ipa-csreplica-manage work with both merged and non-merged DBs
-  Use DN objects for Dogtag configuration
-  ipautil.run: Log the command line before running the command
-  ipa-replica-install: Use configured IPA DNS servers in
   forward/reverse resolution check
-  Make sure the CA is running when starting services
-  Provide explicit user name for Dogtag installation scripts
-  Add Lubomir Rintel to Contributors.txt

.. _petr_vobornik_7_2:

Petr Vobornik (7):
^^^^^^^^^^^^^^^^^^

-  Simpler instructions to generate certificate
-  Fixed incorrect link to browser config after session expiration
-  Web UI: disable global forwarding per zone
-  WebUI: Change of default value of type of new group back to POSIX
-  Editable sshkey, mac address field after upgrade
-  Better licensing information of 3rd party code
-  Better error message for login of users from other realms

.. _rob_crittenden_16_1:

Rob Crittenden (16):
^^^^^^^^^^^^^^^^^^^^

-  Enable transactions by default, make password and modrdn TXN-aware
-  Become IPA 3.1.0
-  Password change in a transaction, ensure passwords are truly expired
-  Don't configure a reverse zone if not desired in interactive
   installer.
-  Fix requesting certificates that contain subject altnames.
-  Improve error messages in ipa-replica-manage.
-  Close connection after each request, avoid NSS shutdown problem.
-  The SECURE_NFS value needs to be lower-case yes on SysV systems.
-  After unininstall see if certmonger is still tracking any of our
   certs.
-  Wait for the directory server to come up when updating the agent
   certificate.
-  Set MLS/MCS for user_u context to what will be on remote systems.
-  Handle the case where there are no replicas with list-ruv
-  Honor the kdb options disabling KDC writes in ipa_lockout plugin
-  Only update the list of running services in the installer or ipactl.
-  Set min for selinux-policy to 3.11.1-60
-  Reorder XML-RPC initialization in ipa-join to avoid segfault.

.. _simo_sorce_7:

Simo Sorce (7):
^^^^^^^^^^^^^^^

-  Add support for using AES for cross-realm TGTs
-  Preserve original service_name in services
-  Save service name on service startup
-  Get list of service from LDAP only at startup
-  Revert "Save service name on service startup"
-  Save service name on service startup/shutdown
-  MS-PAC: Special case NFS services

.. _sumit_bose_7_1:

Sumit Bose (7):
^^^^^^^^^^^^^^^

-  Fix various issues found by Coverity
-  extdom: handle INP_POSIX_UID and INP_POSIX_GID requests
-  Restart httpd if ipa-server-trust-ad is installed or updated
-  ipa-adtrust-install: allow to reset te NetBIOS domain name
-  Lookup the user SID in external group as well
-  Restart sssd after authconfig update
-  Do not recommend how to configure DNS in error message

.. _tomas_babej_5:

Tomas Babej (5):
^^^^^^^^^^^^^^^^

-  Forbid overlapping primary and secondary rid ranges
-  Refactoring of default.conf man page
-  Make service naming in ipa-server-install consistent
-  IPA Server check in ipa-replica-manage
-  Add detection for users from trusted/invalid realms

.. _version_3.0.1_11092012:

Version 3.0.1 (11/09/2012)
==========================

.. _alexander_bokovoy_4:

Alexander Bokovoy (4):
^^^^^^^^^^^^^^^^^^^^^^

-  Remove bogus check for smbpasswd
-  Warn about DNA plugin configuration when working with local ID ranges
-  Resolve external members from trusted domain via Global Catalog
-  Clarify trust-add help regarding multiple runs against the same
   domain

.. _jakub_hrozek_1_5:

Jakub Hrozek (1):
^^^^^^^^^^^^^^^^^

-  ipa-client-automount: Add the autofs service if it doesn't exist yet

.. _jan_cholasta_1_2:

Jan Cholasta (1):
^^^^^^^^^^^^^^^^^

-  Reword description of the --passsync option of ipa-replica-manage.

.. _john_dennis_1_3:

John Dennis (1):
^^^^^^^^^^^^^^^^

-  log dogtag errors

.. _martin_kosek_9:

Martin Kosek (9):
^^^^^^^^^^^^^^^^^

-  Create reverse zone in unattended mode
-  Add fallback for httpd restarts on sysV platforms
-  Report ipa-upgradeconfig errors during RPM upgrade
-  Avoid uninstalling dependencies during package lifetime
-  Remove servertrls and clientctrls options from rename_s
-  Use common encoding in modlist generation
-  Process relative nameserver DNS record correctly
-  Do not require resolvable nameserver in DNS install
-  Disable global forwarding per-zone

.. _nikolai_kondrashov_1_1:

Nikolai Kondrashov (1):
^^^^^^^^^^^^^^^^^^^^^^^

-  Add uninstall command hints to ipa-*-install

.. _petr_viktorin_3:

Petr Viktorin (3):
^^^^^^^^^^^^^^^^^^

-  ipautil.run: Log the command line before running the command
-  ipa-replica-install: Use configured IPA DNS servers in
   forward/reverse resolution check
-  Make sure the CA is running when starting services

.. _petr_vobornik_2:

Petr Vobornik (2):
^^^^^^^^^^^^^^^^^^

-  Simpler instructions to generate certificate
-  Fixed incorrect link to browser config after session expiration

.. _rob_crittenden_11:

Rob Crittenden (11):
^^^^^^^^^^^^^^^^^^^^

-  Use TLS for CA replication
-  Don't configure a reverse zone if not desired in interactive
   installer.
-  Fix requesting certificates that contain subject altnames.
-  Improve error messages in ipa-replica-manage.
-  Close connection after each request, avoid NSS shutdown problem.
-  The SECURE_NFS value needs to be lower-case yes on SysV systems.
-  After unininstall see if certmonger is still tracking any of our
   certs.
-  Wait for the directory server to come up when updating the agent
   certificate.
-  Set MLS/MCS for user_u context to what will be on remote systems.
-  Handle the case where there are no replicas with list-ruv
-  Become IPA 3.0.1

.. _simo_sorce_6:

Simo Sorce (6):
^^^^^^^^^^^^^^^

-  Add support for using AES fo cross-realm TGTs
-  Preserve original service_name in services
-  Save service name on service startup
-  Get list of service from LDAP only at startup
-  Revert "Save service name on service startup"
-  Save service name on service startup/shutdown

.. _sumit_bose_4:

Sumit Bose (4):
^^^^^^^^^^^^^^^

-  Fix various issues found by Coverity
-  extdom: handle INP_POSIX_UID and INP_POSIX_GID requests
-  Restart httpd if ipa-server-trust-ad is installed or updated
-  ipa-adtrust-install: allow to reset te NetBIOS domain name

.. _tomas_babej_4:

Tomas Babej (4):
^^^^^^^^^^^^^^^^

-  Forbid overlapping primary and secondary rid ranges
-  Refactoring of default.conf man page
-  Make service naming in ipa-server-install consistent
-  IPA Server check in ipa-replica-manage

.. _version_3.0.0_ga_10152012:

Version 3.0.0 GA (10/15/2012)
=============================

.. _alexander_bokovoy_7_1:

Alexander Bokovoy (7):
^^^^^^^^^^^^^^^^^^^^^^

-  support multi-line error messages in exceptions
-  Handle NotFound exception when establishing trust
-  Fix wrong RID for Domain Admins in the examples of trust commands
-  Add cifs principal to S4U2Proxy targets only when running
   ipa-adtrust-install
-  Make sure samba{,4}-winbind-krb5-locator package is not used with
   trusts
-  Add instructions support to PublicError
-  Use PublicError instructions support for trust-add case when domain
   is not found

.. _jan_cholasta_1_3:

Jan Cholasta (1):
^^^^^^^^^^^^^^^^^

-  Do not show full SSH public keys in command output by default.

.. _martin_kosek_3_1:

Martin Kosek (3):
^^^^^^^^^^^^^^^^^

-  Minor fixes for default SMB group
-  Move CRL publish directory to IPA owned directory
-  Fix CA CRL migration crash in ipa-upgradeconfig

.. _petr_viktorin_4_1:

Petr Viktorin (4):
^^^^^^^^^^^^^^^^^^

-  ipa-upgradeconfig: Remove the upgrade_httpd_selinux function
-  replica-install: Don't copy Firefox config extension files if they're
   not in the replica file
-  Create Firefox extension on upgrade and replica-install
-  Pull translation files from Transifex

.. _petr_vobornik_1_1:

Petr Vobornik (1):
^^^^^^^^^^^^^^^^^^

-  Add mime type to httpd ipa.conf for xpi exetension

.. _rob_crittenden_6:

Rob Crittenden (6):
^^^^^^^^^^^^^^^^^^^

-  Add uniqueness plugin configuration for sudorule cn
-  Set renewal time for the CA audit certificate to 720 days.
-  Fix CS replication management.
-  Configure the initial CA as the CRL generator.
-  Explicitly disable betxn plugins for the time being.
-  Become IPA 3.0.0

.. _simo_sorce_2_6:

Simo Sorce (2):
^^^^^^^^^^^^^^^

-  Fix trust attributes for ipa trust-add
-  Use stricter requirement for krb5-server

.. _sumit_bose_2:

Sumit Bose (2):
^^^^^^^^^^^^^^^

-  ipa-adtrust-install: create fallback group with ldif file
-  ipadb: reload trust information if domain is not known

.. _tomas_babej_1_1:

Tomas Babej (1):
^^^^^^^^^^^^^^^^

-  Notify user about necessary ports in ipa-client-install

.. _version_3.0.0_rc_2_10082012:

Version 3.0.0 RC 2 (10/08/2012)
===============================

.. _alexander_bokovoy_3_1:

Alexander Bokovoy (3):
^^^^^^^^^^^^^^^^^^^^^^

-  Make sure external group members are listed for the external group
-  Change the way SID comparison is done for belonging to trusted domain
-  Support python-ldap 2.3 way of making LDAP control

.. _martin_kosek_9_1:

Martin Kosek (9):
^^^^^^^^^^^^^^^^^

-  Use custom zonemgr for reverse zones
-  Validate SELinux users in config-mod
-  Improve StrEnum validation error message
-  Add support for unified samba packages
-  Improve DN usage in ipa-client-install
-  Index ipakrbprincipalalias and ipaautomountkey attributes
-  Do not produce unindexed search on every DEL command
-  Only use service PAC type as an override
-  Fill ipakrbprincipalalias on upgrades

.. _petr_viktorin_4_2:

Petr Viktorin (4):
^^^^^^^^^^^^^^^^^^

-  Always handle NotFound error in dnsrecord-mod
-  Don't use bare except: clauses in ipa-client-install
-  Fix NS records in installation
-  Wait for secure Dogtag ports when starting the pki services

.. _petr_vobornik_5_2:

Petr Vobornik (5):
^^^^^^^^^^^^^^^^^^

-  Kerberos authentication extension
-  Kerberos authentication extension makefiles
-  Build and installation of Kerberos authentication extension
-  Configuration pages changed to use new FF extension
-  Removal of delegation-uris instruction from browser config

.. _rob_crittenden_3_1:

Rob Crittenden (3):
^^^^^^^^^^^^^^^^^^^

-  Fix python syntax in ipa-client-automount
-  Clear kernel keyring in client installer, save dbdir on new
   connections
-  Become IPA v3 RC 2 (3.0.0.rc2)

.. _sumit_bose_12:

Sumit Bose (12):
^^^^^^^^^^^^^^^^

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

.. _tomas_babej_2_4:

Tomas Babej (2):
^^^^^^^^^^^^^^^^

-  Improve user addition to default group in user-add
-  Restrict admins group modifications

.. _version_3.0.0_rc_1_09262012:

Version 3.0.0 RC 1 (09/26/2012)
===============================

.. _ade_lee_1_1:

Ade Lee (1):
^^^^^^^^^^^^

-  Modifications to install scripts for dogtag 10

.. _alexander_bokovoy_5:

Alexander Bokovoy (5):
^^^^^^^^^^^^^^^^^^^^^^

-  Add verification of the AD trust
-  validate SID for trusted domain when adding/modifying ID range
-  Fix error messages and use proper ImportError for dcerpc import
-  Add documentation for 'ipa trust' set of commands
-  Document use of external group membership

.. _jan_cholasta_3_1:

Jan Cholasta (3):
^^^^^^^^^^^^^^^^^

-  Add the SSH service to SSSD config file before trying to activate it.
-  Add --no-ssh option to ipa-client-install to disable OpenSSH client
   configuration.
-  SSHPublicKey.fingerprint_dns_sha1 should return unicode value.

.. _martin_kosek_8:

Martin Kosek (8):
^^^^^^^^^^^^^^^^^

-  Fix addattr internal error
-  Add attributeTypes to safe schema updater
-  Amend memberAllowCmd and memberDenyCmd attribute types
-  Run index task in ldap updater only when needed
-  Expand Referential Integrity checks
-  Properly convert DN in ipa-client-install
-  Use default reverse zone consistently
-  Fix idrange plugin help

.. _petr_viktorin_7_1:

Petr Viktorin (7):
^^^^^^^^^^^^^^^^^^

-  ipa-client-install: Obtain host TGT from one specific KDC
-  Fix server installation
-  Use temporary key cache for host key in server installation
-  Update the pot file (translation source)
-  Use Dogtag 10 only when it is available
-  Only stop the main DS instance when upgrading it
-  Use correct Dogtag port in ipaserver.install.certs

.. _petr_vobornik_4_1:

Petr Vobornik (4):
^^^^^^^^^^^^^^^^^^

-  Prevent opening of multiple dirty dialogs on navigation
-  JSON serialization of long type
-  Show trust status in add success notification
-  Fix integer validation when boundary value is empty string

.. _rob_crittenden_3_2:

Rob Crittenden (3):
^^^^^^^^^^^^^^^^^^^

-  Set SELinux default context to unconfined_u:s0-s0:c0.c1023
-  Run the CLEANALLRUV task when deleting a replication agreement.
-  When deleting a master, try to prevent orphaning other servers.

.. _sumit_bose_3_2:

Sumit Bose (3):
^^^^^^^^^^^^^^^

-  ipasam: Fixes build with samba4 rc1
-  Set master_kdc and dns_lookup_kdc to true
-  Update krb5.conf during ipa-adtrust-install

.. _tomas_babej_2_5:

Tomas Babej (2):
^^^^^^^^^^^^^^^^

-  Make sure selinuxusemap behaves consistently to HBAC rule
-  Improves sssd.conf handling during ipa-client uninstall

.. _yuri_chornoivan_1:

Yuri Chornoivan (1):
^^^^^^^^^^^^^^^^^^^^

-  Fix various typos.

.. _version_3.0.0_beta_3_09102012:

Version 3.0.0 Beta 3 (09/10/2012)
=================================

.. _alexander_bokovoy_4_1:

Alexander Bokovoy (4):
^^^^^^^^^^^^^^^^^^^^^^

-  Recover from invalid cached kerberos credentials in ipasam
-  Fix ipasam ipaNThash magic regen to actually fetch updated password
-  Add ACI to allow regenerating ipaNTHash from ipasam
-  Ask for admin password in ipa-adtrust-install

.. _jan_cholasta_1_4:

Jan Cholasta (1):
^^^^^^^^^^^^^^^^^

-  Use OpenSSH-style public keys as the preferred format of SSH public
   keys.

.. _john_dennis_4:

John Dennis (4):
^^^^^^^^^^^^^^^^

-  DN objects hash differently depending on case
-  ipactl exception not handled well
-  ipa user-find --manager does not find matches
-  prevent last admin from being disabled

.. _martin_kosek_12:

Martin Kosek (12):
^^^^^^^^^^^^^^^^^^

-  Read DM password from option in external CA install
-  Fix client-only build
-  Fix managedBy label for DNS zone
-  Update Contributors.txt file
-  Make replica install more robust
-  Add safe updates for objectClasses
-  Allow localhost in zone ACIs
-  Transfer long numbers over XMLRPC
-  Fix DNS SOA serial parameters boundaries
-  Add range safety check for range_mod and range_del
-  Update DNS zone allow-query validation test
-  Cast DNS SOA serial maximum boundary to long

.. _petr_viktorin_3_1:

Petr Viktorin (3):
^^^^^^^^^^^^^^^^^^

-  Internationalization for public errors
-  Run ntpdate in verbose mode, not debug (i.e. no-op) mode
-  Add nsds5ReplicaStripAttrs to replica agreements

.. _petr_vobornik_15:

Petr Vobornik (15):
^^^^^^^^^^^^^^^^^^^

-  Range Web UI
-  Revert change causing failure in test automation
-  Fix issue which broke setup of Web UI unit tests
-  Successful action notification
-  Password policy paging with proper sorting
-  Fixed search in HBAC test
-  Permissions: select only applicable options on type change
-  Notify success on add, delete and update
-  Fixed metadata serialization of Numbers and DNs
-  Added decimal checks to metadata validator
-  Generated metadata for testing updated
-  Fixed problem while deleting entry with unsaved changes
-  Allow localhost in zone ACIs - Web UI
-  Update of confirmation of actions
-  Reflect API change of SSH store in Web UI

.. _rob_crittenden_8:

Rob Crittenden (8):
^^^^^^^^^^^^^^^^^^^

-  Don't generate password history error if history is set to 0.
-  Restrict the SELinux user map user MLS value to 0-1023
-  Support the new Winsync POSIX API.
-  Set minimum of 389-ds-base to 1.2.11.8 to pick up cache warning.
-  Add version to replica prepare file, prevent installing to older
   version
-  Set the e-mail attribute using the default domain name by default
-  Fix some restart script issues found with certificate renewal.
-  Become IPA v3 beta 3 (3.0.0.pre3)

.. _sumit_bose_27:

Sumit Bose (27):
^^^^^^^^^^^^^^^^

-  Use libsamba-security instead of libsecurity
-  ipadb_iterate(): handle match_entry == NULL
-  ipasam: cleanup explicit dependencies to samba libs
-  Make encode_ntlm_keys() public
-  ipasam: remove nt_lm_owf_gen() and dependency to libcliauth.so
-  ipasam: remove sid_peek_rid()
-  ipasam: replace strnequal()
-  ipasam: remove strlower_m()
-  ipasam: remove talloc_asprintf_strupper_m()
-  ipasam: replace sid_copy()
-  ipasam: replace sid_compose()
-  ipasam: Replace is_null_sid()
-  ipasam: Replace dom_sid_compare_domain()
-  ipasam: Replace sid_check_is_our_sam()
-  ipasam: Replace sid_peek_check_rid()
-  ipasam: Replace global_sid_Builtin
-  ipasam: add libsss_idmap context and replace string_to_sid()
-  ipasam: replace get_global_sam_sid()
-  ipasam: remove fetch_ldap_pw()
-  ipasam: replace trim_char() with trim_string()
-  Rename range CLI to idrange
-  IDRange CLI: allow to work without arguments
-  IDRange CLI: Add documentation
-  Do not create trust if murmur hash is not available and base-id not
   given
-  Trust CLI: Return more details when searching trusts
-  Trust CLI: return more details of added trust
-  Trust CLI: mark trust-mod for future use

.. _tomas_babej_5_1:

Tomas Babej (5):
^^^^^^^^^^^^^^^^

-  Adds dependency on samba4-winbind.
-  Improves deletion of PTR records in ipa host-del
-  Fixes different behaviour of permission-mod and show.
-  Change slapi_mods_init in ipa_winsync_pre_ad_mod_user_mods_cb
-  Sort policies numerically in pwpolicy-find

.. _version_3.0.0_beta_2_08172012:

Version 3.0.0 Beta 2 (08/17/2012)
=================================

.. _alexander_bokovoy_11_1:

Alexander Bokovoy (11):
^^^^^^^^^^^^^^^^^^^^^^^

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

.. _david_spångberg_1:

David Spångberg (1):
^^^^^^^^^^^^^^^^^^^^

-  Indirect roles in WebUI

.. _gowrishankar_rajaiyan_1:

Gowrishankar Rajaiyan (1):
^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Adding exit status 3 & 4 to ipa-client-install man page

.. _jan_cholasta_2:

Jan Cholasta (2):
^^^^^^^^^^^^^^^^^

-  Add --{set,add,del}attr options to commands which are missing them.
-  Raise Base64DecodeError instead of ConversionError when base64
   decoding fails in Bytes parameters.

.. _john_dennis_2_3:

John Dennis (2):
^^^^^^^^^^^^^^^^

-  Use DN objects instead of strings
-  Installation fails when CN is set in certificate subject base

.. _martin_kosek_12_1:

Martin Kosek (12):
^^^^^^^^^^^^^^^^^^

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

.. _petr_viktorin_7_2:

Petr Viktorin (7):
^^^^^^^^^^^^^^^^^^

-  Fix batch command error reporting
-  Fix wrong option name in ipa-managed-entries man page
-  Fix updating minimum_connections in ipa-upgradeconfig
-  Framework for admin/install tools, with ipa-ldap-updater
-  Arrange stripping .po files
-  Update translations
-  Create /etc/sysconfig/network if it doesn't exist

.. _petr_vobornik_31:

Petr Vobornik (31):
^^^^^^^^^^^^^^^^^^^

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

.. _rob_crittenden_12:

Rob Crittenden (12):
^^^^^^^^^^^^^^^^^^^^

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

.. _simo_sorce_14:

Simo Sorce (14):
^^^^^^^^^^^^^^^^

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

.. _sumit_bose_4_1:

Sumit Bose (4):
^^^^^^^^^^^^^^^

-  Allow silent build if available
-  ipasam: fixes for clang warnings
-  ipasam: replace testing code
-  Fix typo

.. _tomas_babej_5_2:

Tomas Babej (5):
^^^^^^^^^^^^^^^^

-  Adds check for ipa-join.
-  Permissions of replica files changed to 0600.
-  Handle SSSD restart crash more gently.
-  Corrects help description of selinuxusermap.
-  Improves exception handling in ipa-replica-prepare.

.. _version_3.0.0_beta_1_07022012:

Version 3.0.0 Beta 1 (07/02/2012)
=================================

The development of 3.0 occurred simultaneously with 2.2.0 so there is
some overlap.

.. _adam_young_10:

Adam Young (10):
^^^^^^^^^^^^^^^^

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

.. _alexander_bokovoy_61:

Alexander Bokovoy (61):
^^^^^^^^^^^^^^^^^^^^^^^

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

.. _endi_s._dewata_105:

Endi S. Dewata (105):
^^^^^^^^^^^^^^^^^^^^^

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

.. _jr_aquino_5:

JR Aquino (5):
^^^^^^^^^^^^^^

-  Create Tool for Enabling/Disabling Managed Entry Plugins
-  Replication: Adjust replica installation to omit processing memberof
   computations
-  Improve sudorule documentation
-  Create FreeIPA CLI Plugin for the 389 Auto Membership plugin
-  Move Managed Entries into their own container in the replicated
   space.

.. _jan_cholasta_42:

Jan Cholasta (42):
^^^^^^^^^^^^^^^^^^

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

.. _john_dennis_38:

John Dennis (38):
^^^^^^^^^^^^^^^^^

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

.. _lars_sjostrom_1:

Lars Sjostrom (1):
^^^^^^^^^^^^^^^^^^

-  Add disovery domain if client domain is different from server domain

.. _marko_myllynen_2:

Marko Myllynen (2):
^^^^^^^^^^^^^^^^^^^

-  include <stdint.h> for uintptr_t
-  Don't remove /tmp when removing temp cert dir

.. _martin_kosek_171:

Martin Kosek (171):
^^^^^^^^^^^^^^^^^^^

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

.. _nalin_dahyabhai_5:

Nalin Dahyabhai (5):
^^^^^^^^^^^^^^^^^^^^

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

.. _ondrej_hamada_26:

Ondrej Hamada (26):
^^^^^^^^^^^^^^^^^^^

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

.. _petr_viktorin_60:

Petr Viktorin (60):
^^^^^^^^^^^^^^^^^^^

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

.. _petr_vobornik_158:

Petr Vobornik (158):
^^^^^^^^^^^^^^^^^^^^

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

.. _rob_crittenden_191:

Rob Crittenden (191):
^^^^^^^^^^^^^^^^^^^^^

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

.. _simo_sorce_104:

Simo Sorce (104):
^^^^^^^^^^^^^^^^^

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

.. _sumit_bose_32:

Sumit Bose (32):
^^^^^^^^^^^^^^^^

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

.. _yuri_chornoivan_1_1:

Yuri Chornoivan (1):
^^^^^^^^^^^^^^^^^^^^

-  Fix typos

.. _version_2.2.2_02132013:

Version 2.2.2 (02/13/2013)
==========================

.. _alexander_bokovoy_1_5:

Alexander Bokovoy (1):
^^^^^^^^^^^^^^^^^^^^^^

-  Update plugin to upload CA certificate to LDAP

.. _jan_cholasta_1_5:

Jan Cholasta (1):
^^^^^^^^^^^^^^^^^

-  Pylint cleanup

.. _john_dennis_1_4:

John Dennis (1):
^^^^^^^^^^^^^^^^

-  Use secure method to acquire IPA CA certificate

.. _martin_kosek_3_2:

Martin Kosek (3):
^^^^^^^^^^^^^^^^^

-  Run index task for new indexes
-  Filter suffix in replication management tools
-  Become IPA 2.2.2

.. _rob_crittenden_1_3:

Rob Crittenden (1):
^^^^^^^^^^^^^^^^^^^

-  Do SSL CA verification and hostname validation.

.. _simo_sorce_1_4:

Simo Sorce (1):
^^^^^^^^^^^^^^^

-  Upload CA cert in the directory on install

.. _version_2.2.1_10232012:

Version 2.2.1 (10/23/2012)
==========================

.. _endi_sukma_dewata_1_1:

Endi Sukma Dewata (1):
^^^^^^^^^^^^^^^^^^^^^^

-  Fixed boot.ldif permission.

.. _jan_cholasta_1_6:

Jan Cholasta (1):
^^^^^^^^^^^^^^^^^

-  SSH configuration fixes.

.. _martin_kosek_1_1:

Martin Kosek (1):
^^^^^^^^^^^^^^^^^

-  Become IPA 2.2.1

.. _petr_viktorin_2_2:

Petr Viktorin (2):
^^^^^^^^^^^^^^^^^^

-  replica-install: Don't copy Firefox config extension files if they're
   not in the replica file
-  Create Firefox extension on upgrade and replica-install

.. _petr_vobornik_8:

Petr Vobornik (8):
^^^^^^^^^^^^^^^^^^

-  Host page fixed to work with disabled DNS support
-  Fix jquery error when using '??' in a pkey
-  Kerberos authentication extension
-  Kerberos authentication extension makefiles
-  Build and installation of Kerberos authentication extension
-  Configuration pages changed to use new FF extension
-  Add mime type to httpd ipa.conf for xpi extension
-  RPM spec fix for ffconfig.js and ffconfig_page.js

.. _rob_crittenden_2:

Rob Crittenden (2):
^^^^^^^^^^^^^^^^^^^

-  Check for locked-out user before incrementing lastfail.
-  Index the fqdn attribute.

.. _simo_sorce_2_7:

Simo Sorce (2):
^^^^^^^^^^^^^^^

-  Fix migration code password setting.
-  Add support for disabling KDC writes

.. _version_2.2.0_05032012:

Version 2.2.0 (05/03/2012)
==========================

.. _alexander_bokovoy_1_6:

Alexander Bokovoy (1):
^^^^^^^^^^^^^^^^^^^^^^

-  When changing multiple booleans with setsebool, pass each of them
   separately.

.. _jan_cholasta_9:

Jan Cholasta (9):
^^^^^^^^^^^^^^^^^

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
-  Set the "KerberosAuthentication" option in sshd_config to "no"
   instead of "yes".

.. _john_dennis_7:

John Dennis (7):
^^^^^^^^^^^^^^^^

-  Replace broken i18n shell test with Python test
-  improve handling of ds instances during uninstall
-  Use indexed format specifiers in i18n strings
-  text unit test should validate using installed mo file
-  Validate DN & RDN parameters for migrate command
-  don't append basedn to container if it is included
-  Fix name error in hbactest

.. _lars_sjostrom_1_1:

Lars Sjostrom (1):
^^^^^^^^^^^^^^^^^^

-  Add disovery domain if client domain is different from server domain

.. _martin_kosek_29:

Martin Kosek (29):
^^^^^^^^^^^^^^^^^^

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

.. _ondrej_hamada_7:

Ondrej Hamada (7):
^^^^^^^^^^^^^^^^^^

-  More exception handlers in ipa-client-install
-  Search allowed attributes in superior objectclasses
-  Typos in FreeIPA messages
-  Netgroup nisdomain and hosts validation
-  Confusing default user groups
-  Unable to rename permission object
-  Fix empty external member processing

.. _petr_viktorin_22:

Petr Viktorin (22):
^^^^^^^^^^^^^^^^^^^

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

.. _petr_vobornik_22:

Petr Vobornik (22):
^^^^^^^^^^^^^^^^^^^

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

.. _rob_crittenden_34:

Rob Crittenden (34):
^^^^^^^^^^^^^^^^^^^^

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

.. _simo_sorce_1_5:

Simo Sorce (1):
^^^^^^^^^^^^^^^

-  Fix memleak and silence Coverity defects

.. _version_2.1.90_beta_1_03052012:

Version 2.1.90 Beta 1 (03/05/2012)
==================================

.. _jan_cholasta_1_7:

Jan Cholasta (1):
^^^^^^^^^^^^^^^^^

-  Configure SSH features of SSSD in ipa-client-install.

.. _john_dennis_8:

John Dennis (8):
^^^^^^^^^^^^^^^^

-  update translation pot file and PY_EXPLICIT_FILES list
-  update po files
-  created Transifex resource, adjust tx config file to point to it.
-  Tweak the session auth to reflect developer consensus.
-  Implement session activity timeout
-  Implement password based session login
-  Log a message when returning non-success HTTP result

.. _martin_kosek_21:

Martin Kosek (21):
^^^^^^^^^^^^^^^^^^

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

.. _ondrej_hamada_3:

Ondrej Hamada (3):
^^^^^^^^^^^^^^^^^^

-  Validate attributes in permission-add
-  Migration warning when compat enabled
-  ipa-client-install not calling authconfig

.. _petr_viktorin_6_1:

Petr Viktorin (6):
^^^^^^^^^^^^^^^^^^

-  Make ipausers a non-posix group on new installs
-  Add extra checking function to XMLRPC test framework
-  Add common helper for interactive prompts
-  Make sure the nolog argument to ipautil.run is not a bare string
-  Use stricter semantics when checking IP address for DNS records
-  Use stricter semantics when checking IP address for DNS records
-  Use reboot from /sbin

.. _petr_voborník_18:

Petr Voborník (18):
^^^^^^^^^^^^^^^^^^^

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

.. _rob_crittenden_37:

Rob Crittenden (37):
^^^^^^^^^^^^^^^^^^^^

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

.. _simo_sorce_4:

Simo Sorce (4):
^^^^^^^^^^^^^^^

-  ipa-kdb: Fix ACL evaluator
-  policy: add function to check lockout policy
-  ipa-kdb: fix delegation acl check
-  Fix ticket checks when using either s4u2proxy or a delegated krbtgt

.. _version_2.1.90_alpha_2_02172012:

Version 2.1.90 Alpha 2 (02/17/2012)
===================================

.. _adam_young_4:

Adam Young (4):
^^^^^^^^^^^^^^^

-  remove enrolled column
-  Add priority to pwpolicy list
-  Remove delegation from browser config
-  ignore generated services file.

.. _alexander_bokovoy_14:

Alexander Bokovoy (14):
^^^^^^^^^^^^^^^^^^^^^^^

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

.. _endi_s._dewata_60:

Endi S. Dewata (60):
^^^^^^^^^^^^^^^^^^^^

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

.. _jr_aquino_1_3:

JR Aquino (1):
^^^^^^^^^^^^^^

-  Replication: Adjust replica installation to omit processing memberof
   computations

.. _jan_cholasta_15:

Jan Cholasta (15):
^^^^^^^^^^^^^^^^^^

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

.. _john_dennis_10:

John Dennis (10):
^^^^^^^^^^^^^^^^^

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

.. _marko_myllynen_1:

Marko Myllynen (1):
^^^^^^^^^^^^^^^^^^^

-  include <stdint.h> for uintptr_t

.. _martin_kosek_52:

Martin Kosek (52):
^^^^^^^^^^^^^^^^^^

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

.. _ondrej_hamada_9:

Ondrej Hamada (9):
^^^^^^^^^^^^^^^^^^

-  Misleading Keytab field
-  Sort password policy by priority
-  Client install checks for nss_ldap
-  User-add random password support
-  HBAC test optional sourcehost option
-  localhost.localdomain clients refused to join
-  Leave nsds5replicaupdateschedule parameter unset
-  Fix 'no-reverse' option description
-  Memberof attribute control and update

.. _petr_viktorin_5:

Petr Viktorin (5):
^^^^^^^^^^^^^^^^^^

-  Switch --group and --membergroup in example for delegation
-  Fix/add options in ipa-managed-entries man page
-  Honor default home directory and login shell in user_add
-  Clean up i18n strings
-  Internationalization for HBAC and ipalib.output

.. _petr_voborník_55:

Petr Voborník (55):
^^^^^^^^^^^^^^^^^^^

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

.. _rob_crittenden_54:

Rob Crittenden (54):
^^^^^^^^^^^^^^^^^^^^

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

.. _simo_sorce_77:

Simo Sorce (77):
^^^^^^^^^^^^^^^^

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

.. _version_2.1.90_alpha_1_02072012:

Version 2.1.90 Alpha 1 (02/07/2012)
===================================

This was an unannounced release that formed the basis of the first
Fedora 17 package. It was not well-tested, particularly for upgrades,
which is why it wasn't announced at the time. It was released to meet
Fedora 17 package deadlines.

The changelog is included in the public alpha 2 entry.

.. _version_2.1.4_12062011:

Version 2.1.4 (12/06/2011)
==========================

.. _alexander_bokovoy_4_2:

Alexander Bokovoy (4):
^^^^^^^^^^^^^^^^^^^^^^

-  hbactest fails while you have svcgroup in hbacrule
-  Add support for systemd environments and use it to support Fedora 16
-  Spin for connection success also when socket is not (yet) available
-  Quote multiple workers option

.. _endi_s._dewata_1:

Endi S. Dewata (1):
^^^^^^^^^^^^^^^^^^^

-  Added current password field.

.. _evgeny_sinelnikov_1:

Evgeny Sinelnikov (1):
^^^^^^^^^^^^^^^^^^^^^^

-  ipa_kpasswd: Update selinux policies for ldap and urandom

.. _john_dennis_1_5:

John Dennis (1):
^^^^^^^^^^^^^^^^

-  Unable to Download Certificate with Browser

.. _martin_kosek_8_1:

Martin Kosek (8):
^^^^^^^^^^^^^^^^^

-  Fix client krb5 domain mapping and DNS
-  Fix ipa-managed-entries password option long form
-  Fix ipa-server-install answer cache
-  Fix ipa-replica-conncheck port labels
-  Fix ipa-managed-entries bind procedure
-  Let PublicError accept Gettext objects
-  Enable automember for upgraded servers
-  Make ipa-server-install clean after itself

.. _ondrej_hamada_1:

Ondrej Hamada (1):
^^^^^^^^^^^^^^^^^^

-  Client install root privileges check

.. _rob_crittenden_4_2:

Rob Crittenden (4):
^^^^^^^^^^^^^^^^^^^

-  Fix problems in help system
-  Fix nis netgroup config entry so users appear in netgroup triple.
-  Don't allow default objectclass list to be empty.
-  Require an HTTP Referer header in the server. Send one in ipa tools.
   (CVE-2011-3636)

.. _simo_sorce_1_6:

Simo Sorce (1):
^^^^^^^^^^^^^^^

-  Modify random salt creation for interoperability

.. _version_2.1.3_10192011:

Version 2.1.3 (10/19/2011)
==========================

.. _adam_young_1:

Adam Young (1):
^^^^^^^^^^^^^^^

-  Fix dynamic display of UI tabs based on rights

.. _alexander_bokovoy_8_2:

Alexander Bokovoy (8):
^^^^^^^^^^^^^^^^^^^^^^

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

.. _jan_cholasta_3_2:

Jan Cholasta (3):
^^^^^^^^^^^^^^^^^

-  Disallow deletion of global password policy.
-  Don't leak passwords through kdb5_ldap_util command line arguments.
-  Remove more redundant configuration values from krb5.conf.

.. _john_dennis_1_6:

John Dennis (1):
^^^^^^^^^^^^^^^^

-  Fix Spanish po translation file

.. _martin_kosek_12_2:

Martin Kosek (12):
^^^^^^^^^^^^^^^^^^

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

.. _petr_vobornik_2_1:

Petr Vobornik (2):
^^^^^^^^^^^^^^^^^^

-  Added missing fields to password policy page
-  Fixed: Unable to add external user for RunAs User for Sudo rules

.. _rob_crittenden_12_1:

Rob Crittenden (12):
^^^^^^^^^^^^^^^^^^^^

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

.. _simo_sorce_1_7:

Simo Sorce (1):
^^^^^^^^^^^^^^^

-  updates: Change default limits on ldap searches

.. _version_2.1.2_not_publicly_released_10072011:

Version 2.1.2 (not publicly released, ~ 10/07/2011)
===================================================

.. _adam_young_4_1:

Adam Young (4):
^^^^^^^^^^^^^^^

-  split metadata call
-  Make mod_nss renegotiation configuration a public function
-  Execute pki proxy setup when server is upgraded if needed
-  Force the upgrade of pki-setup when upgrading the RPMS

.. _alexander_bokovoy_13:

Alexander Bokovoy (13):
^^^^^^^^^^^^^^^^^^^^^^^

-  Incorrect name in examples of ipa help hbactest
-  Unroll groups when testing HBAC rules
-  Introduce platform-specific adaptation for services used by FreeIPA.
-  Convert server install code to platform-independent access to system
   services
-  Convert client-side tools to platform-independent access to system
   services
-  Convert installation tools to platform-independent access to system
   services
-  Cleanup whitespace
-  When external host is specified in HBAC rule, allow its use in
   simulation
-  Unroll StrEnum values when displaying help
-  Configure pam_krb5 on the client only if sssd is not configured
-  Setup and restore ntp configuration on the client side properly
-  Fix 'referenced before assignment' warning
-  Before kinit, try to sync time with the NTP servers of the domain we
   are joining

.. _endi_s._dewata_24:

Endi S. Dewata (24):
^^^^^^^^^^^^^^^^^^^^

-  Fixed unit test for entity select widget.
-  Fixed layout problem in permission adder dialog.
-  Fixed sudo rule association dialogs.
-  Fixed missing optional field.
-  Fixed labels for run-as users and groups.
-  Fixed problem opening host adder dialog.
-  Removed entitlement menu.
-  Fixed posix group checkbox.
-  Fixed columns in HBAC/sudo rules list pages.
-  Fixed missing cancel button in unprovisioning dialog.
-  Fixed problem enabling/disabling DNS zone.
-  Fixed problem enrolling member with the same name.
-  Modified dialog to use sections.
-  Removed undo flags from dialog field specs.
-  Fixed problem on combobox with search limit.
-  Fixed problem displaying special characters.
-  Fixed add/delete arrows position.
-  Fixed duplicate entries in enrollment dialog.
-  Updated color scheme.
-  Fixed tab and dialog widths.
-  Disable enroll button if nothing selected.
-  Fixed missing default shell field.
-  I18n clean-up.
-  Disable sudo options Delete button if nothing selected.

.. _jr_aquino_1_4:

JR Aquino (1):
^^^^^^^^^^^^^^

-  25 Create Tool for Enabling/Disabling Managed Entry Plugins

.. _jakub_hrozek_1_6:

Jakub Hrozek (1):
^^^^^^^^^^^^^^^^^

-  Silence a compilation warning in ipa_kpasswd

.. _jan_cholasta_6_1:

Jan Cholasta (6):
^^^^^^^^^^^^^^^^^

-  Check that install hostname matches the server hostname.
-  Fix client install on IPv6 machines.
-  Fix ipa-replica-prepare always warning the user about not using the
   system hostname.
-  Validate name_from_ip parameter of dnszone.
-  Add a function for formatting network locations of the form host:port
   for use in URLs.
-  Work around pkisilent bugs.

.. _jr_aquino_1_5:

JR Aquino (1):
^^^^^^^^^^^^^^

-  Move Managed Entries into their own container in the replicated
   space.

.. _marko_myllynen_1_1:

Marko Myllynen (1):
^^^^^^^^^^^^^^^^^^^

-  Don't remove /tmp when removing temp cert dir

.. _martin_kosek_21_1:

Martin Kosek (21):
^^^^^^^^^^^^^^^^^^

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

.. _nalin_dahyabhai_2:

Nalin Dahyabhai (2):
^^^^^^^^^^^^^^^^^^^^

-  list users from nested groups, too
-  Update man pages to note that PKCS#12 files also contain private
   keys, and that the "pkinit" options refer to the KDC's credentials

.. _petr_vobornik_10:

Petr Vobornik (10):
^^^^^^^^^^^^^^^^^^^

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

.. _rob_crittenden_20:

Rob Crittenden (20):
^^^^^^^^^^^^^^^^^^^^

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

.. _simo_sorce_4_1:

Simo Sorce (4):
^^^^^^^^^^^^^^^

-  ipa-pwd-extop: Fix segfault in password change.
-  ipa-pwd-extop: Enforce old password checks
-  ipa-client-install: Fix joining when LDAP access is restricted
-  replica-prepare: anonymous binds may be disallowed

.. _sumit_bose_2_1:

Sumit Bose (2):
^^^^^^^^^^^^^^^

-  Call standard_logging_setup() before any logging is done
-  ipa-pwd-extop: allow password change on all connections with SSF>1

.. _yuri_chornoivan_1_2:

Yuri Chornoivan (1):
^^^^^^^^^^^^^^^^^^^^

-  Fix typos

.. _version_2.1.1_09082011:

Version 2.1.1 (09/08/2011)
==========================

.. _adam_young_1_1:

Adam Young (1):
^^^^^^^^^^^^^^^

-  enable proxy for dogtag

.. _alexander_bokovoy_1_7:

Alexander Bokovoy (1):
^^^^^^^^^^^^^^^^^^^^^^

-  Propagate environment when it is required.

.. _endi_s._dewata_19:

Endi S. Dewata (19):
^^^^^^^^^^^^^^^^^^^^

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

.. _jan_cholasta_6_2:

Jan Cholasta (6):
^^^^^^^^^^^^^^^^^

-  Make sure messagebus is running prior to starting certmonger.
-  Verify that passwords specified through command line options of
   ipa-server-install meet the length requirement.
-  Add option to install without the automatic redirect to the Web UI.
-  Search for users in all the naming contexts present on the directory
   server.
-  Add subscription-manager dependency for RHEL.
-  Verify that the external CA certificate files are correct.

.. _john_dennis_11:

John Dennis (11):
^^^^^^^^^^^^^^^^^

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

.. _jr_aquino_2:

JR Aquino (2):
^^^^^^^^^^^^^^

-  Improve sudorule documentation
-  Create FreeIPA CLI Plugin for the 389 Auto Membership plugin

.. _martin_kosek_6:

Martin Kosek (6):
^^^^^^^^^^^^^^^^^

-  Add missing attribute labels for sudorule
-  Fix automountkey-mod
-  Fix automountlocation-import conflicts
-  ipa-client-install breaks network configuration
-  Fix sudo help and summaries
-  Let Bind track data changes

.. _petr_vobornik_8_1:

Petr Vobornik (8):
^^^^^^^^^^^^^^^^^^

-  error dialog for batch command
-  Uncheck checkboxes in association after deletion
-  Show error in adding associations
-  Validation of details facet before update
-  Modify serial associator to use batch
-  Modifying sudo options refreshes the whole page
-  Enable update and reset button only if dirty
-  Attributes table not scrollable

.. _rob_crittenden_24:

Rob Crittenden (24):
^^^^^^^^^^^^^^^^^^^^

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
-  Become IPA 2.1.1

.. _simo_sorce_1_8:

Simo Sorce (1):
^^^^^^^^^^^^^^^

-  conncheck: Fix List of ports to check

.. _version_2.1.0_08172011:

Version 2.1.0 (08/17/2011)
==========================

.. _adam_young_62:

Adam Young (62):
^^^^^^^^^^^^^^^^

-  Fixed labels for sudo and hbac rules
-  update metadata with label changes
-  define entities using builder and more declarative syntax
-  default all false no longer default to all: true for searches, only
   specify it for user searches
-  code review fixes
-  make use of new user-find columns.
-  fix JSL error
-  Upgrade to jquery 1.5.2
-  action panel to top tabs
-  remove jquery-cookie library
-  update ipa init a simple script to update the metatdate et alles that
   comes from the ipa_init batch call
-  whitespace and -x removal
-  create entities on demand. fixed changes from code review
-  automount UI
-  redirect on show error.
-  redirect on error Code for redirecting on error has been moved to
   IPA.facet so it can be called from both details and assocaiton
   facets.
-  automount delete key indirect automount maps
-  scrollable content areas
-  dialog scrolling table
-  JSON marshalling list
-  dns multiple records show multiple records that share the same
   dnsname
-  no redirect on search
-  test for dirty
-  test dirty textarea runs the testdirty check before setting the undo
   tag for a textarea
-  test dirty multivalue test the multivalue widgets for changes before
   showing the undo link.
-  test dirty onchange
-  entity select widget for manager
-  hide automount tabs.
-  service host entity select Use the entity select widget for add
   service
-  entity select undo
-  no redirect on unknown error If the error name is indicates a server
   wide error, do not attempt to redirect.
-  editable entity_select
-  ipaddress for host add
-  entity select for password policy
-  tooltips for host add
-  automountkey details
-  identify target as section for permissions
-  optional uid
-  validate required fields
-  Generate record type list from metadata
-  shorten url cache state in a javascript variable, and leave on
   information about the current entity in the URL hash params
-  containing entity pkeys
-  undefined pkeys
-  config fields
-  ipadefaultemaildomain
-  config widgets entity select default group checkbox for migration
-  entity link for password policy
-  validate ints
-  password expiration label
-  HBAC deny warning
-  check required on add
-  clear errors on reset
-  indirect admins
-  entity_select naming
-  remove HBAC warning from static UI
-  dnsrecord-mod ui
-  no dns
-  remove hardcoded DNS label for record name.
-  move dns to identity tab
-  removing setters setup and init
-  dns section header i18n.
-  use other_entity for adder columns

.. _alexander_bokovoy_10:

Alexander Bokovoy (10):
^^^^^^^^^^^^^^^^^^^^^^^

-  Convert Bool to TRUE/FALSE when working with LDAP backend
-  Minor typos in the examples
-  Convert nsaccountlock to always work as bool towards Python code
-  Rearrange logging for NSCD daemon.
-  Fix sssd.conf to always have IPA certificate for the domain.
-  Add hbactest command.
-  Modify /etc/sysconfig/network on a client when IPA manages hostname
-  Make proper LDAP configuration reporting for ipa-client-install
-  Ensure network configuration file has proper permissions
-  Pass empty options as empty arrays for supported dns record types.

.. _endi_s._dewata_114:

Endi S. Dewata (114):
^^^^^^^^^^^^^^^^^^^^^

-  Fixed undefined label in permission adder dialog box.
-  Initial Selenium test cases.
-  Added functional test runner.
-  Refactored action panel and client area.
-  Refactored builder interface.
-  Refactored search facet.
-  Entitlements.
-  Updated Selenium tests.
-  Merged IPA.cmd() into IPA.command().
-  Entitlement registration.
-  Entitlement import.
-  Entitlement download.
-  Moved adder dialog box into entity.
-  Standardized action panel buttons creation.
-  Entitlement quantity validation.
-  Refactored navigation.
-  Use entity names for tab state.
-  Moved entity contents outside navigation.
-  Added facet container.
-  Fixed self-service UI.
-  Updated Selenium tests.
-  Updated Selenium tests.
-  Updated DNS interface.
-  Added Selenium tests for DNS.
-  Added UUID field for entitlement registration.
-  Added Self-Service and Delegation tests.
-  Customizable facet groups.
-  Read-only association facet.
-  jQuery ordered map.
-  Fixed problem disabling HBAC and SUDO rules.
-  Fixed Ajax error handling.
-  Fixed details tests.
-  Fixed adder dialog title.
-  Fixed Add and Edit without primary key.
-  Fixed Selenium tests.
-  Fixed URL parameter parsing.
-  Added Update and Reset buttons into Dirty dialog.
-  Fixed problem deleting value in text field.
-  Added pagination for associations.
-  Fixed pagination problem.
-  Temporary fix for indirect member tabs.
-  Fixed blank dialog box on internal error.
-  Fixed resizing issues.
-  Added selectable option for table widget.
-  Entitlement status.
-  Fixed tab navigation.
-  Fixed build break.
-  Fixed paging for indirect members.
-  Renamed associate.js to association.js.
-  Fixed self-service links.
-  Merged direct and indirect association facets
-  Storing page number in URL.
-  Removed FreeWay font files.
-  Fixed problem with navigation tabs on reload.
-  Converted entity header into facet header.
-  Added navigation breadcrumb.
-  Added record count into association facet tabs.
-  Added singular entity labels.
-  Fixed entity labels.
-  Fixed DNS records page title.
-  Fixed undo all problem.
-  Removed unused images.
-  Fixed hard-coded messages.
-  Added confirmation dialog for user activation.
-  Fixed button style in Entitlements
-  Removed invalid associations.
-  Added arrow icons for details sections.
-  Fixed object_name usage.
-  Fixed HBAC/Sudo rules associations.
-  Fixed blank self-service page.
-  Fixed dirty dialog problems in HBAC/Sudo rules.
-  Fixed test fixture file name.
-  Fixed missing entitlement import button label
-  Added sudo options.
-  Fixed collapsed table in Chrome.
-  Fixed object_name and object_name_plural internationalization
-  Fixed label capitalization
-  Entity select widget improvements
-  Removed reverse zones from host adder dialog.
-  Fixed host details fields.
-  Added checkbox to remove hosts from DNS.
-  Creating reverse zones from IP address.
-  Removed entitlement registration UUID field.
-  Fixed problem loading data in HBAC/sudo details page.
-  Removed HBAC access time code.
-  Removed custom layouts using HTML templates.
-  Refactored IPA.current_facet().
-  Fixed problem with navigation state loading.
-  Fixed navigation problems.
-  Fixed navigation unit test.
-  Fixed click handlers on certificate buttons.
-  New icons for entitlement buttons
-  Fixed problem bookmarking Policy/IPA Server tabs
-  Fixed problem setting host OTP.
-  Fixed hard-coded labels in sudo rules.
-  Fixed hard-coded label in Find button.
-  Fixed missing section header in sudo command group.
-  Fixed problem unprovisioning service.
-  Fixed missing memberof definition in HBAC service.
-  Added association facets for HBAC and sudo.
-  Fixed certificate buttons.
-  Fixed missing icons.
-  Fixed misaligned search icon.
-  Resizable adder dialog box.
-  Linked entries in HBAC/sudo details page.
-  Fixed 3rd level tab style.
-  Fixed facet group labels.
-  Fixed error after login on IE
-  Fixed host adder dialog.
-  Fixed DNS zone adder dialog.
-  Fixed broken links in ipa_error.css and ipa_migration.css.
-  Fixed problem clicking 3rd level tabs.
-  Fixed link style in dialog box.
-  Fixed problem with buttons in enrollment dialog.

.. _jakub_hrozek_1_7:

Jakub Hrozek (1):
^^^^^^^^^^^^^^^^^

-  Remove wrong kpasswd sysconfig

.. _jan_cholasta_34:

Jan Cholasta (34):
^^^^^^^^^^^^^^^^^^

-  Fix wording of error message.
-  Add note about ipa-dns-install to ipa-server-install man page.
-  Fix typo in ipa-server-install.
-  Fix uninitialized variables.
-  Fix double definition of output_for_cli.
-  Add lint script for static code analysis.
-  Fix lint false positives.
-  Remove unused classes.
-  Fix some minor issues uncovered by pylint.
-  Fix uninitialized attributes.
-  Run lint during each build.
-  Several improvements of the lint script.
-  Fix issues found by Coverity.
-  Fix regressions introduced by pylint false positive fixes.
-  Assume ipa help for plugins.
-  Parse netmasks in IP addresses passed to server install.
-  Honor netmask in DNS reverse zone setup.
-  Do stricter checking of IP addressed passed to server install.
-  Fix directory manager password validation in ipa-nis-manage.
-  Improve IP address handling in the host-add command.
-  Verify that the hostname is fully-qualified before accessing the
   service information in ipactl.
-  Remove redundant configuration values from krb5.conf.
-  Replace the 'private' option in netgroup-find with 'managed'.
-  Configure SSSD to store user password if offline.
-  Fix creation of reverse DNS zones.
-  Add ability to specify DNS reverse zone name by IP network address.
-  Fix exit status of ipa-nis-manage enable.
-  Update minimum required version of python-netaddr.
-  Clean up of IP address checks in install scripts.
-  Don't delete NIS netgroup compat suffix on 'ipa-nis-manage disable'.
-  Fix ipa-compat-manage not working after recent ipa-nis-manage change.
-  Make sure that hostname specified by user is not an IP address.
-  Fix external CA install.
-  Ask for reverse DNS zone information in attended install right after
   asking for DNS forwarders, so that DNS configuration is done in one
   place.

.. _john_dennis_9:

John Dennis (9):
^^^^^^^^^^^^^^^^

-  Module for DN objects plus unit test
-  assert_deepequal supports callback for equality testing
-  Add backslash escape support for cvs reader
-  Use DN class in get_primary_key_from_dn to return decoded value
-  Update test_role_plugin test to include a comma in a privilege
-  Ticket 1485 - DN pairwise grouping
-  Make AVA, RDN & DN comparison case insensitive. No need for lowercase
   normalization.
-  Clean up existing DN object usage
-  transifex translation adjustment

.. _jr_aquino_15:

JR Aquino (15):
^^^^^^^^^^^^^^^

-  Escape LDAP characters in member and memberof searches
-  Add memberHost and memberUser to default indexes
-  Optimize and dynamically verify group membership
-  Delete the sudoers entry when disabling Schema Compat
-  Return copy of config from ipa_get_config()
-  Typo in host_nis_groups has been creating 2 CN's
-  Add sudorule and hbacrule to memberof and indirectmemberof attributes
-  Display remaining external hosts when removing from sudorule
-  Raise DuplicateEntry Error when adding a duplicate sudo option
-  Don't add empty tuple to entry_attrs['externalhost']
-  oneliner correct typo in ipasudorunas_group
-  Return correct "RunAs External Group" when removing members
-  remove escapes from the cvs parser in ipaserver/install/ldapupdate
-  Correct behavior for sudorunasgroup vs sudorunasuser
-  Correct sudo runasuser and runasgroup attributes in schema

.. _martin_kosek_68:

Martin Kosek (68):
^^^^^^^^^^^^^^^^^^

-  Inconsistent error message for duplicate user
-  Replica installation fails for self-signed server
-  Remove doc from API.txt
-  Revert "Remove doc from API.txt"
-  Password policy commands do not include cospriority
-  Improve DNS PTR record validation
-  Remove unwanted trimming in text fields
-  Need force option in DNS zone adder dialog
-  IPA replica is not started after the reboot
-  Improve Directory Service open port checker
-  Log temporary files in ipa-client-install
-  Prevent uninstalling client on the IPA server
-  pwpolicy-mod doesn't accept old attribute values
-  Forbid reinstallation in ipa-client-install
-  ipa-client-install uninstall does not work on IPA server
-  LDAP Updater may crash IPA installer
-  NS records not updated by replica
-  Bad return values for ipa-rmkeytab command
-  Update spec with missing BuildRequires for pylint check
-  Let selinux-policy handle port 7390
-  Limit passwd plugin to user container
-  Consolidate man pages and IPA tools help
-  Remove doc from API.txt
-  Improve service manipulation in client install
-  Running ipa-replica-manage as non-root cause errors
-  KDC autodiscovery may fail when domain is not realm
-  A new flag to disable creation of UPG
-  Fix reverse zone creation in ipa-replica-prepare
-  Improve interactive mode for DNS plugin
-  Localization fails for MaxArgumentError
-  Fix forward zone creation in ipa-replica-prepare
-  Connection check program for replica installation
-  Fix support for nss-pam-ldapd
-  Skip know_host check for ipa-replica-conncheck
-  IPA installation with --no-host-dns fails
-  Handle LDAP search references
-  Add ignore lists to migrate-ds command
-  Improve DNS zone creation
-  Add a list of managed hosts
-  Missing krbprincipalname when uid is not set
-  Add port 9443 to replica port checking
-  Fix doc for sudorule runasuser commands
-  Improve IP address handling in IPA option parser
-  Multi-process build problems
-  DNS installation fails when domain and host domain mismatch
-  Fix IPA install for secure umask
-  Allow recursion by default
-  Add DNS record modification command
-  Filter reverse zones in dnszone-find
-  Remove sensitive information from logs
-  Fix ipa-dns-install
-  Fix self-signed replica installation
-  Check IPA configuration in install tools
-  Add new dnszone-find test
-  Fix typo in ipa-replica-prepare
-  Improve long integer type validation
-  Fix sudorule-remove-user
-  Add missing automount summaries
-  Fix man page ipa-csreplica-manage
-  Fix automountkey commands summary
-  Fix invalid issuer in unit tests
-  Hide continue option from automountkey-del
-  Improve error message in ipactl
-  Improve dnszone-add error message
-  Fix idnsUpdatePolicy for reverse zone record
-  Fix client enrollment
-  Update 389-ds-base version
-  Update pki-ca version

.. _nalin_dahyabhai_1:

Nalin Dahyabhai (1):
^^^^^^^^^^^^^^^^^^^^

-  Select a server with a CA on it when submitting signing requests.

.. _pavel_zuna_1:

Pavel Zuna (1):
^^^^^^^^^^^^^^^

-  Fix gidnumber option of user-add command.

.. _petr_vobornik_3:

Petr Vobornik (3):
^^^^^^^^^^^^^^^^^^

-  fixed empty dns record update
-  Fixed adding host without DNS reverse zone
-  Redirection after changing browser configuration

.. _rich_megginson_3:

Rich Megginson (3):
^^^^^^^^^^^^^^^^^^^

-  winsync enables disabled users in AD
-  modify user deleted in AD crashes winsync
-  memory leak in ipa_winsync_get_new_ds_user_dn_cb

.. _rob_crittenden_90:

Rob Crittenden (90):
^^^^^^^^^^^^^^^^^^^^

-  Allow a client to enroll using principal when the host has a OTP
-  Make retrieval of the CA during DNS discovery non-fatal.
-  Cache the value of get_ipa_config() in the request context.
-  Change default gecos from uid to first and last name.
-  Fix ORDERING in some attributetypes and remove other unnecessary
   elements.
-  postalCode should be a string not an integer.
-  Fix traceback in ipa-nis-manage.
-  Suppress --on-master from ipa-client-install command-line and man
   page.
-  Sort entries returned by \*-find by the primary key (if any).
-  The default groups we create should have ipaUniqueId set
-  Always ask members in LDAP*ReverseMember commands.
-  Provide attributelevelrights for the aci components in
   permission_show.
-  Wait for memberof task and DS to start before proceeding in
   installation.
-  Convert manager from userid to dn for storage and back for
   displaying.
-  Modify the default attributes shown in user-find to match the UI
   design.
-  Ensure that the zonemgr passed to the installer conforms to
   IA5String.
-  Handle principal not found errors when converting replication a
   greements
-  Bump version to 2.0.90 to distinguish between 2.0.x
-  Properly handle --no-reverse being passed on the CLI in interactive
   mode
-  Update min nvr for selinux-policy and pki-ca for F-15+
-  Test for forwarded Kerberos credentials cache in wsgi code.
-  Properly configure nsswitch.conf when using the --no-sssd option.
-  Enable 389-ds SSL host checking by default
-  Configure Managed Entries on replicas.
-  Document that deleting and re-adding a replica requires a dirsrv
   restart.
-  Fix migration to work between v2 servers and remove search/size
   limits.
-  Add option to limit the attributes allowed in an entry.
-  Include the word 'member' with autogenerated optional member labels.
-  Do a lazy retrieval of the LDAP schema rather than at module load.
-  Add UID, GID and e-mail to the user default attributes.
-  Fix external CA installation
-  Remove root autobind search restriction, fix upgrade logging & error
   handling
-  Support initializing memberof during replication re-init using GSSAPI
-  Do better detection on status of CA DS instance when installing.
-  Fix indirect member calculation
-  Remove automountinformation as part of the DN for automount.
-  Don't let a JSON error get lost in cascading errors.
-  Add message output summary to sudorule del, mod and find.
-  Return an error message when revocation reason 7 is used
-  Require an imported certificate's issuer to match our issuer.
-  On a master configure sssd to only talk to the local master.
-  The IP address provided to ipa-server-install must be local
-  Do lazy LDAP schema retrieval in json handler.
-  Make data type of certificates more obvious/predictable internally.
-  Update translation files
-  Let the framework be able to override the hostname.
-  Make dogtag an optional (and default un-) installed component in a
   replica.
-  Slight performance improvement by not doing some checking in
   production mode
-  Set the client auth callback after creating the SSL connection.
-  Add pwd expiration notif (ipapwdexpadvnotify) to config plugin def
   attr list
-  Enforce class rules when query=True, continue to not run validators.
-  find_entry_by_attr() should fail if multiple entries are found
-  Fix error in AttrValueNotFound exception example
-  Fix test failure in updater when adding values to a single-value attr
-  Reset failed login count to 0 when admin resets password.
-  Disallow direct modifications to enrolledBy.
-  Document registering to an entitlement server with a UUID as not
   implemented.
-  In sudo labels we should use RunAs and not Run As.
-  Remove the ability to create new HBAC deny rules.
-  Validate that the certificate subject base is in valid DN format.
-  Use information from the certificate subject when setting the NSS
   nickname.
-  Create tool to manage dogtag replication agreements
-  Fix failing tests due to object name changes
-  Set nickname of the RA to 'IPA RA' to avoid confusion with dogtag RA
-  Set the ipa-modrdn plugin precedence to 60 so it runs last
-  Generate a database password by default in all cases.
-  Specify the package name when the replication plugin is missing.
-  Change client enrollment principal prompt to hopefully be clearer.
-  Optionally wait for 389-ds postop plugins to complete
-  A removed external host is shown in output when removing external
   hosts.
-  Don't set krbLastPwdChange when setting a host OTP password.
-  Fix regression when calculating external groups.
-  With the external user/group management fixed, correct the unit
   tests.
-  Set a default minimum value for class Int, handle long values better.
-  Make ipa-client-install error messages more understandable and
   relevant.
-  Add Alexander Bokovoy and Jan Cholasta to contributors file
-  Only call entry_from_entry() after waiting for the new entry.
-  Hide the HBAC access type attribute now that deny is deprecated.
-  Autofill the default revocation reason
-  Don't check for leading/trailing spaces in a File parameter
-  Add an arch-specific Requires on cyrus-sasl-gssapi
-  Revert use of 'can be at least' to 'must be at least' in minvalue
   validator
-  Don't leave dangling map if adding an indirect map fails
-  Fix message in test case for checking minimum values
-  When setting a host password don't set krbPasswordExpiration.
-  Set minimum version of pki-ca to 9.0.10 to pick up new ipa cert
   profile
-  Deprecated managing users and runas user/group in sudorule add/mod
-  Fix date order in changelog.
-  Re-arrange CA configuration code to reduce the number of restarts.

.. _simo_sorce_4_2:

Simo Sorce (4):
^^^^^^^^^^^^^^^

-  Fix resource leaks.
-  ipautil: Preserve environment unless explicitly overridden by caller.
-  install-scripts: avoid using --list with chkconfig
-  Don't set the password expiration to the current time

.. _yuri_chornoivan_1_3:

Yuri Chornoivan (1):
^^^^^^^^^^^^^^^^^^^^

-  Typos in freeIPA messages and man page

.. _kyle_baker_5:

Kyle Baker (5):
^^^^^^^^^^^^^^^

-  Background images and tab hover
-  Search bar style and positioning changes
-  List page spacing changes
-  Tab and spacing on list
-  Facet icon swap and tab sizing

.. _version_2.0.1_05022011:

Version 2.0.1 (05/02/2011)
==========================

-  Fixed undefined label in permission adder dialog box.
-  Add note about ipa-dns-install to ipa-server-install man page.
-  Fix typo in ipa-server-install.
-  Add lint script for static code analysis.
-  Fix lint false positives.
-  Escape LDAP characters in member and memberof searches
-  Add memberHost and memberUser to default indexes
-  Optimize and dynamically verify group membership
-  Delete the sudoers entry when disabling Schema Compat
-  Inconsistent error message for duplicate user
-  Replica installation fails for self-signed server
-  Password policy commands do not include cospriority
-  Improve DNS PTR record validation
-  IPA replica is not started after the reboot
-  Improve Directory Service open port checker
-  Log temporary files in ipa-client-install
-  Prevent uninstalling client on the IPA server
-  pwpolicy-mod doesn't accept old attribute values
-  Fix gidnumber option of user-add command.
-  Allow a client to enroll using principal when the host has a OTP
-  Make retrieval of the CA during DNS discovery non-fatal.
-  Cache the value of get_ipa_config() in the request context.
-  Change default gecos from uid to first and last name.
-  Fix ORDERING in some attributetypes and remove other unnecessary
   elements.
-  postalCode should be a string not an integer.
-  Fix traceback in ipa-nis-manage.
-  Sort entries returned by \*-find by the primary key (if any).
-  The default groups we create should have ipaUniqueId set
-  Provide attributelevelrights for the aci components in
   permission_show.
-  Wait for memberof task and DS to start before proceeding in
   installation.
-  Convert manager from userid to dn for storage and back for
   displaying.
-  Modify the default attributes shown in user-find to match the UI
   design.
-  Ensure that the zonemgr passed to the installer conforms to
   IA5String.
-  Handle principal not found errors when converting replication
   agreements
-  Fix resource leaks.
-  ipautil: Preserve environment unless explicitly overridden by caller.

.. _version_2.0.0_ga_03252011:

Version 2.0.0 GA (03/25/2011)
=============================

-  pwpolicy priority Priority is now a required field in order to add a
   new password policy.
-  Removed nested role from UI.
-  Wait for Directory Server ports to open
-  Prevent stacktrace when DNS AAAA record is added
-  Update translation file (ipa.pot).
-  Always consider domain and server when doing DNS discovery in client.
-  Fix SELinux errors caused by enabling TLS on dogtag 389-ds instance.
-  Ensure that the system hostname is lower-case.
-  Automatically update IPA LDAP on rpm upgrades
-  Domain to Realm Explicitly use the realm specified on the command
   line. Many places were assuming that the domain and realm were the
   same.
-  Fix uninitialized variable.

.. _version_2.0.0_rc_3_03102011:

Version 2.0.0 RC 3 (03/10/2011)
===============================

-  i18n improvements
-  Fixed the self-service page in the WebUI
-  Use TLS for CA replication
-  Setting up Winsync agreements has been fixed

.. _version_2.0.0_rc_2_02282011:

Version 2.0.0 RC 2 (02/28/2011)
===============================

-  Make Indirect membership clearer.
-  Input validation fixes.
-  WebUI improvements.
-  Created default Roles.
-  IPv6 support
-  Documentation updates

.. _version_2.0.0_rc_1_02142011:

Version 2.0.0 RC 1 (02/14/2011)
===============================

-  Installation fixes.
-  DNS improvements.
-  WebUI improvements.

.. _version_2.0.0_beta_2_02032011:

Version 2.0.0 Beta 2 (02/03/2011)
=================================

-  Support of the latest Dogtag packages.
-  Installation fixes.
-  Changes in the DIT structure.
-  New permissions defined against different elements of the tree.
-  Better startup and shutdown handling.
-  Replication improvements.
-  Incremental improvements in IPv6 support.
-  DNS improvements.
-  The package name has been changed to "freeipa" to avoid

collision with IPA v1.x and many others.

.. _version_2.0.0_beta_1_12232010:

Version 2.0.0 Beta 1 (12/23/2010)
=================================

-  FreeIPA has changed its license to GPLv3+
-  Having IPA manage the reverse zone is optional.
-  The access control subsystem was re-written to be more
   understandable. For details see ttp://freeipa.org/page/Permissions
-  Support for SUDO rules
-  There is now a distinction between replicas and their replication
   agreements in the ipa-replica-manage command. It is now much easier
   to manage the replication toplogy.
-  Renaming entries is easier with the --rename option of the mod
   commands.
-  Fix special character handling in passwords, ensure that passwords
   are not logged.
-  Certificates can be saved as PEM files in service-show and host-show
   commands.
-  All IPA services are now started/stopped using the ipactl command.
   This gives us better control over the start/stop order during
   reboot/shutdown.
-  Set up ntpd first so the time is sane.
-  Better multi-valued value handle with --setattr and --addattr.
-  Add support for both RFC2307 and RFC2307bis to migration.
-  UID ranges were reduced by default from 1M to 200k.
-  Add ability to add/remove DNS records when adding/removing a host
   entry.
-  A number of i18n issues have been addressed.

.. _version_2.0.0_alpha_5_11112010:

Version 2.0.0 Alpha 5 (11/11/2010)
==================================

-  Dropped our PKCS#10 parser to use the one provided by python-nss
-  Started enforcing that hosts must be resolvable before adding them
   (use --force if you really want to add them).
-  Provide a reason when adding members to a group fails.
-  Allow de-coupling of user private groups (group-detach).
-  Support for ipa tool failover.
-  Hosts are allowed to retrieve keytabs for their services.
-  More configurable logging, see
   http://freeipa.org/page/IPAv2_config_files
-  Add support for ldap:///self aci rules
-  Use global time and size limit values when searching.
-  Don't include passwords in log files.
-  Make ipactl a lot smarter and add a man page for it.
-  Have certmonger track the IPA service certificates.
-  Initial support for SUDO. You can create the objects but the
   client-side is not done yet.
-  The delete commands now take multiple arguments: ipa user-del user1
   user2 user3 ... usern
-  Remove reliance on 'admin' as a special user. All access control now
   granted via groups.
-  Groups are now created as POSIX by default.
-  Add options to control NTLM hashes. By default LM hash is disabled.
-  Remove the correct password from the history. We were mistakenly
   removing the latest password from the history instead of the oldest.
-  Rename user-lock and user-unlock to user-enable user-disable.
-  The ipa command should return non-zero when something fails.
-  Add gettext support for the C utilities.
-  Add capability to import automount files.
-  Add basic support for user and group renames (more work is needed).
   For now use ipa user-mod --setattr uid=newuser olduser
-  Add flag to group-find to only search on private groups.
-  Set default python encoding to utf-8. This should resolve a number of
   i18n problems.
-  Show indirect members (of groups, hostgroups, netgroups, etc).
-  Remove group nesting from the HBAC service groups.
-  Implement nested netgroups.
-  Add basic support for kerberos lockout policy. You can control how
   many failed attempts are allowed before lockout. What is missing is a
   way to unlock a user. This depends on fixes from MIT Kerberos 1.9.
-  Correct handling of userCategory and hostCategory in netgroups.
-  Updated a lot of man pages.
-  Support Fedora 14.

.. _version_2.0.0_alpha_4_07152010:

Version 2.0.0 Alpha 4 (07/15/2010)
==================================

-  Moved our dogtag SELinux to be installed with the rpm instead of
   during configuration.
-  Fedora 13 moved to gpg2 and dropped gpg. Fix our invocation so we
   work with either (this was preventing replica installations).
-  Query remote server during replica installation to see if the replica
   already exists. This prevents lots of really strange errors during
   replica installation.
-  Fixed SSL error in client enrollment.
-  Changed the way services are handled in HBAC. There is now a separate
   service and servicegroup object that you associate with HBAC rules.
   sssd is already using this new mechanism.
-  First pass at per-command documentation. It still needs a lot of
   work.
-  Fix aci-mod command. It wasn't really working well in almost all
   cases.
-  Add replication version checking. This is one step in better control
   during updates.
-  Don't try to convert a host's password into a keytab with bulk
   enrollment (this was causing krbPasswordExpiration to be set).
-  Add support for User-Private Groups.
-  Worked on error handling in mod_wsgi. Now hopefully a shorter and
   less scary backtrace will be thrown when things go bump in the night.
-  Add new API to disable service and host principals.
-  Significant cleanup of crypto code. Using python-nss for a lot more
   (and more to come).
-  Fixed some errors in and made ipa-compat-manage and ipa-nis-manage
   more bullet-proof.
-  Fixed netgroups plugin, it was generating the wrong attributes.
-  Other minor polish and bug fixes.

.. _version_2.0.0_alpha_3_05072010:

Version 2.0.0 Alpha 3 (05/07/2010)
==================================

-  better i18n support including a few translations
-  use mod_wsgi instead of mod_python
-  the CA is a required component and is now configured by default. Pass
   --selfsign to the installer to use the old self-signed CA
-  A default Host-Based Access Control (HBAC) rule is created that
   grants all users the ability to log into any host from any host. This
   was done to simplify initial testing, it is expected this rule,
   allow_all, will be removed before you deploy.
-  We no longer enable nscd, sssd handles caching now

.. _version_2.0.0_alpha_2_02182010:

Version 2.0.0 Alpha 2 (02/18/2010)
==================================

-  Draft Web-based UI
-  Simplified migration of the users from IPA v1 or external LDAP server
-  IPA client component to configure SSSD to integrate with IPA
-  Integration with "certmonger" certificate tracking utility. The
   utility allows automatic provisioning, tracking and renewal of
   certificates on a member server.
-  General improvements and enhancements across the whole project.

.. _version_2.0.0_alpha_1_10282009:

Version 2.0.0 Alpha 1 (10/28/2009)
==================================

-  Pluggable and extensible framework for UI/CLI
-  Optionally installable DNS server
-  Optionally installable Certificate Authority to manage server
   certificates
-  NIS compatibility plug-in

.. _version_1.2.1:

Version 1.2.1
=============

-  Add ipa-compat-manage utility
-  Ensure the CA cert is always included when preparing a replica
-  Fix error in validation when editing new groups via the UI
   `471808 <https://bugzilla.redhat.com/show_bug.cgi?id=471808>`__
-  Fixed some crash conditions in the password plugin

.. _version_1.2.0:

Version 1.2.0
=============

-  Active Directory User Synchronization
-  Schema Compatibility Plug-in (native Solaris nss_ldap now works)
-  Fix group mapping /etc/ldap.conf so getent works
   `431603 <https://bugzilla.redhat.com/show_bug.cgi?id=431603>`__
-  The ipa-addservice command failed if the realm name was included in
   the principal name.
   `437566 <https://bugzilla.redhat.com/show_bug.cgi?id=437566>`__
-  The ipa_webgui service did not start after the initial installation.
   `440475 <https://bugzilla.redhat.com/show_bug.cgi?id=440475>`__
-  IPA does not handle group names with spaces properly.
   `450613 <https://bugzilla.redhat.com/show_bug.cgi?id=450613>`__
-  The ipa-moduser -f command may not change the appearance of the
   user's first name when shown as the full name.
   `451318 <https://bugzilla.redhat.com/show_bug.cgi?id=451318>`__
-  The potential existed for Directory Server to crash if you nested
   groups too deeply.
   `451358 <https://bugzilla.redhat.com/show_bug.cgi?id=451358>`__
-  IPA replicas did not fully synchronize in single-master, dual-replica
   topology environments.
   `468732 <https://bugzilla.redhat.com/show_bug.cgi?id=468732>`__
-  Fix error in validation when adding new groups via the UI
-  Add list of DNs that are not controlled by password policy.
   `471130 <https://bugzilla.redhat.com/show_bug.cgi?id=471130>`__

.. _version_1.1.0:

Version 1.1.0
=============

-  Ensure that the realm name is upper-case.
-  When an LDAP connection fails, display the host one is trying to
   connect to.
   `450111 <https://bugzilla.redhat.com/show_bug.cgi?id=450111>`__
-  Add our own SIGTERM handler to ipa_webgui so we can do clean
   shutdowns.
   `450211 <https://bugzilla.redhat.com/show_bug.cgi?id=450211>`__
-  Make it clear which packages are being configured and which aren't.
   `450175 <https://bugzilla.redhat.com/show_bug.cgi?id=450175>`__
-  Add -p/--password option so the DM password can be passed on the
   command-line.
-  Don't make the search criteria lower-case so one can do
   case-sensitive searches (such as looking for HTTP principals).
   `449975 <https://bugzilla.redhat.com/show_bug.cgi?id=449975>`__
-  Man page improvements.
-  Fix issue of double logging in ipa_error.log.
-  Add a Not Found (404) template
-  Only print a traceback on 500 errors.
-  Don't prompt regarding previous DS installations in unattended mode.
-  Add two new options, --addattr and --setattr, to allow arbitrary
   attributes to be added and set when a new user or group is created.
   `449006 <https://bugzilla.redhat.com/show_bug.cgi?id=449006>`__
-  Make password not mandatory in ipa-adduser
-  Make ipa_kpasswd listen on each single interface explicitly instead
   of 0.0.0.0.
-  Fix the case where domain != lower(REALM) add the domain to the
   ipa.conf file for apps that need to know. This should fix a bug in
   the replica setup.
-  Move admin into cn=users,cn=accounts
-  Move non-user-configurable configuration elements to TurboGears
   app.cfg file.
   `432908 <https://bugzilla.redhat.com/show_bug.cgi?id=432908>`__
-  Change file mode of log files to 600.
   `446869 <https://bugzilla.redhat.com/show_bug.cgi?id=446869>`__
-  Ensure hostnames are lower during installation and when adding
   service princs.
   `447381 <https://bugzilla.redhat.com/show_bug.cgi?id=447381>`__
-  Remove broken link for IE configuration and replace sample
   domain/realm. Also fix some HTML errors.
   `447445 <https://bugzilla.redhat.com/show_bug.cgi?id=447445>`__
-  Do uniqueness check on phone numbers and cn entered via the UI.
   `445286 <https://bugzilla.redhat.com/show_bug.cgi?id=445286>`__
-  Don't pass the Directory Manager password on the command-line to
   ldapmodify.
   `446865 <https://bugzilla.redhat.com/show_bug.cgi?id=446865>`__
-  Use split instead of find as split does not fail to provide a
   complete component if no '.' is found. This should better handle a
   realm with no periods in it.
-  Improve DNA plugin and ensure that the numbers it hands out are
   unique.
-  Don't ask the user again if he wants to replace bind configuration
   files if he specified --setup-bind.
   `430090 <https://bugzilla.redhat.com/show_bug.cgi?id=430090>`__
-  Make sure all services are stopped during uninstall.
   `440322 <https://bugzilla.redhat.com/show_bug.cgi?id=440322>`__
-  Hack to not require a First Name in the UI for the admin user since
   it lacks the inetOrgPerson objectclass.
-  Display information on how to uninstall a partially installed server.
   `442454 <https://bugzilla.redhat.com/show_bug.cgi?id=442454>`__
-  Include information on where to look if a hostname resolves to
   localhost.
   `442812 <https://bugzilla.redhat.com/show_bug.cgi?id=442812>`__
-  On IPA Servers configure PAM and nss_ldap to connect to ourselves
   using localhost.
-  Detect existing DS instances and prompt for removal during replica
   install.
-  Don't allow the IPA server service principals to be removed.
-  Move entire web space to be rooted in /ipa
-  Add --verbose option so the HTTP headers and XML request/response can
   be seen in the ipa-\* tools.
   `443987 <https://bugzilla.redhat.com/show_bug.cgi?id=443987>`__
-  Fixed various memory leaks in memberOf plug-in.
-  Make sure we always have the [domain-realm] section or kerberos libs
   misbehave.

.. _version_1.0.0:

Version 1.0.0
=============

Lots of bug fixes

.. _version_0.99:

Version 0.99
============

Feature complete

`Category:Documentation <Category:Documentation>`__
`Category:Features <Category:Features>`__
