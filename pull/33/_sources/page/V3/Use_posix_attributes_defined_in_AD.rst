Use_posix_attributes_defined_in_AD
==================================

\__NOTOC_\_

Overview
========

The first implementation of the trusts solution takes SID of the user
entry and uses a special algorithm to create unique UID for the user.
The similar approach is used for GID. This works fine for the
deployments that do not have POSIX attributes defined in AD.



Use Cases
=========

In the cases when POSIX attributes are defined in AD there are usually
already clients that leverage these attributes so switching to the SID
-> UID/GID model is not acceptable. Instead IPA and SSSD should be able
to respect the POSIX attributes defined in the AD.

Design
======



Exteding ID Range schema to support various types of ID Ranges
----------------------------------------------------------------------------------------------

Schema
^^^^^^

The schema has been extended to support various range types. A new
attribute **ipaRangeType** has been added to the schema.

``attributeTypes: (2.16.840.1.113730.3.8.11.41 NAME 'ipaRangeType' DESC 'Range type' EQUALITY caseIgnoreIA5Match SUBSTR caseIgnoreIA5SubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 X-ORIGIN 'IPA v3' )``

The **ipaIDrange** objectclass now requires the **ipaRangeType**
attribute:

``objectClasses: (2.16.840.1.113730.3.8.12.15 NAME 'ipaIDrange' ABSTRACT MUST ( cn $ ipaBaseID $ ipaIDRangeSize $ ipaRangeType ) X-ORIGIN 'IPA v3' )``



Allowed values
^^^^^^^^^^^^^^

The **ipaRangeType** must obtain exactly one of the following values:

-  *ipa-local* for a local domain range
-  *ipa-ad-winsync* for Active Directory winsync range
-  *ipa-ad-trust*: for Active Directory domain range
-  *ipa-ad-trust-posix* for Active Directory trust range with POSIX
   attributes
-  *ipa-ipa-trust* for IPA trust range

This is enforced via CLI and UI validation, not in LDAP itself.



Detecting the ID range type for the AD trusted domain
----------------------------------------------------------------------------------------------

To avoid the need of specifying the range parameters, such as type,
range size and base, we query the AD DC.

Depending on whether the users/groups defined have specified UID and GID
POSIX attributes, we create a range of appropriate type during the
execution of *trust-add* command.



Connecting to the AD
^^^^^^^^^^^^^^^^^^^^

This code will need to access AD Global Catalog service which implies
authentication. Such authentication should be done using Kerberos ticket
with MS-PAC attached, or trusted account like we currently do.

MS-PAC is not currently attached to HTTP/fqdn or host/fqdn tickets and
we cannot use user's ticket due to fact that our KDC does not allow wide
open S42Proxy relay to other domain for HTTP/fqdn. See the related
ticket: https://fedorahosted.org/freeipa/ticket/3651



Obtaining information on use of POSIX attributes in AD
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Information on the can be obtained conducting the following ldapsearch:

::

   | ``# ``\ ``, ypservers, ypServ30, RpcServices, System, ``
   | ``dn: CN=idm,CN=ypservers,CN=ypServ30,CN=RpcServices,CN=System,DC=idm,DC=com``
   | ``objectClass: top``
   | ``objectClass: msSFU30DomainInfo``
   | ``cn: idm``
   | ``distinguishedName: CN=idm,CN=ypservers,CN=ypServ30,CN=RpcServices,CN=System,DC=idm,DC=com``
   | ``instanceType: 4``
   | ``whenCreated: 20130625220322.0Z``
   | ``whenChanged: 20130626190645.0Z``
   | ``uSNCreated: 53308``
   | ``uSNChanged: 61507``
   | ``showInAdvancedViewOnly: TRUE``
   | ``name: idm``
   | ``objectGUID:: 7ntK3glNqE2KGmHY1SePWQ==``
   | ``objectCategory: CN=msSFU-30-Domain-Info,CN=Schema,CN=Configuration,DC=idm,DC=com``
   | ``dSCorePropagationData: 16010101000000.0Z``
   | ``msSFU30MasterServerName: tbad``
   | **``msSFU30OrderNumber``**\ ``: 10000``
   | ``msSFU30Domains: idm``
   | **``msSFU30MaxGidNumber``**\ ``: 10001``
   | **``msSFU30MaxUidNumber``**\ ``: 10002``



ID base detection
^^^^^^^^^^^^^^^^^

Attribute **msSFU30OrderNumber** holds the base from which AD starts
automatically assigning UID/GID numbers. Please note that any manual
intervention will not be reflected in this attribute, so changing an
user's UID to 9800 manually will not change the value of this attribute.



Range size detection
^^^^^^^^^^^^^^^^^^^^

Attributes **msSFU30MaxUidNumber** and **msSFU30MaxGidNumber** hold the
value of next available UID/GID.



Detecting the range type
^^^^^^^^^^^^^^^^^^^^^^^^

When dealing with the AD trust, we need to only decide between
*ipa-ad-trust* type and *ipa-ad-trust-posix* type.

We can use presence of **msSFU30MaxUidNumber** and
**msSFU30MaxGidNumber** attributes as an indicator that there exists an
user with POSIX attributes defined.

Implementation
==============

N/A



Feature Management
==================

UI

TODO: **ipaRangeType** needs to be made available via UI.
https://fedorahosted.org/freeipa/ticket/3759

TODO: **range_type** needs to be made available via UI.
https://fedorahosted.org/freeipa/ticket/3049

CLI



idrange-add
-----------

An *--type* option has been added to the *idrange-add* command. Note
that *idrange-mod* does not have this option. Since *--type* corrseponds
to **ipaRangeType** attribute, the allowed value is any of the allowed
values for **ipaRangeType** attribute.



trust-add
---------

An *--range-type* option has been added to the *trust-add* command. All
range types except *ipa-local* are allowed as values. Further validation
is based on the trust type. For AD trust, only one of *ipa-ad-trust* or
*ipa-ad-trust-posix* is allowed.

Please note that setting this attribute overrides any detection-based
decision that is otherwise performed. You can set *ipa-ad-trust-posix*
range type using this option for a trust with AD which does not have IdM
for Unix support and therefore would not get any users from the AD. The
reasoning behind this behaviour is that admin should have authoritative
way to set the range type, since the detection might fail. Generally,
you should not need to force the range type using the --range-type
option.



Major configuration options and enablement
==========================================

N/A

Replication
===========

N/A



Updates and Upgrades
====================

On package update (or whenever ipa-upgradeconfig is ran), all ID ranges
that do not have the **ipaRangeType** attribute set, have the attribute
value filled in according to their objectclass:

-  ranges with *ipatrustedaddomainrange* objectclass are assigned
   *ipa-ad-trust* type
-  ranges with *ipadomainidrange* objectclass are assigned *ipa-local*
   type

Dependencies
============

N/A



External Impact
===============

N/A



Backup and Restore
==================

N/A



Test Plan
=========

Test scenarios that will be transformed to test cases for FreeIPA
Continuous Integration during implementation or review phase.



Common assumptions
------------------

-  FreeIPA server: ipa.example.org
-  Active Directory: ad.example.org

These tests assume AD with POSIX support. More detailed info about the
particular setup steps can be found in the test cases below.



RFE Author
==========

`User:Tbabej <User:Tbabej>`__