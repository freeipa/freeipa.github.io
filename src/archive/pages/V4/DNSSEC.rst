Overview
--------

The Domain Name System Security Extensions (DNSSEC) technology is a set
of extensions to `DNS <DNS>`__ allowing clients to check denial of
existence and data integrity of the DNS query results.

Design
------

FreeIPA `4.0.0 <Releases/4.0.0>`__ introduced `experimental DNSSEC
implementation <Releases/4.0.0#Experimental_DNSSEC_Support>`__ which
provided only `minimal user
interface <https://fedorahosted.org/bind-dyndb-ldap/wiki/BIND9/Design/DNSSEC#FeatureManagement>`__
and depends on manual key management (done by administrator).

FreeIPA `4.1.0 <Releases/4.1.0>`__ and newer provides automatic key
management (`bind-dyndb-ldap's design
page <https://fedorahosted.org/bind-dyndb-ldap/wiki/BIND9/Design/DNSSEC/Keys/Shortterm>`__).
Disadvantage of this approach is that one replica is
single-point-of-failure (for key management). More information available
`here <V4/DNSSEC_Support>`__.



RFE Author
==========

`pspacek <User:Pspacek>`__ (`talk <User_talk:Pspacek>`__)
