\__NOTOC_\_

Overview
========

DNS is an integral part of IPA, particularly when AD trust is involved.
It should remain an optional feature but rather than relying on users to
remember to pass an option at install time, it should be prompted for.

.. _use_cases10h:

Use Cases
=========

.. _basic_ipa_installation:

Basic IPA installation
----------------------

Even in a simple, one master test installation letting IPA manage its
own domain has advantages like automatic SRV record management. Adding
additional masters is simplified and makes failover easier since DNS is
automatically maintained.

.. _ad_trust:

AD Trust
--------

DNS is even more critical for a working AD trust configuration. There
are a number of different ways it can be configured (parent of the
domain, peer, delegated subdomain), all of which are easier and more
automatically managed if IPA manages DNS itself.

Design
======

The proposed solution. This may include but is not limited to:

Add a prompt for configuring bind as the first prompt in an interactive
install.

``Do you want to configure integrated DNS (BIND)? [no]:``

The default value will be no.

Implementation
==============



Feature Management
==================

UI
~~

How the feature will be manged via the UI

CLI
~~~



Major configuration options and enablement
==========================================

Replication
===========

None



Updates and Upgrades
====================

None

Dependencies
============

None



External Impact
===============

None



RFE Author
==========

Adam Young
