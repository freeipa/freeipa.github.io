\__NOTOC_\_

Overview
========

Ticket `3869 <https://fedorahosted.org/freeipa/ticket/3869>`__

The CLI options for ipa-server-certinstall are not ideal. Clean them up.

All changes are backwards compatible.

.. _use_cases10b:

Use Cases
=========

Same as for ipa-server-certinstall

Design
======

Before this change, ``ipa-replica-certinstall -h`` output was:

| ``   Usage: ipa-server-certinstall [options]``
| ``   ``
| ``   Options:``
| ``   -h, --help            show this help message and exit``
| ``   -d, --dirsrv          install certificate for the directory server``
| ``   -w, --http            install certificate for the http server``
| ``   --dirsrv_pin=DIRSRV_PIN``
| ``                           The password of the Directory Server PKCS#12 file``
| ``   --http_pin=HTTP_PIN   The password of the Apache Server PKCS#12 file``

The actual allowed usages were:

| ``   ipa-server-certinstall -h``
| ``   ipa-server-certinstall -d --dirsrv_pin ``\ `` ``
| ``   ipa-server-certinstall -w --http_pin ``\ `` ``
| ``   ipa-server-certinstall -d --dirsrv_pin ``\ `` -w --http_pin ``\ `` ``

This change:

-  Adds a ``--pin`` option that replaces ``--dirsrv_pin`` and
   ``--http_pin``. The old options will remain as deprecated aliases.

-  Adds a ``-p, --dirman-password`` option to specify the directory
   manager password (necessary for replacing the DS cert).

-  Mentions in the usage string that

   -  a PKCS#12 cert argument is required
   -  at least one of ``-d`` or ``-w`` is required

Implementation
==============

It's possible that only a part of this proposal, namely adding the
``--pin`` option, will be be implemented in 3.3. In that case the rest
will be atted later.



Feature Management
==================

UI
~~

N/A

CLI
~~~

See Design



Major configuration options and enablement
==========================================

N/A

Replication
===========

N/A



Updates and Upgrades
====================

N/A

Dependencies
============

N/A



External Impact
===============

Docs and tests will need to be updated, although since the change is
backwards compatible, there's no rush.



Backup and Restore
==================

N/A



Test Plan
=========

Tests are to be included in the CA-less test plan,
`CA-less_install <CA-less_install>`__, and any future test cases that
exercise ipa-server-certinstall.



RFE Author
==========

`Petr Viktorin <User:pviktorin>`__
