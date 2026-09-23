IPAv3_300_beta3
===============

The FreeIPA team is proud to announce version FreeIPA v3.0.0 beta 3.

It can be downloaded from http://www.freeipa.org/page/Downloads.

A build is available onlly for Fedora 17 via the freeipa-devel repo on
www.freeipa.org: http://freeipa.org/downloads/freeipa-devel.repo . To
install in Fedora 17 the updates repo repository needs to be enabled as
well.

For additional information see the AD Trust design page
http://freeipa.org/page/IPAv3_AD_trust and the AD Trust testing page
http://freeipa.org/page/IPAv3_testing_AD_trust.



Highlights since 3.0.0 beta 2
-----------------------------

-  Cooperate with new 389-ds-base winsync POSIX plugin so that AD POSIX
   attribute can be synced with IPA.
-  Improvements to schema upgrade process.
-  Prevent last admin from being disabled.
-  Exclude some attributes from replication.
-  Notify success on add, delete and update in UI.
-  Set the e-mail attribute on new users by default.
-  Rename range commands to idrange,
-  Improvements to idrange command-line.
-  SSH public key format has been changed to OpenSSH-style public keys.

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



Detailed changelog
------------------

Alexander Bokovoy (4):

-  Recover from invalid cached kerberos credentials in ipasam
-  Fix ipasam ipaNThash magic regen to actually fetch updated password
-  Add ACI to allow regenerating ipaNTHash from ipasam
-  Ask for admin password in ipa-adtrust-install

Jan Cholasta (1):

-  Use OpenSSH-style public keys as the preferred format of SSH public
   keys.

John Dennis (4):

-  DN objects hash differently depending on case
-  ipactl exception not handled well
-  ipa user-find --manager does not find matches
-  prevent last admin from being disabled

Martin Kosek (12):

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

Petr Viktorin (3):

-  Internationalization for public errors
-  Run ntpdate in verbose mode, not debug (i.e. no-op) mode
-  Add nsds5ReplicaStripAttrs to replica agreements

Petr Vobornik (15):

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

Rob Crittenden (8):

-  Don't generate password history error if history is set to 0.
-  Restrict the SELinux user map user MLS value to 0-1023
-  Support the new Winsync POSIX API.
-  Set minimum of 389-ds-base to 1.2.11.8 to pick up cache warning.
-  Add version to replica prepare file, prevent installing to older
   version
-  Set the e-mail attribute using the default domain name by default
-  Fix some restart script issues found with certificate renewal.
-  Become IPA v3 beta 3 (3.0.0.pre3)

Sumit Bose (27):

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

Tomas Babej (5):

-  Adds dependency on samba4-winbind.
-  Improves deletion of PTR records in ipa host-del
-  Fixes different behaviour of permission-mod and show.
-  Change slapi_mods_init in ipa_winsync_pre_ad_mod_user_mods_cb
-  Sort policies numerically in pwpolicy-find