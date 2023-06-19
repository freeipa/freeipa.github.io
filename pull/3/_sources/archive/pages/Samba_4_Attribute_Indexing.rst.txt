Overview
========

Samba relies on the LDAP backend to do attribute indexing. Currently the
provisioning tool can already configure the indexing on OpenLDAP, but it
still needs to be modified to configure the indexing on DS.

.. _attribute_indexing:

Attribute Indexing
==================

AD schema uses the fATTINDEX bit in the searchFlags attribute of an
attribute type to indicate whether the attribute will be indexed. For
example:

::

   cn: Alt-Security-Identities
   searchFlags: fATTINDEX

Finding the attributes that need to be indexed can be done by searching
the schema subtree using the following filter:

::

   (&(objectclass=attributeSchema)(searchFlags:1.2.840.113556.1.4.803:=1))

There are 114 attributes that need indexing in AD schema.

.. _current_code:

Current Code
============

.. _openldap_configuration:

OpenLDAP Configuration
----------------------

Indexing an attribute in OpenLDAP can be done by specifying the
following directive in slapd.conf:

::

   index ${ATTR} eq

.. _provisioning_tool:

Provisioning Tool
-----------------

The provision_openldap_backend() uses the following code to configure
attribute indexing in OpenLDAP:

::

   index_config = ""

   // get indexed attributes
   attrs = ["linkID", "lDAPDisplayName"]
   res = schema.ldb.search(
       expression="(&(objectclass=attributeSchema)(searchFlags:1.2.840.113556.1.4.803:=1))",
       base=names.schemadn,
       scope=SCOPE_ONELEVEL,
       attrs=attrs)

   // for each indexed attribute
   for i in range (0, len(res)):
       index_attr = res[i]["lDAPDisplayName"][0]

       // map objectGUID to entryUUID
       if index_attr == "objectGUID":
           index_attr = "entryUUID"

       // generate indexing configuration
       index_config += "index " + index_attr + " eq\n"

.. _default_indexes:

Default Indexes
---------------

The following attributes are indexed by default in DS:

-  aci
-  cn
-  entrydn
-  entryusn
-  givenName
-  mail
-  mailAlternateAddress
-  mailHost
-  member
-  memberOf
-  nsUniqueId
-  ntUniqueId
-  ntUserDomainId
-  numsubordinates
-  objectclass
-  owner
-  parentid
-  seeAlso
-  sn
-  telephoneNumber
-  uid
-  uniquemember

All except aci and numsubordinates have an equality index.

.. _linked_attributes:

Linked Attributes
-----------------

The following attributes are linked, so they need to have an equality
index. See also `this page <Obsolete:Samba_4_Attribute_Linking>`__.

-  bridgeheadTransportList
-  frsComputerReference
-  fRSMemberReference
-  hasMasterNCs
-  hasPartialReplicaNCs
-  managedBy
-  manager
-  member
-  msCOM-PartitionLink
-  msCOM-UserPartitionSetLink
-  msDFSR-ComputerReference
-  msDFSR-MemberReference
-  msDS-AuthenticatedAtDC
-  msDS-HasDomainNCs
-  msDS-hasFullReplicaNCs
-  msDS-hasMasterNCs
-  msDS-KrbTgtLink
-  msDS-MembersForAzRole
-  msDS-NC-RO-Replica-Locations
-  msDS-NonMembers
-  msDS-ObjectReference
-  msDS-OperationsForAzRole
-  msDS-OperationsForAzTask
-  msDS-PSOAppliesTo
-  msDS-TasksForAzRole
-  msDS-TasksForAzTask
-  msSFU30PosixMember
-  netbootServer
-  nonSecurityMember
-  owner
-  privilegeHolder
-  queryPolicyObject
-  serverReference
-  siteObject

The member and owner are already defined in the default indexes and have
an equality index.

.. _proposed_changes:

Proposed Changes
================

.. _ds_configuration:

DS Configuration
----------------

Indexing an attribute in DS can be done by adding the following
configuration entry:

::

   dn: cn=${ATTR},cn=default indexes,cn=config,cn=ldbm database,cn=plugins,cn=config
   objectClass: top
   objectClass: nsIndex
   cn: ${ATTR}
   nsSystemIndex: false
   nsIndexType: eq

This template should be stored in source4/setup/fedorads-index.ldif.

.. _provisioning_tool_1:

Provisioning Tool
-----------------

The provision_fds_backend() should use the following code to configure
attribute indexing in DS. First it will configure the indexes for all
linked attributes, then it will configure the indexes for all indexed
attributes as defined in AD schema. The code might generate duplicate
indexes, but they will be ignored during instance creation.

::

   index_config = ""

   // get linked attributes
   lnkattr = get_linked_attributes(names.schemadn,schema.ldb)

   // for each linked attribute
   for attr in lnkattr.keys():

       // generate indexing configuration
       index_config += read_and_sub_file(
           setup_path("fedorads-index.ldif"),
           { "ATTR" : attr })

   // get indexed attributes
   attrs = ["linkID", "lDAPDisplayName"]
   res = schema.ldb.search(
       expression="(&(objectclass=attributeSchema)(searchFlags:1.2.840.113556.1.4.803:=1))",
       base=names.schemadn,
       scope=SCOPE_ONELEVEL,
       attrs=attrs)

   // for each indexed attribute
   for i in range (0, len(res)):

       attr = res[i]["lDAPDisplayName"][0]

       // map objectGUID to nsUniqueId
       if attr == "objectGUID":
           attr = "nsUniqueId"

       // generate indexing configuration
       index_config += read_and_sub_file(
           setup_path("fedorads-index.ldif"),
           { "ATTR" : attr })

.. _samba_patches:

Samba Patches
=============

-  `s4:provision - replaced linked_attributes with FDS
   plugins <http://gitweb.samba.org/?p=samba.git;a=commit;h=cf77bf338260e33e7353f1176210d5cac5a6048d>`__

References
==========

-  `Search-Flags
   Attribute <http://msdn.microsoft.com/en-us/library/ms679765%28VS.85%29.aspx>`__
-  `Attribute
   searchFlags <http://msdn.microsoft.com/en-us/library/cc220851%28PROT.13%29.aspx>`__
-  `How to query Active Directory by using a bitwise
   filter <http://support.microsoft.com/kb/269181>`__

`Category:Obsolete <Category:Obsolete>`__
