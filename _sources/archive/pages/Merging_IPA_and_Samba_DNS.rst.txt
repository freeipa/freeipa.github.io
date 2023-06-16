Overview
========

IPA and Samba use DNS records to inform the clients about their
services. Some of these records use the same name, but they can be
merged because they are pointing to the same service (e.g. Kerberos
service). However, there are some records that use the same name but
pointing to different services (e.g. IPA LDAP service vs. Samba LDAP
service).

.. _ipa_dns_records:

IPA DNS Records
===============

::

   _ldap._tcp              IN SRV 0 100 389        ipa

   _kerberos               IN TXT EXAMPLE.COM

   _kerberos._tcp          IN SRV 0 100 88         ipa
   _kerberos._udp          IN SRV 0 100 88         ipa
   _kerberos-master._tcp   IN SRV 0 100 88         ipa
   _kerberos-master._udp   IN SRV 0 100 88         ipa
   _kpasswd._tcp           IN SRV 0 100 464        ipa
   _kpasswd._udp           IN SRV 0 100 464        ipa

   _ntp._udp               IN SRV 0 100 123        ipa

.. _samba_dns_records:

Samba DNS Records
=================

::

   gc._msdcs               IN CNAME        samba
   27f515e4-f5af-4396-bc93-130013076ab7._msdcs     IN CNAME        samba

   _gc._tcp                IN SRV 0 100 3268       samba
   _gc._tcp.Default-First-Site-Name._sites IN SRV 0 100 3268       samba
   _ldap._tcp.gc._msdcs    IN SRV 0 100 389        samba
   _ldap._tcp.Default-First-Site-Name._sites.gc._msdcs     IN SRV 0 100 389 samba

   _ldap._tcp              IN SRV 0 100 389        samba
   _ldap._tcp.dc._msdcs    IN SRV 0 100 389        samba
   _ldap._tcp.pdc._msdcs   IN SRV 0 100 389        samba
   _ldap._tcp.b168ccf1-d862-4146-8cea-0021f3c88feb IN SRV 0 100 389        samba
   _ldap._tcp.b168ccf1-d862-4146-8cea-0021f3c88feb.domains._msdcs          IN SRV 0 100 389 samba
   _ldap._tcp.Default-First-Site-Name._sites               IN SRV 0 100 389 samba
   _ldap._tcp.Default-First-Site-Name._sites.dc._msdcs     IN SRV 0 100 389 samba

   _kerberos._tcp          IN SRV 0 100 88         samba
   _kerberos._tcp.dc._msdcs        IN SRV 0 100 88 samba
   _kerberos._tcp.Default-First-Site-Name._sites   IN SRV 0 100 88 samba
   _kerberos._tcp.Default-First-Site-Name._sites.dc._msdcs IN SRV 0 100 88 samba
   _kerberos._udp          IN SRV 0 100 88         samba

   _kerberos-master._tcp           IN SRV 0 100 88         samba
   _kerberos-master._udp           IN SRV 0 100 88         samba

   _kpasswd._tcp           IN SRV 0 100 464        samba
   _kpasswd._udp           IN SRV 0 100 464        samba

   _kerberos               IN TXT  EXAMPLE.COM

.. _proposed_solution:

Proposed Solution
=================

IPA's and Samba's LDAP services will be bound to different virtual
hostnames. The DNS record for IPA's LDAP service will be renamed and
point to IPA's virtual hostname. The DNS record for Samba's LDAP service
will remain the same and point to Samba's virtual hostname. The IPA
clients will be modified to look for the new DNS record. Samba clients
will continue to use the existing DNS record.

::

   _ldap._tcp._ipa         IN SRV 0 100 389        ipa
   _ldap._tcp              IN SRV 0 100 389        samba

There will be only 1 Kerberos service with could be accessed using IPA's
and Samba's virtual hostname. Both IPA and Samba clients will use the
same KDC.

::

   _kerberos               IN TXT EXAMPLE.COM

   _kerberos._tcp          IN SRV 0 100 88         ipa
   _kerberos._tcp.dc._msdcs        IN SRV 0 100 88 ipa
   _kerberos._tcp.Default-First-Site-Name._sites   IN SRV 0 100 88 ipa
   _kerberos._tcp.Default-First-Site-Name._sites.dc._msdcs IN SRV 0 100 88 ipa
   _kerberos._udp          IN SRV 0 100 88         ipa

   _kerberos-master._tcp   IN SRV 0 100 88         ipa
   _kerberos-master._udp   IN SRV 0 100 88         ipa

   _kpasswd._tcp           IN SRV 0 100 464        ipa
   _kpasswd._udp           IN SRV 0 100 464        ipa

`Category:Obsolete <Category:Obsolete>`__
