SolarisNonTlsProfile
====================



Sample Non TLS Profile for Solaris
==================================

Back to `Configuring UNIX Clients <ConfiguringUnixClients>`__

::

   #****************************************
   # Sample Non TLS Profile File for Solaris
   #****************************************
   dn: ou=profile,dc=example,dc=com
   objectClass: top
   objectClass: organizationalUnit
   ou: profile

   dn: cn=default,ou=profile,dc=example,dc=com
   objectClass: top
   objectClass: DUAConfigProfile
   defaultServerList: ldap01.example.com
   defaultSearchBase: dc=example,dc=com
   authenticationMethod: simple
   followReferrals: TRUE
   defaultSearchScope: sub
   searchTimeLimit: 30
   profileTTL: 43200
   cn: default
   credentialLevel: proxy
   bindTimeLimit: 2
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