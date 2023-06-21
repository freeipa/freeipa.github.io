Samba_4_Disabling_Heimdal_Service
=================================

Overview
========

IPA uses MIT KDC to provide Kerberos and kpasswd services for IPA
clients. Samba uses Heimdal KDC to provide Kerberos and kpasswd services
for Samba clients. Only one of the KDC's can run on a machine because
they both listen to the same ports: 88 and 464 for Kerberos and kpasswd
services, respectively.

In the integrated environment, MIT KDC was chosen to provide Kerberos
and kpasswd services for both IPA and Samba clients. However, Samba
still needs Heimdal KDC service to perform some operations internally.
To avoid conflicts Samba needs to be modified so that Heimdal KDC
service can continue running but it does not listen to the ports.



Current Code
============



Port Configuration
------------------

Currently Samba supports configuring the ports for Kerberos and kpasswd
services in smb.conf. By default they will be set to 88 and 464 in
source4/param/loadparam.c.

::

   struct loadparm_context *loadparm_init(TALLOC_CTX *mem_ctx)
   {
       lp_do_global_parameter(lp_ctx, "krb5 port", "88");
       lp_do_global_parameter(lp_ctx, "kpasswd port", "464");
   }



KDC Initialization
------------------

The code for Heimdal KDC service is located in kdc_task_init() in
source4/kdc/kdc.c.

::

   static void kdc_task_init(struct task_server *task)
   {
       load_interfaces(task, lp_interfaces(task->lp_ctx), &ifaces);

       smb_krb5_init_context(kdc, task->event_ctx, task->lp_ctx,
           &kdc->smb_krb5_context);

       krb5_add_et_list(kdc->smb_krb5_context->krb5_context,
           initialize_hdb_error_table_r);

       krb5_kdc_get_config(kdc->smb_krb5_context->krb5_context, 
           &kdc->config);

       hdb_samba4_create_kdc(kdc, task->event_ctx, task->lp_ctx, 
           kdc->smb_krb5_context->krb5_context, 
           &kdc->config->db[0]);

       krb5_plugin_register(kdc->smb_krb5_context->krb5_context, 
           PLUGIN_TYPE_DATA, "hdb",
           &hdb_samba4);

       krb5_kt_register(kdc->smb_krb5_context->krb5_context, &hdb_kt_ops);

       krb5_plugin_register(kdc->smb_krb5_context->krb5_context, 
           PLUGIN_TYPE_DATA, "windc",
           &windc_plugin_table);

       krb5_kdc_windc_init(kdc->smb_krb5_context->krb5_context);

       // create sockets
       kdc_startup_interfaces(kdc, task->lp_ctx, ifaces);

       IRPC_REGISTER(task->msg_ctx, irpc, KDC_CHECK_GENERIC_KERBEROS, 
           kdc_check_generic_kerberos, kdc);
   }



Socket Creation
---------------

The kdc_startup_interfaces() creates sockets for all interfaces:

::

   static NTSTATUS kdc_startup_interfaces(struct kdc_server *kdc,
           struct loadparm_context *lp_ctx,
           struct interface *ifaces)
   {
       num_interfaces = iface_count(ifaces);
       
       for (i=0; i<num_interfaces; i++) {
           address = talloc_strdup(tmp_ctx, iface_n_ip(ifaces, i));
           kdc_add_socket(kdc, address, lp_krb5_port(lp_ctx), 
               lp_kpasswd_port(lp_ctx));
       }
   }

The kdc_add_socket() creates TCP and UDP sockets for kdc and kpasswd
services:

::

   static NTSTATUS kdc_add_socket(struct kdc_server *kdc, const char *address,
                      uint16_t kdc_port, uint16_t kpasswd_port)
   {
       // listen to kdc UDP port
       kdc_address = socket_address_from_strings(kdc_socket,
           kdc_socket->sock->backend_name, address, kdc_port);

       socket_listen(kdc_socket->sock, kdc_address, 0, 0);

       // listen to kpasswd UDP port
       kpasswd_address = socket_address_from_strings(kpasswd_socket,
           kpasswd_socket->sock->backend_name, address, kpasswd_port);

       socket_listen(kpasswd_socket->sock, kpasswd_address, 0, 0);

       // create process model
       model_ops = process_model_startup(kdc->task->event_ctx, "single");

       // listen to kdc TCP port
       stream_setup_socket(kdc->task->event_ctx, 
           kdc->task->lp_ctx,
           model_ops, 
           &kdc_tcp_stream_ops, 
           "ip", address, &kdc_port, 
           lp_socket_options(kdc->task->lp_ctx), 
           kdc);

       // listen to kpasswd TCP port
       stream_setup_socket(kdc->task->event_ctx, 
           kdc->task->lp_ctx,
           model_ops, 
           &kpasswdd_tcp_stream_ops, 
           "ip", address, &kpasswd_port, 
           lp_socket_options(kdc->task->lp_ctx), 
           kdc);
   }



Proposed Solution
=================



Port Configuration
------------------

Heimdal ports could be disabled by setting them to 0 in smb.conf:

::

   [global]
       krb5 port = 0
       kpasswd port = 0

Note: No Samba source code will be changed for this.



KDC Initialization
------------------

The kdc_task_init() should not be modified so that it will continue to
initialize Heimdal service.



Socket Creation
---------------

The kdc_startup_interfaces() should be changed such that it does not
create the sockets when the ports are set to 0.

::

   static NTSTATUS kdc_startup_interfaces(struct kdc_server *kdc,
           struct loadparm_context *lp_ctx,
           struct interface *ifaces)
   {
       // create process model
       model_ops = process_model_startup(kdc->task->event_ctx, "single");

       num_interfaces = iface_count(ifaces);
       
       for (i=0; i<num_interfaces; i++) {
           address = talloc_strdup(tmp_ctx, iface_n_ip(ifaces, i));
           kdc_port = lp_krb5_port(lp_ctx);
           kpasswd_port = lp_kpasswd_port(lp_ctx);

           if (kdc_port) {
               kdc_add_kdc_socket(kdc, model_ops, address, kdc_port);
           }

           if (kpasswd_port) {
               kdc_add_passwd_socket(kdc, model_ops, address, kpasswd_port);
           }
       }
   }

The kdc_add_socket() should be split into kdc_add_kdc_socket() and
kdc_add_kpasswd_socket() as follows:

::

   static NTSTATUS kdc_add_kdc_socket(struct kdc_server *kdc,
           const struct model_ops *model_ops,
           const char *address,
           uint16_t kdc_port)
   {
       // listen to kdc UDP port
       kdc_address = socket_address_from_strings(kdc_socket,
           kdc_socket->sock->backend_name, address, kdc_port);

       socket_listen(kdc_socket->sock, kdc_address, 0, 0);

       // listen to kdc TCP port
       stream_setup_socket(kdc->task->event_ctx, 
           kdc->task->lp_ctx,
           model_ops, 
           &kdc_tcp_stream_ops, 
           "ip", address, &kdc_port, 
           lp_socket_options(kdc->task->lp_ctx), 
           kdc);
   }

   static NTSTATUS kdc_add_kpasswd_socket(struct kdc_server *kdc,
           const struct model_ops *model_ops,
           const char *address,
           uint16_t kpasswd_port)
   {
       // listen to kpasswd UDP port
       kpasswd_address = socket_address_from_strings(kpasswd_socket,
           kpasswd_socket->sock->backend_name, address, kpasswd_port);

       socket_listen(kpasswd_socket->sock, kpasswd_address, 0, 0);

       // listen to kpasswd TCP port
       stream_setup_socket(kdc->task->event_ctx, 
           kdc->task->lp_ctx,
           model_ops, 
           &kpasswdd_tcp_stream_ops, 
           "ip", address, &kpasswd_port, 
           lp_socket_options(kdc->task->lp_ctx), 
           kdc);
   }

Patches
=======

The following patch has been applied to the source repository:

-  `s4:kdc - Disable KDC port when it's set to
   0 <http://gitweb.samba.org/?p=samba.git;a=commit;h=c93fc3a10a8839752eb4c1d1e91c1b455c974eef>`__
-  `s4:kdc - Merged kdc_add_kdc_socket() and
   kdc_add_kpasswd_socket() <http://gitweb.samba.org/?p=samba.git;a=commit;h=0c89a6f2aa433e54d7af99d9214ddc186784af97>`__
-  `s4:kdc - Merged kdc_tcp_accept() and
   kpasswdd_tcp_accept() <http://gitweb.samba.org/?p=samba.git;a=commit;h=9ce7e9ab8401e038b36d53e477fcb658d1c54f80>`__

`Category:Obsolete <Category:Obsolete>`__