Samba_4_Attribute_Dereferencing
===============================

Overview
========

Samba relies on the LDAP backend to perform attribute dereferencing to
generate extended DN. Samba uses an LDB module to send the request and
handle the response. Currently the module assumes the backend will be
OpenLDAP. It needs to be modified to work with DS.

This is a prerequisite for `storing SID in string
format <Obsolete:Samba_4_Storing_SID_in_String_Format>`__.



Current Code
============



Provisioning Tool
-----------------

The provisioning tool uses extended_dn_out_dereference module for both
OpenLDAP and DS:

::

   if ldap_backend.ldap_backend_type == "fedora-ds":
       tdb_modules_list = ["extended_dn_out_dereference"]

   elif ldap_backend.ldap_backend_type == "openldap":
       tdb_modules_list = ["extended_dn_out_dereference"]



Module Definition
-----------------

The module is defined in
source/dsdb/samdb/ldb_modules/extended_dn_out.c:

::

   _PUBLIC_ const struct ldb_module_ops ldb_extended_dn_out_dereference_module_ops = {
       .name         = "extended_dn_out_dereference",
       .search       = extended_dn_out_search,
       .init_context = extended_dn_out_dereference_init,
   };



Initialization Method
---------------------

The initialization method is contains OpenLDAP-specific attribute
entryUUID.

::

   static int extended_dn_out_dereference_init(struct ldb_module *module) {
       for (cur = schema->attributes; cur; cur = cur->next) {
           static const char *attrs[] = {
               "entryUUID",
               "objectSID",
               NULL
           };
       }
   }



Search Method
-------------

The search method specifies a callback function when constructing a
search request.

::

   static int extended_dn_out_search(struct ldb_module *module, struct ldb_request *req)
   {
       ret = ldb_build_search_req_ex(&down_req,
           ldb_module_get_ctx(module), ac,
           req->op.search.base,
           req->op.search.scope,
           req->op.search.tree,
           const_attrs,
           req->controls,
           ac, extended_callback,
           req);
   }



Callback Function
-----------------

The callback function calls a handler function.

::

   static int extended_callback(struct ldb_request *req, struct ldb_reply *ares)
   {
       ret = handle_dereference(dn, 
           dereference_control->attributes,
           msg->elements[i].name,
           &msg->elements[i].values[j]);
   }



Handler Function
----------------

The handler function uses OpenLDAP-specific attribute entryUUID.

::

   static int handle_dereference(struct ldb_dn *dn,
       struct dsdb_openldap_dereference_result **dereference_attrs, 
       const char *attr, const DATA_BLOB *val)
   {
       entryUUIDblob = ldb_msg_find_ldb_val(&fake_msg, "entryUUID");
       if (entryUUIDblob) {        
           GUID_from_data_blob(entryUUIDblob, &guid);      
           ndr_push_struct_blob(&guid_blob, NULL, NULL, &guid,
               (ndr_push_flags_fn_t)ndr_push_GUID);
           ldb_dn_set_extended_component(dn, "GUID", &guid_blob);
       }

       sid_blob = ldb_msg_find_ldb_val(&fake_msg, "objectSID");
       if (sid_blob) {
           ldb_dn_set_extended_component(dn, "SID", sid_blob);
       }
   }



Proposed Changes
================



Provisioning Tool
-----------------

The provisioning tool should be changed to use different modules for
OpenLDAP and DS:

::

   if ldap_backend.ldap_backend_type == "fedora-ds":
       tdb_modules_list = ["extended_dn_out_fds"]

   elif ldap_backend.ldap_backend_type == "openldap":
       tdb_modules_list = ["extended_dn_out_openldap"]



Module Definition
-----------------

The extended_dn_out_dereference module should be replaced by
extended_dn_out_openldap and extended_dn_out_fds modules:

::

   _PUBLIC_ const struct ldb_module_ops ldb_extended_dn_out_openldap_module_ops = {
       .name         = "extended_dn_out_openldap",
       .search       = extended_dn_out_openldap_search,
       .init_context = extended_dn_out_openldap_init,
   };

   _PUBLIC_ const struct ldb_module_ops ldb_extended_dn_out_fds_module_ops = {
       .name         = "extended_dn_out_fds",
       .search       = extended_dn_out_fds_search,
       .init_context = extended_dn_out_fds_init,
   };



Initialization Method
---------------------

The original initialization method should be generalized to take an
attribute list. There will be separate initialization methods for
OpenLDAP and DS which will call the generic initialization method and
supply the appropriate attribute list for the backend.

::

   static int extended_dn_out_dereference_init(struct ldb_module *module,
       const char *attrs[]) {

       for (cur = schema->attributes; cur; cur = cur->next) {
           ...
       }
   }

   static int extended_dn_out_openldap_init(struct ldb_module *module) {
       static const char *attrs[] = {
           "entryUUID",
           "objectSID",
           NULL
       };
       return extended_dn_out_dereference_init(module, attrs);
   }

   static int extended_dn_out_fds_init(struct ldb_module *module) {
       static const char *attrs[] = {
           "nsUniqueId",
           "objectSID",
           NULL
       };
       return extended_dn_out_dereference_init(module, attrs);
   }



Search Method
-------------

The original search method should be changed to take a callback
function. There will be separate search methods for OpenLDAP and DS
which will call the generic search method and supply the appropriate
callback function for the backend.

::

   static int extended_dn_out_search(
       struct ldb_module *module, struct ldb_request *req,
       int (*callback)(struct ldb_request *req, struct ldb_reply *ares))
   {
       ret = ldb_build_search_req_ex(&down_req,
           ldb_module_get_ctx(module), ac,
           req->op.search.base,
           req->op.search.scope,
           req->op.search.tree,
           const_attrs,
           req->controls,
           ac, callback,
           req);
   }

   static int extended_dn_out_openldap_search(struct ldb_module *module, struct ldb_request *req)
   {
       return extended_dn_out_search(module, req, extended_callback_openldap);
   }

   static int extended_dn_out_fds_search(struct ldb_module *module, struct ldb_request *req)
   {
       return extended_dn_out_search(module, req, extended_callback_fds);
   }



Callback Function
-----------------

The original callback function should be changed to take a handler
function. There will be separate callback functions for OpenLDAP and DS
which will call the generic callback function and supply the appropriate
handler function.

::

   static int extended_callback(struct ldb_request *req, struct ldb_reply *ares,
       int (*handle_dereference)(struct ldb_dn *dn,
           struct dsdb_openldap_dereference_result **dereference_attrs, 
           const char *attr, const DATA_BLOB *val))
   {
       ret = handle_dereference(dn, 
           dereference_control->attributes,
           msg->elements[i].name,
           &msg->elements[i].values[j]);
   }

   static int extended_callback_openldap(struct ldb_request *req, struct ldb_reply *ares)
   {
       return extended_callback(req, ares, handler_dereference_openldap);
   }

   static int extended_callback_fds(struct ldb_request *req, struct ldb_reply *ares)
   {
       return extended_callback(req, ares, handler_dereference_fds);
   }



Handler Function
----------------

The original handler function that reads the entryUUID attribute should
be used for OpenLDAP only. A new handler function that reads nsUniqueId
should be used for DS.

::

   static int handle_dereference_openldap(struct ldb_dn *dn,
       struct dsdb_openldap_dereference_result **dereference_attrs, 
       const char *attr, const DATA_BLOB *val)
   {
       entryUUIDblob = ldb_msg_find_ldb_val(&fake_msg, "entryUUID");
       if (entryUUIDblob) {        
           GUID_from_data_blob(entryUUIDblob, &guid);      
           ndr_push_struct_blob(&guid_blob, NULL, NULL, &guid,
               (ndr_push_flags_fn_t)ndr_push_GUID);
           ldb_dn_set_extended_component(dn, "GUID", &guid_blob);
       }

       sid_blob = ldb_msg_find_ldb_val(&fake_msg, "objectSID");
       if (sid_blob) {
           ldb_dn_set_extended_component(dn, "SID", sid_blob);
       }
   }

   static int handle_dereference_fds(struct ldb_dn *dn,
       struct dsdb_openldap_dereference_result **dereference_attrs, 
       const char *attr, const DATA_BLOB *val)
   {
       nsUniqueIdBlob = ldb_msg_find_ldb_val(&fake_msg, "nsUniqueId");
       if (nsUniqueIdBlob) {       
           NS_GUID_from_string((char *)nsUniqueIdBlob->data, &guid);       
           ndr_push_struct_blob(&guid_blob, NULL, NULL, &guid,
               (ndr_push_flags_fn_t)ndr_push_GUID);
           ldb_dn_set_extended_component(dn, "GUID", &guid_blob);
       }

       sid_blob = ldb_msg_find_ldb_val(&fake_msg, "objectSID");
       if (sid_blob) {
           ldb_dn_set_extended_component(dn, "SID", sid_blob);
       }
   }

Patches
=======

The following patch has been applied into the source repository:

-  `s4:dsdb - Fixed attribute dereferencing for 389
   DS <http://gitweb.samba.org/?p=samba.git;a=commit;h=1fc19ee7d0021e963923911bb440463aa79184fc>`__

References
==========

-  `LDAP Dereference
   Control <http://www.openldap.org/devel/cvsweb.cgi/~checkout~/doc/drafts/draft-masarati-ldap-deref-xx.txt>`__

`Category:Obsolete <Category:Obsolete>`__