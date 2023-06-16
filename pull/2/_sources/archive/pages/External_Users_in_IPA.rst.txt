Overview
========

`External Collaboration Domains <External_Collaboration_Domains>`__
defines a use case where organizations host resources in a public-facing
network space in support of cross-organization collaboration. A key
requirement of that use case is that users be capable of logging into
machines (i.e., via ssh), accessing file shares, and participating in
POSIX groups for authorization and file permission purposes. This
request proposes IPA support this use case via the ability to synthesize
and harmonize POSIX attributes for external users without creating a new
account in the collaboration domain.

There are two objectives to this enhancement:

#. Ensure all users have the required POSIX attributes, that the POSIX
   attributes are unique, and that services in the domain have access to
   these attributes.
#. Ensure that locally defined POSIX groups can add individual external
   users.

IPA uses Kerberos for authentication and LDAP to store user attributes.
Users created within IPA are fully functional: they may log in to
websites, perform console logins, and participate in group membership.
External users, defined here as identities not allocated within the
local IPA server, are not fully functional. In general, they may not
participate in POSIX groups because they have no DN defined within the
local LDAP server.

To address "participation in local POSIX groups", this RFE proposes
creating and maintaining individual entries in the local LDAP server for
every external user. This process should not create another password for
the user to manage

Further, it is proposed that IPA centralize the functions of importing,
harmonizing, and synthesizing POSIX attributes for the local domain. The
explicit intent is that all services and hosts in the local realm obtain
POSIX attributes from the local IPA.

These two actions are envisioned to happen automatically upon first
encounter with an external user.

.. _use_cases14:

Use Cases
=========

See `External Collaboration Domains <External_Collaboration_Domains>`__
for the context driving this RFE.

Design
======

.. _overview_1:

Overview
--------

The crux of this proposal is the provisioning and storage of user
attributes in LDAP for external users, while *not* creating a
corresponding identity in the local Kerberos store. Automatic
provisioning of user attributes on first access is desirable.
Persistence of these attributes from login to login is required. It is
also desirable to map and synchronize attributes from the user's home
realm when they: a] exist in the user's home realm; and b] have not been
overridden by the local IPA to prevent collisions.

Currently, ``sssd`` on hosts within the realm may be configured to
recognize multiple domains. The intent is to allow each machine to
authenticate directly against the user's home domain and receive user
attribute information directly from the source. Each machine must be
individually configured, the configuration must explicitly whitelist
every permitted domain from which users may connect, and each machine's
configuration must be individually maintained. ``sssd`` is capable of
deriving POSIX-specific identity attributes from windows-specific ones,
but is not currently capable of synthesizing POSIX attributes (or
windows attributes) if they are completely absent. ``sssd``, in its
current state, is not a good solution for configuring an open,
collaborative environment which accepts identities using a wide range of
technologies and providers.

.. figure:: KrbUserAttributes.png
   :alt: KrbUserAttributes.png

   KrbUserAttributes.png

The above figure shows common configurations of realms which are
designed to provide console logins and which utilize Kerberos as the
authentication technology. Users have two identities with a mapping
between them. The LDAP store represents the user's identity as a
Distinguished Name (DN), which is essentially an absolute path to the
user's identity entry from the root of the directory. The Kerberos store
represents the user's identity as "name@realm". The mapping between the
two identities may be stored in the user's entry in the LDAP store using
the ``krbPrincipalName`` attribute. This attribute is defined in the
expired internet draft
`draft-skibbie-krb-kdc-ldap-schema-02 <http://tools.ietf.org/html/draft-skibbie-krb-kdc-ldap-schema-02>`__.
The relationship between the two identities may also be embodied in the
way configuration of hosts is managed in the source realm.

.. _provisioning_external_users_in_ldap:

Provisioning External Users in LDAP
-----------------------------------

For a stand-alone domain which allows no cross-realm operation, there is
a 1:1 relationship between the users defined in the LDAP store and the
users defined in Kerberos. For reliable and robust cross realm operation
which is centrally managed for the local realm, it is necessary to add
an external entry to LDAP without adding the corresponding entry to the
local KDC. Two identities are still in play: the user still has a
Kerberos identity, it's just not a *local* identity. The mapping still
needs to be maintained, and should be made explicit using the
``krbPrincipalName`` LDAP attribute.

.. figure:: KrbExtUserAttributes.png
   :alt: KrbExtUserAttributes.png

   KrbExtUserAttributes.png

Due to the fact that IPA would require substantial work to allocate
external users in a separate OU, no such provision is specified in this
RFE. Directory entries for external users will be allocated in the same
OU as entries for users created by the IPA realm. This feature, however,
does impose the constraint that directory entries for external users
have a globally unique RDN. In other words, the RDN must include both
the ``name`` and ``realm`` part of the external user's kerberos
identity. Failure to do so would result in collisions between principals
having the same name, but from two different realms.

It is necessary that hosts and services within the realm be readily
configured to understand the mapping between LDAP and Kerberos user
identities.

.. _attributes_from_the_realm_of_origin:

Attributes from the realm of origin
-----------------------------------

The focus of this design is to consolidate the formerly distributed sssd
configuration to provide equivalent functionality by pointing all hosts
and services in the local realm at the IPA server. While the Kerberos
server orchestrates which external principals are permitted to
authenticate to services within the local realm, the LDAP server should
provide the local realm's clients access to as much information about
those Kerberos principals as it can. Administrators should be able to
configure the hostname of remote LDAP servers as well as a mapping from
the remote schema to the IPA schema.

User attributes should be classified as being under local control or
remote control. IPA takes local control of an attribute for all
identities connecting to the local realm when there is a potential for
conflict. Candidates for local control are the POSIX UID, and GID. The
Windows SID is also a candidate for local control, particularly when the
realm of origin does not provide such a value (i.e., is a POSIX domain
or web-oriented IdP.) IPA may also take local control over an attribute
if the value defined in the realm of origin is irrelevant to the local
realm: such as ``homeDirectory``.

Group information from the realm(s) of origin are potentially
interesting. However, it is more likely that new groups relevant to
specific collaborative partnerships will be more interesting. These
groups may be composed within the local IPA realm using identities from
any available source without regard to whether the contributing identity
sources have a direct relationship with one another. Harvesting group
information from donor realms is explicitly *not* a goal of this RFE.

LDAP requests for information about external identities may be filled by
relaying the request to the realm of origin (if configured) and cached
in the local LDAP server. Periodically, the cache should be refreshed in
order to pick up changes. This can be configured to happen on a "just in
time" basis (i.e., when a new request for old cached data is received),
or it can be configured to happen on a schedule (nightly or weekly).
Just as group information from donor realms is not obviously critical,
synchronizing all identities from every potential donor realm is
unlikely to serve a useful purpose. The only identities important to the
local realm are those identities which access services within the realm.

.. _external_user_participation_in_posix_groups:

External User participation in POSIX groups
-------------------------------------------

Automatically provisioning external users in LDAP provides the local DN
required to directly add them to POSIX groups. In the case of AD trusts,
it will no longer be necessary to manually mirror AD groups to IPA and
then add them as a member to POSIX groups. However, while this method
offers administrators another method of handling users from a tightly
coordinated AD domain, it does not conflict with the current way of
integrating AD users into an IPA domain.

The limitation on this approach is that the external user must login to
the domain once (to create their user attribute entry in the LDAP
server) prior to being added to a group.

Implementation
==============

Any additional requirements or changes discovered during the
implementation phase.

**TBD**



Feature Management
==================

UI
~~

How the feature will be manged via the UI

**TBD**

CLI
~~~

Overview of the CLI commands

**TBD**



Major configuration options and enablement
==========================================

-  Globally enable or disable cross-realm operation.
-  Allow admin to provide a set of default values for automatically
   provisioned external users.
-  Allow admin to configure the identity server in the realm of origin
   when known. This should include both the hostname and a mapping from
   the original schema to IPA's schema. It should be possible to
   save/load mappings to a file to share them with other admins.
-  Allow admin to select an allocation strategy for POSIX IDs within the
   realm (one pool of external IDs for the realm? one pool per donor
   realm?)
-  Allow control over which external Kerberos realms may provide
   identities for use in this realm. This may consist of a whitelist,
   blacklist, or both.

Replication
===========

New configuration must be replicated.

"external users" exist only in LDAP and should leverage the existing
replication mechanism.



Updates and Upgrades
====================

When users upgrade to IPA with this feature, the default configuration
should be to have cross-realm operation turned off.

Dependencies
============

no change



External Impact
===============

This work should make it possible for Ipsilon to adapt external users
into a form that the local IPA realm can transparently use.

The production of certificates by FreeIPA is not currently compatible
with PKINIT usage. FreeIPA, DogTag, and nss need to be modified to
provide this support (`Ticket
#521 <https://fedorahosted.org/freeipa/ticket/521>`__). This
functionality is required for the OTP effort. This RFE describes the
complementary operation which could consume kx509 tickets, should they
become available. As such, implementation of this RFE need not depend on
completion of kx509 certificate support.



Backup and Restore
==================

If the extra configuration creates new configuration files, these will
need to be included in a backup/restore scheme.

External user attribute entries in LDAP should be covered by existing
backup/restore strategies.



Test Plan
=========

-  Verify the KDB plugin correctly audits PKINIT transactions from the
   AS.
-  Verify the KDB plugin correctly audits requests using cross realm
   TGTs in the TGS.
-  Verify that IPA can automatically provision an external user account
   given no information other than ``cname`` and ``crealm``. This should
   respect the default values provided by the admin.
-  Verify that IPA's attribute mapper can adapt remote user attribute
   schemas to the local IPA schema.
-  Verify that upstream changes to a user attribute in the realm of
   origin are reflected in the local realm.



RFE Author
==========

bnordgren@fs.fed.us
