Overview
========

Samba currently uses several methods to maintain links between objects
in the directory:

-  linked_attributes LDB module for TDB and DS backend
-  memberof and refint overlay for OpenLDAP backend

DS provides Linked Attributes and Referential Integrity plugins which
can be used to achieve the same goal. Samba provisioning tool should be
modified to utilize these plugins.

.. _attribute_linking:

Attribute Linking
=================

Attribute linking is defined using the linkID attribute in Active
Directory schema. An attribute with linkID n is linked to another
attribute with linkID n+1. See the following example:

::

   cn: Member
   ldapDisplayName: member
   linkID: 2

   cn: Is-Member-Of-DL
   ldapDisplayName: memberOf
   linkID: 3

In this example the member attribute is linked to the memberOf
attribute.

Suppose in the DIT there's already an entry as follows:

::

   dn: CN=Administrator,CN=Users,DC=example,DC=com

Suppose a new entry is added with a link pointing to the above entry:

::

   dn: CN=Enterprise Admins,CN=Users,DC=example,DC=com
   member: CN=Administrator,CN=Users,DC=example,DC=com

A new attribute will be added to the target entry pointing back to the
new entry:

::

   dn: CN=Administrator,CN=Users,DC=example,DC=com
   memberOf: CN=Enterprise Admins,CN=Users,DC=example,DC=com

.. _current_code:

Current Code
============

.. _openldap_configuration:

OpenLDAP Configuration
----------------------

The template for the memberof overlay configuration is stored in
source4/setup/memberof.conf:

::

   overlay memberof
   memberof-dn cn=samba-admin,cn=samba
   memberof-dangling error
   memberof-refint TRUE
   memberof-group-oc top
   memberof-member-ad ${MEMBER_ATTR}
   memberof-memberof-ad ${MEMBEROF_ATTR}
   memberof-dangling-error 32

The template for the refint overlay configuration is stored in
source4/setup/refint.conf:

::

   overlay refint
   refint_modifiersName cn=samba-admin,cn=samba
   refint_attributes ${LINK_ATTRS}

.. _provisioning_tool:

Provisioning Tool
-----------------

The provision_openldap_backend() uses the following code to configure
attribute linking in OpenLDAP:

::

   lnkattr = get_linked_attributes(names.schemadn,schema.ldb)

   refint_attributes = ""
   memberof_config = ""

   for att in  lnkattr.keys():

       refint_attributes = refint_attributes + " " + att 

       memberof_config += read_and_sub_file(setup_path("memberof.conf"),
           { "MEMBER_ATTR" : att ,
             "MEMBEROF_ATTR" : lnkattr[att] })

.. _proposed_changes:

Proposed Changes
================

.. _ds_configuration:

DS Configuration
----------------

The Linked Attributes Plugin should be configured as follows:

::

   dn: cn=${MEMBER_ATTR} to ${MEMBEROF_ATTR},cn=Linked Attributes,cn=plugins,cn=config
   objectclass: extensibleObject
   cn: ${MEMBER_ATTR} to ${MEMBEROF_ATTR}
   linkType: ${MEMBER_ATTR}
   managedType: ${MEMBEROF_ATTR}

The Referential Integrity Plugin should be configured as follows:

::

   dn: cn=referential integrity postoperation,cn=plugins,cn=config
   nsslapd-pluginArg0: 0
   nsslapd-pluginArg1: %log_dir%/referint
   nsslapd-pluginArg2: 0
   nsslapd-pluginArg3: ${ATTR1}
   nsslapd-pluginArg4: ${ATTR2}
   nsslapd-pluginArg5: ${ATTR3}
   ...

The attributes must have an equality index. See also `this
page <Obsolete:Samba_4_Attribute_Indexing>`__.

.. _provisioning_tool_1:

Provisioning Tool
-----------------

The provision_fds_backend() should uses a similar code to configure
attribute linking in DS.

Issues
======

There is a minor issue in DS:

-  `Use proper attribute names for plugin
   parameters <https://bugzilla.redhat.com/show_bug.cgi?id=527500>`__

.. _samba_patches:

Samba Patches
=============

-  `s4:provision - replaced linked_attributes with FDS
   plugins <http://gitweb.samba.org/?p=samba.git;a=commit;h=cf77bf338260e33e7353f1176210d5cac5a6048d>`__

References
==========

-  `Linked Attributes
   Design <http://directory.fedoraproject.org/wiki/Linked_Attributes_Design>`__
-  `MemberOf
   Plugin <http://directory.fedoraproject.org/wiki/MemberOf_Plugin>`__
-  `Reverse Group Membership
   Maintenance <http://www.openldap.org/doc/admin24/overlays.html#Reverse%20Group%20Membership%20Maintenance>`__
-  `slapo-memberof(5) <http://linux.die.net/man/5/slapo-memberof>`__
-  `Maintaining Referential
   Integrity <http://www.redhat.com/docs/manuals/dir-server/8.1/admin/Creating_Directory_Entries-Maintaining_Referential_Integrity.html>`__
-  `The MemberOf Plug-in
   Syntax <http://www.redhat.com/docs/manuals/dir-server/8.1/admin/Advanced_Entry_Management-Using_Groups.html#memberof-syntax>`__
-  `Managing
   Indexes <http://www.redhat.com/docs/manuals/dir-server/8.1/admin/Managing_Indexes.html>`__
-  `Creating Standard
   Indexes <http://www.redhat.com/docs/manuals/dir-server/8.1/admin/Managing_Indexes-Creating_Indexes.html>`__

`Category:Obsolete <Category:Obsolete>`__
