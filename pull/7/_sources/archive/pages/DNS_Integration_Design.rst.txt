Summary
-------

The section `Machine
Identity <V2BPRD#1._Machine_Identity_and_Authentication>`__ in the
`IPAv2 PRD <V2BPRD>`__ requires us to be able allow a machine to
dynamically update its DNS record. This is pretty easy, but in order to
do it in a secure manner, we have to ensure that a machine can only
manipulate it's own record. For this, we have to establish the machines
identity via kerberos. It is possible to use kerberos to authenticate
the machine with DNS server using
`GSS-TSIG <http://en.wikipedia.org/wiki/GSS-TSIG>`__ (RFC 3645). With
the authenticity established, we will only allow the machine to edit
it's own DNS record. The changes will then be stored in LDAP.

.. _dynamic_updates_with_gss_tsig:

Dynamic updates with GSS-TSIG
-----------------------------

GSS-TSIG (Generic Security Service Algorithm for Secret Key Transaction)
is an authentication protocol for DNS, which is the extension to TSIG
Protocol. The GSS-TSIG is a framework of GSS-API to provide
authentication, integrity and confidentiality. It is defined in RFC
3645.

.. _support_in_dnspython:

Support in dnspython
~~~~~~~~~~~~~~~~~~~~

The `dnspython <http://www.dnspython.org/>`__ library doesn't have
support for GSS-TSIG. As a workaround, we could just make a simple
wrapper for the nsupdate utility, but in the long-term, we should make a
patch and work with upstream community on its adoption.

.. _storing_dns_records_in_ldap:

Storing DNS records in LDAP
---------------------------

Problem with BIND and LDAP is that there is currently no way of both
reading and writing. There are two database interfaces, SDB and DLZ. SDB
reads the data when BIND starts and holds them in cache. DLZ will look
into the database with each query it gets. There are numerous drivers
for each of these, LDAP included. Our choice is DLZ. The current LDAP
driver for DLZ does not suit our need, mostly because of the schema it
uses. We will need to write our own driver after we implement the
write-back support for DLZ. We will design our own schema, inspired by
the one SDB uses. Currently, all DLZ drivers are statically linked with
BIND. We will patch BIND in such a way that these drivers can be loaded
dynamically. This will allow us better control of schema we use and we
will ship the driver ourselves.

.. _ldap_driver:

LDAP driver
~~~~~~~~~~~

Here are some notes and things we should keep in mind when writing the
driver.

-  LDAP driver should make as little ldap queries as possible, ldapi://
   would be very handy.

-  The Serial information from the SOA record needs to be automatically
   generated whenever a record in the zone is changed. This would only
   be important for admins that run slaves for caching. Date would
   probably be best to use as a value. LDAP driver could update this
   with each change, but this might be slow, since we need to do
   additional query. Other way of solving this would be to have a DS
   plugin.

-  Every host is represented by one object, which can contain multiple
   DNS records. However, we should be prepared for multiple objects so
   that we allow different TTLs. It's not really important to also
   support writing such records, just to be able to handle them.

-  The LDAP driver should update the PTR record (if the zone exists and
   updates to the zone hosting the PTR are allowed). When A DNS Update
   is received first an update of the A(AAAA/A6) Record is performed. If
   that is successful, then a search for PTRRecord= is performed. If the
   record is found and the IP address matches the record name, no
   further action is performed. If the IP address does not match and is
   of the same type as the update being performed (A/A6/AAAA) then it is
   deleted. Then a search for the provided IP address is performed to
   find a record, if is it found and it does not match it is updated.
   Finally if a PTR record is not found, a new one is created. WARNING:
   automatic PTR address removal might not be possible. Depending on how
   BIND is built internally we may not be able to know if a client is
   sending multiple IP addresses (multihomed system) so removing "older"
   PTR records may simply cause us to remove a PTR record we just set.

-  Careful with private IPv4 address. We need to explore if the driver
   has information about the ip address of the client that sent an
   update. In this case then we would never set the PTR record based on
   the sent ip address but only based on the ip address of the dns
   packet. This way there is no risk for client in a remote network
   using the VPN to try to update the wrong IP address (stupid private
   IP address spaces can cause problems). For IPV6 there is no such
   problem, so for A6/AAAA records we use the client provided
   information.

Schema
~~~~~~

This schema is based on the `SDB
Schema <http://www.venaas.no/ldap/bind-sdb/dnszone-schema.txt>`__.

| ``attributetype ( 1.3.6.1.4.1.2428.20.0.0``
| ``   NAME 'dNSTTL'``
| ``   DESC 'An integer denoting time to live'``
| ``   EQUALITY integerMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.27``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.0.1``
| ``   NAME 'dNSClass'``
| ``   DESC 'The class of a resource record'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.0.2``
| ``   NAME 'zoneName'``
| ``   DESC 'The name of a zone, i.e. the name of the highest node in the zone'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.0.3``
| ``   NAME 'relativeDomainName'``
| ``   DESC 'The starting labels of a domain name'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.12``
| ``   NAME 'pTRRecord'``
| ``   DESC 'domain name pointer, RFC 1035'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.13``
| ``   NAME 'hInfoRecord'``
| ``   DESC 'host information, RFC 1035'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.14``
| ``   NAME 'mInfoRecord'``
| ``   DESC 'mailbox or mail list information, RFC 1035'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.16``
| ``   NAME 'tXTRecord'``
| ``   DESC 'text string, RFC 1035'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.18``
| ``   NAME 'aFSDBRecord'``
| ``   DESC 'for AFS Data Base location, RFC 1183'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.24``
| ``   NAME 'SigRecord'``
| ``   DESC 'Signature, RFC 2535'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.25``
| ``   NAME 'KeyRecord'``
| ``   DESC 'Key, RFC 2535'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.28``
| ``   NAME 'aAAARecord'``
| ``   DESC 'IPv6 address, RFC 1886'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.29``
| ``   NAME 'LocRecord'``
| ``   DESC 'Location, RFC 1876'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.30``
| ``   NAME 'nXTRecord'``
| ``   DESC 'non-existant, RFC 2535'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.33``
| ``   NAME 'sRVRecord'``
| ``   DESC 'service location, RFC 2782'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.35``
| ``   NAME 'nAPTRRecord'``
| ``   DESC 'Naming Authority Pointer, RFC 2915'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.36``
| ``   NAME 'kXRecord'``
| ``   DESC 'Key Exchange Delegation, RFC 2230'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.37``
| ``   NAME 'certRecord'``
| ``   DESC 'certificate, RFC 2538'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.38``
| ``   NAME 'a6Record'``
| ``   DESC 'A6 Record Type, RFC 2874'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.39``
| ``   NAME 'dNameRecord'``
| ``   DESC 'Non-Terminal DNS Name Redirection, RFC 2672'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.43``
| ``   NAME 'dSRecord'``
| ``   DESC 'Delegation Signer, RFC 3658'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.44``
| ``   NAME 'sSHFPRecord'``
| ``   DESC 'SSH Key Fingerprint, draft-ietf-secsh-dns-05.txt'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.46``
| ``   NAME 'rRSIGRecord'``
| ``   DESC 'RRSIG, RFC 3755'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 1.3.6.1.4.1.2428.20.1.47``
| ``   NAME 'nSECRecord'``
| ``   DESC 'NSEC, RFC 3755'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``)``
| ``attributetype ( 2.16.840.1.113730.3.8.3.0``
| ``   NAME 'idnsName'``
| ``   DESC 'DNS FQDN'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``   SINGLE-VALUE``
| ``)``
| ``attributetype ( 2.16.840.1.113730.3.8.3.1``
| ``   NAME 'idnsAllowDynUpdate'``
| ``   DESC 'permit dynamic updates on this zone'``
| ``   EQUALITY booleanMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| ``   SINGLE-VALUE``
| ``)``
| ``attributetype ( 2.16.840.1.113730.3.8.3.2``
| ``   NAME 'idnsZoneActive'``
| ``   DESC 'define if the zone is considered in use'``
| ``   EQUALITY booleanMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| ``   SINGLE-VALUE``
| ``)``
| ``attributetype ( 2.16.840.1.113730.3.8.3.3``
| ``   NAME 'idnsSOAmName'``
| ``   DESC 'SOA Name'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``   SINGLE-VALUE``
| ``)``
| ``attributetype ( 2.16.840.1.113730.3.8.3.4``
| ``   NAME 'idnsSOArName'``
| ``   DESC 'SOA root Name'``
| ``   EQUALITY caseIgnoreIA5Match``
| ``   SUBSTR caseIgnoreIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26``
| ``   SINGLE-VALUE``
| ``)``
| ``attributetype ( 2.16.840.1.113730.3.8.3.5``
| ``   NAME 'idnsSOAserial'``
| ``   DESC 'SOA serial number'``
| ``   EQUALITY numericStringMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.36``
| ``   SINGLE-VALUE``
| ``)``
| ``attributetype ( 2.16.840.1.113730.3.8.3.6``
| ``   NAME 'idnsSOArefresh'``
| ``   DESC 'SOA refresh value'``
| ``   EQUALITY numericStringMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.36``
| ``   SINGLE-VALUE``
| ``)``
| ``attributetype ( 2.16.840.1.113730.3.8.3.7``
| ``   NAME 'idnsSOAretry'``
| ``   DESC 'SOA retry value'``
| ``   EQUALITY numericStringMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.36``
| ``   SINGLE-VALUE``
| ``)``
| ``attributetype ( 2.16.840.1.113730.3.8.3.8``
| ``   NAME 'idnsSOAexpire'``
| ``   DESC 'SOA expire value'``
| ``   EQUALITY numericStringMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.36``
| ``   SINGLE-VALUE``
| ``)``
| ``attributetype ( 2.16.840.1.113730.3.8.3.9``
| ``   NAME 'idnsSOAminimum'``
| ``   DESC 'SOA minimum value'``
| ``   EQUALITY numericStringMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.36``
| ``   SINGLE-VALUE``
| ``)``
| ``objectclass ( 2.16.840.1.113730.3.8.4.0``
| ``   NAME 'idnsRecord'``
| ``   DESC 'dns Record, usually a host'``
| ``   SUP top``
| ``   STRUCTURAL``
| ``   MUST idnsName``
| ``   MAY ( cn $ idnsAllowDynUpdate $ DNSTTL $ DNSClass $ ARecord $``
| ``       AAAARecord $ A6Record $ NSRecord $ CNAMERecord $ PTRRecord $``
| ``       SRVRecord $ TXTRecord $ MXRecord $ MDRecord $ HINFORecord $``
| ``       MINFORecord $ AFSDBRecord $ SIGRecord $ KEYRecord $ LOCRecord $``
| ``       NXTRecord $ NAPTRRecord $ KXRecord $ CERTRecord $ DNAMERecord $``
| ``       DSRecord $ SSHFPRecord $ RRSIGRecord $ NSECRecord``
| ``   )``
| ``)``
| ``objectclass ( 2.16.840.1.113730.3.8.4.1``
| ``   NAME 'idnsZone'``
| ``   DESC 'Zone class'``
| ``   SUP idnsRecord``
| ``   STRUCTURAL``
| ``   MUST ( idnsName $ idnsZoneActive $ idnsSOAmName $ idnsSOArName $``
| ``       idnsSOAserial $ idnsSOArefresh $ idnsSOAretry $ idnsSOAexpire $``
| ``       idnsSOAminimum``
| ``   )``
| ``)``

.. _see_also:

See also
--------

-  `Dynamic updates with
   GSS-TSIG <FreeIPAv2:Dynamic_updates_with_GSS-TSIG>`__

.. _useful_links:

Useful links
------------

-  RFC 3645 GSS-TSIG Generic Security Service Algorithm for Secret Key
   Transaction Authentication for DNS (GSS-TSIG)
-  http://directory.fedoraproject.org/wiki/Howto:BIND Some tips on how
   to integrate BIND with Fedora DS
-  http://www.blue-giraffe.com/zone2ldap/ zone2ldap utility, writes DNS
   records from flat files to LDAP
-  http://projects.alkaloid.net/e107_plugins/content/content.php?content.5
   ldap2dns utility, converts DNS records from LDAP to flat files
