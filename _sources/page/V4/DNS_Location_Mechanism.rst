DNS_Location_Mechanism
======================

Overview
--------

**Author**: Simo Sorce with help from Petr Spacek, Martin Basti, and
others

Forewords
----------------------------------------------------------------------------------------------

This is a new proposal (Jan 2013) to support Location Based discovery in
FreeIPA. It was inspired by `earlier
proposal <FreeIPAv2:DNS_Location_Discovery>`__ made a while ago. The
main difference is that it simplifies the whole management by
eliminating IP subnets and special client code while still maintaining a
great deal of flexibility. The key insight being that different
locations can configure the network to use different FreeIPA DNS
servers, that are local to the location being considered.

This document describes versions
`2 <#Design_(Version_2:_DNAME_per_sub-tree)>`__ and
`3 <#Design_(Version_3:_CNAME_per_service_name)>`__ of the proposal
which are easier to implement. Version 1 which adds support for
per-client overrides is described in `separate
document <V4/DNS_Location_Mechanism_with_per_client_override>`__. Only
`version 3 <#Design_(Version_3:_CNAME_per_service_name)>`__ will be
implemented at this time. Please see `comparison of
proposals <#Comparison_of_proposals>`__ for further information about
the differences.

Introduction
----------------------------------------------------------------------------------------------

Service Discovery is a process whereby a client uses information stored
in the network to find servers that offer a specific service. It is
useful for two reasons. First, it places the information required for
the location of servers in the network, lessening the burden of client
configuration. Second, it gives system and network administrators a
central point of control that can be used to to define an optimized
policy that determines which clients should use which server and in what
order.

A standard for service discovery is defined in `RFC
2782 <http://www.rfc-archive.org/getrfc.php?rfc=RFC2782>`__. This
standard defines a DNS RR with the mnemonic SRV and usage rules around
it. It allows a client to discover servers that offer a given service
over a given protocol.

A drawback of SRV Records is that it assumes clients know the correct
DNS domain to use as the query target. Without any further information,
the client's options includes using their own DNS domain and the name of
the security domain in which the client operates (such as the domain
name associated to the Kerberos REALM the client refers to). Neither
option is likely to yield optimal results however. One key service
discovery requirement, especially in large distributed enterprises, is
to choose a server that is close to a client. However in many situation
binding the client name to a specific zone that is tied to a physical
location is not possible. And even if it were it would not address the
needs of a roaming client that moves across the networks. We cannot have
the name of a client change when it move across networks as this break
the Kerberos protocol that associates keys to a fixed host name for
Service Principals.

The incompleteness of `RFC
2782 <http://www.rfc-archive.org/getrfc.php?rfc=RFC2782>`__ is
acknowledged by systems such as `Active
Directory <http://en.wikipedia.org/wiki/Active_Directory>`__ that
contain extra functionality to augment SRV lookups to make them site
aware. The basic idea is to use a target in SRV service discovery that
is specific to a location or "site" in AD parlance. Unfortunately AD
clients rely on additional configuration or side protocols to determine
the client "site" and it is quite specific to Microsoft technologies so
the method they use is not easily portable and reusable in other
contexts nor documented in Standards or informative IETF RFCs.

This document specifies a protocol for location discovery based
exclusively on DNS. While the protocol itself can be fully implemented
with a normal DNS the FreeIPA mechanism is augmented by additional
'location' awareness and will depend on additional computation done by
the FreeIPA DNS plugins.

Howto document: `IPA Locations <Howto/IPA_locations>`__

Assumptions
-----------

-  Clients are doing DNS SRV lookup (`RFC
   2782 <http://tools.ietf.org/html/rfc2782>`__) or DNS URI lookup (`RFC
   7553 <http://tools.ietf.org/html/rfc7553>`__) to find out information
   about servers they should connect to.
-  There are sets of servers which are functionally equivalent but
   communication costs to different sets of servers vary.
-  For a client, it is 'cheap' to communicate with servers 'near' the
   client and it is more 'expensive' to communicate with 'remote'
   servers (higher latency, smaller throughput, pay-per-byte etc.).

   -  Special case: Some clients are able to reach only subset of
      servers due to firewall restrictions etc.

-  Clients can roam among various networks so optimal set of servers
   changes from client's perspective.

   -  For this reason only one set of static DNS SRV records cannot
      suffice.
   -  The *DNS Locations* feature will work only if TTL of DNS records
      is shorter than time which clients need to move between two
      locations.

-  At least one of the FreeIPA servers in each location is a DNS enabled
   server.

   -  In principle this might be relaxed to "DNS view per location" but
      then IPA needs integration with DNS views on external servers.

-  Clients in each ``location`` are configured to use a local DNS
   server, which is (or recursively queries or forwards to) a local
   FreeIPA DNS server. In other words, the assumption is that a client
   in a specific location will generally query a FreeIPA server
   (possibly through forwarding or recursion) in the same location for
   location-specific information.

   -  If DNS recursion is involved, we assume that recursor is
      intelligent enough to query *nearest* FreeIPA DNS server. This is
      commonly done by querying servers with shortest round trip time.
      The implicit logic in DNS recursors can be overriden by explicit
      forwarding configuration.
   -  In principle this might be done using "DNS view per location" but
      then IPA needs integration with DNS views on external servers.

-  **To sum up**: 1 ``location`` is defined as set of servers which
   should be prefered by associated DNS clients.

   -  Typically the servers have the same functionality and
      communication costs (as seen from client's perspective).
   -  DNS resolution chain typically looks like
      ``client->(any number of recursive resolvers)->authoritative server (or its view)``.
      All clients which end up querying the same location-specific DNS
      server (or its DNS view if views are in use) are associated with
      the same DNS location and will try to contact the same set of
      servers.



Current use of SRV records
----------------------------------------------------------------------------------------------

Client queries for SRV records located in the domain/realm
(``example.com`` in the following example) and gets back all servers.
Assuming that all DNS servers responsible for given zone are returning a
static set of records, there is no way how to prioritize a server
'nearest' to the client at the moment. Consequenty, all FreeIPA servers
typically have the same priority (0) and weight (100):

::

   ``;; QUESTION SECTION:``
   ``;_ldap._tcp.``\ **``example.com.``**\ `` IN SRV``
   ``;; ANSWER SECTION:``
   ``_ldap._tcp.example.com. SRV ``\ **``0``\ ````\ ``100``**\ `` 389 ipa-brno.example.com.``
   ``_ldap._tcp.example.com. SRV ``\ **``0``\ ````\ ``100``**\ `` 389 ipa-london.example.com.``



Use Cases
---------

-  Roaming clients should connect to set of 'nearest' servers so
   communication is 'cheap'. When none of nearest servers is reachable,
   failover to remote servers might be desired.
-  Some clients might want to connect always to the same set of servers
   for some reason, i.e. server can be pinned to a location.
-  **Future extension - out of scope for FreeIPA 4.4:** The mechanism
   can be re-used for non-IPA services. Generally any service which is
   using ``SRV`` records can be hacked in this way. This might require
   specifying location on per-zone basics.



Feature Management
------------------

Howto document: `IPA Locations <Howto/IPA_locations>`__

Current proposal offers limited configuration capabilities on purpose to
limit user inteface complexity.

One location can contain 1 or more FreeIPA servers and 1 server can be
assigned to at most 1 location. Mechanism generating DNS SRV records
will ensure that clients always prefer servers assigned to location over
all other FreeIPA servers in topology (servers in client's location will
have static priority set to a value higher than all other servers).

Load-balancing among servers in one location is based on
`weight <http://tools.ietf.org/html/rfc2782#page-3>`__ which can be
defined by administrator. By default the load will be equally
distributed among all servers in the location.

When none of servers assigned to particular location can be contacted,
the client will use remaining servers (i.e. servers not assigned to the
particular location) as fallback. These fallback servers always have
smaller priority that all other servers assigned to the location by
administrator so clients should return back to local servers as soon as
they become available. (This depends on particular implementation on the
client side.)

UI

It seems as natural fit to somehow show locations in Topology Graph.
Details TBD

.. figure:: Locations-v2-topology-graph.png
   :alt: Locations-v2-topology-graph.png

   Locations-v2-topology-graph.png

-  Drag & drop servers between locations?
-  How to add/delete/edit locations?

CLI

 =========================================================================== ======================= ======================= =================== 
  +-----------------------+-----------------------+-----------------------+                                                                      
 =========================================================================== ======================= ======================= =================== 
  Command                                                                     Options                 Meaning                                    
  +=======================+=======================+=======================+                                                                      
  location-add                                                                LOCATION_NAME           Add empty IPA                              
                                                                              [--desc=text]           location [with                             
                                                                                                      optional                                   
                                                                                                      description].                              
  +-----------------------+-----------------------+-----------------------+                                                                      
  location-del                                                                LOCATION_NAME           Delete IPA location.                       
                                                                                                      All servers in given                       
                                                                                                      location will stay                         
                                                                                                      unassigned and will                        
                                                                                                      be used only as                            
                                                                                                      backup servers for                         
                                                                                                      other locations.                           
  +-----------------------+-----------------------+-----------------------+                                                                      
  location-find                                                               [SEARCH_TERM]           Get locations with                         
                                                                                                      name or description                        
                                                                                                      matching given                             
                                                                                                      SEARCH_TERM. List all                      
                                                                                                      locations if no                            
                                                                                                      SEARCH_TERM was                            
                                                                                                      specified.                                 
  +-----------------------+-----------------------+-----------------------+                                                                      
  location-show                                                               LOCATION_NAME           Show location name,                        
                                                                                                      description, and list                      
                                                                                                      of all member servers                      
                                                                                                      including their                            
                                                                                                      weights + weights                          
                                                                                                      recalculated to                            
                                                                                                      relative number in                         
                                                                                                      percents. Mark IPA                         
                                                                                                      DNS servers in the                         
                                                                                                      output so it is easy                       
                                                                                                      to see which servers                       
                                                                                                      advertise this                             
                                                                                                      location.                                  
                                                                                                                                                 
                                                                                                                              ``Example:``       
                                                                                                                              descri           
                                                                                                      ption: IPA location                     
                                                                                                                                                 
                                                                                                      dvertised by serve                      
                                                                                                      rs: server1.example                      
                                                                                                                              ``Servers:``       
                                                                                                                              ``  serv           
                                                                                                      er: server1.example``                      
                                                                                                                              ``  weight 100``   
                                                                                                                              ``  re             
                                                                                                      lative weight: 33 %``                      
                                                                                                                              ``  Roles: DN      
                                                                                                      S server, CA server``                      
                                                                                                                              ``  serv           
                                                                                                      er: server2.example``                      
                                                                                                                              ``  weight: 200``  
                                                                                                                              ``  re             
                                                                                                      lative weight: 67 %``                      
                                                                                                                              ``  Role           
                                                                                                      s: CA server, NTP ser                      
                                                                                                      ver, AD trust agent,                       
                                                                                                      AD trust controller``                      
  +-----------------------+-----------------------+-----------------------+                                                                      
  dns-                                                                        [--dry-run]             This command is not                        
  update-system-records                                                                               necessary if IPA DNS                       
                                                                                                      is used and no                             
                                                                                                      hand-tweaking is ever                      
                                                                                                      done by user.                              
                                                                                                      Re-generate all DNS                        
                                                                                                      records. This will be                      
                                                                                                      especially useful if                       
                                                                                                      someone manually                           
                                                                                                      tweaks DNS records in                      
                                                                                                      a wrong way or when                        
                                                                                                      external DNS is used.                      
                                                                                                      Option --dry-run will                      
                                                                                                      print the records                          
                                                                                                      without actually                           
                                                                                                      modifying them.                            
  +-----------------------+-----------------------+-----------------------+                                                                      
  server-mod                                                                  --l                     Add IPA server into                        
                                                                              ocation=LOCATION_NAME   specified location.                        
                                                                              [--loca                 The server will be                         
                                                                              tion-weight=0..65535]   advertised in DNS SRV                      
                                                                                                      records for given                          
                                                                                                      location. One server                       
                                                                                                      can be member of at                        
                                                                                                      most 1 location.                           
                                                                                                                                                 
                                                                                                      All weights in one                         
                                                                                                      location detemine how                      
                                                                                                      requests from clients                      
                                                                                                      are distributed among                      
                                                                                                      IPA servers. Example:                      
                                                                                                      Location has three                         
                                                                                                      servers with weights                       
                                                                                                      50, 25, 25. First                          
                                                                                                      server will receive                        
                                                                                                      50 % of all requests                       
                                                                                                      and second and third                       
                                                                                                      server will receive                        
                                                                                                      25 % requests,                             
                                                                                                      respectively.                              
                                                                                                      Default: 100, i.e.                         
                                                                                                      requests are evenly                        
                                                                                                      distributed among all                      
                                                                                                      servers.                                   
  +-----------------------+-----------------------+-----------------------+                                                                      
  server-show                                                                 FQDN                    Show assigned                              
                                                                                                      location and weight                        
                                                                                                      for particular                             
                                                                                                      server.                                    
  +-----------------------+-----------------------+-----------------------+                                                                      
 =========================================================================== ======================= ======================= =================== 


Notes:

-  server-mod command should print a warning if non-empty location has
   zero advertising (IPA DNS) servers assigned

   -  A warning should be printed if location has less than 2 DNS
      servers: "For redundancy configure at least two advertising DNS
      servers for this location."

Configuration
----------------------------------------------------------------------------------------------

IPA DNS servers will automatically generate distinct DNS SRV and DNAME
records for each location as necessary. To function properly, this
feature depends on optimal routing of DNS queries from clients to
nearest IPA DNS servers.

This auto-configuration depends on three conditions:

-  Number of IPA DNS servers >= number of configured IPA locations
-  All advertising IPA DNS servers are listed in NS records of IPA DNS
   zone
-  Server selection algorithm used by recursors (typically something
   based on round-trip time) selects nearest IPA DNS server which has to
   advertise nearest IPA location for given client

In standard configuration this should work automatically as long as all
IPA DNS servers and their slaves are listed in NS records and recursors
follow `RFC 1035 section
7.2 <http://tools.ietf.org/html/rfc1035#page-44>`__.

Explicit DNS query forwarding overrides normal server selection and can
be used to fine-tune client-to-location assignment (or to
unintentionally break auto-configuration described above).



Design (Version 1: DNAME per client)
------------------------------------

`First version of this
proposal <V4/DNS_Location_Mechanism_with_per_client_override>`__ which
allowed per-client override was split to separate page. This version is
being deferred for now.



Design (Version 2: DNAME per sub-tree)
--------------------------------------

An alternative is to put DNAME redirection onto ``_udp`` and ``_tcp``
(and possibly other) sub-trees of the main domain and redirect these to
location-specific sub-tree.

Similarly to per-client ``_location`` records, this DNAME redirection
can be different on each server.

This should work out of the box.



Interaction with hand-made records
----------------------------------------------------------------------------------------------

Side-effect of DNAME-redirecting ``_udp`` and ``_tcp`` subdomains is
that all original names under these subdomains will become
occluded/invisible to clients (see `RFC 6672 section
2.4 <https://tools.ietf.org/html/rfc6672#section-2.4>`__).

This effectively means that hand-made records in the IPA DNS domain will
become invisible. E.g. following record will disappear when DNS
locations are configured and enabled on IPA domain ``ipa.example``:

``_userservice._udp.ipa.example.  SRV record: 0 100 123 own-server.somewhere.example``

This behavior is in fact necessary for seamless upgrade of replicas
which do not understand the new template LDAP entries in DNS tree. Old
replicas will ignore the template entries and use original sub-tree (and
ignore ``_locations`` sub-tree). New replicas will uderstand the entry,
generate DNAME records and thus occlude old names and use only new ones
(in ``_locations`` sub-tree).

Note: This would be unnecessary if IPA used standard DNS update protocol
against standard DNS server with non-replicated zones because we would
not need to play DNAME tricks. In that case we could instead update
records on each server separately. With current LDAP schema we cannot do
that without adding non-replicated part of LDAP tree to each DNS server.

-  If we added non-replicated sub-tree to each IPA DNS server we would
   have another set of problems because hand-made entries would not be
   replicated among IPA servers.

Handling of hand-made records adds some interesting questions:

-  How to find hand-made records?

   -  Blacklist on name-level or record-data level? What record fields
      should we compare?

-  How to handle collisions with IPA-generated records?

   -  Ignore hand-made records?
   -  Add hand-made records?
   -  Replace IPA generated ones with hand-made ones?

-  What triggers hand-made record synchronization?

   -  Should the user or IPA framework call *ipa
      location-update-records* after each hand-made change to DNS
      records?
   -  How is this synchronization supposed to work with DNS update
      protocol? Currently we do not have means to trigger an action when
      a records is changed in LDAP.

-  How it affects interaction with older IPA DNS servers (see above)?

There are several options:

-  For first version, document that enabling DNS location will hide
   hand-made records in IPA domain.
-  Add non-replicated sub-trees for IPA records and somehow solve
   replication of hand-made records.

   -  What is the proper granularity? Create 20 backends so we can
      filter on name-level?

-  Do 'something' which prevents replication of IPA-generated DNS
   records among servers while still using one LDAP suffix.

   -  With this in place we can mark IPA-generated records as
      non-replicable while still replicating hand-made records as usual.
      (An object class like ``idnsRecordDoNotReplicate``?) This would
      mean that we can drop whole ``_locations`` sub-tree and each
      server will hold only its own copy of DNS records.

-  Find, filter and copy hand-made records from main tree into the
   ``_locations`` sub-trees. This means that every hand-made record
   needs to be copied and synchronized N-times where N = number of IPA
   locations.

Example
----------------------------------------------------------------------------------------------

Clients will query name ``_ldap._tcp.example.com.`` as usual but this
name will be redirected to location-specific sub-tree:

| ``;; QUESTION SECTION:``
| ``;_ldap._tcp.example.com. IN SRV``
| ``;; ANSWER SECTION:``
| ``_tcp.example.com. DNAME _tcp.cz._locations.example.com.``
| ``_ldap._tcp.example.com. CNAME _ldap._tcp.cz._locations.example.com.``
| ``_ldap._tcp.cz._locations.example.com. SRV 0 100 389 ipa-brno.example.com.``
| ``_ldap._tcp.cz._locations.example.com. SRV 3 100 389 ipa-london.example.com.``

.. figure:: ExampleLocationsV2.svg
   :alt: ExampleLocationsV2.svg

   ExampleLocationsV2.svg

-  **(A)** The LDAP database contains records per each location
   ("Y.$LOCATION._location.$SUFFIX") and default records (*Y.$SUFFIX*)
-  **(B)** The DNAME record that overrides the default locations in
   format
   *\_location.$HOSTNAME*\ **DNAME**\ *$LOCATION._locations.$SUFFIX*
-  **(C)** The DNS server in location using *bind-dyndb-ldap* generates
   DNAME records per protocol for IPA domain. A client from location
   **cz** will get SRV records with priority set for this location.
-  **(D)** The DNS server in location using *bind-dyndb-ldap* generates
   DNAME records per protocol for IPA domain. A client from location
   **uk** will get SRV records with priority set for this location.
-  **(E)** Configuration for client2 has been overridden. The client is
   configured to contact location **uk** but DNS server returns results
   for location **cz**. Client has to be configured to ask in format
   **\_service._proto._location.$CLIENT_HOSTNAME** to be overridden
   effective.

-  **[1]** Client wants to connect to the closest LDAP server. (No extra
   configuration is required.)
-  **[2]** Client send DNS query in format *\_ldap._tcp.$SUFFIX* to
   server in its location.
-  **[3]** DNAME records for each protocol for IPA domain has been
   dynamically created on DNS server.
-  **[4]** Server returns DNAME and CNAME (for old clients) records, the
   client has to ask server again to receive SRV records for the name
   returned by DNAME (CNAME).
-  **[5]** Server returns SRV records configured for this location
   (priority for servers located in CZ (Brno))



Compatibility tests
----------------------------------------------------------------------------------------------

-  FreeIPA client installer:

   -  Fedora 23: ``freeipa-client-4.2.3-2.fc23.x86_64`` works
   -  RHEL 5.11: ``ipa-client-2.1.3-7.el5`` works

-  SSSD (ipa provider):

   -  Fedora 23: ``sssd-1.13.1-3.fc23.x86_64`` works
   -  RHEL 5.11: ``sssd-1.5.1-71.el5``

      -  it seems that SSSD has a generic bug which inverts priority in
         DNS SRV records or does something else - the discovery found
         correct servers but the order was weird
      -  discovery works with version 2
      -  ``_srv_``\ discovery does not respect ``dns_discovery_domain``
         option so version 1 will not work

-  nss_ldap (``nss_srv_domain`` option)

   -  RHEL 5.11: ``nss_ldap-253-52.el5_11.2`` works and respects
      priority properly

-  nss-pam-ldapd (``uri DNS`` option)

   -  RHEL 6.7: ``nss-pam-ldapd-0.7.5-20.el6_6.3.x86_64`` works

-  MIT Kerberos libs:

   -  Fedora 23: ``krb5-libs-1.13.2-13.fc23.x86_64``,
      ``krb5-libs-1.14-6.fc23.x86_64`` works
   -  RHEL 5.11: ``krb5-libs-1.6.1-80.el5_11`` works

-  OpenLDAP libs (``-H dc=...`` parameter):

   -  Fedora 23: ``openldap-clients-2.4.40-14.fc23.x86_64`` works
   -  RHEL 5.11: ``openldap-clients-2.3.43-29.el5_11`` does not support
      DNS SRV lookup at all

-  Microsoft Active Directory

   -  Windows Server 2008 R2 with updates released up to 2016-01-29, AD
      functional level 2008 (without R2): works in cross-forest scenario
      and respects SRV priorities - is it a way to cheap DNS sites for
      AD?



Design (Version 3: CNAME per service name)
------------------------------------------

Version 2 poorly integrates with hand-made records which can be
potentially used by users for non-IPA services in primary IPA DNS
domain. Version 3 attempts to mitigate this problem at the expense of
more complex aliasing and record handling in bind-dyndb-ldap and IPA
framework.

IPA will generate ``_locations`` DNS sub-tree in the same way as for
`version 1 <V4/DNS_Location_Mechanism_with_per_client_override>`__ and
`version 2 <#Design_(Version_2:_DNAME_per_sub-tree)>`__.

The main difference in comparison with `version
2 <#Design_(Version_2:_DNAME_per_sub-tree)>`__ is the in way how
redirection from ``_kerberos._udp.$SUFFIX`` to
``_kerberos._udp.$LOCATION._locations.$SUFFIX`` is done.

IPA framework will add a "template" object class and attributes for each
and every DNS name containing managed service records. I.e. a template
will be added to:

-  \_kerberos._udp.$SUFFIX
-  \_kerberos._tcp.$SUFFIX
-  \_ldap._tcp.$SUFFIX
-  \_ldap._tcp.Default-First-Site-Name._sites.dc._msdcs.$SUFFIX
-  ... all of them

The template will generate CNAME redirection from original name to the
location-specific name (which can be different on each DNS server).
Example:

| ``;; QUESTION SECTION:``
| ``;_ldap._tcp.example.com. IN SRV``
| ``;; ANSWER SECTION:``
| ``_ldap._tcp.example.com. CNAME _ldap._tcp.cz._locations.example.com.``
| ``_ldap._tcp.cz._locations.example.com. SRV 0 100 389 ipa-brno.example.com.``
| ``_ldap._tcp.cz._locations.example.com. SRV 3 100 389 ipa-london.example.com.``

Servers which are not assigned to a location (or are too old to
understand the template) will ignore the template and use the original
value in ``*Record`` attributes.

For more information about mechanism generating the records see
`bind-dyndb-ldap design
page <https://fedorahosted.org/bind-dyndb-ldap/wiki/Design/RecordGenerator>`__.



Example
----------------------------------------------------------------------------------------------

.. figure:: ExampleLocationsV3.svg
   :alt: ExampleLocationsV3.svg

   ExampleLocationsV3.svg

-  **(A)** The LDAP database contains records per each location
   ("$Y.$LOCATION._location.$SUFFIX") and default records (*Y.$SUFFIX*)
-  **(B)** The CNAME record that overrides the default locations in
   format *$Y.$SUFFIX*\ **CNAME**\ *$Y.$LOCATION._locations.$SUFFIX*
-  **(C)** The DNS server in location using *bind-dyndb-ldap* generates
   CNAME records per SRV record of IPA service for IPA domain. A client
   from location **cz** will get SRV records with priority set for this
   location.
-  **(D)** The DNS server in location using *bind-dyndb-ldap* generates
   CNAME records per SRV record of IPA service for IPA domain. A client
   from location **uk** will get SRV records with priority set for this
   location.
-  **(E)** Configuration for client2 has been overridden. The client is
   configured to contact location **uk** but DNS server returns results
   for location **cz**. Client has to be configured to ask in format
   **\_service._proto._location.$CLIENT_HOSTNAME** to be overridden
   effective. Also, these records are not automatically generated so
   administrator has to manually configure CNAME record template for
   this client.

-  **[1]** Client wants to connect to the closest LDAP server. (No extra
   configuration is required.)
-  **[2]** Client send DNS query in format *\_ldap._tcp.$SUFFIX* to
   server in its location.
-  **[3]** CNAME records for each protocol for IPA domain has been
   dynamically created on DNS server.
-  **[4]** Server returns CNAME records, the client has to ask server
   again to receive SRV records for the name returned by CNAME.
-  **[5]** Server returns SRV records configured for this location
   (priority for servers located in CZ (Brno))



Interaction with hand-made records
----------------------------------------------------------------------------------------------

Auto-generated CNAME records avoid problem with occluded/invisible
subdomains in ``_udp`` and ``_tcp`` sub-trees.

Hand-made records with names which are not managed by IPA will be
visible as usual because IPA will not add template object class to them.
E.g. following record will stay as is when DNS locations are configured
and enabled on IPA domain ``ipa.example``:

``_userservice._udp.ipa.example.  SRV record: 0 100 123 own-server.somewhere.example.``

This allows the user to use hand-made records as long as they do not
reside under the same DNS name which is managed by IPA. All hand-made
records under IPA-managed names (e.g. ``_kerberos._udp.$SUFFIX``) will
be ignored.

This approach also avoids synchronization problem because hand-made
records do not need to be copied into ``_locations`` sub-tree.

Compatibility
----------------------------------------------------------------------------------------------

Given that client's will see only CNAME, from client's perspective
version 3 should have the same or better properties than `version
2 <#Design_(Version_2:_DNAME_per_sub-tree)>`__. Version 2 used
DNAME+CNAME and worked pretty well so I assume that version 2 should
have the same or better compatibility with clients.



Comparison of proposals
-----------------------

 ======================================================================= ================ ================ ================ 
  +----------------+----------------+----------------+----------------+                                                     
 ======================================================================= ================ ================ ================ 
  Property                                                                v1: `DNAME per   v2: `DNAME per   v3: `CNAME per  
                                                                          client           sub-tree <#      service         
                                                                          <V4/DNS_Locat    Design_(Versio   name <#Desi     
                                                                          ion_Mechanism_   n_2:_DNAME_per   gn_(Version_3:  
                                                                          with_per_clien   _sub-tree)>`__   _CNAME_per_ser  
                                                                          t_override>`__                    vice_name)>`__  
  +================+================+================+================+                                                     
  Requires                                                                yes              no               no              
  client-side                                                                                                               
  support                                                                                                                   
  +----------------+----------------+----------------+----------------+                                                     
  Risk of                                                                 zero             small            small           
  i                                                                                        `1 <#fn1>`__     `2 <#fn2>`__    
  ncompatibility                                                                                                            
  with old                                                                                                                  
  clients                                                                                                                   
  +----------------+----------------+----------------+----------------+                                                     
  Per client                                                              yes              no               no              
  override                                                                                                                  
  +----------------+----------------+----------------+----------------+                                                     
  Works as                                                                no               yes              yes             
  cross-realm                                                             `3 <#fn3>`__     `4 <#fn4>`__     `5 <#fn5>`__    
  optimization                                                                                                              
  +----------------+----------------+----------------+----------------+                                                     
  Implementation                                                          hard             easy             harder than     
  with standard                                                           `6 <#fn6>`__     `7 <#fn7>`__     v2 <#          
  DNS server                                                                                                Design
                                                                                                            but much        
                                                                                                            easier than     
                                                                                                            v1 
                                                                                                            `8 <#fn8>`__    
  +----------------+----------------+----------------+----------------+                                                     
  DNS query                                                               1 extra hop      1 extra hop      1 extra hop     
  overhead                                                                                                                  
  +----------------+----------------+----------------+----------------+                                                     
  DNS zone size                                                           factor ~ 2.3     negligible       negligible\     
  overhead                                                                                 \ `9 <#fn9>`__   `10 <#fn10>`__  
  +----------------+----------------+----------------+----------------+                                                     
  Zone signing                                                            factor ~ 2.3     negligible\      negligible\     
  CPU overhead                                                                             `11 <#fn11>`__   `12 <#fn12>`__  
  +----------------+----------------+----------------+----------------+                                                     
 ======================================================================= ================ ================ ================ 




Comparison with Microsoft Active Directory Sites
----------------------------------------------------------------------------------------------

Some administrators might be familiar with concept of `Active Directory
Sites <https://technet.microsoft.com/en-us/library/cc754697.aspx>`__.
Please note that FreeIPA's *DNS Locations* are different in several
aspects:

-  FreeIPA's replication topology is not affected in any way by *DNS
   Locations*
-  There is no concept of intra-site links between *DNS Locations*
-  Client's location is determined by DNS server used by the client for
   making DNS queries for records in FreeIPA primary DNS domain

   -  All clients using particular DNS server always belong to one *DNS
      Location*

-  In current implementation, there is no way to statically configure a
   client to always use particular location
-  Clients are using standard DNS queries and generally do not need to
   be aware of concept of locations

   -  Consequently, the facility will work with any standard-compliant
      client (please see `#Assumptions <#Assumptions>`__)

One thing is common to AD Sites and FreeIPA DNS Locations:

-  Set of servers assigned to one site (in case of FreeIPA servers with
   highest priority) are assumed to be *optimal* choice for clients
   assigned to that particular site.



Security Considerations
-----------------------

As always DNS replies can be spoofed relatively easily (unless the zone
is DNSSEC signed and records are validated on the client).

We recommend that SRV records resolution is used only for those clients
that normally use an additional security protocol to talk to network
resources and can use additional mechanisms to authenticate these
resources. For example a client that uses an LDAP server for security
related information like user identity information should only trust SRV
record discovery for the LDAP service if LDAPS or STARTTLS over LDAP are
mandatory and certificate verification is fully turned on, or if
SASL/GSSAPI is used with mutual authentication, integrity and
confidentiality options required.

Use of DNSSEC and full DNS signature verification may be considered an
additional requirement in some cases.



Summmary of meeting 2016-02-04
------------------------------

-  Participants: Simo Sorce, Petr Spacek, Martin Basti
-  We will start with per sub-tree approach and deffer `V4/DNS Location
   Mechanism with per client override <per-client_overrides>`__ for now.
-  Keep in mind that bind-dyndb-ldap might get rid of GSSAPI. LDAPI
   mapping to a principal may change results from LDAP whoami.
-  LDAP schema and user interface has to be defined.

   -  We should think about supporting DNS locations per (server & zone)
      so different zones can be assigned to different locations.

Implementation
--------------

The implementation consists of several phases (preferably in this
order):

-  Add `per-IPA DNS server configuration
   capabilities <https://fedorahosted.org/bind-dyndb-ldap/wiki/Design/PerServerConfigInLDAP>`__
   to bind-dyndb-ldap
-  Add `Per bind-dyndb-ldap instance record
   generation <https://fedorahosted.org/bind-dyndb-ldap/wiki/Design/RecordGenerator>`__
-  Cleanup and unification of DNS record generators in FreeIPA framework
   and installers
-  Add location management capabilities to FreeIPA (location-\*
   commands)
-  Combine new record generators in FreeIPA framework with locations
-  Add support for default TTL value into bind-dyndb-ldap and FreeIPA
   (so roaming clients are not stuck with cached records)
-  Add management UI for per-DNS server configuration (to make it more
   manageable)



DNS server configuration
----------------------------------------------------------------------------------------------

This FreeIPA feature depends on two sub-features of bind-dyndb-ldap:

-  `Per-server configuration in
   LDAP <https://fedorahosted.org/bind-dyndb-ldap/wiki/Design/PerServerConfigInLDAP>`__
-  `Per bind-dyndb-ldap instance record
   generation <https://fedorahosted.org/bind-dyndb-ldap/wiki/Design/RecordGenerator>`__

Both features contain a new attributes and related access controls. For
details please see separate pages.

When a FreeIPA server is assigned to a location (which will be
advertised to clients), the DNS name of the location will be put into
``idnsSubstitutionVariable;ipaLocation`` attribute in
``idnsServerConfigObject`` representing the DNS server.



CNAME data generation
----------------------------------------------------------------------------------------------

FreeIPA must create ``idnsTemplateObject`` at all SRV records belongs to
IPA services in FreeIPA primary DNS domain.

All these objects need to contain attribute
``idnsTemplateAttribute;CNAMERecord`` which will instruct
bind-dyndb-ldap to generate the CNAME records for the particular
location.



Example
^^^^^^^

Following example will instruct bind-dyndb-ldap to generate
``CNAMERecord`` attribute with value constructed from prefix ``_udp.``,
user-defined variable ``ipalocation``, and suffix ``._locations``.

| ``dn: idnsName=_ldap._tcp,idnsname=example.com.,cn=dns,dc=example,dc=com``
| ``objectClass: idnsTemplateObject``
| ``objectClass: top``
| ``objectClass: idnsRecord``
| ``idnsName: _ldap._tcp``
| ``srvrecord: 0 100 389 ipa.example.com.``
| ``idnsTemplateAttribute;cnamerecord: _ldap._tcp.\{substitutionvariable_ipalocation\}._locations``



Records generated for IPA services
----------------------------------------------------------------------------------------------

**One in IPA domain:**

``_kerberos TXT {IPA_REALM}``

**For each IPA master:**

| ``_ldap._tcp SRV 0 100 389 {hostname}``
| ``_kerberos._tcp SRV 0 100 88 {hostname}``
| ``_kerberos._udp SRV 0 100 88 {hostname}``
| ``_kerberos-master._tcp SRV 0 100 88 {hostname}``
| ``_kerberos-master._udp SRV 0 100 88 {hostname}``
| ``_kpasswd._tcp SRV 0 100 464 {hostname}``
| ``_kpasswd._udp SRV 0 100 464 {hostname}``

**For each IPA CA server:**

| ``ipa-ca A {ipv4 address of server}``
| ``ipa-ca AAAA {ipv6 address of server}``

**For each IPA NTP server:**

``_ntp._udp SRV 0 100 123 {hostname}``

**For each ADTrust controller**:

| ``_ldap._tcp.Default-First-Site-Name._sites.dc._msdcs SRV  0 100 389 {hostname}``
| ``_ldap._tcp.dc._msdcs SRV 0 100 389 {hostname}``
| ``_kerberos._tcp.Default-First-Site-Name._sites.dc._msdcs SRV 0 100 88 {hostname}``
| ``_kerberos._udp.Default-First-Site-Name._sites.dc._msdcs SRV 0 100 88 {hostname}``
| ``_kerberos._tcp.dc._msdcs SRV 0 100 88 {hostname}``
| ``_kerberos._udp.dc._msdcs SRV 0 100 88 {hostname}``



Location data generation
----------------------------------------------------------------------------------------------

We have to modify FreeIPA Python code responsible for generating DNS
records in installers etc. so FreeIPA is able to automatically generate
DNS records for each possible combination (service, location).

Preferably, there should be a standardized way for a service to yield
set of records which should be placed into the DNS so this set of
records can be further transformed and either placed into FreeIPA DNS or
sent as update to an external DNS server.

This will require refactoring described in FreeIPA ticket `#5620:
Centralize DNS record creation in IPA
services <https://fedorahosted.org/freeipa/ticket/5620>`__.

The record generator will be executed for the FreeIPA primary DNS domain
and then again with modified priority and weight for each DNS location.



LDAP Data structure
----------------------------------------------------------------------------------------------



Objectclasses and attributes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Bind-dyndb-related (`Record
generator <https://fedorahosted.org/bind-dyndb-ldap/wiki/Design/RecordGenerator>`__,
`Per server config in
LDAP <https://fedorahosted.org/bind-dyndb-ldap/wiki/Design/PerServerConfigInLDAP>`__),
all located in DNS subtree

``attributeTypes: ( 2.16.840.1.113730.3.8.5.31 NAME 'idnsServerId' DESC 'DNS server identifier' EQUALITY caseIgnoreMatch SINGLE-VALUE SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 X-ORIGIN 'IPA v4.4' )``

``attributeTypes: ( 2.16.840.1.113730.3.8.5.30 NAME 'idnsSubstitutionVariable' DESC 'User defined variable for DNS plugin' EQUALITY caseIgnoreIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 X-ORIGIN 'IPA v4.4' )``

``objectClasses: ( 2.16.840.1.113730.3.8.6.6 NAME 'idnsServerConfigObject' DESC 'DNS server configuration options' STRUCTURAL MUST ( idnsServerId ) MAY ( idnsSubstitutionVariable $ idnsSOAmName $ idnsForwarders $ idnsForwardPolicy ) X-ORIGIN 'IPA v4.4' )``

``attributeTypes: ( 2.16.840.1.113730.3.8.5.29 NAME 'idnsTemplateAttribute' DESC 'Template attribute for dynamic attribute generation' EQUALITY caseIgnoreIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 X-ORIGIN 'IPA v4.4' )``

``objectClasses: ( 2.16.840.1.113730.3.8.6.5 NAME 'idnsTemplateObject' DESC 'Template object for dynamic DNS attribute generation' AUXILIARY MUST ( idnsTemplateAttribute ) X-ORIGIN 'IPA v4.4' )``

IPA locations part, in cn=etc subtree:

``attributeTypes: ( 2.16.840.1.113730.3.8.5.32 NAME 'ipaLocation' DESC 'Reference to IPA location' EQUALITY distinguishedNameMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 SINGLE-VALUE X-ORIGIN 'IPA v4.4' )``

``attributeTypes: ( 2.16.840.1.113730.3.8.5.33 NAME 'ipaLocationWeight' DESC 'Weight for the server in IPA location' EQUALITY integerMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.27 SINGLE-VALUE X-ORIGIN 'IPA v4.4' )``

``objectClasses: (  2.16.840.1.113730.3.8.6.7 NAME 'ipaLocationObject' DESC 'Object for storing IPA server location' STRUCTURAL MUST ( idnsName ) MAY ( description ) X-ORIGIN 'IPA v4.4' )``

``objectClasses: (  2.16.840.1.113730.3.8.6.8 NAME 'ipaLocationMember' DESC 'Member object of IPA location' AUXILIARY MAY ( ipaLocation $ ipaLocationWeight ) X-ORIGIN 'IPA v4.4' )``



Locations LDAP structure
^^^^^^^^^^^^^^^^^^^^^^^^

| ``DN: cn=locations, cn=etc, $SUFFIX``
| ``objectlcass: nsContainer``
| ``cn: locations``

| ``DN: idnsName=prague, cn=locations, cn=etc, $SUFFIX``
| ``objectclass: ipaLocationObject``
| ``idnsName: prague``
| ``description: Servers in Prague area``



Servers LDAP structure
^^^^^^^^^^^^^^^^^^^^^^

| ``DN: cn=ipa-server.example.com,cn=masters,cn=ipa,cn=etc, $SUFFIX``
| ``objectclass: top``
| ``objectclass: nsContainer``
| ``objectclass: ipaSupportedDomainLevelConfig``
| ``objectclass: ipaReplTopoManagedService``
| **``objectclass:``\ ````\ ``ipaLocationMember``**
| ``cn: ipa-server.example.com``
| ``ipaMaxDomainLevel: 1``
| ``ipaMinDomainLevel: 0``
| ``ipaReplTopoManagedSuffix: o=ipaca``
| ``ipaReplTopoManagedSuffix: $SUFFIX``
| **``ipaLocation:``\ ````\ ``idnsName=prague,cn=locations,cn=etc,$SUFFIX``**
| **``ipaLocationWeight:``\ ````\ ``100``**



IPA commands affected by this feature
----------------------------------------------------------------------------------------------

When following commands are executed, resulting of that commands might
result into a need to update location records



server-del
^^^^^^^^^^

system records should be updated



server-mod
^^^^^^^^^^

system records should be updated only if *location* or *weight* have
been changed



ipa-replica-manage del
^^^^^^^^^^^^^^^^^^^^^^

system records should be updated



ipa-server-install
^^^^^^^^^^^^^^^^^^

system records should be updated



ipa-replica-install
^^^^^^^^^^^^^^^^^^^

system records should be updated



location-add
^^^^^^^^^^^^

TBD



location-del
^^^^^^^^^^^^

system records should be updated, unused location records should be
removed



permissions and privileges
----------------------------------------------------------------------------------------------

*DNS Administrator* privilege must have read and write access to
locations

*DNS Servers* privilege must have read access to new container in cn=DNS
subtree



New permissions
^^^^^^^^^^^^^^^

-  **System: Read IPA Locations**: allows to read locations in location
   container
-  **System: Add IPA Locations**: allows to add new locations into
   locations container
-  **System: Remove IPA Locations**: allows to remove location from
   locations container
-  **System: Modify IPA Locations**: allows to modify locations in
   location container (just description)
-  **System: Read Locations of IPA Servers**: allows to read assigned
   location to server in masters container and weight attribute of
   server
-  **System: Read Server Roles**: allows to read which roles belong to
   IPA servers (this is needed for proper generation of DNS SRV records)

Upgrade
-------

-  bind-dyndb-ldap's feature `Per-server configuration in
   LDAP <https://fedorahosted.org/bind-dyndb-ldap/wiki/Design/PerServerConfigInLDAP>`__
   describe which options from ``named.conf`` should be migrated to LDAP
   tree during upgrade
-  For each upgraded server, ``cn=DNS`` entry in ``cn=masters`` should
   be extended with ``ipaConfigString`` = ``dnsLocationsVersion 1``
   which will make it easy to check if particular server supports
   locations or not.



How to Test
-----------

-  Install at least two IPA DNS servers
-  Create at least two locations:

| ``ipa location-add loc1``
| ``ipa location-add loc2``

-  Assign one or more FreeIPA servers to each location
-  Assign first FreeIPA DNS server to one location (must have non-empty
   set of servers)

``ipa server-mod server1.example --location=loc1``

-  Assign second FreeIPA DNS server to second location (must have
   non-empty set of servers)

``ipa server-mod server2.example --location=loc2``

-  Query SRV records from the first FreeIPA DNS server:

``$ dig @$FIRST_IPA_DNS_SERVER _kerberos._udp.$PRIMARYDNSDOMAIN SRV``

The answer must contain FreeIPA server assigned to first location with
higher priority (smaller number) and the second server must have lower
priority (higher number).



Test Plan
---------

`DNS Location Mechanism V4.4 test
plan <V4/DNS_Location_Mechanism/Test_Plan>`__

References
----------

SRV Records: `RFC
2782 <http://www.rfc-archive.org/getrfc.php?rfc=RFC2782>`__

DNAME Records: `RFC
6672 <http://www.rfc-archive.org/getrfc.php?rfc=RFC6672>`__

--------------

#. Some clients can be theoretically confused when ordinary query for
   ``_ldap._tcp`` returns ``CNAME`` pointing to a location sub-tree.
   `#Compatibility_tests <#Compatibility_tests>`__ suggest that this
   should be rare.\ `↩︎ <#fnref1>`__

#. ``_location.$HOSTNAME`` domain can contain only ``SRV`` records for
   client's realm. Consequently, clients which only query for
   ``_location.$HOSTNAME`` does not have a way to find tailored ``SRV``
   records from other realms.\ `↩︎ <#fnref3>`__

#. Clients from different realms will obtain tailored ``SRV`` records
   from "nearest" DNS server. This was tested with Microsoft AD 2008 R2,
   see
   `#Compatibility_tests <#Compatibility_tests>`__.\ `↩︎ <#fnref4>`__

#. Per client approach requires some mechanism which creates ``DNAME``
   record for every new ``A/AAAA`` record created on the server. This
   does not sound as an easy task with a general-purpose DNS
   server.\ `↩︎ <#fnref6>`__

#. General-purpose DNS server can be manually configured with ``DNAME``
   records for sub-trees. Alternativelly these records can be
   dynamically updated by IPA framework.\ `↩︎ <#fnref7>`__

#. General-purpose DNS server can be manually configured with ``CNAME``
   records for each service name. Alternativelly these records can be
   dynamically updated by IPA framework.\ `↩︎ <#fnref8>`__

#. 2 DNAME records per zone\ `↩︎ <#fnref9>`__

#. Each service name requires one CNAME\ `↩︎ <#fnref10>`__