SolarisTlsProfile
=================



Sample TLS Profile File for Solaris
===================================

Back to `Configuring UNIX Clients <ConfiguringUnixClients>`__

::

   #************************************
   # Sample TLS Profile File for Solaris
   #************************************
   dn: cn=tls_default,ou=profile,dc=example,dc=com
   ObjectClass: top
   ObjectClass: DUAConfigProfile
   defaultServerList: ldap01.example.com
   defaultSearchBase: dc=example, dc=com
   authenticationMethod: tls:simple
   followReferrals: FALSE
   defaultSearchScope: sub
   searchTimeLimit: 30
   profileTTL: 43200
   bindTimeLimit: 10
   cn: tls_default
   credentialLevel: proxy
   serviceSearchDescriptor: passwd: dc=example,dc=com?sub
   serviceSearchDescriptor: group: dc=example,dc=com?sub
   serviceSearchDescriptor: shadow: dc=example,dc=com?sub
   serviceSearchDescriptor: netgroup: dc=example,dc=com?sub
   serviceSearchDescriptor: auto_master: automountMapName=auto_master, dc=example,dc=com?one
   serviceSearchDescriptor: auto_automnt: automountMapName=auto_automnt, dc=example,dc=com?one
   objectclassMap: automount: automount=nisObject
   objectclassMap: automount: automountMap=nisMap
   attributeMap: automount: automountInformation=nisMapEntry
   attributeMap: automount: automountKey=cn
   attributeMap: automount: automountMapName=nisMapName