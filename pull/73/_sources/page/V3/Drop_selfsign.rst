Drop_selfsign
=============

\__NOTOC_\_

Not to be confused with
`V3/Drop_selfsign_functionality <V3/Drop_selfsign_functionality>`__, a
more complex RFE to remove the selfsign functionality altogether.

Overview
========

Ticket `3534 <https://fedorahosted.org/freeipa/ticket/3534>`__ Remove
the --selfsign option

In a future, we would like to support 2 flavors of certificate
management in IPA:

-  IPA with pki-ca (dogtag) with either a self-signed certificate or
   with a certificate signed by external CA (--external-ca option)
-  IPA with no pki-ca installed with certificates signed and provided by
   an external CA.

Installation with --selfsign (selfsigned certificate managed in local
NSS database on server) is rather troublesome and not even supported -
it should be dropped.



Use Cases
=========

#. User tries passing the --selfsign option to ipa-server-install.
#. The install fails as there is no such option.

#. User upgrades a server that uses the self-signed CA
#. The CA continues to work normally

Design
======

The --selfsign option to ipa-server-install will be removed.

Existing self-signed CAs should continue working for now, but the
functionality is untested, and may be removed entirely in the near
future.

Implementation
==============

No additional requirements or changes discovered during the
implementation phase.



Feature Managment
=================

N/A



Major configuration options and enablement
==========================================

N/A

Replication
===========

No impact, self-signed CAs are incapable of replication



Updates and Upgrades
====================

Self-signed CAs should continue to work after upgrading to the new
version. As before, they are neither tested nor supported.

Dependencies
============

N/A



External Impact
===============

QE will need to drop tests for the self-signed CA, if they have any.

Documentation may need updating.



RFE Author
==========

`pviktori <User:pviktorin>`__