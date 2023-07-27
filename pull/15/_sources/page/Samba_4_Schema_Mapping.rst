Samba_4_Schema_Mapping
======================

Overview
========

Some of the attribute types and object classes in Active Directory
schema are incompatible with the standard LDAP schema. When Samba is
running by itself on DS it only includes the core standard LDAP schema
so there is no issue. However, when integrated with IPA this becomes a
problem because IPA uses the full standard LDAP schema so there are some
conflicts.

This problem can be solved by renaming the conflicting AD schema. Samba
already has a mechanism to translates object classes and attribute
types. It needs to be expanded to include all conflicting schema.

For example, the standard the person object class requires cn and sn.
However, in AD schema the object class person only requires cn. With
this solution the AD person will be renamed to samba4Person when stored
in DS.

.. figure:: Samba4-schema-mapping.png
   :alt: Samba4-schema-mapping.png

   Samba4-schema-mapping.png

For other AD attribute types and object classes that have
identical/compatible definitions in the standard LDAP schema, Samba
should just use the standard LDAP schema.



Schema Mapping
==============



DS Schema
---------

The following AD attributes are compatible with DS schema.

==================== ====================
AD Attribute         DS Attribute
==================== ====================
name                 name
objectClasses        objectClasses
createTimeStamp      createTimeStamp
attributeTypes       attributeTypes
objectClass          objectClass
userPassword         userPassword
seeAlso              seeAlso
modifyTimeStamp      modifyTimeStamp
distinguishedName    distinguishedName
description          description
cn                   cn
dITContentRules      dITContentRules
top                  top
homePostalAddress    homePostalAddress
info                 info
displayName          displayName
employeeName         employeeName
employeeType         employeeType
personalTitle        personalTitle
co                   co
unixHomeDirectory    **homeDirectory**
userSMIMECertificate userSMIMECertificate
==================== ====================



Samba 3 Schema
--------------

The following AD attributes are compatible with `Samba 3
schema <Obsolete:Samba_3_Schema>`__.

==================== =====================
AD Attribute         Samba 3 Attribute
==================== =====================
pwdLastSet           sambaPwdLastSet
lastLogon            sambaLogonTime
lastLogoff           sambaLogoffTime
badPwdCount          sambaBadPasswordCount
logonHours           sambaLogonHours
homeDrive            sambaHomeDrive
scriptPath           sambaLogonScript
profilePath          sambaProfilePath
userWorkstations     sambaUserWorkstations
homeDirectory        sambaHomePath
groupType            sambaGroupType
nextRid              sambaNextRid
privilegeDisplayName sambaPrivName
==================== =====================



Conflicting Attribute Types
---------------------------

The following AD attributes are incompatible with DS attributes. The
solution is to rename the attributes and/or change the OID's.

+----------------------+----------------------+----------------------+
| AD Attribute         | DS Attribute         | Solution             |
+======================+======================+======================+
| fRSDirectoryFilter   | calOtherCAPURIs      | fRSDirectoryFilter   |
| OID:                 | (60rfc2739.ldif)     | OID:                 |
| 1.                   | OID:                 | fR                   |
| 2.840.113556.1.4.484 | 1.                   | SDirectoryFilter-oid |
|                      | 2.840.113556.1.4.484 |                      |
+----------------------+----------------------+----------------------+
| fRSFileFilter OID:   | calOtherFBURLs       | fRSFileFilter OID:   |
| 1.                   | (60rfc2739.ldif)     | fRSFileFilter-oid    |
| 2.840.113556.1.4.483 | OID:                 |                      |
|                      | 1.                   |                      |
|                      | 2.840.113556.1.4.483 |                      |
+----------------------+----------------------+----------------------+
| fRSUpdateTimeout     | calOtherCalAdrURIs   | fRSUpdateTimeout     |
| OID:                 | (60rfc2739.ldif)     | OID:                 |
| 1.                   | OID:                 | fRSUpdateTimeout-oid |
| 2.840.113556.1.4.485 | 1.                   |                      |
|                      | 2.840.113556.1.4.485 |                      |
+----------------------+----------------------+----------------------+
| thumbnailLogo OID:   | nsLicensedFor        | thumbnailLogo OID:   |
| 2.16                 | (02common.ldif) OID: | thumbnailLogo-oid    |
| .840.1.113730.3.1.36 | 2.16                 |                      |
|                      | .840.1.113730.3.1.36 |                      |
+----------------------+----------------------+----------------------+
| thumbnailPhoto OID:  | changeLog            | thumbnailPhoto OID:  |
| 2.16                 | (02common.ldif) OID: | thumbnailPhoto-oid   |
| .840.1.113730.3.1.35 | 2.16                 |                      |
|                      | .840.1.113730.3.1.35 |                      |
+----------------------+----------------------+----------------------+
| schemaUpdate OID:    | calCalAdrURI         | schemaUpdate OID:    |
| 1.                   | (60rfc2739.ldif)     | schemaUpdate-oid     |
| 2.840.113556.1.4.481 | OID:                 |                      |
|                      | 1.                   |                      |
|                      | 2.840.113556.1.4.481 |                      |
+----------------------+----------------------+----------------------+



Conflicting Object Classes
--------------------------

The following AD object classes are incompatible with DS object classes.
The solution is to rename the object class and/or change the OID's.

+----------------------+----------------------+----------------------+
| AD Object Class      | DS Object Class      | Solution             |
+======================+======================+======================+
| domain               | domain               | samba4Domain         |
|                      | (05rfc4524.ldif)     |                      |
+----------------------+----------------------+----------------------+
| rFC822LocalPart OID: | rFC822localPart      | s                    |
| 0.9.23               | (05rfc4524.ldif)     | amba4RFC822LocalPart |
| 42.19200300.100.4.14 | OID:                 | OID:                 |
|                      | 0.9.23               | samba                |
|                      | 42.19200300.100.4.14 | 4RFC822LocalPart-oid |
+----------------------+----------------------+----------------------+
| mailRecipient        | mailRecipient        | samba4MailRecipient  |
|                      | (50ns-mail.ldif)     |                      |
+----------------------+----------------------+----------------------+
| nisMap               | nisMap               | samba4NisMap         |
|                      | (10rfc2307.ldif)     |                      |
+----------------------+----------------------+----------------------+
| person OID: 2.5.6.6  | person (00core.ldif) | samba4Person OID:    |
|                      | OID: 2.5.6.6         | samba4Person-oid     |
+----------------------+----------------------+----------------------+
| residentialPerson    | residentialPerson    | sam                  |
| OID: 2.5.6.7         | (00core.ldif) OID:   | ba4ResidentialPerson |
|                      | 2.5.6.7              | OID:                 |
|                      |                      | samba4R              |
|                      |                      | esidentialPerson-oid |
+----------------------+----------------------+----------------------+
| organizationalPerson | organizationalPerson | samba4               |
| OID: 2.5.6.7         | (00core.ldif) OID:   | OrganizationalPerson |
|                      | 2.5.6.7              | OID:                 |
|                      |                      | samba4Orga           |
|                      |                      | nizationalPerson-oid |
+----------------------+----------------------+----------------------+
| inetOrgPerson OID:   | inetOrgPerson        | samba4InetOrgPerson  |
| 2.1                  | (0                   | OID:                 |
| 6.840.1.113730.3.2.2 | 6inetorgperson.ldif) | sam                  |
|                      | OID:                 | ba4InetOrgPerson-oid |
|                      | 2.1                  |                      |
|                      | 6.840.1.113730.3.2.2 |                      |
+----------------------+----------------------+----------------------+



Current Code
============



Schema Conversion
-----------------

Some of the schema conversions are already configured at
source4/setup/schema-map-fedora-ds-1.0. The file uses the following
format:

::

   # Skip attribute/object class
   <attribute/object class>
   ...
   # Rename OID/attribute/object class
   <old OID/attribute/object class>:<new OID/attribute/object class>
   ...

The conversion code is located in
source4/dsdb/schema/schema_convert_to_ol.c:

::

   char *dsdb_convert_schema_to_openldap(struct ldb_context *ldb,
       char *target_str, const char *mappings) 
   {
   }

Current the code doesn't map the object class parent.



Mapping Module
--------------

The mapping module is located at
source4/dsdb/samdb/ldb_modules/simple_ldap_map.c. It maintains a
separate mapping configuration:

+----------------+----------------+----------------+----------------+
| Local Name     | Remote Name    | Convert Local  | Convert Remote |
+================+================+================+================+
| objectGUID     | nsuniqueid     | guid_ns_string | encode_ns_guid |
+----------------+----------------+----------------+----------------+
| objectSid      | objectSid      | sid            | val_copy       |
|                |                | _always_binary |                |
+----------------+----------------+----------------+----------------+
| whenCreated    | c              |                |                |
|                | reateTimestamp |                |                |
+----------------+----------------+----------------+----------------+
| whenChanged    | m              |                |                |
|                | odifyTimestamp |                |                |
+----------------+----------------+----------------+----------------+
| objectCategory | objectCategory | objectCate     | val_copy       |
|                |                | gory_always_dn |                |
+----------------+----------------+----------------+----------------+
| dis            | entryDN        |                |                |
| tinguishedName |                |                |                |
+----------------+----------------+----------------+----------------+
| primaryGroupID | primaryGroupID | normali        | val_copy       |
|                |                | se_to_signed32 |                |
+----------------+----------------+----------------+----------------+
| groupType      | groupType      | normali        | val_copy       |
|                |                | se_to_signed32 |                |
+----------------+----------------+----------------+----------------+
| user           | user           | normali        | val_copy       |
| AccountControl | AccountControl | se_to_signed32 |                |
+----------------+----------------+----------------+----------------+
| sAMAccountType | sAMAccountType | normali        | val_copy       |
|                |                | se_to_signed32 |                |
+----------------+----------------+----------------+----------------+
| systemFlags    | systemFlags    | normali        | val_copy       |
|                |                | se_to_signed32 |                |
+----------------+----------------+----------------+----------------+
| usnChanged     | m              | us             | ti             |
|                | odifyTimestamp | n_to_timestamp | mestamp_to_usn |
+----------------+----------------+----------------+----------------+
| usnCreated     | c              | us             | ti             |
|                | reateTimestamp | n_to_timestamp | mestamp_to_usn |
+----------------+----------------+----------------+----------------+

The attribute mapping is stored in the following structure:

::

   static const struct ldb_map_attribute nsuniqueid_attributes[] = 
   {
       {
           .local_name = "...",
           .type = MAP_CONVERT | MAP_RENAME | MAP_KEEP,
           .u = {
               .convert = {
                   .remote_name = "...",
                   .convert_local = ...,
                   .convert_remote = ...,
               }
           }
       },
       {
           .local_name = NULL
       }
   };

Currently there is no object class mapping for DS.

The module is initialized in the following method:

::

   static int nsuniqueid_init(struct ldb_module *module)
   {
       ldb_map_init(module, nsuniqueid_attributes, NULL,
           nsuniqueid_wildcard_attributes, "extensibleObject", NULL);

       return ldb_next_init(module);
   }



Proposed Changes
================



Adding Samba 3 Schema
---------------------

Samba 3 schema and its dependencies have to be included during DS
instance creation. The following lines should be added into
source4/setup/fedorads.inf:

::

   SchemaFile=/etc/dirsrv/schema/10rfc2307.ldif
   SchemaFile=/etc/dirsrv/schema/05rfc4523.ldif
   SchemaFile=/etc/dirsrv/schema/05rfc4524.ldif
   SchemaFile=/etc/dirsrv/schema/06inetorgperson.ldif
   SchemaFile=/usr/share/dirsrv/data/60samba3.ldif



Schema Conversion
-----------------

The following schema conversion should be added:

::

   #Standard FDS attributes
   homePostalAddress
   info
   displayName
   employeeNumber
   employeeType
   personalTitle
   co
   userSMIMECertificate

   #Remap into existing schema
   unixHomeDirectory
   unixHomeDirectory:homeDirectory
   pwdLastSet
   pwdLastSet:sambaPwdLastSet
   lastLogon
   lastLogon:sambaLogonTime
   lastLogoff
   lastLogoff:sambaLogoffTime
   badPwdCount
   badPwdCount:sambaBadPasswordCount
   logonHours
   logonHours:sambaLogonHours
   homeDrive
   homeDrive:sambaHomeDrive
   scriptPath
   scriptPath:sambaLogonScript
   profilePath
   profilePath:sambaProfilePath
   userWorkstations
   userWorkstations:sambaUserWorkstations
   homeDirectory
   homeDirectory:sambaHomePath
   groupType
   groupType:sambaGroupType
   nextRid
   nextRid:sambaNextRid
   privilegeDisplayName
   privilegeDisplayName:sambaPrivName

   #Resolve conflicting attributes
   1.2.840.113556.1.4.484:fRSDirectoryFilter-oid
   1.2.840.113556.1.4.483:fRSFileFilter-oid
   1.2.840.113556.1.4.485:fRSUpdateTimeout-oid
   2.16.840.1.113730.3.1.36:thumbnailLogo-oid
   2.16.840.1.113730.3.1.35:thumbnailPhoto-oid
   1.2.840.113556.1.4.481:schemaUpdate-oid

   #Resolve conflicting object classes
   domain:samba4Domain
   rFC822LocalPart:samba4RFC822LocalPart
   mailRecipient:samba4MailRecipient
   nisMap:samba4NisMap
   0.9.2342.19200300.100.4.14:samba4RFC822LocalPart-oid
   person:samba4Person
   2.5.6.6:samba4Person-oid
   organizationalPerson:samba4OrganizationalPerson
   2.5.6.7:samba4OrganizationalPerson-oid
   residentialPerson:samba4ResidentialPerson
   2.5.6.10:samba4ResidentialPerson-oid
   inetOrgPerson:samba4InetOrgPerson
   2.16.840.1.113730.3.2.2:samba4InetOrgPerson-oid

The conversion code should be modified map the object class parent:

::

   static char *print_schema_recursive(
       char *append_to_string, struct dsdb_schema *schema, const char *print_class,
       enum dsdb_schema_convert_target target, 
       const char **attrs_skip, const struct attr_map *attr_map, const struct oid_map *oid_map) 
   {
       for (j=0; subClassOf && attr_map && attr_map[j].old_attr; j++) {
           if (strcasecmp(subClassOf, attr_map[j].old_attr) == 0) {
               subClassOf =  attr_map[j].new_attr;
               break;
           }
       }
   }



Mapping Module
--------------

The following attribute mapping should be modified:

========== ================== ===================== ==============
Local Name Remote Name        Convert Local         Convert Remote
========== ================== ===================== ==============
groupType  **sambaGroupType** normalise_to_signed32 val_copy
========== ================== ===================== ==============

The following attribute mapping should be added:

==================== ===================== ============= ==============
Local Name           Remote Name           Convert Local Convert Remote
==================== ===================== ============= ==============
unixHomeDirectory    homeDirectory                       
pwdLastSet           sambaPwdLastSet                     
lastLogon            sambaLogonTime                      
lastLogoff           sambaLogoffTime                     
badPwdCount          sambaBadPasswordCount               
logonHours           sambaLogonHours                     
homeDrive            sambaHomeDrive                      
scriptPath           sambaLogonScript                    
profilePath          sambaProfilePath                    
userWorkstations     sambaUserWorkstations               
homeDirectory        sambaHomePath                       
nextRid              sambaNextRid                        
privilegeDisplayName sambaPrivName                       
==================== ===================== ============= ==============

The following object class mapping should be added:

==================== ==========================
Local Name           Remote Name
==================== ==========================
domain               samba4Domain
rFC822LocalPart      samba4RFC822LocalPart
mailRecipient        samba4MailRecipient
nisMap               samba4NisMap
person               samba4Person
organizationalPerson samba4OrganizationalPerson
residentialPerson    samba4ResidentialPerson
inetOrgPerson        samba4InetOrgPerson
==================== ==========================

The object class mapping should stored in the following structure:

::

   const struct ldb_map_objectclass nsuniqueid_objectclasses[] =
   {
       {
           .local_name = "...",
           .remote_name = "..."
       },
       {
           .local_name = NULL
       }
   };

The module initialization should be changed to use the object class
mapping:

::

   static int nsuniqueid_init(struct ldb_module *module)
   {
       ldb_map_init(module, nsuniqueid_attributes, nsuniqueid_objectclasses,
           nsuniqueid_wildcard_attributes, "extensibleObject", NULL);

       return ldb_next_init(module);
   }

Patches
=======

The following patch has been applied to the source repository:

-  `s4 - Mapped AD schema to existing FDS
   schema <http://gitweb.samba.org/?p=samba.git;a=commit;h=8097280b468b7bcf26a0e17fdcaaccfb34d06415>`__
-  `s4:provision - Removed dependency on full Samba 3 schema from
   FDS <http://gitweb.samba.org/?p=samba.git;a=commit;h=8e5f5e3f05f9d2cd6ef1553deacce88c2a8c4d2e>`__

`Category:Obsolete <Category:Obsolete>`__