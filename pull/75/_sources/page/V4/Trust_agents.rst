Trust_agents
============

Overview
========

FreeIPA supports trusted relationships with Active Directory via
cross-forest trust. Currently all functionality to support trusted
relationships with Active Directory must be present on every IPA master
which controls IPA clients where access to AD users is desired. There is
certain difference between uses of the IPA infrastructure which allow to
reduce requirements towards IPA masters involved in providing trust
features.

A trust controller is a FreeIPA master which runs following services:

-  LDAP server with sigden, extdom, and cldap plugins
-  KDC with IPA driver
-  Samba configured with ipasam PASSDB module
-  SSSD with ipa_server_mode=True
-  Global Catalog instance (a separate LDAP instance with an
   AD-compatible schema)

A trust agent is a FreeIPA master which runs following services

-  LDAP server with sigden and extdom plugins
-  KDC with IPA driver
-  SSSD with ipa_server_mode=True

Trust agent is a master that relies on SSSD to do resolution of IDs.
Trust controller is used for managing trust: add trust agreements,
enable/disable separate domains from a trusted forest to access FreeIPA
resources, etc. Trust controller is also what Active Directory's domain
controllers contact when validating the trust by means of SMB protocol
using LSA calls which implies running a Samba server.



Use cases
=========

Two major use cases are:

-  IPA replicas are used to resolve AD users and groups but not used to
   administer trust (trust agents)
-  IPA replicas are used to establish trust and manipulate their
   properties

In case of trust agent, not running Samba suite reduces exposure for
possible security breaches. Additionally, such lightweight IPA replica
reduces possibility to negatively influence performance when trust is
validated as it will not be used by AD DCs to contact to validate trust.
AD DCs consider IPA replicas to contact based on DNS SRV records. If
lightweight IPA replica is not present in the corresponding MSDCS SRV
record, it will not be contacted and thus possible replication delays
will not cause validation issues. This is an important consideration
because currently FreeIPA does not have concept of sites and site-local
communication.



Design and Implementation
=========================

Primary ticket: https://fedorahosted.org/freeipa/ticket/4951 No major
code changes required, only defaults need to be changed on which plugins
are enabled and how they are used.

In order to support trust agents role, FreeIPA has changes
ipa-adtrust-install tool and updates to convert existing IPA masters
without ipa-adtrust-install into the ones that enable extdom and sidgen
plugins if any of ipa-adtrust-install artefacts are in the replicated
tree.

Running **ipa-adtrust-install --add-agents** at the IPA master which
handles trust setup is required to convert specific IPA masters. There
is now no need to run ipa-adtrust-install utility on all IPA masters, it
should be run only on those masters where trust is supposed to be
established.

Additionally, RPM dependencies need to change to allow installing Samba
libraries which are in use by SSSD components and extdom/sidgen plugins.

Enablement of slapi-nis to provide AD users' access should depend on
whether compat tree is enabled at this IPA master already or not.



Feature management
==================

CLI

no changes



Web UI----

no changes

Replication
-----------

no changes

Upgrades
--------

Upgrade code automatically enables sidgen and extdom plugins.