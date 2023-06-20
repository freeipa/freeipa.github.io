DNS_Location_Mechanism_with_per_client_override
===============================================

Overview
--------

**Author**: Simo Sorce with help from Petr Spacek, Martin Basti, and
others

Forewords
----------------------------------------------------------------------------------------------

This is a new proposal (Jan 2013) to support Location Based discovery in
FreeIPA. It was inspired by `this <FreeIPAv2:DNS_Location_Discovery>`__
earlier proposal made a while ago. The main difference is that it
simplifies the whole management by eliminating IP subnets and special
client code while still maintaining a great deal of flexibility. The key
insight being that different locations can configure the network to use
different FreeIPA DNS servers, that are local to the location being
considered.

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
'location' awarness and will depend on additional computation done by
the FreeIPA DNS plugins.

Assumptions
-----------

-  Clients are doing DNS SRV lookup specified in `RFC
   2782 <http://tools.ietf.org/html/rfc2782>`__ to find out information
   about servers they should connect to.
-  There are sets of servers which are functionally equivalent but
   communication costs to different sets of servers vary.
-  For a client, it is 'cheap' to communicate with servers 'near' the
   client and it is more 'expensive' to communicate with 'remote'
   servers (higher latency, smaller throughput, pay-per-byte etc.).

   -  Special case: Some clients are able to reach only subset of
      servers due to firewall restrictions etc.

-  Clients can roam among various networks so optimal set of servers
   changes from client's perspective. For this reason only one set of
   static DNS SRV records cannot suffice.

   -  The *DNS Locations* feature will work only if TTL of DNS records
      is shorter than time which clients need to move between two
      locations.

-  Each ``location`` corresponds to a different network.
-  At least one of the FreeIPA servers in each location is a DNS enabled
   server.

   -  In principle this might be relaxed to "DNS view per location" but
      then IPA needs integration with DNS views on external servers.

-  Clients in each ``location`` are configured to use local DNS server,
   which is (or forwards to) a local FreeIPA DNS server. In other words,
   the assumption is that a client in a specific location will generally
   query a FreeIPA server (possibly through forwarding or recursion) in
   the same location for location-specific information.

   -  If DNS recursion is involved, we assume that recursor is
      intelligent enough to query *nearest* FreeIPA DNS server. This is
      commonly done by querying servers with shortest round trip time.
      The implicit logic in DNS recursors can be overridden by explicit
      forwarding configuration.
   -  In principle this might be done using "DNS view per location" but
      then IPA needs integration with DNS views on external servers.



Current use of SRV records
----------------------------------------------------------------------------------------------

Client queries for SRV records located in the domain/realm
(``example.com`` in the following example) and gets back all servers.
Assuming that all DNS servers responsible for given zone are returning a
static set of records, there is no way how to prioritize a server
'nearest' to the client at the moment. Consequently, all FreeIPA servers
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
-  The mechanism can be re-used for non-IPA services. Generally any
   service which is using ``SRV`` records can be hacked in this way.
   This might require specifying location on per-zone basics.



Design (so-called DNAME per client)
-----------------------------------



The Discovery Protocol
----------------------------------------------------------------------------------------------

The main point about this protocol is the recognition that all we really
need is to find a specific (set of) server(s) that a specific client
needs to find. The second point is that the client has 1 bit of
information that is unequivocal: its own fully qualified name. Whether
this is fixed or assigned by the network (via DHCP) is not important,
what matters is that the name unequivocally identifies this specific
host in the network.

Based on these two points the idea is to make the client query for a
special set of SRV records keyed on the client's own DNS name. From the
client perspective this is the simplest protocol possible, it requires
no knowledge or hard decisions about what DNS domain name to query or
how to discover it. At the same time is allows the Domain Administrators
a lot of flexibility on how to configure these records per-client.

The failure mode for this protocol is to simply keep using the previous
heuristics, we will not define these heuristics as they are not
standardized and are implementation and deployment specific to some
extent. Suffice to say that this new protocol should not impact in any
way on previous heuristics and DNS setups and can be safely implemented
in clients with no ill effects save for an additional initial query.
Local negative caching may help in avoiding excessive queries if the
administrator chooses not to configure the servers to support per client
SRV Records and otherwise adds little overhead.



Client Implementation
----------------------------------------------------------------------------------------------

Because currently used SRV records are multiple and to allow the case
where a host may actually be using a domain name that is also already
used as a zone name (ie the name X.example.com identifies both an actual
host and is a sub-domain where clients Y.X.example.com normally searches
for SRV records) we group all per-client location SRV records under the
``_location.`` sub name.

So for example, a client named X.example.com would search for its own
per-client records for the LDAP service over the TCP protocol by using
the name: ``_ldap._tcp._location.X.example.com``

With current practices a client normally looks for
``_ldap._tcp.example.com`` instead.

It is a simple as that, the only difference between a client supporting
this new mechanism and a generic client is only about what name is used
as the 'base domain name'. Everything else is identical. Many clients
can probably be already configured to use this new base domain. And
clients that may not support it (either because the base domain is
always derived in some way and not directly configurable or because
clients refuse to use \_location as a valid bade DNS name component due
to the leading '_' character) can be easily changed. Those that can't be
changed will simply fall back to use the classic SRV records on the base
domain and will simply not be location aware.

The additional advantage of using this scheme is that clients can now
use per-client SRV searches by default if they so choose because there
is no risk of ending up using unrelated servers due to unfortunate host
naming. If the administrator took the pain to configure per-client SRV
records there is an overwhelming chance those are indeed the records the
client is supposed to use. By using this as default it is possible to
make client configuration free by default which is a real boon on
networks with many hosts.

Changing defaults requires careful consideration of security
implications, please read the `#Security
Considerations <#Security_Considerations>`__ section for more
information.



Server side implementation
----------------------------------------------------------------------------------------------



Basic solution
^^^^^^^^^^^^^^

The simplest way to implement this scheme on the server side is to just
create a set of records for each client. However this is a very
heavyweight and error prone process as it requires the creation of many
records for each client.



A more rational solution
^^^^^^^^^^^^^^^^^^^^^^^^

A simple but more manageable solution may be to use DNAME records as
defined by `RFC
6672 <http://www.rfc-archive.org/getrfc.php?rfc=RFC6672>`__. The
administrator in this case can set up a single set of SRV records per
location and then use a DNAME record to glue each client to this
subtree.

This solution is much more lightweight and less error prone as each
client would need one single additional record that points to a well
maintained subtree.

So a client X.example.com could have a DNAME record like this:
``_location.X.example.com. DNAME Y._locations.example.com.``

When the client X tries to search for its own per-client records for the
LDAP service over the TCP protocol by using the name
``_ldap._tcp._location.X.example.com`` it would be automatically
redirected to the record ``_ldap._tcp.Y._locations.example.com``



Advanced FreeIPA solution
^^^^^^^^^^^^^^^^^^^^^^^^^

Although the above implementation works fine for most cases it has 2
major drawbacks. The first one is poor support for roaming clients as
they would be permanently referring to a specific location even when
they travel across potentially very geographically dispersed locations.
The other big drawback is that admins will have to create the DNAME
records for each client which is a lot of work. In FreeIPA we can have
more smarts given we can influence the bind-dyndb-ldap plugin behavior.

So one first very simple yet very effective simplification would be to
change the bind-dyndb-ldap plugin to create a phantom per-client
location DNAME record that points to a 'default' location.

This means DNAME records wouldn't be directly stored in LDAP but would
be synthesized by the driver if not present using a default
configuration. However to make this more useful the plugin shouldn't
just use one single default, but should have a default 'per server'.



Related tickets (incomplete list)
'''''''''''''''''''''''''''''''''

-  `bind-dyndb-ldap ticket
   #126 <https://fedorahosted.org/bind-dyndb-ldap/ticket/126>`__
-  `FreeIPA ticket #2008:
   [RFE <https://fedorahosted.org/freeipa/ticket/2008>`__ IPA should
   support and manage DNS sites]



Roaming/Remote clients
''''''''''''''''''''''

Roaming clients or Remote clients have one big problem, although they
may have a default preferred location they move across networks and the
definition of 'location' and 'closest' server changes as they move. Yet
their name is still fixed. With a classic Bind setup this problem can
somewhat be handled by using views and changing the DNAME returned or
directly the SRV records depending on the client IP address. However
using source IP address is not always a good indicator. Clients may be
behind a NAT or maybe IP addressing is shared between multiple logical
locations within a physical network. or the client may be getting the IP
address over a VPN tunnel and so on. In general relying on IP address
information may or may not work. (There is also the minor issue that we
do not yet support views in the bind-dyndb-ldap plugin.)



Addressing the multiple locations problem
'''''''''''''''''''''''''''''''''''''''''

The reason to define multiple locations is that we want to redirect
clients to different servers depending on the location they belong to.
This only really makes sense if each location has its own (set of)
FreeIPA server(s).

Also usually a location corresponds to a different network so it can be
assumed the if at least one of the FreeIPA servers in each location is a
DNS enabled server and the local network configuration (DHCP) server
serves this DNS server as the primary server for the client then we can
make the reasonable assumption that a client in a specific location will
generally query a FreeIPA server in that same location for
location-specific information.

If this holds true then changing the 'default' location base on the
server's own location would effectively make clients stick to the local
servers (Assuming the location's SRV records are properly configured to
contain only local server, which we can insure through appropriate
checks in the framework)

This is another simple optimization and works for a lot of cases but not
necessarily all. However this optimization leads to another problem.
What if the client needs to belong to a specific location indipendetly
from what server they ask to, or what if we really only have a few
FreeIPA DNS servers but want to use more locations ?

One way of course is to create a fixed DNAME record for these clients,
so the defaults do not kick in. However this is rather final. Maybe the
clients needs a preference but that preference can be overridden in some
circumstances.



Choosing the right location
'''''''''''''''''''''''''''

So the right location for a client may be a combination of a preference
and a set of requirements. One example of a requirement that can trump
any preference is a bandwidth constrained location.

Assume we have a client that normally resides in a large location. This
location has been segmented in small sub-locations to better distribute
load so it has a preferred location. If we use a fixed DNAME to
represent this preference when this client roams to a bandwidth
constrained network it will try to use the slow link to call 'home' to
his usual location. This may be a serious problem.

However if we generate the default location dynamically we can easily
have rules on the bandwidth constrained location DNS servers that no
matter what is the preference any client asking for location based SRV
records will always be redirected to the local location which includes
only local servers in their SRV records.

This is quite powerful and would neatly solve many issues connected with
roaming clients.



DNS Slave server problem
''''''''''''''''''''''''

Dynamically choosing locations may cause issues with DNS Slaves servers,
as they wouldn't be able implement this dynamic mechanism.

One way to handle this problem is to operate in a 'degraded' mode where
DNAME records are effectively created and the location is not dynamic
per-client anymore. We can still have 'different' defaults per server if
we decide to filter DNAME records from replication. However filtering
DNAME records is also a problem because we would not be able to filter
only location based ones, it would be an all or nothing filter, which
would render DNAME records unusable for any other purpose. This
restriction is a bit extreme.

Another way might be to always compute all zone DNAME records based on
the available host records on the fly at DNS server startup, and then
keep them cached (and updated) by the bind-dyndb-ldap plugin, which will
include these records in AXFR transfers but will not write them back to
the LDAP server keeping them local. This solution might be the golden
egg, as it might allow all the advantages of dynamic generation, as well
as response performance and solve the slave server issue and perhaps
even DNSSEC related issues. It has a major drawback, it would make the
code a lot more compicated and critical.



Overall implementation proposal
----------------------------------------------------------------------------------------------

Given that the basic solution is relatively simple and require minimal
if no client changes we should consider implementing at least part of
this proposal as soon as possible. Implementing DNAME record support in
bind-dyndb-ldap seem a prerequisite and adding client support in the
SSSD IPA provider would allow to test at least with the basic setup.
This basic support should be implemented sooner rather than later so
that full dynamic support can lately be easily added to bind-dyndb-ldap
support as well as adding the necessary additional schema and UI to the
freeipa framework to mark and group clients and locations.



Security Considerations
----------------------------------------------------------------------------------------------

TBD



Client Implementation
^^^^^^^^^^^^^^^^^^^^^

As always DNS replies can be spoofed relatively easily. We recommend
that SRV records resolution is used only for those clients that normally
use an additional security protocol to talk to network resources and can
use additional mechanisms to authenticate these resources. For example a
client that uses an LDAP server for security related information like
user identity information should only trust SRV record discovery for the
LDAP service if LDAPS or STARTTLS over LDAP are mandatory and
certificate verification is fully turned on, or if SASL/GSSAPI is used
with mutual authentication, integrity and confidentiality options
required. Use of DNSSEC and full DNS signature verification may be
considered an additional requirement in some cases.



Server Implementation
^^^^^^^^^^^^^^^^^^^^^

Given current integration with BIND (using bind-dyndb-ldap), the only
way how to handle DNSSEC is to pre-generate all ``_location`` records
for each client name at zone loading time. DNSSEC signing will then sign
all the data as usual.

As a consequence, this pre-generation increases memory consumption and
CPU time spent on signing by factor of ~ 2.3. Tested on zone with 10000
names using ``dnssec-signzone`` from BIND
``bind-9.10.3-7.P2.fc23.x86_64`` with 2048 bit ZSK, 3072 KSK:

| ``$ dnssec-keygen -a RSASHA256 -3 -b 3072 -f KSK -r /dev/urandom test.``
| ``$ dnssec-keygen -a RSASHA256 -3 -b 2048 -r /dev/urandom test.``
| ``$ time dnssec-signzone -3 0123456789 -S -K . -o test. dname.db``
| ``user   0m56.584s``
| ``$ time dnssec-signzone -3 0123456789 -S -K . -o test. nodname.db``
| ``user   0m24.881s``

Zone file sizes:

| ``23150974  dname.db.signed``
| ``10097345  nodname.db.signed``

Example
----------------------------------------------------------------------------------------------

Version 1 of this proposal introduces separate sets of SRV records for
each location.

Location ``cz`` will have one set of SRV records:

::

   ``;; QUESTION SECTION:``
   ``;_ldap._tcp.cz._locations.example.com. IN  SRV``
   ``;; ANSWER SECTION:``
   ``_ldap._tcp.cz._locations.example.com. SRV ``\ **``0``\ ````\ ``100``**\ `` 389 ipa-brno.example.com.``
   ``_ldap._tcp.cz._locations.example.com. SRV ``\ **``3``\ ````\ ``100``**\ `` 389 ipa-london.example.com.``

Location ``uk`` will have different set of SRV records (possibly with
different priorities, weights, or even servers):
::

   ``;; QUESTION SECTION:``
   ``;_ldap._tcp.uk._locations.example.com. IN  SRV``
   ``;; ANSWER SECTION:``
   ``_ldap._tcp.uk._locations.example.com. SRV ``\ **``0``\ ````\ ``50``**\ `` 389 ipa-brno.example.com.``
   ``_ldap._tcp.uk._locations.example.com. SRV ``\ **``0``\ ````\ ``200``**\ `` 389 ipa-london.example.com.``

Clients are querying SRV records under client's FQDN prefixed with label
``_location`` name. This record contains redirection to a location into
which the client is assigned. (From client's perspective is does not
matter how the DNS server generated the redirection.)

::

   ``;; QUESTION SECTION:``
   ``;_ldap._tcp.``\ **``_location.client2.example.com.``**\ `` IN SRV``
   ``;; ANSWER SECTION:``
   **``_location.client2``**\ ``.example.com. DNAME ``\ **``cz._locations``**\ ``.example.com.``
   ``_ldap._tcp._location.client2.example.com. CNAME _ldap._tcp.cz._locations.example.com.``
   ``_ldap._tcp.cz._locations.example.com. SRV ``\ **``3``\ ````\ ``100``**\ `` 389 ipa-london.example.com.``
   ``_ldap._tcp.cz._locations.example.com. SRV ``\ **``0``\ ````\ ``100``**\ `` 389 ipa-brno.example.com.``

Following diagram summarizes proposed behavior (version 1):
|ExampleLocationsV1.svg|

-  **(A)** The LDAP database contains records per each location
   ("Y.$LOCATION._location.$SUFFIX") and default records (*Y.$SUFFIX*)
-  **(B)** The DNAME record that overrides the default locations in
   format
   *\_location.$HOSTNAME*\ **DNAME**\ *$LOCATION._locations.$SUFFIX*
-  **(C)** The DNS server in location using *bind-dyndb-ldap* generates
   DNAME records per host which replace client hostnames with **cz**
   location. A client from location **cz** will get SRV records with
   priority set for this location.
-  **(D)** The DNS server in location using *bind-dyndb-ldap* generates
   DNAME records per host which replace client hostnames with **uk**
   location. A client from location **uk** will get SRV records with
   priority set for this location. Please note DNAME record for
   **client2** that has been overridden with the record stored in the
   LDAP database.
-  **(E)** Configuration for client2 has been overridden. The client is
   configured to contact location **uk** but DNS server returns results
   for location **cz**.

-  **[1]** Client is configured to use DNS *locations* and wants to
   connect to the closest LDAP server.
-  **[2]** Client send DNS query in format
   *\_ldap._tcp._location.$CLIENT_HOSTNAME* to server in its location.
-  **[3]** DNAME records for each client has been dynamically created on
   DNS server (except override records).
-  **[4]** Server returns DNAME and CNAME (for old clients) records, the
   client has to ask server again to receive SRV records for the name
   returned by DNAME (CNAME).
-  **[5]** Server returns SRV records configured for this location
   (priority for servers located in CZ (Brno))



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



Summary of meeting 2016-02-04
-----------------------------

-  Participants: Simo Sorce, Petr Spacek, Martin Basti
-  We will start with `per sub-tree
   approach <V4/DNS_Location_Mechanism>`__ and deffer per-client
   overrides for now.
-  Keep in mind that bind-dyndb-ldap might get rid of GSSAPI. LDAPI
   mapping to a principal may change results from LDAP whoami.
-  LDAP schema and user interface has to be defined.

   -  We should think about supporting DNS locations per (server & zone)
      so different zones can be assigned to different locations.

Implementation
--------------

TBD

UI

TBD

CLI

TBD

Configuration
----------------------------------------------------------------------------------------------

TBD

Upgrade
-------

TBD



How to Test
-----------

TBD



Test Plan
---------

`DNS Location Mechanism with per client override V4.4 test
plan <V4/DNS_Location_Mechanism_with_per_client_override/Test_Plan>`__

References
----------

SRV Records: `RFC
2782 <http://www.rfc-archive.org/getrfc.php?rfc=RFC2782>`__

DNAME Records: `RFC
6672 <http://www.rfc-archive.org/getrfc.php?rfc=RFC6672>`__

.. |ExampleLocationsV1.svg| image:: ExampleLocationsV1.svg