Overview
--------

Global Catalog is a special service that exposes information about
objects in FreeIPA domain in a way expected by the clients enrolled to
Active Directory environments. Active Directory domain controllers and
other systems use Global Catalog to search across the potentially
multi-domain forest for the information about users, groups, machines,
and other types of objects.

A search in the Global Catalog service allows machines enrolled to an
Active Directory environment to efficiently maintain a relationship
between internal access control lists (ACLs) associated with the
resources provided by the machines, and the visual representation of the
access controls for interaction with users, locally and remotely.

The information stored in the Global Catalog represents a subset of the
information available at the specific domain controllers in Active
Directory.

A detailed overview of Global Catalog structure and design in Active
Directory can be found at the following
[https://msdn.microsoft.com/en-us/library/how-global-catalog-servers-work(v=ws.10).aspx
MSDN article].

Since no native FreeIPA clients are expected to use Global Catalog for
write purposes, it is only accessible in read-only mode. Any change to
the information shall come through the domain controllers responsible
for a domain in question.

.. _supported_naming_contexts:

Supported naming contexts
-------------------------

There are three naming contexts which exist in Active Directory: Schema
Naming Context, Configuration Naming Context, and Domain Naming Context.
The scope for Global Catalog service in FreeIPA is limited to Domain
Naming Context in initial implementation.

.. _supported_ports_and_protocols:

Supported ports and protocols
-----------------------------

FreeIPA Global Catalog service is only available over TCP port 3268 in
the first implementation phase. Retrieval and update of universal group
membership cache over TCP port 135 is not planned to be supported. The
latter is a feature of Active Directory used by its domain controllers
over DCE RPC protocol stack. Samba implements corresponding
functionality for user and group name lookups with the help of the
primary FreeIPA LDAP server instance.

.. _global_catalog_provisioning:

Global Catalog provisioning
---------------------------

The data in Global Catalog is provisioned from the primary LDAP server
instance running on the same FreeIPA master. A SYNCREPL mechanism is
used to retrieve the changes and a modified **slapi-nis** module is used
to transform FreeIPA original data to a schema compatible with Global
Catalog in Active Directory. Unlike the original **slapi-nis** module,
the data is stored in a proper LDAP backend so it is persistent across
the directory server restarts.

.. _gc_tree_structure:

GC tree structure
-----------------

In Active Directory Global Catalog contains directory data for all
domains in a forest. For an Active Directory forest with multiple forest
roots this means multiple domain suffixes exist in the Global Catalog
tree. For FreeIPA there is only a single domain that corresponds to the
forest tree and thus only a single domain suffix is hosted in the Global
Catalog.

Since FreeIPA is built around a flat DIT, there are no organizational
units available. As result, object-specific containers are children of
the forest domain suffix: e.g. for users a subtree of
**cn=users,dc=example,dc=com** is used.

.. _gc_schema_mapping:

GC schema mapping
-----------------

There are two separate mapping processes: first, a data from the primary
FreeIPA LDAP instance needs to be mapped to a schema expected by Active
Directory clients consuming Global Catalog service. However, the Global
Catalog schema itself requires transformation so that it could be used
in 389-ds directory server environment.

.. _freeipa_schema_to_gc_schema_mapping:

FreeIPA schema to GC schema mapping
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To perform data transformation from the primary FreeIPA LDAP instance to
GC instance, a plugin that consumes original LDAP objects from the
primary FreeIPA LDAP instance will be written. This plugin will use an
infrastructure to transform LDAP objects developed as part of the
**slapi-nis** plugin.

Schema compatibility plugin (**slapi-nis**) has a special language to
define transformations of the LDAP objects. A detailed description of
the formatting language is provided in the `slapi-nis
documentation <https://git.fedorahosted.org/cgit/slapi-nis.git/tree/doc/format-specifiers.txt>`__.

.. _equality_and_attribute_syntax_rules_mapping:

Equality and attribute syntax rules mapping
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Global Catalog instance schema is derived from a schema published by
Microsoft through WSPP program as MS-ADSC document,
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

.. _auxiliary_classes:

Auxiliary classes
-----------------

Active Directory schema supports multiple inheritance through use of
auxiliaryClass and systemAuxiliaryClass attributes. 389-ds does not
support mechanism to specify multiple superior classes in the schema. In
result, we need to explicitly add these classes to the objects of a
specific objectClass type on creation.

.. _tree_structure_correctness:

Tree structure correctness
--------------------------

Active Directory schema describes types of objects that may contain the
object of a type through systemPossSuperiors and possSuperiors
attributes. 389-ds does not support this type enforcement. In result, we
need to explicitly check these requirements on the object creation.
