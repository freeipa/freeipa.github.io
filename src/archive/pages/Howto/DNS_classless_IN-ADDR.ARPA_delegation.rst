

Classless IN-ADDR.ARPA delegation
=================================

This page page explains how to deal with ``in-addr.arpa.`` DNS sub-tree
when you use
`classless <http://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing>`__
networks.

The main idea comes from `RFC
2317 <http://tools.ietf.org/html/rfc2317>`__ and is not specific to
FreeIPA in any way. FreeIPA just provides user interface but all
standard DNS principles apply.

NOTE: this HOWTO as it is does not work with FreeIPA 3.2 or later
anymore due to commit 42c401a87795fe3a2067155460ae276ad2d3e360 which
removed ability to have CNAME+PTR record co-existence.

Theory
------

-  DNS clients always look for names like ``w.x.y.z.in-addr.arpa``
   because clients can't possibly know how networks are partitioned.
-  The main trick is to use CNAME/DNAME records to redirect clients from
   ``w.x.y.z.in-addr.arpa`` to some other name. The target name can be
   arbitrary DNS name, i.e. the name can belong to different zone and
   normal delegation rules will apply.
-  A common practice is to create auxiliary sub-domains in
   ``in-addr.arpa.`` sub-tree but it is not strictly required.

Example
-------

-  We want to delegate zone for classless network: ``198.51.100.0/26``

   -  We want to host records for this (smaller) network on server
      ``ipa1.example.``.

-  Nearest classfull network (with netmask at byte boundary) is:
   ``198.51.100.0/24``

   -  Records for this network should be hosted on server
      ``ipa2.example.``.



On server ipa2.example.
----------------------------------------------------------------------------------------------

-  Create auxiliary zone ``0/26.100.51.198.in-addr.arpa.``:

   -  ``ipa dnszone-add 0/26.100.51.198.in-addr.arpa.``

-  Add all PTR records to this new zone:

   -  ``ipa dnsrecord-add 0/26.100.51.198.in-addr.arpa. 1 --ptr-rec=my.machine.example.``
   -  ... (add all other records as necessary)
   -  ``ipa dnsrecord-add 0/26.100.51.198.in-addr.arpa. 62 --ptr-rec=another.machine.example.``



On server ipa1.example.
----------------------------------------------------------------------------------------------

-  Create classfull zone ``100.51.198.in-addr.arpa.``:

   -  ``ipa dnszone-add 100.51.198.in-addr.arpa.``

-  Add CNAMEs for all PTR records belonging to the smaller zone:

   -  ``ipa dnsrecord-add 100.51.198.in-addr.arpa. 1 --cname-rec=1.0/26``
   -  ... (add all other records as necessary)
   -  ``ipa dnsrecord-add 100.51.198.in-addr.arpa. 62 --cname-rec=62.0/26``

-  Delegate sub-zone ``0/26.100.51.198.in-addr.arpa.`` from
   ``ipa2.example.`` to ``ipa1.example.``:

   -  ``ipa dnsrecord-add 100.51.198.in-addr.arpa. 0/26 --ns-rec=ipa1.example.``

Testing
----------------------------------------------------------------------------------------------

Following command should always return the same results, no matter what
DNS server you queried:

::

   $ dig -t PTR 1.100.51.198.in-addr.arpa.

   ;; QUESTION SECTION:
   ;1.100.51.198.in-addr.arpa. IN  PTR

   ;; ANSWER SECTION:
   1.100.51.198.in-addr.arpa. 86400 IN CNAME   1.0/26.100.51.198.in-addr.arpa.
   1.0/26.100.51.198.in-addr.arpa. 86400 IN PTR    my.machine.example.
