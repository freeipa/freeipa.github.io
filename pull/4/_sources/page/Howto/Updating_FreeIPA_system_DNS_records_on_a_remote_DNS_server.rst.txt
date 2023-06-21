Updating_FreeIPA_system_DNS_records_on_a_remote_DNS_server
==========================================================



Short feature description
-------------------------

FreeIPA with integrated DNS updates dynamically its own DNS records
after changes in topology. This can be also executed manually by calling
command ``ipa dns-update-system-records``.

However this is not so simple with external DNS services. For this case
the option ``--dry-run`` can provide list of required DNS records and
option ``--out FILE`` can export data in *nsupdate* util format. This
can be used directly with ``nsupdate`` util.



Supported in versions
---------------------

-  **4.4+** ``dns-update-system-records`` command
-  **4.5+** ``dns-update-system-records`` with ``--out`` option

Examples
--------



Show list of required records
----------------------------------------------------------------------------------------------

This can be handy in a case you have hosted external DNS and management
is done via GUI to see records that have to be updated manually.

The list of records can look for example like this:

| ``[user@ipa ~]$ ipa dns-update-system-records --dry-run``
| `` IPA DNS records:``
| ``   _kerberos-master._tcp.example.com. 86400 IN SRV 0 100 88 ipa.example.com.``
| ``   _kerberos-master._udp.example.com. 86400 IN SRV 0 100 88 ipa.example.com.``
| ``   _kerberos._tcp.example.com. 86400 IN SRV 0 100 88 ipa.example.com.``
| ``   _kerberos._udp.example.com. 86400 IN SRV 0 100 88 ipa.example.com.``
| ``   _kerberos.example.com. 86400 IN TXT "EXAMPLE.COM"``
| ``   _kpasswd._tcp.example.com. 86400 IN SRV 0 100 464 ipa.example.com.``
| ``   _kpasswd._udp.example.com. 86400 IN SRV 0 100 464 ipa.example.com.``
| ``   _ldap._tcp.example.com. 86400 IN SRV 0 100 389 ipa.example.com.``
| ``   _ntp._udp.example.com. 86400 IN SRV 0 100 123 ipa.example.com.``
| ``   ipa-ca.example.com. 86400 IN A 192.0.2.36``
| ``   ipa-ca.example.com. 86400 IN AAAA 2001:db8:0:224e:21a:4aff:fe23:1523``



Generating a file with FreeIPA DNS data for *nsupdate* utility
----------------------------------------------------------------------------------------------

Option ``--out FILE`` will store DNS data in *nsupdate* format in file
*FILE*.

::

   | ``[user@ipa ~]$ ipa dns-update-system-records --dry-run --out ipa-records.nsupdate``
   | ``  IPA DNS records:``
   | ``   ...``
   | ``[user@ipa ~]$ cat ipa-records.nsupdate ``
   | ``; IPA DNS records``
   | ``update delete _kerberos-master._tcp.example.com. SRV``
   | ``update add _kerberos-master._tcp.example.com. 86400 IN SRV 0 100 88 ipa.example.com.``
   | ``update delete _kerberos-master._udp.example.com. SRV``
   | ``update add _kerberos-master._udp.example.com. 86400 IN SRV 0 100 88 ipa.example.com.``
   | ``.....``
   | ``update delete ipa-ca.example.com. AAAA``
   | ``update add ipa-ca.example.com. 86400 IN AAAA 2001:db8::0:224e:21a:4aff:fe23:1523``
   | ``send``



Notes about exported *nsupdate* file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By default records exported by ``dns-update-system-records`` relies on
auto-detection of the zone where records should be updated and the
authoritative server of that zone. In majority cases this should just
work. However in non-standard DNS setup or missing zone delegations,
*nsupdate* may not be able to find the right zone and server. For these
cases the exported file must be amended by following options (at the
beginning of the file):

-  ``server``\ *``servername``*\ ``[``\ *``por``*\ ``t]`` where
   *servername* is an authoritative DNS server where records should be
   sent
-  ``zone``\ *``zonename``* where *zonename* is zone where FreeIPA
   records should be placed

Example:

::

   | ``[user@ipa ~]$ cat ipa-records.nsupdate ``
   | ``zone example.com.``
   | ``server 192.0.2.222``
   | ``; IPA DNS records``
   | ``update delete _kerberos-master._tcp.example.com. SRV``
   | ``update add _kerberos-master._tcp.example.com. 86400 IN SRV 0 100 88 ipa.example.com.``
   | ``...``

For more details please see *man nsupdate*.



Using *nsupdate* with TSIG
----------------------------------------------------------------------------------------------

`TSIG <https://tools.ietf.org/html/rfc2845>`__ mechanism allows to use
nsupdate utility with securely by using a shared key. The DNS server
must be configured and both server and client must have the particular
shared key to allow updates.

Server configuration examples:

-  `BIND <ftp://ftp.isc.org/www/bind/arm95/Bv9ARM.ch04.html#tsig>`__
-  `PowerDNS <https://doc.powerdns.com/md/authoritative/dnsupdate/#dns-update-how-to-setup-dyndnsrfc2136-with-dhcpd>`__
-  `Knot DNS
   1 <https://www.knot-dns.cz/docs/2.x/html/configuration.html#dynamic-updates>`__
   + `Knot DNS
   2 <https://www.knot-dns.cz/docs/2.x/html/configuration.html#access-control-list-acl>`__
   + `Knot DNS
   3 <https://www.knot-dns.cz/docs/2.x/html/man_keymgr.html#tsig-commands>`__,

Run *nsupdate* with the ``-k keyfile`` option:

``[user@ipa ~]$ nsupdate -k tsig-key.keyfile ipa-records.nsupdate``

or with ``-y algorithm:keyname:secret`` option:

``[user@ipa ~]$ nsupdate -y algorithm:keyname:secret ipa-records.nsupdate``

More details about *nsupdate* with TSIG and how to generate keyfiles can
be found `here <Howto/DNS_updates_and_zone_transfers_with_TSIG>`__



Using *nsupdate* with GSS-TSIG
----------------------------------------------------------------------------------------------

`GSS-TSIG <https://tools.ietf.org/html/rfc3645>`__ mechanism uses
`GSS-API <https://tools.ietf.org/html/rfc2743>`__ for getting secret
TSIG key. Details about GSS-API is out of scope of this document, for
simplification we will assume *Kerberos V5* as used technology for
GSS-API ("kerberized" DNS servers are usually the most used).

Examples of server configuration:

-  `BIND <http://ddiguru.com/blog/136-how-to-implement-gss-tsig-on-isc-bind>`__
-  `PowerDNS <https://doc.powerdns.com/md/authoritative/gss-tsig/>`__
-  `Windows
   DNS <https://technet.microsoft.com/en-us/library/cc961412.aspx>`__

Run *nsupdate* with option ``-g``

| ``[user@ipa ~]$ kinit principal-allowed-to-update-records@REALM``
| ``[user@ipa ~]$ nsupdate -g ipa-records.nsupdate``



Using *nsupdate* without authentication
----------------------------------------------------------------------------------------------

Using *nsupdate* without authentication is discouraged. However if you
really need this, then set up DNS server to allow dynamic updates from
the particular IP address/IP range.

Server configuration examples:

-  `BIND <http://www.zytrax.com/books/dns/ch7/xfer.html#allow-update>`__
-  `PowerDNS <https://doc.powerdns.com/md/authoritative/dnsupdate/#allow-dnsupdate-from>`__
-  `Knot
   DNS <https://www.knot-dns.cz/docs/2.x/html/configuration.html#dynamic-updates>`__

Run *nsupdate* without options:

``[user@ipa ~]$ nsupdate ipa-records.nsupdate``