\__NOTOC_\_

Overview
========

Related tickets:

-  `3289: Make SID checks for MS-PAC filter
   configurable <https://fedorahosted.org/freeipa/ticket/3289>`__
-  `3231: Need to relax MS-PAC
   checks <https://fedorahosted.org/freeipa/ticket/3231>`__

Microsoft Windows 2012 slightly changed what it sends in the MS-PAC, and
it sends a special SID in the ExtraSids buffer. We used to not accept
this MS-PAC and raise validation error, but ticket
`3231 <https://fedorahosted.org/freeipa/ticket/3231>`__ introduced a
static list of SIDs that are filtered and are excluded from the MS-PAC
to avoid this error. A target of this RFE is to include this list in
LDAP to allow Administrator to change the list and add or remove SID to
filter.

.. _use_cases113:

Use Cases
=========

Windows 2012 user from a trusted AD domain tries to authenticate to IPA
domain, but his ticket is refused due to MS-PAC check. Administrator may
want to extend the default list of SID so that the colliding SID is
filtered.

Design
======

.. _configuration_granularity:

Configuration granularity
-------------------------

SID blacklist should be configured per-trust. Administrator should be
able to configure a blacklist for both *incoming* MS-PAC (i.e.
authentication from a trusted domain to IPA domain) and *outgoing*
MS-PAC (i.e. for transitive authentication of a user from IPA trusted
domain trying to authenticate to other domain trusted by IPA, but which
is not trusted directly by this domain).

.. _schema_updates:

Schema updates
--------------

The feature will introduce 2 new *attributeTypes* which will be added to
*MAY* list of *ipaNTTrustedDomain* object class:

| ``attributetypes: ( 2.16.840.1.113730.3.8.11.38 NAME 'ipaNTSIDBlacklistIncoming'``
| `` DESC 'Extra SIDs filtered out from incoming MS-PAC'``
| `` EQUALITY caseIgnoreIA5Match``
| `` SUBSTR caseIgnoreIA5SubstringsMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 X-ORIGIN 'IPA v3' )``
| ``attributetypes: ( 2.16.840.1.113730.3.8.11.39 NAME 'ipaNTSIDBlacklistOutgoing'``
| `` DESC 'Extra SIDs filtered out from outgoing MS-PAC'``
| `` EQUALITY caseIgnoreIA5Match``
| `` SUBSTR caseIgnoreIA5SubstringsMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 X-ORIGIN 'IPA v3' )``

.. _ipa_kdb_changes:

ipa-kdb Changes
---------------

Currently, ipa-kdb uses ``mspac_well_known_sids`` static list of SIDs to
filter SIDs from MS-PAC. Changes:

-  ``struct ipadb_mspac`` will be changed:

   -  ``well_known_sids`` will be renamed to ``sid_blacklist_incoming``
   -  new ``sid_blacklist_outgoing`` will be added

-  ``ipadb_mspac_fill_well_known_sids`` function will be updated to read
   these new attributes from LDAP and if it finds these attributes, it
   will fill their value to ``mspac->sid_blacklist_incoming`` or
   ``mspac->sid_blacklist_outgoing`` respectively. If the attributes for
   the trust is missing, it will use the default value in
   ``mspac_well_known_sids``.
-  Note that ``mspac->sid_blacklist_outgoing`` **will be unused** until
   the transitive trusts functionality is implemented.

These new attributes should not cause high LDAP load as
``ipadb_reinit_mspac`` it is run at most once per minute.

Implementation
==============

N/A

.. _feature_managment:

Feature Managment
=================

UI
~~

UI will need to allow updating these new attributes in *Settings* tab in
*IPA Server* -> *Trusts* section.

CLI
~~~

CLI should allow editing of these new attributes. They should not be
displayed by default in *trust-show* or *trust-find* command, but only
with *--all* option to keep clarity of trust entries in these commands.



Major configuration options and enablement
==========================================

N/A

Replication
===========

New attributes will be replicated.



Updates and Upgrades
====================

The 2 new *attributeTypes* will be added and one *ipaNTTrustedDomain*
object class will be updated.

**QUESTION**: We can either fill *ipaNTSIDBlacklistIncoming* and
*ipaNTSIDBlacklistIncoming* for all current trusts during updates or
fill them only for re-established and new trusts. The latter would avoid
unnecessary update plugin.

The prefilled list should be equal to ``mspac_well_known_sids`` list in
``ipa_kdb_mspac.c``.

Dependencies
============

N/A



External Impact
===============

N/A
