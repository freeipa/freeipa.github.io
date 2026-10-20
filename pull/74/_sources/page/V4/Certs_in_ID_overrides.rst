Certs_in_ID_overrides
=====================

Overview
--------

To allow IPA to provide the user certificates for the user identities
coming from the Active Directory.



Use Cases
---------

The feature will be used by administrators to manually provide the user
certificates for the users whose entries do not belong to the FreeIPA.

Design
------

For FreeIPA, extend the 'ipaUserOverride' objectclass with a new
multivalued 'usercertificate' binary attribute storing the certificate.
Provide standard CLI and WebUI interface to allow administrators to
manage the values.

Implementation
--------------

The implementation uses the standard FreeIPA schema interface and LDAP
object framework.



Feature Management
------------------

UI

The Web UI will be extended to support the new 'usercertificate'
attribute in a consistent manner in respect with the other LDAP objects
that are providing it.

CLI

========================== ==============
Command                    Options
========================== ==============
idoverrideuser-add         --certificate=
idoverrideuser-add-cert    --certificate=
idoverrideuser-remove-cert --certificate=
========================== ==============

Configuration
----------------------------------------------------------------------------------------------

This feature has no configuration impacts on the FreeIPA side.

Upgrade
----------------------------------------------------------------------------------------------

The server's schema will be updated to support the 'usercertificate'
attribute for the ID override objects as an extension to the
'ipaUserOverride' object class. This happens transparently during a
regular FreeIPA upgrade, hence no custom action needs to be taken.



How to Test
-----------

Ideally, this feature should be tested with its SSSD counterpart.

To test in isolation, the following steps should be taken:

-  (optional) Establish a trust with an Active Directory domain
-  Create an ID view
-  Make sure the ID view applies to the host being used for the testing
-  Create an user ID override for the selected user identity
-  Add/Remove/Read the certificate from the user ID override entry



Test Plan
---------

`Certs in ID overrides V4.4 test
plan <V4/Certs_in_ID_overrides/Test_Plan>`__