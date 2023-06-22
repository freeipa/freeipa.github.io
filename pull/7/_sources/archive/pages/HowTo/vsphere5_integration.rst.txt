I'm running vSphere 6.0 (hot off presses last month) and these
instructions against a vSphere 6 installation and FreeIPA 4.1 (on CentOS
7.1) do not appear to work anymore. I'm troubleshooting now, but it's
not obvious what's going on and I'm pretty new to troubleshooting this
sort of problem.

What I've seen thus far is that the ldap access logs on my FreeIPA
server show signs that vCenter is logging in just fine to my FreeIPA
LDAP server:

::

   [17/Apr/2015:13:27:29 +0100] conn=2438 op=0 BIND dn="uid=#removed#,cn=users,cn=compat,dc=#removed#,dc=uk" method=128 version=3
   [17/Apr/2015:13:27:29 +0100] conn=2438 op=0 RESULT err=0 tag=97 nentries=0 etime=0 dn="uid=#removed#,cn=users,cn=accounts,dc=#removed#,dc=uk"
   [17/Apr/2015:13:27:29 +0100] conn=2438 op=1 SRCH base="cn=users,cn=compat,dc=#removed#,dc=uk" scope=2 filter="(objectClass=inetOrgPerson)" attrs="uid description givenName sn mail useraccountcontrol pwdaccountlockedtime entryuuid"
   [17/Apr/2015:13:27:29 +0100] conn=2438 op=1 RESULT err=0 tag=101 nentries=4 etime=0 notes=P
   [17/Apr/2015:13:27:29 +0100] conn=2438 op=2 UNBIND

And on the VCSA 6.0 appliance is the vmware-sts-idmd.log I see this
corresponding exception:

::

   [2015-04-17T19:17:42.252Z #removed#.local  f4468c9c-220c-43d1-99af-cf8fd6a7d926 WARN ] [LdapErrorChecker] Error received by LDAP client: com.vmware.identity.interop.ldap.LinuxLdapClientLibrary, error code: -13
   [2015-04-17T19:17:42.252Z #removed#.local  f4468c9c-220c-43d1-99af-cf8fd6a7d926 ERROR] [LinuxLdapClientLibrary] Exception when calling ldap_one_paged_search: base=cn=users,cn=compat,dc=#removed#,dc=uk, scope=2, filter=(objectClass=inetOrgPerson), attrs=[Ljava.lang.String;@94e9177, attrsonly=0, sizelimit=0
   com.vmware.identity.interop.ldap.ControlNotFoundLdapException: Control not found
   LDAP error [code: -13]
           at com.vmware.identity.interop.ldap.LdapErrorChecker$56.RaiseLdapError(LdapErrorChecker.java:768)
           at com.vmware.identity.interop.ldap.LdapErrorChecker.CheckError(LdapErrorChecker.java:826)
           at com.vmware.identity.interop.ldap.LinuxLdapClientLibrary.CheckError(LinuxLdapClientLibrary.java:1066)
           at com.vmware.identity.interop.ldap.LinuxLdapClientLibrary.ldap_one_paged_search(LinuxLdapClientLibrary.java:859)
           at com.vmware.identity.interop.ldap.LdapConnection$5.call(LdapConnection.java:651)
           at com.vmware.identity.interop.ldap.LdapConnection$5.call(LdapConnection.java:648)
           at com.vmware.identity.interop.ldap.LdapConnection.execute(LdapConnection.java:73)
           at com.vmware.identity.interop.ldap.LdapConnection.search_one_page_internal(LdapConnection.java:647)
           at com.vmware.identity.interop.ldap.LdapConnection.paged_search(LdapConnection.java:597)
           at com.vmware.identity.idm.server.provider.ldap.LdapProvider.ldap_search(LdapProvider.java:2236)
           at com.vmware.identity.idm.server.provider.ldap.LdapProvider.findUsersInternal(LdapProvider.java:782)
           at com.vmware.identity.idm.server.provider.ldap.LdapProvider.findUsers(LdapProvider.java:725)
           at com.vmware.identity.idm.server.provider.ldap.LdapProvider.find(LdapProvider.java:1407)
           at com.vmware.identity.idm.server.IdentityManager.find(IdentityManager.java:4443)
           at com.vmware.identity.idm.server.IdentityManager.find(IdentityManager.java:8963)
           at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
           at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
           at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
           at java.lang.reflect.Method.invoke(Unknown Source)
           at sun.rmi.server.UnicastServerRef.dispatch(Unknown Source)
           at sun.rmi.transport.Transport$1.run(Unknown Source)
           at sun.rmi.transport.Transport$1.run(Unknown Source)
           at java.security.AccessController.doPrivileged(Native Method)
           at sun.rmi.transport.Transport.serviceCall(Unknown Source)
           at sun.rmi.transport.tcp.TCPTransport.handleMessages(Unknown Source)
           at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run0(Unknown Source)
           at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run(Unknown Source)
           at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
           at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
           at java.lang.Thread.run(Unknown Source)
   [2015-04-17T19:17:42.253Z #removed#.local  f4468c9c-220c-43d1-99af-cf8fd6a7d926 ERROR] [IdentityManager] Failed to find objects [Criteria : search String=, domain=#removed#.uk] in tenant [#removed#.local]
   [2015-04-17T19:17:42.253Z #removed#.local  f4468c9c-220c-43d1-99af-cf8fd6a7d926 ERROR] [ServerUtils] Exception 'com.vmware.identity.interop.ldap.ControlNotFoundLdapException: Control not found
   LDAP error [code: -13]'
   com.vmware.identity.interop.ldap.ControlNotFoundLdapException: Control not found
   LDAP error [code: -13]
           at com.vmware.identity.interop.ldap.LdapErrorChecker$56.RaiseLdapError(LdapErrorChecker.java:768)
           at com.vmware.identity.interop.ldap.LdapErrorChecker.CheckError(LdapErrorChecker.java:826)
           at com.vmware.identity.interop.ldap.LinuxLdapClientLibrary.CheckError(LinuxLdapClientLibrary.java:1066)
           at com.vmware.identity.interop.ldap.LinuxLdapClientLibrary.ldap_one_paged_search(LinuxLdapClientLibrary.java:859)
           at com.vmware.identity.interop.ldap.LdapConnection$5.call(LdapConnection.java:651)
           at com.vmware.identity.interop.ldap.LdapConnection$5.call(LdapConnection.java:648)
           at com.vmware.identity.interop.ldap.LdapConnection.execute(LdapConnection.java:73)
           at com.vmware.identity.interop.ldap.LdapConnection.search_one_page_internal(LdapConnection.java:647)
           at com.vmware.identity.interop.ldap.LdapConnection.paged_search(LdapConnection.java:597)
           at com.vmware.identity.idm.server.provider.ldap.LdapProvider.ldap_search(LdapProvider.java:2236)
           at com.vmware.identity.idm.server.provider.ldap.LdapProvider.findUsersInternal(LdapProvider.java:782)
           at com.vmware.identity.idm.server.provider.ldap.LdapProvider.findUsers(LdapProvider.java:725)
           at com.vmware.identity.idm.server.provider.ldap.LdapProvider.find(LdapProvider.java:1407)
           at com.vmware.identity.idm.server.IdentityManager.find(IdentityManager.java:4443)
           at com.vmware.identity.idm.server.IdentityManager.find(IdentityManager.java:8963)
           at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
           at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
           at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
           at java.lang.reflect.Method.invoke(Unknown Source)
           at sun.rmi.server.UnicastServerRef.dispatch(Unknown Source)
           at sun.rmi.transport.Transport$1.run(Unknown Source)
           at sun.rmi.transport.Transport$1.run(Unknown Source)
           at java.security.AccessController.doPrivileged(Native Method)
           at sun.rmi.transport.Transport.serviceCall(Unknown Source)
           at sun.rmi.transport.tcp.TCPTransport.handleMessages(Unknown Source)
           at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run0(Unknown Source)
           at sun.rmi.transport.tcp.TCPTransport$ConnectionHandler.run(Unknown Source)
           at java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
           at java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
           at java.lang.Thread.run(Unknown Source)

Any ideas?
