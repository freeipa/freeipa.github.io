.. _short_feature_description:

Short feature description
-------------------------

IPA (DNS) Locations feature can be used to distribute load from clients.
This is done by SRV records with the proper priority set and CNAME
aliases. Servers in IPA location redirect DNS queries using CNAME to
location records.

IPA servers outside location work as backup servers. In case that all
IPA servers in the particular location failed to respond, servers from
outside of the location will be used.

.. _information_good_to_know:

Information good to know
~~~~~~~~~~~~~~~~~~~~~~~~

-  Only IPA system services are supported.
-  IPA DNS subsystem must be installed.
-  IPA DNS must be authoritative for primary IPA DNS domain.
-  IPA locations use SRV records, without functional DNS server inside a
   location, this location effectively stops working.
-  IPA locations use priority field in SRV records to prefer servers in
   the location. Servers with priority 0 belongs to the location,
   servers with priority 50 are backup servers from outside of the
   location.
-  Membership of client in a location depends on DNS server which is
   configured as resolver for client. (Directly or indirectly through
   recursors etc.)
-  Each location must contain at leas one IPA DNS server.
-  Multiple IPA DNS servers per location are recommended (backup DNS
   servers inside one location).
-  Be careful with caching DNS servers. Caching DNS servers may cause
   delay in propagation of changes in locations. This can be solved by
   lowering TTL for location records in advance (IPA default is one
   day).
-  Clients must be able to work with SRV and CNAME records to work with
   IPA locations properly.

Example
-------

Topology
~~~~~~~~

.. figure:: Dns_location_topology.svg
   :alt: dns_location_topology.svg

   dns_location_topology.svg

.. _installation_server:

Installation (server)
~~~~~~~~~~~~~~~~~~~~~

Please note the following steps are just examples, in real world, do not
forget to add additional replication agreements as backup to prevents
disconnect topology.

| ``[root@berlin ~]# ipa-server-install --setup-dns``
| ``[root@prague1 ~]# ipa-replica-install --setup-dns``
| ``[root@prague2 ~]# ipa-replica-install --setup-dns``
| ``[root@paris1 ~]# ipa-replica-install --setup-dns``
| ``[root@paris2 ~]# ipa-replica-install``
| ``[root@paris3 ~]# ipa-replica-install --setup-dns``

.. _adding_locations:

Adding locations
~~~~~~~~~~~~~~~~

Use **location-add** command to create a new location.

| ``[root@berlin ~]# ipa location-add czechrep --description="Czech republic clients"``
| ``[root@berlin ~]# ipa location-add france --description="France clients"``

.. _adding_servers_to_locations:

Adding servers to locations
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use **server-mod** command to add server into location. Server can be
member of only one location.

| ``[root@berlin ~]# ipa server-mod prague1.example.com --location=czechrep``
| ``[root@berlin ~]# ipa server-mod prague2.example.com --location=czechrep``
| ``[root@berlin ~]# ipa server-mod paris1.example.com --location=france``
| ``[root@berlin ~]# ipa server-mod paris2.example.com --location=france``
| ``[root@berlin ~]# ipa server-mod paris3.example.com --location=france``

.. _advanced_configuration:

Advanced configuration
~~~~~~~~~~~~~~~~~~~~~~

.. _setting_service_weight:

Setting service weight
^^^^^^^^^^^^^^^^^^^^^^

Using weight attribute in SRV records, you can customize load between
servers in a location (for servers without location too). Servers with
better performance can have assigned higher weight for IPA services and
those servers will receive more queries than others. To set weight for
IPA system SRV records, please use **server-mod**. For further
information about weight please see `RFC 2783 page
3 <https://tools.ietf.org/html/rfc2782#page-3>`__.

``[root@berlin ~]# ipa server-mod paris2.example.com --service-weight=2000``

.. _lowering_ttl:

Lowering TTL
^^^^^^^^^^^^

In case you use caching DNS servers, you may need to lower default TTL
(86400) of the IPA domain. Any change in locations or in system records
will be reflected on caching servers after the TTL of original records
expires. Lowering TTL in advance is good for planned changes in
topology.

To change default TTL use **dnszone-mod** command.

``[root@berlin ~]# ipa dnszone-mod example.com. --default-ttl=3600``

.. _installation_client:

Installation (client)
~~~~~~~~~~~~~~~~~~~~~

Client must have set the proper resolver which belongs to the location
where clients are supposed to be located in. Preferred is to configure
resolvers using DHCP, as we expect that the network topology respects
geographical location, and the DHCP provides the closest DNS server IP
addresses, thus this should be done without any extra configuration.

Please note that all nameservers configured on client should be in the
same location. A configuration with nameservers from various location
will work, but it may result into inefficient resolving.

| ``[root@client-prague ~] # cat /etc/resolv.conf``
| ``nameserver 10.10.0.1``
| ``nameserver 10.10.0.2``
| ``[root@client-paris ~]# cat /etc/resolv.conf``
| ``nameserver 10.50.0.1``
| ``nameserver 10.50.0.3``
| ``[root@client-berlin ~]# cat /etc/resolv.conf``
| ``nameserver 10.30.0.1``
| ``[root@client-oslo ~]# cat /etc/resolv.conf``
| ``nameserver 10.30.0.1``

If resolvers are properly set, you can install clients by using
**ipa-client-install**.

Verification
~~~~~~~~~~~~

We can use **dig** to verify returned DNS records

Server/client without locations

| ``[root@berlin ~]# dig +short _ldap._tcp.example.com SRV``
| ``0 100 389 berlin.example.com.``
| ``0 100 389 prague1.example.com.``
| ``0 100 389 prague2.example.com.``
| ``0 100 389 paris1.example.com.``
| ``0 2000 389 paris2.example.com.``
| ``0 100 389 paris3.example.com.``

Server/client inside *czechrep* location

| ``[root@client-prague ~]# dig +short _ldap._tcp.example.com SRV``
| ``_ldap._tcp.czechrep._locations.example.com.    # CNAME alias _ldap._tcp --> _ldap._tcp.czechrep._locations``
| ``50 100 389 berlin.example.com.    # server with lower priority (50), outside of location``
| ``0 100 389 prague1.example.com.    # server inside location``
| ``0 100 389 prague2.example.com.``
| ``50 100 389 paris1.example.com.``
| ``50 2000 389 paris2.example.com.``
| ``50 100 389 paris3.example.com.``

Server/client inside *france* location

| ``[root@client-paris ~]# dig +short _ldap._tcp.example.com SRV``
| ``_ldap._tcp.france._locations.example.com.    # CNAME alias _ldap._tcp --> _ldap._tcp.france._locations``
| ``50 100 389 berlin.example.com.    # server with lower priority (50), outside of location``
| ``50 100 389 prague1.example.com.``
| ``50 100 389 prague2.example.com.``
| ``0 100 389 paris1.example.com.    # server inside location``
| ``0 2000 389 paris2.example.com.``
| ``0 100 389 paris3.example.com.``

.. _get_list_of_all_required_records:

Get list of all required records
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can use command **ipa dns-update-system-records --dry-run** to print
list of all required system records, and location records. Eventually if
some records are missing in IPA domain, you can use this command
**without --dry-run** option to fix missing system records.

.. _example_with_non_freeipa_dns_servers:

Example with non-FreeIPA DNS servers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The first example assumed that you have at least one FreeIPA DNS server
in each location. With same effort the same feature can be implemented
also using external DNS servers. Following example is using InfoBlox's
support for DNS view to simulate multiple DNS servers in each location:

*  On Infoblox, create DNS view **for each location**:
* Data Management -> DNS -> Zones -> click to Plus sign to add DNS view
* Add DNS view step 1 -> fill-in name, use the same name as for IPA location, e.g. "czechrep" 
* Add DNS view step 2 -> specify Match Clients rule so that all clients in "czechrep" location domain belong to this DNS view
* Add DNS view step 3 -> Save
* In each DNS view, create two zones with names: "_udp.", "_tcp."

These zones (specific for each DNS view) will be filled-in with records
produced by the IPA server.

-  On a IPA server, run command:

``[root@berlin ~]# ipa dns-update-system-records --dry-run``

It will produce a lot of DNS records. We are interested in records
listed in section **IPA location records**:

| `` IPA location records:``
| ``   _kerberos-master._tcp.czechrep._locations.example.com. ...``
| ``   _kerberos-master._udp.france._locations.example.com. ...``
| ``...``

Each IPA location has own set of records. Records specific to given
location contain ``._locations`` in their name.

-  For each DNS location/DNS view, select relevant records from
   ``ipa dns-update-system-records --dry-run``'s output and transform
   them to form suitable for general purpose DNS server. E.g.:

| ``LOCATION=czechrep``
| ``[root@berlin ~]# ipa dns-update-system-records --dry-run | grep $LOCATION._locations | sed "s/\.$LOCATION\._locations//"``

This way, you will obtain list of records for each location. Each list
contains records with the same names (the left side) but different data
(the right side).

-  As the last step, take this list of records and for each location,
   copy it into particular Infoblox's DNS view.

Now you are done. Clients using DNS discovery to find IPA servers will
prefer the local servers automatically (as soon as DNS TTL expires). Of
course, the procedure needs to be repeated each time you reconfigure IPA
location or do a modification to a IPA server topology.
