Resolving_Conflicting_LDAP_Port
===============================

Overview
========

Both IPA and Samba are providing LDAP services on port 389. If installed
on the same machine, these services will conflict with each other. To
solve this problem, the machine must have multiple IP addresses, then
the services must bind to different addresses.



Configuring IPA
===============

IPA can listen to a specific address by specifying the
nsslapd-listenhost parameter:

::

   dn: cn=config
   nsslapd-listenhost: ipa.example.com
   nsslapd-port: 389
   nsslapd-securelistenhost: ipa.example.com
   nsslapd-secureport: 636

Note that IPA can only listen to one IP address.

The socket is created in ldap/servers/slapd/main.c and
ldap/servers/slapd/daemon.c:

::

   int main(int argc, char **argv) {

       // get port
       n_port = config_get_port();
       ports_info.n_port = (unsigned short)n_port;

       // get listen address     
       slapd_listenhost2addr(config_get_listenhost(), &ports_info.n_listenaddr);

       // create sockets
       daemon_pre_setuid_init(&ports_info);

       slapd_daemon(&ports_info);
   }

   int daemon_pre_setuid_init(daemon_ports_t *ports) {

       // create sockets
       ports->n_socket = createprlistensockets(ports->n_port, ports->n_listenaddr, 0, 0);
   }

   void slapd_daemon(daemon_ports_t *ports) {

       // get sockets
       n_tcps = ports->n_socket;

       // iterate through each socket
       for (fdesp = n_tcps; fdesp && *fdesp; fdesp++) {

           // listen to socket
           PR_Listen(*fdesp, DAEMON_LISTEN_SIZE);
       }
   }



Configuring Samba
=================

Samba services can listen to specific interfaces only by adding the
following parameters in smb.conf:

::

   [global]
       interfaces           = eth1, eth4
       bind interfaces only = Yes

Note that the port number is not configurable. Each interface
corresponds to one IP address.

The socket is created in source4/ldap_server/ldap_server.c:

::

   NTSTATUS server_service_ldap_init() {
       return register_server_service("ldap", ldapsrv_task_init);
   }

   static void ldapsrv_task_init(struct task_server *task) {

       // if "interfaces" and "bind interfaces only" are defined
       if (lp_interfaces(task->lp_ctx) && lp_bind_interfaces_only(task->lp_ctx)) {

           // load interfaces
           load_interfaces(task, lp_interfaces(task->lp_ctx), &ifaces);
           num_interfaces = iface_count(ifaces);

           // iterate through each interface
           for(i = 0; i < num_interfaces; i++) {

               // get interface address
               const char *address = iface_n_ip(ifaces, i);

               // listen to address
               status = add_socket(task->event_ctx, task->lp_ctx, model_ops,
                   address, ldap_service);
           }
       }
   }



Creating Virtual IP Address
===========================

Make sure the machine has an network interface with a static IP address.
Create a new network interface as its child:

::

   % cd /etc/sysconfig/network-scripts
   % cp ifcfg-eth0 ifcfg-eth0:0

Edit ifcfg-eth0:0:

::

   DEVICE=eth0:0
   IPADDR=<new IP address>

Restart networking service:

::

   % service network restart

References
==========

-  `nsslapd-listenhost (Listen to IP
   Address) <http://www.redhat.com/docs/manuals/dir-server/8.1/cli/Configuration_Command_File_Reference-Core_Server_Configuration_Reference-Core_Server_Configuration_Attributes_Reference.html#Configuration_Command_File_Reference-cnconfig-nsslapd_listenhost_Listen_to_IP_Address>`__
-  `nsslapd-port (Port
   Number) <http://www.redhat.com/docs/manuals/dir-server/8.1/cli/Configuration_Command_File_Reference-Core_Server_Configuration_Reference-Core_Server_Configuration_Attributes_Reference.html#Configuration_Command_File_Reference-cnconfig-nsslapd_port_Port_Number>`__
-  `nsslapd-securelistenhost <http://www.redhat.com/docs/manuals/dir-server/8.1/cli/Configuration_Command_File_Reference-Core_Server_Configuration_Reference-Core_Server_Configuration_Attributes_Reference.html#Configuration_Command_File_Reference-cnconfig-nsslapd_securelistenhost>`__
-  `nsslapd-securePort (Encrypted Port
   Number) <http://www.redhat.com/docs/manuals/dir-server/8.1/cli/Configuration_Command_File_Reference-Core_Server_Configuration_Reference-Core_Server_Configuration_Attributes_Reference.html#Configuration_Command_File_Reference-cnconfig-nsslapd_securePort_Encrypted_Port_Number>`__
-  `Multiple
   Interfaces <http://us6.samba.org/samba/docs/man/Samba-HOWTO-Collection/NetworkBrowsing.html#id2583165>`__

`Category:Obsolete <Category:Obsolete>`__