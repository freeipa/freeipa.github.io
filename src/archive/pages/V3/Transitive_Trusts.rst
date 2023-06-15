\__NOTOC_\_

Overview
========

FreeIPA v3 supports cross-forest trusts to Active Directory domains. By
their nature, cross-forest trusts are established at the forest root
level. Each forest root domain's domain controller is responsible to
expose subordinate domains to the trusting party.

FreeIPA v3 currently does not support fetching transitive trusts
information and providing its own view of a FreeIPA forest.



Use Cases
=========

Following use cases considered:

-  Active Directory forest with a resource domain and user domain where
   the user domain is not the forest root, and users in AD need to
   access resources in FreeIPA forest
-  Active Directory forest with a resource domain and user domain where
   the resource domain is not the forest root, and users in FreeIPA need
   to access resources in AD forest

Design
======

Main tracking ticket: https://fedorahosted.org/freeipa/ticket/3510

Transitive trusts enquiry requires that information about other domains
is available through the Global Catalog service in the forest root
domain. A trusting party can query the GC to perform SID-to-name and
name-to-SID resolution operations. Additionally, FreeIPA will need to
retrieve transitive trust information from the AD side.

The following should be implemented:

-  A GC service on at least one FreeIPA master, exposing an AD
   compatible schema and the necessary set of information derived from
   the main LDAP instance
-  Discovery of all the trusted domains within the trusted AD forest
-  CLI and Web UI to manage filtering of trusted domain's trusts and
   refreshing list of domains

Implementation
==============

.. _global_catalog_service:

Global Catalog service
----------------------

The Global Catalog service will be implemented through a separate
directory server (389-ds) instance. In this instance, a separate
AD-compatible schema will be used and a read-only view of the original
data will be given according to an AD-compatible DIT.

A special plugin will be used to connect to the original FreeIPA
directory server instance to notice any changes and propagate them to
the GC instance. The plugin will rely on the forthcoming SyncRepl
feature of 389-ds (ticket `"RFE Support 'Content Synchronization
Operation' (SyncRepl) - RFC
4533" <https://fedorahosted.org/389/ticket/47388>`__)

A specialized tool to bootstrap the GC instance based on an
AD-compatible schema and an existing FreeIPA directory service instance
needs to be developed for testing and development purposes. This tool
will use the existing 'ipaserver.install.dsinstance' code and configure
the instance based on existing parameters for the original instance.

.. _discovery_of_the_trusted_domains_partnerships:

Discovery of the trusted domain's partnerships
----------------------------------------------

FreeIPA framework will need to fetch list of the domains in the trusted
forest and their trusted domains, and configure both domain and range
objects for these additional domains. This is achieved by the use of
netr_DsrEnumerateDomainTrusts DCE RPC call.

All imported domains need to have SID filtration set up for them
according to the rules described in `this MSDN
article <http://technet.microsoft.com/en-us/library/cc755673%28v=ws.10%29.aspx>`__

.. _kerberos_dal_driver_modifications:

Kerberos DAL driver modifications
---------------------------------

Kerberos DAL driver will need to be updated to understand transitive
trusts. It includes understanding that a ticket from a child domain
should be usable with a cross-realm TGT belonging to the forest root
domain. This is supposedly done by implementing check_transited_realms()
method in the driver (though little examples exist for this method).

Since forest-level trusts with Active Directory can only be established
with root level domain of the foreign forest, cross-realm TGT will
always be issued by the root level domain's DC. This ticket is already
trusted by FreeIPA Kerberos DAL driver.

However, when accessing services in the non-root level domains in the
trusted forest, a request for a ticket for those services should be
forwarded to the DC of the root domain. We need to ensure proper
realm/domain and capaths mappings are in place.

When non-root level domains in the trusted forest are hierarchical to
the root domain, MIT Kerberos KDC already knows how to transit trust
paths to then. When there are non-heirarchical domains in the trusted
forest, [capaths] should be set up on all IPA clients to establish
proper transit trust path towards non-root level domains in the trusted
forest.

In SSSD following tickets were opened to track these changes:
https://fedorahosted.org/sssd/ticket/2080 and
https://fedorahosted.org/sssd/ticket/2093.

.. _realm_and_domain_mapping_for_forest_domains:

Realm and domain mapping for forest domains
-------------------------------------------

When trust to an AD forest is established, we trust explicitly root
level domain in the forest. For all other domains in the forest trust
path should go through the root level domain. However, FreeIPA needs to
know about the domains first.

-  List of the domains belonging to a forest needs to be fetched upon
   establishing the trust
-  List of the domains belonging to the trusted domain may be refreshed
   after trust is established. The update process should be triggered by
   an admin manually since adding new domains is rare
-  Information about domains from the trusted forest needs to be stored
   in FreeIPA LDAP storage
-  For each domain from the trusted forest there should be identity
   range created in order to allow user and group enumeration for the
   domain
-  FreeIPA PDB module to Samba should be able to retrieve the list of
   the domains and provide it to other Samba components similar to how
   we provide list of our own additional DNS domains
-  SSSD should be able to fetch the list of the domains and expose it to
   IPA clients for proper configuration of the Kerberos applications via
   domain/realm mapping and auth_to_local regular expressions in
   krb5.conf. In MIT Kerberos 1.12 there will be new API added to allow
   bulk auth_to_local ruleset handling.



Feature Management
==================

UI
~~

TODO.

CLI
~~~

Since discovery of the domains belonging to the trusted forest can be
run by an administrator at will, new CLI command is required. **ipa
trust-domain** command family will contain the commands to operate on
the information about trusted forest domains.

ipa trust-fetch-domains
   Fetches the list of domains belonging to the forest trust. If there
   are already domains defined for the trust with the same name, no
   changes are done to them)
ipa trustdomain-find
   Displays list of domains associated with the trust. Root level domain
   is always included
ipa trustdomain-del
   Removes domain from the trust. As result, Kerberos principals
   belonging to the domain will not be allowed to access IPA resources.
   Root level domain cannot be removed through this command as it makes
   trust inactive.
ipa trustdomain-disable
   Adds SID of the domain from the list of filtered SIDs for this trust.
   SID filtering per trust is already implemented in **trust-mod**. SID
   filtering can be used to temporarily block access to IPA resources
   from the domain.
ipa trustdomain-enable
   Removes SID of the domain from the list of filtered SIDs for this
   trust. SID filtering per trust is already implemented in
   **trust-mod**.



Major configuration options and enablement
==========================================

TODO

Replication
===========

TODO



Updates and Upgrades
====================

TODO

Dependencies
============

SSSD will need to understand transitive trusted domains membership.



External Impact
===============

Part of the implementation relies on SSSD being able to understand
transitive trusted domain membership for incoming users



Backup and Restore
==================

TODO



Test Plan
=========

TODO



RFE Author
==========

Alexander Bokovoy
