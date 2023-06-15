\__NOTOC_\_

Overview
========

Related ticket: `#3544 <https://fedorahosted.org/freeipa/ticket/3544>`__

When AD trust is established, upon validation AD DC is capable to fetch
information about trusted forest configuration, including additional
name suffixes to use when creating routing to trusted domain. This gives
an opportunity to make multiple DNS domain configurations supported in
AD trust.

When AD DC knows about these additional DNS domains, it is capable to
properly ask our KDCs for tickets for services in those domains and
trust them via our trust.

This is how it looks from Windows side:

.. figure:: Win2012-multiple-suffixes.png
   :alt: Win2012-multiple-suffixes.png

   Win2012-multiple-suffixes.png

For the domains that Windows manages as a DNS server, it exposes the
same list via the same SMB interface.

Realmdomains information will also be used by SSSD to fetch and
configure mapping between DNS domains and realms within IPA.

So from our side it means we need to hook into ``'ipa dnszone-add'`` and
``'ipa dnszone-del'`` to call ``'ipa realmdomains-mod'`` for
non-forwarded zones. Any error when removing the domain with
``'ipa realmdomains-mod --del-domain'`` should be ignored because the
domain in question might not be in the list of realm's domains by
admin's intention.

Also, a ``_kerberos TXT`` record containing the realm name should be
added to the DNS zone as a part of this process.

.. _use_cases2:

Use Cases
=========

-  Admin creates a new DNS zone in IPA using the ``ipa dnszone-add``
   command. As a result, a new entry is added to realmdomains and
   ``_kerberos TXT`` record is added to the DNS zone.
-  Admin creates a new forwarded zone in IPA. No other side effects
   happen since the zone is not managed by IPA.
-  Admin adds a new domain ``new-domain.com`` to the realmdomains
   object, using ``ipa realmdomains-mod --add-domain=new-domain.com`` or
   ``ipa realmdomains-mod --domain={ipa-domain.org,new-domain.com}``. If
   IPA manages the DNS zone for ``new-domain.com``, ``_kerberos TXT``
   record is added to the zone.
-  Admin deletes a domain ``new-domain.com`` from the realmdomains
   object, using ``ipa realmdomains-mod --del-domain=new-domain.com`` or
   ``ipa realmdomains-mod --domain={ipa-domain.org}``. If IPA manages
   the DNS zone for ``new-domain.com`` and ``_kerberos TXT`` record
   exists in this zone and contains the name of the realm, the record is
   deleted.
-  Admin deletes a DNS zone using the ``ipa dnszone-del`` command. If a
   domain with the same name as the zone exists in the realmdomains
   object, it is deleted.

Design
======

Add functionality to the ``post_callback()`` methods of ``dnszone_add``
and ``dnszone_del``. These methods should now include the calls to
``realmdomains-mod --add-domain`` and ``realmdomains-mod --del-domain``,
respectively, in order to implement the addition/deletion of entries to
the realmdomain object upon DNS zone creation. The call to
``realmdomains-mod`` will not be executed in the following cases:

-  for our own domain
-  for forwarded zones
-  for reverse zones

Implement the ``execute()`` method of ``realmdomains_mod``. This method
should include the calls to ``dnsrecord_add`` and ``dnsrecord_del``, in
order to implement the addition/deletion of the ``_kerberos TXT`` record
upon modification of the realmdomains object. The call to
``dnsrecord_del`` will not be executed in the case of our own domain.

Add a normalizer to all the options of ``realmdomains_mod``. The
normalizer should lowercase the arguments and strip the trailing dot.

Add new unit tests to cover these newly added functionalities.

.. _feature_management2:

Feature Management
==================

UI
~~

N/A

CLI
~~~

N/A

.. _major_configuration_options_and_enablement2:

Major configuration options and enablement
==========================================

N/A

Replication
===========

N/A

.. _updates_and_upgrades2:

Updates and Upgrades
====================

N/A

Dependencies
============

N/A

.. _external_impact2:

External Impact
===============

N/A

.. _rfe_author2:

RFE Author
==========

`akrivoka <User:Akrivoka>`__ 06:13, 12 April 2013 (EDT)
