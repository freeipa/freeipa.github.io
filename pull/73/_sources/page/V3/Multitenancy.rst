Multitenancy
============

Description
===========

Multi-tenancy is an aspect of Identity Management (IdM) where multiple
parties use the same resource without learn any information about each
other. The example is two rival companies who both operate servers
hosted in a public cloud. Neither company should be aware of the
existance of the other users presence in the web using, and they
definitely should not be able to enumerate either the users or the hosts
of the other company due to information leaks inside the cloud services.

The entities stored under each tenant have a relatively high likelihood
of reusing the same names. User names and group names are often have a
high number of names that are commonly used. For example, both companies
might have someone with the user name of asmith and a group named
developers. However, in the first company, asmith has a UID of 512 and
developers has a GID of 2100, but in the other company asmith has a UID
of 87687 and developers has a GID of 6332. Both need to exist in the IdM
server and be distinguishable from the other.



IPA Status
----------

IPA currently has a completely flat data model. All data is stored in
two instances: one for the IPA user and host database, and one for the
PKI CA. The Kerberos server would only serve out tickets for a single
domain. The Directory server has the vast majority of its data open read
only available to anonymous binds. The user Admin has full control over
the entire set of entites in the system.

FreeIPA stores the data it manages inside a Directory Server (389)
instance in a subtree that, by default, mirrors the domain name of the
server. Thus the server ipa.example.com would store in the subtree
cn=example,cn=com. The layout of this subtree will be referred to as the
current Directory Information Tree (DIT) Layout. The database suffix of
the subtree will be referred to as the current baseDN.

In addition, each IPA server manages a Kerberos Realm. By default the
name of the realm matches the domain name, but in all capitals. The zone
example.com would have a corresponding realm EXAMPLE.COM.



Required Changes
================

These are the changes necessary to the FreeIPA server to support a cloud
deployment and multi-tenancy.



Directory Server DIT Layout
---------------------------

Each tenant in the Identity Management system would be limited to a
subtree named after the tenant: cn={TENANT},cn=tenants,$suffix. This
subtree would have a DIT Layout similar to the main DIT Layout; it will
include all identity related subtrees necessary to create users, groups
and tenant specific policies (HBAC, SUDO etc...) . The initial
(undeletable) suffix is the Identity Infrastructure Owner and used for
the management of the tenants. The other subtrees will be referred to as
tenant trees.

When performing an action in the IPA server, the user is identified by
their principal, which contains the Kerberos Realm name. This can be
used to distinguish which domain, and by extension, which subtree, the
action should use for data. Operations are limited to the current tenant
subtree. Cloud administrators will need the ability to explicitly
specify a tenant in order to perform maintenance operations on the
tenants.

The directory will no longer be world readable. Instead, ACIs will limit
the users ability to read only the subtree in which they are enrolled.
LDAP operations will require an authenticated bind.

When updating FreeIPA, DIT layout changes changes need to be applied to
each of the the tenant trees. However the DIT should not and cannot
really change, as that would tend to break tenant configurations if you
do. Additions can be made but they should be very limited and optional.
Instead FreeIPA should allow tenants to opt-in in new features when that
involves touching their subtree rather then forcing it on them.



New API
-------

ipa run-as will be used by the administrative users to run commands for
a tenat. For example, to add a user to a tenant named "community":

ipa run-as community user-add gwashington --first George --last
Washington

The implementation should follow the techniques used by the batch
command in ipalib/plugins/batch.py.



Directory Server Plugins
------------------------

Some configuration changes will need to be made around a number of the
Directory Server plug-ins with regards to scope. We will likely need
separate configuration entries to restrict the plug-ins to each tenant
subtree. This includes the following plug-ins (and maybe others as
well):

#. memberOf
#. DNA
#. Managed Entries
#. Auto-Membership
#. Attribute Uniqueness



BaseLDAP plugin
---------------

Instead of just reading the BaseDN out of the configuration file, the
base DN needs to be calculated based on the Kerberos principal of the
calling user. This value should be cached in the WSGI application so
that it does not always require multiple round trips to the LDAP server.

Kerberos
--------

Each tenant would get its own Kerberos Realm, but each would trust the
Cloud Provider's realm. The httpd/mod_auth_kerberos needs to be capable
of accepting new realms. When adding a new tenant, we might need to
update krb5.conf on the IPA server to know about the new realm.

DNS
---

The BIND DynDB back end code would need to be extended to read zone
information from multiple subtrees in order to read the DNS entires in
the tenant trees.

It is also possible to keep the current approach, and then add the
ability to identify which users and tenants can manage and control DNS
entries. The simpler solution is to modify BIND Dyn DB.



Certificate Authority
---------------------

We would continue to have only one CA, and have IPA submit all
Certificate requests as a single Agent. The IPA server will ensure that
a user can only request certificates for entities within their own
subtree.



IPA-CLIENT
----------

When enrolling a host, the principal used to authenticate with the IPA
server will determine which subtree will hold host. This involves
changing the LDAP configuration information in the following client
components:

#. SSSD
#. nss_ldap
#. Kerberos.