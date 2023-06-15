\__NOTOC_\_

Overview
========

A Kerberos principal can have a slew of flags set on it. This is managed
in the krbTicketFlags attribute as an integer value, where specific bits
represent specific flags.

We will not add support for all available flags in Kerberos at once.
Support for flags will be added gradually, on a use-case basis.

.. _use_cases10c:

Use Cases
=========

Windows requires the ok_as_delegate flag be set in order to forward
credentials. See ticket https://fedorahosted.org/freeipa/ticket/3329

There may be flags required for upcoming OTP work.

Design
======

Add a Bool option to service and host commands for every flag, e.g.
--ok-as-delegate.

Implementation
==============

Add a Bool virtual attribute for every flag to service and host plugins.
Convert values of these virtual attributes to/from krbTicketFlags in
service and host commands.

.. _feature_managment:

Feature Managment
=================

UI
~~

A new checkbox is needed to set/display the status of this flag.

CLI
~~~

Add a new Bool option to the service and host commands, ok-as-delegate.

::

   # ipa service-mod --ok-as-delegate=TRUE ldap/ipa.example.com

One can confirm that the principal was updated by using kadmin.local to
get a Kerberos-side view of the principal.



Major configuration options and enablement
==========================================

None.

Replication
===========

None.



Updates and Upgrades
====================

None.

Dependencies
============

None.



External Impact
===============

None.



Test Plan
=========

Assumptions:

-  IPA server fqdn: ``server.example.com``
-  IPA service host: ``host.example.com``

| 
| Install a FreeIPA server with ``ipa-server-install``.

``OK_AS_DELEGATE`` flag will be used as an example Kerberos flag in this
test plan.
