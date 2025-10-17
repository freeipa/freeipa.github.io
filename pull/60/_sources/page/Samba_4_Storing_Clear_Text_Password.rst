Samba_4_Storing_Clear_Text_Password
===================================

Overview
========

In the initial implementation the password synchronization will be
implemented by storing the clear text password in the LDAP backend. In
the final implementation this will be replaced using an LDB module. See
also `this page <Obsolete:IPAv3_Password_Synchronization>`__.

By default Samba encrypts user password before storing it into the
backend. Samba can be configured to store clear text password by
enabling the DOMAIN_PASSWORD_STORE_CLEARTEXT flag in the pwdProperties
attribute of the domain object.

The flag is defined as follows:

::

   #define DOMAIN_PASSWORD_STORE_CLEARTEXT ( 0x00000010 )



Current Code
============



Password Hash Module
--------------------

The password encryption is done in
source4/dsdb/samdb/ldb_modules/password_hash.c.

When a new password is being set using LDAP add or modify operation, the
module will check the password policy using the
build_domain_data_request() function:

::

   // search password policy in domain object
   attrs[] = { "pwdProperties", "pwdHistoryLength", NULL };
   return ldb_build_search_req(&ac->dom_req, ldb, ac,
       ldb_get_default_basedn(ldb),
       LDB_SCOPE_BASE,
       NULL, attrs,
       NULL,
       ac, get_domain_data_callback,
       ac->req);

The get_domain_data_callback() function will check the
DOMAIN_PASSWORD_STORE_CLEARTEXT bit:

::

   data->pwdProperties = samdb_result_uint(ares->message, "pwdProperties", 0);
   data->store_cleartext = data->pwdProperties & DOMAIN_PASSWORD_STORE_CLEARTEXT;

The clear text password will be stored in the supplementalCredentials
attribute by the setup_supplemental_field() function:

::

   // check password policy
   if (io->domain->store_cleartext &&
       (io->u.user_account_control & UF_ENCRYPTED_TEXT_PASSWORD_ALLOWED)) {
       do_cleartext = true;
   }

   // add a package for clear text password
   if (do_cleartext) {
       /* Packages */
       pp = &packages[num_packages++];

       /* Primary:CLEARTEXT */
       nc = &names[num_names++];
       pc = &packages[num_packages++];
   }

   // store clear text password into the package
   if (pc) {
       *nc     = "CLEARTEXT";
       pcb.cleartext   = *io->n.cleartext_utf16;
       ndr_err = ndr_push_struct_blob(&pcb_blob, io->ac, 
           lp_iconv_convenience(ldb_get_opaque(ldb, "loadparm")),
           &pcb,
           (ndr_push_flags_fn_t)ndr_push_package_PrimaryCLEARTEXTBlob);

       pcb_hexstr = data_blob_hex_string_upper(io->ac, &pcb_blob);

       pc->name    = "Primary:CLEARTEXT";
       pc->reserved    = 1;
       pc->data    = pcb_hexstr;
   }

   // store package into supplementalCredentials
   scb.sub.num_packages    = num_packages;
   scb.sub.packages    = packages;

   ndr_push_struct_blob(&io->g.supplemental, io->ac, 
           lp_iconv_convenience(ldb_get_opaque(ldb, "loadparm")),
           &scb,
           (ndr_push_flags_fn_t)ndr_push_supplementalCredentialsBlob);



Encoder and Decoder Functions
-----------------------------

The clear text encoder & decoder functions are defined in
librpc/gen_ndr/ndr_drsblobs.c:

::

   _PUBLIC_ enum ndr_err_code ndr_push_package_PrimaryCLEARTEXTBlob(struct ndr_push *ndr, int ndr_flags, const struct package_PrimaryCLEARTEXTBlob *r)
   {
       if (ndr_flags & NDR_SCALARS) {
           NDR_CHECK(ndr_push_align(ndr, 4));
           {
               uint32_t _flags_save_DATA_BLOB = ndr->flags;
               ndr_set_flags(&ndr->flags, LIBNDR_FLAG_REMAINING);
               NDR_CHECK(ndr_push_DATA_BLOB(ndr, NDR_SCALARS, r->cleartext));
               ndr->flags = _flags_save_DATA_BLOB;
           }
           NDR_CHECK(ndr_push_trailer_align(ndr, 4));
       }
       if (ndr_flags & NDR_BUFFERS) {
       }
       return NDR_ERR_SUCCESS;
   }

   _PUBLIC_ enum ndr_err_code ndr_pull_package_PrimaryCLEARTEXTBlob(struct ndr_pull *ndr, int ndr_flags, struct package_PrimaryCLEARTEXTBlob *r)
   {
       if (ndr_flags & NDR_SCALARS) {
           NDR_CHECK(ndr_pull_align(ndr, 4));
           {
               uint32_t _flags_save_DATA_BLOB = ndr->flags;
               ndr_set_flags(&ndr->flags, LIBNDR_FLAG_REMAINING);
               NDR_CHECK(ndr_pull_DATA_BLOB(ndr, NDR_SCALARS, &r->cleartext));
               ndr->flags = _flags_save_DATA_BLOB;
           }
           NDR_CHECK(ndr_pull_trailer_align(ndr, 4));
       }
       if (ndr_flags & NDR_BUFFERS) {
       }
       return NDR_ERR_SUCCESS;
   }



Proposed Solution
=================

There is no code changes required in Samba. However, after installing
Samba the clear text password needs to be enabled by executing the
following command:

::

   % ldapmodify -x -D "cn=Administrator,cn=Users,dc=samba,dc=example,dc=com" -W
   dn: dc=samba,dc=example,dc=com
   changetype: modify
   replace: pwdProperties
   pwdProperties: 17

References
==========

-  `DOMAIN_PASSWORD_INFORMATION
   Structure <http://msdn.microsoft.com/en-us/library/aa375371%28VS.85%29.aspx>`__
-  `Primary:CLEARTEXT
   Property <http://msdn.microsoft.com/en-us/library/cc245682%28PROT.10%29.aspx>`__
-  `clearTextPassword <http://msdn.microsoft.com/en-us/library/cc245686%28PROT.13%29.aspx>`__
-  `DCE 1.1:Remote Procedure Call - Transfer Syntax
   NDR <http://www.opengroup.org/onlinepubs/9629399/chap14.htm>`__

`Category:Obsolete <Category:Obsolete>`__