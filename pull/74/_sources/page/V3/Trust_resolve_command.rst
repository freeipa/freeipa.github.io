Trust_resolve_command
=====================

\__NOTOC_\_

Overview
========

Related tickets `#3302 <https://fedorahosted.org/freeipa/ticket/3302>`__

We expose an interface to resolve Security Identifiers (SIDs) from a
foreign trusted domain. The command is used by Web UI to resove the SIDs
added as external group members.



Use Cases
=========

-  Trust administrator adds external group and adds several external
   members to it from a trusted domain. These members are stored
   internally as SIDs and should be shown as proper names of groups and
   users in Web UI.

Design
======

Add ``trust-resolve`` command to implement a query interface. Query is
resolved via SSSD. SSSD encapsulates actual resolution mechanism under
libsss_nss_idmap API. Details on SSSD design are available at
https://fedorahosted.org/sssd/wiki/DesignDocs/NSSResponderIDMappingCalls



Feature Management
==================

UI

Since SID resolution may potentially take time at server side, SID
resolution should be done as asynchronous task. This is implemented via
special SID facet in Web UI -- that is, content of any field with facet
type 'sid' will be used to query 'trust-resolve' command and it in case
of successful resolution will be replaced by the returned value.

CLI



trust-resolve
----------------------------------------------------------------------------------------------

``trust-resolve`` displays SIDs and names they resolved to.

Required command options:

-  ``--sids``: comma-separated list of SIDs to resolve.



Major configuration options and enablement
==========================================

The command will be only functional when the trusts are configured.

Replication
===========

N/A



Updates and Upgrades
====================

N/A

Dependencies
============

SSSD 1.10 is required. In particular, Python bindings to
libsss_nss_idmap are used, packaged as libsss_nss_idmap-python in Fedora
19.



External Impact
===============

N/A



Design page authors
===================

`Alexander Bokovoy <User:ab>`__