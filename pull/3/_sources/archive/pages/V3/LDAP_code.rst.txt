\__NOTOC_\_

Overview
========

Ticket `#2660 <https://fedorahosted.org/freeipa/ticket/2660>`__
installer code should use ldap2

This is important to do. We really should have just one API and set of
classes for dealing with LDAP. For the DN work we had to refactor a fair
amount of code in order to force most things to funnel through one
common code location. Because ldap is so decentralized and we had so
many different APIs, classes, etc it was a large chunk of work and is
only partially completed, it got a lot better but it wasn't finished.

The primary thing which needs to be resolved is our use of Entity and
Entry classes. There never should have been two almost identical
classes. One or both of Entity/Entry needs to be removed.

As it stands now we have two basic ways we access ldap results. In the
installer code it's mostly via Entity/Entry objects. But in the server
code (ldpa2) it's done by accessing the data as returned by the python
ldap module (e.g. list of (DN, attr_dict) tuples).

We need to decide which of the two basic interfaces we're going to use
and converge on it. Each approach has merits. But 3 different API's for
interacting with ldap is 2 too many.

.. _use_cases10d:

Use Cases
=========

N/A

Design
======

.. _entry_representation:

Entry representation
--------------------

LDAP entries will be encapsulated in objects. These will perform type
checking and validation (ticket #2357). They should grow from
ldap2.LDAPEntry (which is currently just a "dn, data" namedtuple).

These objects will behave like a dict of lists:

| `` entry[attrname] = [value]``
| `` attrname in entry``
| `` del entry[attrname]``
| :literal:` entry.keys(), .values(), .items()  # but NOT `for key in entry`, see below`

The keys are case-insensitive but case-preserving.

We'll use lists for all attributes, even single-valued ones, because
"single-valuedness" can change.

The object should also "rembember" its original set of attributes, so we
don't have to retrieve them from LDAP again when it's updated.

.. _the_connectionbackend_class:

The connection/backend class
----------------------------

We'll continue to use the ldap2 plugin for "normal" connections, and the
IPAdmin class for connections that need special authentication or happen
before the server is configured (or after is unconfigured). Most of the
code and API will be common, and inherited from a new
ipapython.ipaldap.LDAPConnection class. The common API will match the
current ldap2 API. The common code will move to ipapython, so that it is
also available to the client installer.

ldap2 has some overly specific "helper" methods like
remove_principal_key or modify_password. We shouldn't add new ones, and
the existing ones should be moved away eventually.

.. _backwards_compatibility_porting:

Backwards compatibility, porting
--------------------------------

For compatibility with existing plugins, the LDAPEntry object will
unpack to a tuple:

`` dn, entry_attrs = entry``

(the entry_attrs can be entry itself, so we keep the object's validation
powers.)

The legacy Entry interface (toTupleList, toDict, setValues, data,
origDataDict, ...) will be temporarily added to LDAPEntry. Code will be
moved away from using it.

The backwards compatibility stuff should be removed as soon as it's
unused.

Of course code using the raw python-ldap API will also be converted to
ldap2.

Implementation
==============

No additional requirements or changes discovered during the
implementation phase.

.. _feature_managment:

Feature Managment
=================

N/A



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

N/A

.. _design_page_authors:

Design page authors
===================

`Pviktorin <User:Pviktorin>`__, jdennis
