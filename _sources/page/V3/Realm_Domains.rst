Realm_Domains
=============

\__NOTOC_\_

Overview
========

Related tickets:
`#2945 <https://fedorahosted.org/freeipa/ticket/2945>`__,
`#2848 <https://fedorahosted.org/freeipa/ticket/2848>`__,
`#3407 <https://fedorahosted.org/freeipa/ticket/3407>`__

We want to allow administrators to maintain a list of domains associated
with IPA Kerberos realm. This list will be stored in LDAP, under
**``cn=Realm``\ ````\ ``Domains,cn=ipa,cn=etc,$SUFFIX``**.

We need to expose an interface to display and modify this list via IPA
commands:

-  **``realmdomains-show``**
-  **``realmdomains-mod``**



Use Cases
=========

-  IPA administrator can display/modify the list of domains associated
   with IPA realm
-  ``ipa dnszone-add`` command can be hooked to ``realmdomains-mod``, to
   automatically add domain to the list of domains associated with IPA
   realm if this is not a reverse domain and not a pure forwarder
-  Trust code can use this list to expose to trusted parties

Design
======

Update LDAP schema to add the **``Realm``\ ````\ ``Domains``**
container. The default value for **``associatedDomain``** attribute will
be the DNS domain of the IPA server:

| ``dn: cn=Realm Domains,cn=ipa,cn=etc,$SUFFIX``
| ``default:objectClass: domainRelatedObject``
| ``default:objectClass: nsContainer``
| ``default:objectClass: top``
| ``default:cn: Realm Domains``
| ``default:associatedDomain: $DOMAIN``

Add two new IPA commands:

-  **``realmdomains-show``**, to display the current list of realm
   domains
-  **``realmdomains-mod``**, to modify the list



Feature Managment
=================

UI

A new page needs to be added to UI. This page will be able to handle all
mentioned operations with realm domains: display the current list, add a
new domain, remove an existing domain. The new page will be added under
'Identity' section.

CLI



realmdomains-show
^^^^^^^^^^^^^^^^^

**``realmdomains-show``** will display the current list of realm
domains, stored in
**``cn=Realm``\ ````\ ``Domains,cn=ipa,cn=etc,$SUFFIX``**.



realmdomains-mod
^^^^^^^^^^^^^^^^

**``realmdomains-mod``** will modify the list of realm domains.
Modifications can be performed in several ways:

-  To replace the list of realm domains with a new list (or a single
   value):

| **``realmdomains-mod``\ ````\ ``--domain=ourdomain.com``**
| **``realmdomains-mod``\ ````\ ``--domain={ourdomain.com,domain2.com,domain3.com}``**

-  To add a domain to the list:

**``realmdomains-mod``\ ````\ ``--add-domain=newdomain.com``**

-  To delete a domain from the list:

**``realmdomains-mod``\ ````\ ``--del-domain=olddomain.com``**

It will be possible to use either the ``--domain`` option, or a
combination of ``--add-domain`` and ``--del-domain``.

The following checks will be performed:

-  Check that we are not deleting the DNS domain of the IPA server from
   the list
-  Check that domain is a valid DNS domain name
-  Check that domain is accessible through DNS (provide --force option
   to skip this check)

Questions
---------



dnszone-\*
^^^^^^^^^^

-  Should **``dnszone-del``** delete **``associatedDomain``** when whole
   DNS zone is being deleted?
-  Should **``dnszone-add``** offer an option to create
   **``associatedDomain``** attribute for the new zone?

`pspacek <User:Pspacek>`__ 03:26, 7 February 2013 (EST)

Update regarding DNS integration: DNS <-> realmdomains integration has
been implemented. Details are covered in `this design
page <http://www.freeipa.org/page/V3/DNS_realmdomains_integration>`__.

`akrivoka <User:Akrivoka>`__ 05:05, 7 May 2013 (EDT)



Major configuration options and enablement
==========================================

N/A

Replication
===========

N/A



Updates and Upgrades
====================

Container **``cn=Realm``\ ````\ ``Domains,cn=ipa,cn=etc,$SUFFIX``**
needs to be created in LDAP. This will be achieved by adding a new
update file **``install/updates/40-realm_domains.update``**, with the
following contents:

``# Add the Realm Domains container``

| ``dn: cn=Realm Domains,cn=ipa,cn=etc,$SUFFIX``
| ``default:objectClass: domainRelatedObject``
| ``default:objectClass: nsContainer``
| ``default:objectClass: top``
| ``default:cn: Realm Domains``
| ``default:associatedDomain: $DOMAIN``

and referencing this file in ``install/updates/Makefile.am``.

A reference to this container will also be added to the
``DEFAULT_CONFIG`` variable in ``ipalib/constants.py``

Dependencies
============

N/A



External Impact
===============

N/A



Design page authors
===================

`akrivoka <User:Akrivoka>`__ 12:15, 6 February 2013 (EST)