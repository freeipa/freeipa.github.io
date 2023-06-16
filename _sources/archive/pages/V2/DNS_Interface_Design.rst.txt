Summary
-------

This page present a summary of proposed and acknowledged design for
FreeIPA DNS module.

Motivation
----------

There are many structured DNS RR types where DNS data is not just a
scalar value, for example an IP address or a domain name, but a data
structure which may be often complex. A good example is a LOC record
[RFC1876] which consist of many mandatory and optional parts (degrees,
minutes, seconds of latitude and longtitude, altitude or precision).
This is an example of adding a LOC DNS record:

``ipa dnsrecord-add example.com @ --loc-rec "49 11 42.4 N 16 36 29.6 E 227.64m"``

It may be difficult to enter such DNS records to FreeIPA without making
an error which would lead in errors in *bind-dyndb-ldap* plugin which
process data from LDAP and passes them to BIND DNS server.

The new API shall present a convenient way to enter there record for
user without an obligation to know and apply all RFCs specifying DNS RR
types format.

Resolution
----------

IPA v3.0 will follow these steps to improve working with all these
complex steps

.. _new_per_type_structured_api:

1. New per-type structured API
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  New API shall be implemented for all supported resource record types

   -  It will take form of new options. Every part of DNS RR which will
      be set separately shall have its own option. For example, MX
      record have 2 parts: *PREFERENCE* and *EXCHANGER*. I.e. there
      should be 2 new options which can be used to create/modify MX
      record: *--mx-preference* and *--mx-exchanger*
   -  The list of all new options can be retrieved using standard
      *--help* option of chosen command

-  New API shall be integrated to current commands for dnsrecord
   manipulation. That is:

   -  *dnsrecord-show*
   -  *dnsrecord-add*
   -  *dnsrecord-mod*
   -  *dnsrecord-del*

-  All these commands should have a new interactive mode which should
   help with record manipulation. The interactive mode shall be started
   when the dnsrecord command is executed with no option.

-  DNSSEC RR types: DS, KEY, NSEC, RRSIG, SIG. These are not meant to be
   set manually and will be generated automatically in the future.

-  Unsupported RR types shall be removed from CLI at all: APL, DHCID,
   DLV, DNSKEY, HIP, IPSECKEY, NSEC3, NSEC3PARAM, RP, TA, TKEY, TSIG

-  The new API shall be implemented for both structured DNS RR types but
   also DNS RR types with scalar values to provide a base for other RR
   type related functionality. A good example may be a generation of
   IPv6 address from MAC:

``ipa dnsrecord-add example.com client1 --aaaa-from-mac=00:1D:BA:06:37:64``

.. _dnsrecord_add_command:

dnsrecord-add command
^^^^^^^^^^^^^^^^^^^^^

New per-type options can be used to add DNS RR, each part separately.
When any new option for a DNS RR type is detected, the command assumes
that the new API is used and all required DNS RR type options have to be
passed.

Adding of MX record using a standard ADD operation:

| ``ipa dnsrecord-add example.com @ --mx-rec="0 server1.example.com."``
| ``  Record name: example.com``
| ``  MX record: 0 server1.example.com.``
| ``  NS record: ns.example.com.``

Adding MX record using new API:

| ``ipa dnsrecord-add example.com @ --mx-preference=0 --mx-exchanger= server1.example.com.``
| ``  Record name: example.com``
| ``  MX record: 0 server1.example.com.``
| ``  NS record: ns.example.com.``

Adding MX record using new interactive help:

| ``ipa dnsrecord-add example.com @``
| ``Please choose a type of DNS resource record to be added``
| ``The most common types for this type of zone are: NS, MX, LOC``

.. _dnsrecord_mod_command:

dnsrecord-mod command
^^^^^^^^^^^^^^^^^^^^^

The command should offer an easy way to modify only specific part(s) of
a DNS record. It shall operate in 2 modes:

-  An old mode when just a new set of DNS RRs is passed, e.g.

``ipa dnsrecord-mod example.com @ --mx-rec="0 server1.example.com.","1 server2.example.com."``

-  A new mode where a specified DNS RR will be updated (per-part). This
   mode shall be triggered when any of the new option is passed. In this
   case, the old *---rec* will serve just as a pointer to the modified
   record:

``ipa dnsrecord-mod example.com @ --mx-rec="1 server2.example.com." --mx-preference=2``

A full example of old-style record modification:

| ``ipa dnsrecord-mod example.com @ --mx-rec="0 server1.example.com.","1 server2.example.com."``
| ``  Record name: example.com``
| ``  MX record: 0 server1.example.com., 1 server2.example.com.``
| ``  NS record: ns.example.com.``

A full example of a modification using the new API:

| ``ipa dnsrecord-mod example.com @ --mx-rec="1 server2.example.com." --mx-preference=2``
| ``  Record name: example.com``
| ``  MX record: 0 server1.example.com., 2 server2.example.com.``
| ``  NS record: ns.example.com.``

An example of new interactive mode:

| ``ipa dnsrecord-mod example.com @``
| ``No option to modify specific record provided.``
| ``Current DNS record contents:``
| ``MX record: 0 server1.example.com., 2 server2.example.com.``
| ``NS record: ns.example.com.``
| ``Modify MX record '0 server1.example.com.'? Yes/No (default No): ``
| ``Modify MX record '2 server2.example.com.'? Yes/No (default No): y``
| ``MX Preference [2]: 3``
| ``MX Exchanger [server2.example.com.]: ``
| ``Modify NS record 'ns.example.com.'? Yes/No (default No): ``
| ``  Record name: example.com``
| ``  MX record: 0 server1.example.com., 3 server2.example.com.``
| ``  NS record: ns.example.com.``

.. _dnsrecord_del_command:

dnsrecord-del command
^^^^^^^^^^^^^^^^^^^^^

Neither API nor the interactive mode need to be changed.

.. _improved_output:

Improved output
^^^^^^^^^^^^^^^

A new option *--structured* has been implemented which can be useful for
displaying more complex records:

| ``ipa dnsrecord-show example.com @ --structured``
| ``  Record name: @``
| ``  Records: ``
| ``    Record type: MX``
| ``    Record data: 0 server1.example.com.``
| ``    MX Preference: 0``
| ``    MX Exchanger: server1.example.com.``

| ``    Record type: MX``
| ``    Record data: 3 server2.example.com.``
| ``    MX Preference: 3``
| ``    MX Exchanger: server2.example.com.``

| ``    Record type: NS``
| ``    Record data: ns.example.com.``
| ``    NS Hostname: ns.example.com.``

The output then shows all record in a structured format including the
record type, raw DNS record data and an attribute for every part of the
DNS record.

.. _improved_validation:

2. Improved validation
~~~~~~~~~~~~~~~~~~~~~~

DNS record validation should be improved so that most common user errors
are detected and reported by IPA client and by *bind-dyndb-ldap* plugin
failing to serve the record.

A better help with a pointer to further information (RFC) should be
produced when validation fails:

| ``ipa dnsrecord-add example.com @ --mx-rec=BADRECORD``
| ``ipa: ERROR: invalid 'mx_rec': format must be specified as "PREFERENCE EXCHANGER"  (see RFC 1035 for details)``

| ``ipa dnsrecord-add example.com @ --loc-rec=BADRECORD``
| ``ipa: ERROR: invalid 'loc_rec': format must be specified as``
| ``    "d1 [m1 [s1]] {"N"|"S"}  d2 [m2 [s2]] {"E"|"W"} alt["m"] [siz["m"] [hp["m"] [vp["m"]]]]"``
| ``    where:``
| ``       d1:     [0 .. 90]            (degrees latitude)``
| ``       d2:     [0 .. 180]           (degrees longitude)``
| ``       m1, m2: [0 .. 59]            (minutes latitude/longitude)``
| ``       s1, s2: [0 .. 59.999]        (seconds latitude/longitude)``
| ``       alt:    [-100000.00 .. 42849672.95] BY .01 (altitude in meters)``
| ``       siz, hp, vp: [0 .. 90000000.00] (size/precision in meters)``
| ``    See RFC 1876 for details``
