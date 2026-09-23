Trust_GC_support
================

\__NOTOC_\_

Overview
========

Global Catalog service support for AD trusts

Primary ticket: https://fedorahosted.org/freeipa/ticket/3125



Use Cases
=========

-  Active Directory administrators want to share resources with IPA
   users

   -  Setting up access rights in Security tab of a properties dialog on
      Windows machine
   -  Log-on to Windows Machines using IPA user credentials

Design
======

Global Catalog service is a read-only view of the users and groups in an
Active Directory implementation. It is exposed as an LDAP server on a
separate non-standard port 3268 and contains full information about the
domain the controller manages and partial read-only information of the
other domains from the same forest. The information includes schema and
user/group information for the respective domains.

Details on how Global Catalog service works in Microsoft's Active
Directory implementation can be found here:
http://technet.microsoft.com/en-us/library/how-global-catalog-servers-work%28v=ws.10%29.aspx

We differentiate below Global Catalog service as implemented in
Microsoft's Active Directory and Global Catalog service in FreeIPA by
talking about \*GC service intance\* in the latter case.

FreeIPA implementation of GC service will use following approach:

-  a separate Directory Server instance will be created to contain
   read-only memory-only copy of the data about IPA's domain

   -  Directory Server cannot listen on multiple ports for the same
      protocol (389 vs 3268)

-  GC service instance will use a different schema than original FreeIPA
   schema since GC schema is incompatible with IPA schema.
-  GC service instance will use modified slapi-nis plugin to search an
   original IPA directory server instance for entries instead of an
   internal search

   -  Updates from the original IPA directory server instance will be
      tracked with the help of SYNCREPL protocol

-  slapi-nis plugin will be configured to present entries from the
   original IPA directory server instance as GC service instance entries
-  additional plugins will be created to handle schema correctness
   enforcement as multiple inheritance (auxiliary classes) and tree
   structure cannot be expressed through 389-ds schema syntax

Implementation
==============



GC schema mapping
-----------------

Global Catalog instance schema is derived from a schema published by
Microsoft through WSPP programme as MS-ADSC document,
http://msdn.microsoft.com/en-us/library/cc221630.aspx. Schema files are
available in LDIF format from
http://go.microsoft.com/fwlink/?LinkId=212555 and processed with the
help of a converting script. The script to convert schema to 389-ds
format is based on a similar script from Samba (ms_schema.py) which only
supports non-validating output for LDB database. FreeIPA's version has
to generate valid schema in 389-ds format and thus adds mapping between
schema attribute definitions existing in 389-ds and MS-ADSC. In
particular, attribute types, their ordering and matching functions
mapped to those of 389-ds.

.. table:: Equality rules mapping

   ======================= ======================
   Original syntax         Mapped syntax
   ======================= ======================
   2.5.5.8                 booleanMatch
   2.5.5.9                 integerMatch
   2.5.5.16                integerMatch
   2.5.5.14                distinguishedNameMatch
   1.3.12.2.1011.28.0.702  octetStringMatch
   1.2.840.113556.1.1.1.12 distinguishedNameMatch
   2.5.5.7                 octetStringMatch
   2.6.6.1.2.5.11.29       octetStringMatch
   1.2.840.113556.1.1.1.11 octetStringMatch
   2.5.5.13                caseIgnoreMatch
   2.5.5.10                octetStringMatch
   2.5.5.3                 caseIA5Match
   2.5.5.5                 caseIA5Match
   2.5.5.15                octetStringMatch
   2.5.5.6                 numericStringMatch
   2.5.5.2                 objectIdentifierMatch
   2.5.5.10                octetStringMatch
   2.5.5.17                caseExactMatch
   2.5.5.4                 caseIgnoreMatch
   2.5.5.12                caseIgnoreMatch
   2.5.5.11                generalizedTimeMatch
   ======================= ======================

.. table:: Attribute syntax mapping

   ======================= =============================
   Original syntax         Mapped syntax
   ======================= =============================
   2.5.5.8                 1.3.6.1.4.1.1466.115.121.1.7
   2.5.5.9                 1.3.6.1.4.1.1466.115.121.1.27
   2.5.5.16                1.3.6.1.4.1.1466.115.121.1.27
   2.5.5.14                1.3.6.1.4.1.1466.115.121.1.12
   1.3.12.2.1011.28.0.702  1.3.6.1.4.1.1466.115.121.1.5
   1.2.840.113556.1.1.1.12 1.3.6.1.4.1.1466.115.121.1.12
   2.5.5.7                 1.3.6.1.4.1.1466.115.121.1.12
   2.6.6.1.2.5.11.29       1.3.6.1.4.1.1466.115.121.1.12
   1.2.840.113556.1.1.1.11 1.3.6.1.4.1.1466.115.121.1.12
   2.5.5.13                1.3.6.1.4.1.1466.115.121.1.43
   1.3.12.2.1011.28.0.732  1.3.6.1.4.1.1466.115.121.1.43
   2.5.5.10                1.3.6.1.4.1.1466.115.121.1.5
   1.2.840.11.3556.1.1.1.6 1.3.6.1.4.1.1466.115.121.1.5
   ======================= =============================



Auxiliary classes
-----------------

Active Directory schema supports multiple inheritance through use of
auxiliaryClass and systemAuxiliaryClass attributes. 389-ds does not
support mechanism to specify multiple superior classes in the schema. In
result, we need to explicitly add these classes to the objects of a
specific objectClass type on creation.



Tree structure correctness
--------------------------

Active Directory schema describes types of objects that may contain the
object of a type through systemPossSuperiors and possSuperiors
attributes. 389-ds does not support this type enforcement. In result, we
need to explicitly check these requirements on object creation.



Feature Management
==================

UI

How the feature will be manged via the UI

CLI

Overview of the CLI commands



Major configuration options and enablement
==========================================

Any configuration options? Any commands to enable/disable the feature or
turn on/off its parts?

Replication
===========

Any impact on replication?



Updates and Upgrades
====================

Any impact on updates and upgrades?

Dependencies
============

Any new package and library dependencies.



External Impact
===============

Impact on other development teams and components



Backup and Restore
==================

Any files or configuration that needs to be taken care of in backup or
restore procedure.



Test Plan
=========

Test scenarios that will be transformed to test cases for FreeIPA
Continuous Integration during implementation or review phase.



RFE Author
==========

`Alexander Bokovoy <User:Ab>`__