DNS_in_isolated_networks
========================

**Supported in IPA version 4.1 and newer**

Theory
------

Public internet contains root zone, handled by root servers. Recursive
servers needs the root hints, list of known addresses of root servers,
to be able send queries to the root servers.

In isolated you might want to have own root zone. This can be done by
setting up root zone (.) as master zone on IPA DNS server. If zone is
authoritative (master) root hints are not used by the IPA server.

Keep in mind that root hints have to be updated on all servers which are
not authoritative for the root zone, and also on recursive clients
inside the isolated network.



Example how to create root zone
-------------------------------

IPA DNS server: *ipa.example.com.*

Add root zone (.):

``$ ipa dnszone-add .``

Without proper delegation of zone where nameserver is located, BIND will
show following error and will not load the root zone.

| ``# journalctl -b -u named[-pkcs11]``
| ``  zone ./IN: NS 'ipa.example.com' has no address records (A or AAAA)``
| ``  zone ./IN: not loaded due to errors.``

**Note:** IPA adds all DNS capable IPA replicas as default NS records to
zone apex.

Add NS delegation for all child zones on server.

::

    | ``$ ipa dnszone-find --pkey-only``
    | ``  Zone name: .``
    | ``  Zone name: ``\ **``2.0.192.in-addr.arpa.``**
    | ``  Zone name: ``\ **``example.com.``**
    | ``----------------------------``
    | ``Number of entries returned 3``
    | ``----------------------------``
    | ``$ ipa dnsforwardzone-find --pkey-only``
    | ``  Zone name: ``\ **``fwzone.test.``**
    | ``----------------------------``
    | ``Number of entries returned 1``
    | ``----------------------------``
    | ``$ ipa dnsrecord-add . ``\ **``2.0.192.in-addr.arpa.``**\ `` --ns-rec=ipa.example.com.``
    | ``$ ipa dnsrecord-add . ``\ **``example.com.``**\ `` --ns-rec=ipa.example.com.``
    | ``$ ipa dnsrecord-add . ``\ **``fwzone.test.``**\ `` --ns-rec=ipa.example.com.``



Updating root hints
-------------------

This procedure updates root hints for machines which are not
authoritative for the root zone. It has to be repeated every time you
change IP addresses of your root servers.

In this example, the IP address ``192.0.2.1`` is IP address of one of
your private root servers. Root hints file usually has the same format
as zone file, so output from dig command can be used directly.

For BIND 9:

``$ dig @192.0.2.1 . NS > /var/named/named.ca``

For Unbound:

| ``$ dig @192.0.2.1 . NS > /var/unbound/hints.ca``
| ``# add option "roothints: /var/unbound/hints.ca" to "server" section``
| ``# in /etc/unbound/unbound.conf``

For Unbound utilities (drill etc.):

| ``$ dig @192.0.2.1 . NS > /tmp/hints.ca``
| ``# use -r option for every call of the command``
| ``$ drill -r /tmp/hints.ca``

DNSSEC
------

Details how to use DNSSEC in isolated networks are described in article
`DNSSEC in isolated
networks <Howto/DNSSEC#DNSSEC_in_isolated_networks>`__.

`Category:How to <Category:How_to>`__ `Category:Draft
documentation <Category:Draft_documentation>`__