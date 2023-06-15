Overview
--------

Support for external trust to a domain from Active Directory forest

An external trust is a trust relationship between Active Directory
domains that are in different Active Directory forests. While forest
trust always requires to establish trust between root domains of the
Active Directory forests, external trust can be established to any
domain within the forest.



Use Cases
---------

As an Active Directory domain admin, I want to establish trust between
IPA and my domain only. The trust between IPA and an external Active
Directory domain will be non-transitive as no users or groups from other
Active Directory domains will have access to IPA resources.

Design
------

External trust between Active Directory domains is by definition
non-transitive and enforces SID filtering between the domain boundaries.
This means only users and groups with SIDs from the trusted domain can
use the resources and be visible on IPA systems. None of other users and
groups from domains the trusted domain trusts within its own Active
Directory forest or other externally trusted domains will be allowed to
access IPA resources.

Implementation
--------------

External trust feature re-uses existing forest trust infrastructure.
There are several specific changes to allow supporting external trust:

-  **Non-transitivity**: since external trust is non-transitive by
   definition, any attempt to set transitivity feature of the trust link
   with LSA SetInformationTrustedDomain() command will fail. Thus, there
   is no need to set transitivity for the external trust.
-  **Trust attributes**: external trust can be detected by looking into
   absense of ipaNTTrustAttributes LDAP attribute of the trusted domain
   object.



Feature Management
------------------

UI
~~

An option 'external trust' needs to be added to Web UI, corresponding to
'--external' flag in 'trust-add' command in CLI.

CLI
~~~

An external trust creation can be requested by passing additional flag
'--external=true' to the 'trust-add' command. The flag defaults to
'false', e.g. no external trust would be created.

========= =====================
Command   Options
========= =====================
trust-add --external=true/false
========= =====================

Configuration
~~~~~~~~~~~~~

No configuration options needed.

Upgrade
-------

No changes on upgrades. The trust properties are only set up at trust
creation time.

.. _how_to_test26:

How to Test
-----------

In order to test the external trust, attempt to create a trust to
non-root domain in an Active Directory forest. It should fail without
'--external=true' option and should be able to establish the external
trust with '--external=true' option to 'trust-add' command.

A type of the trust can be seen with 'trust-show' command.

.. _test_plan26:

Test Plan
---------

`External trust to AD V4.4 test
plan <V4/External_trust_to_AD/Test_Plan>`__
