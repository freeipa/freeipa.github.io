.. _sample_non_tls_profile_for_hpux:

Sample Non TLS Profile for HPUX
===============================

Back to `Configuring UNIX Clients <ConfiguringUnixClients>`__

::

   #*************************************
   # Sample Non TLS Profile File for HPUX
   #*************************************
   version: 1
   dn: cn=ldapuxprofile-tls,ou=profile,dc=example,dc=com
   serviceSearchDescriptor: passwd:dc=example,dc=com?sub?(objectclass=posixaccount)
   serviceSearchDescriptor: shadow:dc=example,dc=com?sub?(objectclass=shadowaccount)
   serviceSearchDescriptor: group:dc=example,dc=com?sub?(objectclass=posixgroup)
   serviceSearchDescriptor: pam:dc=example,dc=com?sub?(objectclass=posixaccount)
   serviceSearchDescriptor: rpc:dc=example,dc=com?sub?(objectclass=oncrpc)
   serviceSearchDescriptor: protocols:dc=example,dc=com?sub?(objectclass=ipprotocol)
   serviceSearchDescriptor: networks:dc=example,dc=com?sub?(objectclass=ipnetwork)
   serviceSearchDescriptor: hosts:dc=example,dc=com?sub?(objectclass=iphost)
   serviceSearchDescriptor: services:dc=example,dc=com?sub?(objectclass=ipservice)
   serviceSearchDescriptor: netgroup:dc=example,dc=com?sub?(objectclass=nisnetgroup)
   serviceSearchDescriptor: printers:dc=example,dc=com?sub?(objectclass=printerlpr)
   serviceSearchDescriptor: automount:dc=example,dc=com?sub?(objectclass=automount)
   attributeMap: passwd:userpassword=*NULL*
   attributeMap: shadow:userpassword=*NULL*
   credentialLevel: proxy anonymous
   authenticationMethod: tls:simple
   bindTimeLimit: 5
   defaultSearchBase: dc=example,dc=com
   preferredServerList: ldap01.example.com
   objectClass: top
   objectClass: duaconfigprofile
   cn: ldapuxprofile-tls
