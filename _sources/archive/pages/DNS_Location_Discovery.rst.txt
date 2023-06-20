DNS_Location_Discovery
======================

**This design is obsolete and was replaced
by**\ `V3/DNS_Location_Mechanism <V3/DNS_Location_Mechanism>`__\ **.**



A DNS RR FOR IP BASED LOCATION DISCOVERY (DRAFT)
================================================

Simo Sorce, Red Hat Geert Jansen, Red Hat

Introduction
------------

Service Discovery is a process whereby a client uses information stored
in the network to find servers that offer a specific service. It is
useful for two reasons. First, it places the information required for
the location of servers in the network, lessening the burden of client
configuration. Second, it gives system and network administrators a
central point of control that can be used to to define an optimized
policy that determines which clients should use which server and in what
order.

A standard for service discovery is defined in [RFC2782]. This standard
defines a DNS RR with the mnemonic SRV and usage rules around it. It
allows client to discover servers that offer a given service over a
given protocol.

A drawback of [RFC2782] is that it assumes clients know the correct DNS
domain to use as the query target. Without any further information, the
client's options include using their own DNS domain and the name of the
security domain in which the client operates (such as the AD domain name
or the Kerberos REALM). Neither option is likely to yield optimal
results however. One key service discovery requirement, especially in
large distributed enterprises, is to choose a server that is close to a
client. Doing this correctly can make the difference between a working
and a non-working service. However, different considerations underlie
the grouping of systems for the purpose of DNS domains and security
contexts, and for the purpose of deciding network locality. For example,
for locality purposes system administrators may want to group systems
into groups corresponding to network subnets, but for security purposes
they may want to group systems across organisational boundaries.
Whatever option the client decides to take for a target for SRV based
service discovery, if this target is not specifically destined for
service discovery, an undesirable coupling is created between service
discovery and the other use for the target.

The incompleteness of [RFC2782] is acknowledged by systems such as [AD]
that contain extra functionality to augment SRV lookups to make them
site aware. The basic idea is to use a target in SRV service discovery
that is specific to a location or "site" as AD parlance. This introduces
location awareness into the SRV service discovery. Unfortunately though,
the protocol that AD uses to determine a site for a client is not
submitted as an Internet standard and also contains Microsoft specific
features.

This document specifies a protocol for location discovery based on DNS.
It introduces two new RRs for this, with the mnemonics SUBNET and
SUBNET6. Clients can use the protocol to find out their location, based
on their IP address and DNS domain name. The result of the location
discovery is a DNS name that can then be used as a target for a
subsequent, location aware, DNS SRV based service discovery.



The SUBNET and SUBNET6 records
------------------------------

The format of the SUBNET and SUBNET6 RR is as follows.

::

   ``   ``\ `` SUBNET  TTL CLASS ``
   ``   ``\ `` SUBNET6 TTL CLASS ``

The components are explained below.

A name for the network that is being described.

The start IP address for the network. This must be an IPv4 address for
the SUBNET RR and an IPv6 address for the SUBNET6 RR.

The length of the network prefix in bits. Together,

and form a CIDR subnet.

If does not start with an underscore ('_') then this is the name of the
location that corresponds to the subnet. The location is a plain DNS
name that can be used as the target for a subsequent SRV query.

If the location starts with an underscore, then this is not a location
but a reference to an intermediate SUBNET instead. SUBNET references are
used to build index structures that make lookups more efficient in the
case of a large number of subnets.



The Discovery Protocol
----------------------

Clients use a DNS domain for location discovery. A SUBNET aware DNS
domain MUST have a SUBNET record published under the name "_network".
This record directly or indirectly points to all locations that are
known for the domain. The name "_network" MUST resolve into one or more
SUBNET records. Each record MUST NOT have overlap with any of the other
records. In case the part of the SUBNET RR does not start with an
underscore, it points to the final location for this subnet. This
location is used as the target for a SRV lookup.

If starts with an underscore then it MUST be a DNS name that resolves to
further SUBNET records. We call the SUBNET that refers to the other
subnets the parent and the subnets that are referred to by the parent
the children. Based on this parent/child relationship the SUBNET records
in a DNS domain form a multiway tree that is rooted under "_network".
This tree is called the network tree. All parents in the network tree
MUST have a CIDR subnet that is equal to the envelope of the CIDR
subnets all its children. Each parent MUST have at least two children
but SHOULD have more children up to an implementation defined optimum to
ensure the tree forms an efficient search index.

Locations SHOULD be stored under the \_locations node in the same domain
as the network tree. Locations SHOULD have human readable names.

An example zone file that implements a network with 3 locations and 4
subnets is given below. One location (Boston) is the main location and
contains the ldap server ldap.example.net. The two other locations are
both Satellite locations and contain an ldap proxy server and a fallback
to the main location.

| ``   $TTL 1d``
| ``   $ORIGIN example.com.``
| ``   ``
| ``   _network        SUBNET IN 192.168.1.0 24 boston._locations``
| ``   _network        SUBNET IN 192.168.2.0 24 boston._locations``
| ``   _network        SUBNET IN 192.168.3.0 24 newyork._locations``
| ``   _network        SUBNET IN 192.168.4.0 24 thehague._locations``
| ``   ``
| ``   _ldap._tcp.boston._locations        SRV IN 10 10 389 ldap.example.net.``
| ``   _ldap._tcp.newyork._locations       SRV IN 10 10 389 proxy-ny.examle.net.``
| ``   _ldap._tcp.newyork._locations       SRV IN 20 10 389 ldap.example.net.``
| ``   _ldap._tcp.thehague._locations      SRV IN 10 10 389 proxy-hag.example.net.``
| ``   _ldap._tcp.thehague._locations      SRV IN 20 10 389 ldap.example.net.``

A SUBNET aware client SHOULD use the following procedure to determine
its location on the network. The location SHOULD be used as input into a
subsequently SRV query.

#. Set the SUBNET query target to the client DNS domain name.
#. The client makes a DNS query with QNAME=_network., QCLASS=IN and
   QTYPE=SUBNET. Here, is the current SUBNET query target.
#. For each entry in the query result:

   #. Determine whether the client IP address is in the CIDR network
      formed by the

      and parts of the SUBNET RR. If it is not, continue with the next
      entry.

   #. Check if starts with an underscore ('_'). If it does not, is the
      result of our location discovery and exit succesfully.

   #. If we were called recursively, ensure that is smaller than in our
      caller. If not, abort with an error. This step protects from
      infinite loops that can arise in wrongly configured DNS zones.

   #. Set the current SUBNET query target to , and jump to step 2.

Advice
------



Advice for Server Implementors
----------------------------------------------------------------------------------------------

Nothing special is required to support the SUBNET and SUBNET6 RRs in a
DNS server software.

For larger networks, it becomes ineffecient to list all SUBNET records
under the signle "_network" node in a DNS domain, and a tree should be
constructured. Current thinking suggest that no more than 20 SUBNET RRs
should be added to a single node in the tree, which makes this also the
limit for the root node.

Manually creating the network tree is inefficient and error-prone and
therefore we suggest that server implementors provide functionality to
facilitate this. Here we describe how such functionality can look like.

The input to the tree building software is a flat database containing
(subnet, location) tuples. The procedure to create the tree is given
below.

#. First, all overlaps need to be removed from the subnet. This can be
   done by splitting overlapping regions into multiple regions and by
   deciding for each region what location will be the final location.
   Typically, this will be the location of the smallest (= most
   specific) subnet for that region.
#. All non-overlapping regions are inserted in a data structure that is
   similar to a B+ tree but instead of working with keys and pointers it
   works with intervals. Each node contains a maximum number of
   intervals and intervals can point to other nodes. The standard B-tree
   procedures for splitting and merging nodes are trivially ported to
   the interval based approach. Using a B+ tree ensures that the tree
   will be balanced (ensuring efficient lookup) and that all leaves will
   be on the same, bottom level (because we cannot store both a location
   and a pointer in a node).
#. The tree is dumped. Each internal and leaf node get a random name
   starting with an underscore assigned to it. All pointers are resolved
   using these names. All nodes are published under \_network (the root
   node as \_network).
#. All locations are published under \_locations.



Advice for DNS Administrators
----------------------------------------------------------------------------------------------

Because of efficience considerations, DNS administrators are encouraged
to publish the network tree only once under a DNS domain of their
choice. Each other DNS (sub)domain that needs to use the network and
location tree can be pointed to this using a "pointer" SUBNET record.
The example below illustates a domain "corp.example.com" that uses the
network tree and location databsea of the domain "example.com".

``   _network.corp.example.com.   SUBNET IN 0.0.0.0 0 _network.example.net.``

This requires one DNS record to be published in each DNS zone in the
network which in our view is a manageable overhead.



Advice for Client Implementors
----------------------------------------------------------------------------------------------

Location discovery requires a number of successive DNS queries to
succeed. If efficient network trees are used with e.g. 20 subnets per
node, the number of queries should not exceed 4 for even the largest
networks. Nevertheless this discovery will take time and therefore we
recommend that locator software caches the result of site discovery.

Another question is how to handle multi-homed sites. There is not a
unique answer to this question and much depends on the context. Locator
software could use the first network interface of the system to
determine the IP address, or could accept a configuration setting
indicating the system's primary IP address.



Alternative Solutions
---------------------

DHCP
----------------------------------------------------------------------------------------------

DHCP could be extended to include an option that tells the client the
site it is in. The granularity of such an approach would be reasonable
as most subnets are contained to physical sites (the notable exception
being strechted subnets for high availability purposes). Nevertheless,
we don't think DHCP is a valid option because there are many systems
that do not use it an use static IP configuration instead.



The resolver "sortlist" option
----------------------------------------------------------------------------------------------

Some DNS resolvers recognize an option called "sortlist" that specifies
a set of subnets that are "local" to the client. The resolve uses this
list to order the results of queries that have multiple results so that
matching IP addresses are put first. A location aware service discovery
protocol could be constructed by agreeing that for a service everybody
uses the same name, and let the resolver put the local server first.
However, this option does not allow for the specification of the
load-balancing parameters priority and weight which makes it unsuitable
as a general purpose service discovery protocol in our view.

Another disadvantage is that the subnet as seen from he network may
actually be different from the subnet that the administrator defined
from a location point of view.



Use DNS subdomains
----------------------------------------------------------------------------------------------

Each location could have its associated DNS subdomain, which could be
used to publish SRV records. This approach has the drawbacks that many
DNS domains are required, and that there is again an undesirable
coupling between grouping for naming purposes and grouping for location
purposes.



Use LDAP for location discovery
----------------------------------------------------------------------------------------------

Instead of DNS, LDAP could be used to store the location and subnet
information. In this case, the LDAP server could also take care of the
indexing removing the burder of the creation of the network tree.
Howver, as LDAP servers often contain interesting data, many deployments
do not allow unauthenticated connections to it (apart from a few
internal attributes on the LDAP root). This is a problem, as we'd like
to use service discovery protocol to resolve servers for our identity
service.



Use Remote Procedure Calls
----------------------------------------------------------------------------------------------

Instead of a client resolving its site, a remote procedure call approach
could be used. This would solve the unauthenicated access to the
location database problem. This is also the approach taken in [AD] where
clients make a connectionless LDAP request to a domain controller which
is in fact is just an RPC. The disadvantage of this approach is that yet
another protocol is introduce. the advantage is that the location and
subnet database does not need to be public, it only needs to be
available to the RPC server.



Transition Period
-----------------

It is expected that it will take quite a while for DNS server to catch
up and implement the SUBNET RR. Until this time, client implementations
MAY use [RFC1464] style TXT records to store keys named "subnet" and
"subnet6" in TXT records.



Security Considerations
-----------------------

Publishing the tree of SUBNET nodes under a well known location allows
for anonymous discovery of all the subnets and location names. Although
the data disclosed is not as relevant as what is discolsed via a zone
transfer it may still be perceived as a security issue. An organization
may use features in their DNS server to provide different results
depending on the querying IP source address (views) so that this
information is not available outside the internal organization networks.

References
----------

[RFC1464]

[RFC2782]

[AD]