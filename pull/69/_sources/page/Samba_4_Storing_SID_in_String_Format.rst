Samba_4_Storing_SID_in_String_Format
====================================

Overview
========

In order to guarantee uniqueness among multiple Samba instances backed
by DS, DNA plugin will be used to allocate SID. See also `this
page <Obsolete:Samba_4_SID_Allocation_using_DNA_Plugin>`__.

Currently the DNA plugin can only support generating integer or string
values whereas Samba is storing the SID in binary format in the DS. In
order to utilize the plugin without any changes, Samba code needs to be
changed such that it stores the SID in the string format. The SID will
be stored in sambaSID attribute which is part of Samba 3 schema.



Current Code
============

In Samba code the SID may exist in 3 different formats:

-  SID structure (struct dom_sid)
-  Binary format (NDR-encoded DATA_BLOB)
-  String format (LDIF-encoded char*)

The mapping code always converts the SID into binary format before
storing it into the DS.



SID Structure
-------------

The SID structure is defined in librpc/gen_ndr/security.h:

::

   struct dom_sid {
       uint8_t sid_rev_num;
       int8_t num_auths;
       uint8_t id_auth[6];
       uint32_t sub_auths[15];
   };



Conversion Methods
------------------

The binary conversion methods are defined in librpc/ndr/ndr.c:

::

   // convert SID structure to binary
   _PUBLIC_ enum ndr_err_code ndr_push_dom_sid(
       struct ndr_push *ndr, int ndr_flags, const struct dom_sid *r);

   // convert binary to SID structure
   _PUBLIC_ enum ndr_err_code ndr_pull_dom_sid(
       struct ndr_pull *ndr, int ndr_flags, struct dom_sid *r);

The string conversion methods are defined in libcli/security/dom_sid.c:

::

   // convert string to SID structure
   bool dom_sid_parse(const char *sidstr, struct dom_sid *ret);

   // convert SID structure to string
   char *dom_sid_string(TALLOC_CTX *mem_ctx, const struct dom_sid *sid);



Attribute Syntax
----------------

The SID syntax is defined in source4/lib/ldb-samba/ldif_handlers.c:

::

   static const struct ldb_schema_syntax samba_syntaxes[] = {
       {
           .name         = LDB_SYNTAX_SAMBA_SID,
           .ldif_read_fn     = ldif_read_objectSid,
           .ldif_write_fn    = ldif_write_objectSid,
           .canonicalise_fn  = ldif_canonicalise_objectSid,
           .comparison_fn    = ldif_comparison_objectSid
       }
   }

The handlers are defined in source4/lib/lib-samba/ldif_handlers.c:

::

   static int ldif_read_objectSid(
       struct ldb_context *ldb, void *mem_ctx,
       const struct ldb_val *in, struct ldb_val *out)
   {
       // convert string to SID structure
       sid = dom_sid_parse_length(mem_ctx, in);

       // convert SID structure to binary
       ndr_err = ndr_push_struct_blob(out, mem_ctx, NULL, sid,
           (ndr_push_flags_fn_t)ndr_push_dom_sid);
   }

   static int ldif_write_objectSid(
       struct ldb_context *ldb, void *mem_ctx,
       const struct ldb_val *in, struct ldb_val *out)
   {
       // convert binary to SID structure
       ndr_err = ndr_pull_struct_blob_all(in, sid, NULL, sid,
           (ndr_pull_flags_fn_t)ndr_pull_dom_sid);

       // convert SID structure to string
       *out = data_blob_string_const(dom_sid_string(mem_ctx, sid));
   }

The ldif_canonicalise_objectSid() makes sure that the SID is converted
into binary.

::

   static int ldif_canonicalise_objectSid(
       struct ldb_context *ldb, void *mem_ctx,
       const struct ldb_val *in, struct ldb_val *out)
   {
       // if SID is string
       if (ldif_comparision_objectSid_isString(in)) {

           // convert string to binary
           if (ldif_read_objectSid(ldb, mem_ctx, in, out) != 0) {

               // in case of error return a copy
               return ldb_handler_copy(ldb, mem_ctx, in, out);
           }
           return 0;
       }

       // if not string return a copy
       return ldb_handler_copy(ldb, mem_ctx, in, out);
   }



Attribute Mapping
-----------------

The SID mapping is defined in
source4/dsdb/samdb/ldb_modules/simple_ldap_map.c:

::

   // mapping for OpenLDAP
   static const struct ldb_map_attribute entryuuid_attributes[] = {
       {
           .local_name = "objectSid",
           .type = MAP_CONVERT,
           .u = {
               .convert = {
                   .remote_name = "objectSid", 
                   .convert_local = sid_always_binary,
                   .convert_remote = val_copy,
               },
           },
       }
   }

   // mapping for DS
   static const struct ldb_map_attribute nsuniqueid_attributes[] = {
       {
           .local_name = "objectSid",
           .type = MAP_CONVERT,
           .u = {
               .convert = {
                   .remote_name = "objectSid", 
                   .convert_local = sid_always_binary,
                   .convert_remote = val_copy,
               }
           }
       },
   }

The handlers are also defined in
source4/dsdb/samdb/ldb_modules/simple_ldap_map.c:

::

   static struct ldb_val sid_always_binary(
       struct ldb_module *module, TALLOC_CTX *ctx, const struct ldb_val *val)
   {
       // get SID syntax
       a = ldb_schema_attribute_by_name(ldb, "objectSid");

       // canonicalize SID
       a->syntax->canonicalise_fn(ldb, ctx, val, &out);

       return out;
   }

   static struct ldb_val val_copy(
       struct ldb_module *module, TALLOC_CTX *ctx, const struct ldb_val *val)
   {
       // no conversion because SID is already in binary format
       return ldb_val_dup(ctx, val);
   }



Attribute Dereferencing
-----------------------

The handler function in extended_dn_out_fds module reads the binary SID
value.

::

   static int handle_dereference_fds(struct ldb_dn *dn,
       struct dsdb_openldap_dereference_result **dereference_attrs, 
       const char *attr, const DATA_BLOB *val)
   {
       sid_blob = ldb_msg_find_ldb_val(&fake_msg, "objectSID");
       if (sid_blob) {
           ldb_dn_set_extended_component(dn, "SID", sid_blob);
       }
   }

Schema
------

The provisioning tool generates the objectSid attribute in 99_ad.ldif.
The attribute uses Octet String (binary) syntax.

::

   attributeTypes: (
     1.2.840.113556.1.4.146
     NAME 'objectSid'
     EQUALITY octetStringMatch
     SYNTAX 1.3.6.1.4.1.1466.115.121.1.40
     SINGLE-VALUE
     )



Proposed Changes
================

One option is to change the ldif_canonicalise_objectSid() to convert
from binary to string. However, this method is used in many places and
by different backends as well.

To minimize the risks, the changes should be done specifically for DS
only.



Attribute Mapping
-----------------

The mapping for DS should be changed as follows:

::

   static const struct ldb_map_attribute nsuniqueid_attributes[] = {
       {
           .local_name = "objectSid",
           .type = MAP_CONVERT,
           .u = {
               .convert = {
                   .remote_name = "sambaSID", 
                   .convert_local = sid_always_string,
                   .convert_remote = sid_always_binary,
               }
           }
       },
   }

Then the following method should be added:

::

   static struct ldb_val sid_always_string(
       struct ldb_module *module, TALLOC_CTX *ctx, const struct ldb_val *val)
   {
       // if SID is string
       if (ldif_comparision_objectSid_isString(in)) {

           // return a copy
           return ldb_handler_copy(ldb, mem_ctx, in, out);

       } else {

           // convert binary to string
           if (ldif_write_objectSid(ldb, mem_ctx, in, out) != 0) {

               // in case of error return a copy
               return ldb_handler_copy(ldb, mem_ctx, in, out);
           }
           return 0;
       }
   }



Attribute Dereferencing
-----------------------

The handler function in extended_dn_out_fds module should be changed to
read the string SID value and convert it into SID structure.

::

   static int handle_dereference_fds(struct ldb_dn *dn,
       struct dsdb_openldap_dereference_result **dereference_attrs, 
       const char *attr, const DATA_BLOB *val)
   {
       sidBlob = ldb_msg_find_ldb_val(&fake_msg, "sambaSID");
       if (sidBlob) {
           // convert string into SID structure
           sid = dom_sid_parse_length(NULL, sidBlob);

           // convert SID structure into binary
           ndr_push_struct_blob(&sid_blob, NULL, NULL, sid,
               (ndr_push_flags_fn_t)ndr_push_dom_sid);

           ldb_dn_set_extended_component(dn, "SID", sid_blob);
       }
   }



Schema
------

The provisioning tool should be configured such that it doesn't generate
the objectSid attribute but instead it uses the sambaSID attribute. The
schema conversion is located at source4/setup/schema-map-fedora-ds-1.0:

::

   objectSid
   objectSid:sambaSID

Issues
======



Attribute Dereferencing
-----------------------

In order to change the storage format in DS without affecting the format
in OpenLDAP, a new attribute deferencing module needs to be created for
the DS. See also `this
page <Obsolete:Samba_4_Attribute_Dereferencing>`__.



Schema Mapping
--------------

Samba 3 schema has a dependency on InetOrgPerson schema which is
conflicting with AD schema. To solve this the AD schema needs to be
renamed. See also `this page <Obsolete:Samba_4_Schema_Mapping>`__.

Patches
=======

The following patch has been applied to the source repository:

-  `s4:dsdb - Store SID as string in
   FDS <http://gitweb.samba.org/?p=samba.git;a=commit;h=bf01937549cd1ebaf327a709ecb104bfc0e0705c>`__

`Category:Obsolete <Category:Obsolete>`__