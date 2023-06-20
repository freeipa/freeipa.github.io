Forward_zones
=============

Overview
--------

Forward zones allow to configure the name server to use forwarding only
for a particular zone.

In older versions of IPA, to allow forwarding per zone, was required to
create master zone with forward statement together with unnecessary
records. Current implementation allows separate forward and master zones
in FreeIPA `DNS <DNS>`__ to follow BIND design and achieve consistent
behavior.



Use Cases
---------



Forwarding queries per a zone
----------------------------------------------------------------------------------------------

Generally using global forwarders is all or nothing solution. Using
forward zones give more control to admin, which zones and how should be
forwarded.



Forwarding policy in forward and master zones
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Forwarding follows BIND design, here is short overview in table bellow,
for more information please read `BIND
manual <http://ftp.isc.org/isc/bind9/cur/9.9/doc/arm/Bv9ARM.ch06.html#id2583443>`__.

+-----------------------------------+-----------------------------------+
| IPA forward-policy                | behavior                          |
+===================================+===================================+
| none                              | #. search local database for an   |
|                                   |    authoritative answer           |
|                                   | #. if local server is             |
|                                   |    authoritative, return the      |
|                                   |    answer (including NXDOMAIN if  |
|                                   |    DNS name was not found)        |
|                                   | #. if local server is not         |
|                                   |    authoritative, try only        |
|                                   |    recursion (if recursion is     |
|                                   |    allowed), no forwarding        |
|                                   | #. return an answer obtained by   |
|                                   |    recursion                      |
+-----------------------------------+-----------------------------------+
| first (or as default)             | #. search local database for an   |
|                                   |    authoritative answer           |
|                                   | #. if local server is             |
|                                   |    authoritative, return the      |
|                                   |    answer (including NXDOMAIN if  |
|                                   |    DNS name was not found)        |
|                                   | #. if local server is not         |
|                                   |    authoritative, try to query    |
|                                   |    forwarders                     |
|                                   | #. if query to forwarders failed, |
|                                   |    try recursion (if recursion is |
|                                   |    allowed)                       |
|                                   | #. return an answer from          |
|                                   |    forwarder or recursion         |
+-----------------------------------+-----------------------------------+
| only                              | #. search local database for an   |
|                                   |    authoritative answer           |
|                                   | #. if local server is             |
|                                   |    authoritative, return the      |
|                                   |    answer (including NXDOMAIN if  |
|                                   |    DNS name was not found)        |
|                                   | #. if local server is not         |
|                                   |    authoritative, try to query    |
|                                   |    forwarders                     |
|                                   | #. return an answer from          |
|                                   |    forwarders or return SERVFAIL  |
|                                   |    (if designated forwarders did  |
|                                   |    not reply)                     |
+-----------------------------------+-----------------------------------+



Avoid an ineffective forward zone
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Be careful using following set of zones, where a master zone exists for
a forward zone.

| ``$ ipa dnszone-add example.com.``
| ``$ ipa dnsforwardzone-add  fw.example.com. --forwarder 192.0.2.1``

In this case, forwarding will not work, because zone *example.com.* is
authoritative, and server return NXDOMAIN answer. To make forward zone
effective, you need delegate this forward domain using NS record to
different name server.

To add proper NS record use following command:

``$ ipa dnsrecord-add example.com. fw --ns-rec=``

Design
------

No changes for master zones required ('idnsZone' objectclass).

Forward zone will use own objectclass 'idnsForwardZone'. (see below).

| ``objectClass ( 2.16.840.1.113730.3.8.6.3``
| ``   NAME 'idnsForwardZone'``
| ``   DESC 'Forward Zone class'``
| ``   SUP top``
| ``   STRUCTURAL``
| ``   MUST ( idnsName $ idnsZoneActive )``
| ``   MAY ( idnsForwarders $ idnsForwardPolicy ) )``

The idnsName, idnsZoneActive, idnsForwarders, idnsForwardPolicy are the
same attributes as the 'idnsZone' objectclass uses.

Both types are stored in LDAP, under ``idnsname=$DOMAIN,cn=dns,$SUFFIX``
Only one zone type can be stored under the same name. (To add forward
zone with same name you must remove master zone and vice versa)

Forward zones are enabled by default.



Forward Policy
----------------------------------------------------------------------------------------------

``forward-policy = {first, only, none}``, default: **first**

-  **none** - disable forwarding, forwarders are not required
-  **only** - forward queries to forwarders (**require** to specify
   **forwarders**)
-  **first** - forward queries, if unanswered try to answer the query
   (**require** to specify **forwarders**)

+-----------------------------------+-----------------------------------+
| IPA                               | Bind                              |
+===================================+===================================+
| none                              | | ``zone "fwd.test." in {``       |
|                                   | | ``  type forward;``             |
|                                   | | ``  forwarders { };``           |
|                                   | | ``};``                          |
+-----------------------------------+-----------------------------------+
| first (or as default)             | | ``zone "fwd.test." in {``       |
|                                   | | ``  type forward;``             |
|                                   | | ``  forwarders { ``\ `` };``    |
|                                   | | ``};``                          |
+-----------------------------------+-----------------------------------+
| only                              | | ``zone "fwd.test." in {``       |
|                                   | | ``  type forward;``             |
|                                   | | ``  forward only;``             |
|                                   | | ``  forwarders { ``\ `` };``    |
|                                   | | ``};``                          |
+-----------------------------------+-----------------------------------+



New IPA commands
----------------------------------------------------------------------------------------------

-  ``dnsforwardzone-add``, to add a new forward zone
-  ``dnsforwardzone-mod``, to modify a forward zone
-  ``dnsforwardzone-show``, to show a forward zone
-  ``dnsforwardzone-find``, to find a forward zone(s)
-  ``dnsforwardzone-del``, to delete a forward zone(s)
-  ``dnsforwardzone-enable``, to enable a forward zone(s)
-  ``dnsforwardzone-disable``, to disable a forward zone(s)
-  ``dnsforwardzone-add-permission``, to add the permission for per
   forward zone access delegation
-  ``dnsforwardzone-remove-permission``, to remove the permission for
   per forward zone access delegation



Feature Management
------------------

UI

A new page *Network Services/DNS/DNS Forward Zones* in WebUI. This page
handle all required operations: show current list of forward zones, add
a new forward zone, delete a forward zone, display a forward zone,
allows to modify forwarders and forward policy per a forward zone,
disable/enable a forward zone.

Forward zone consists of a name, forwarders, forwarding policy, and
enabled/disabled status

CLI



dnsforwardzone-\*
^^^^^^^^^^^^^^^^^

Args ``--forwarder``, ``--forward-policy``, ``--name-from-ip`` have same
behavior as they have in dnszone-\* commands.

Forward zone name has same restrictions as in the master zone
(dnszone-\*).



dnsforwardzone-add
^^^^^^^^^^^^^^^^^^

will add a new forward zone. Is required to specify at least one
forwarder if forward-policy is not 'none'.

| ``dnsforwardzone-add zone.test. --forwarder=172.16.0.1 --forwarder=172.16.0.2 --forward-policy=first``
| ``  Zone name: zone.test.``
| ``  Zone forwarders: 172.16.0.1, 172.16.0.2``
| ``  Forward policy: first``



dnsforwardzone-mod
^^^^^^^^^^^^^^^^^^

will modify a forward zone. Is required to specify at least one
forwarder if forward-policy is not 'none'. Modifications can be
performed in several ways.

| ``dnsforwardzone-mod zone.test. --forwarder=172.16.0.3``
| ``  Zone name: zone.test.``
| ``  Zone forwarders: 172.16.0.3``
| ``  Forward policy: first``

| ``dnsforwardzone-mod zone.test. --forward-policy=only``
| ``  Zone name: zone.test.``
| ``  Zone forwarders: 172.16.0.3``
| ``  Forward policy: only``



dnsforwardzone-show
^^^^^^^^^^^^^^^^^^^

will show specified forward zone

| ``dnsforwardzone-show zone.test.``
| ``  Zone name: zone.test.``
| ``  Zone forwarders: 172.16.0.5``
| ``  Forward policy: first``



dnsforwardzone-find
^^^^^^^^^^^^^^^^^^^

will find specified forward zone

| ``dnsforwardzone-find zone.test.``
| ``  Zone name: zone.test.``
| ``  Zone forwarders: 172.16.0.3``
| ``  Forward policy: first``
| ``----------------------------``
| ``Number of entries returned 1``
| ``----------------------------``



dnsforwardzone-del
^^^^^^^^^^^^^^^^^^

will delete specified forward zone(s)

::

   | ``dnsforwardzone-del zone.test. ``
   | ``----------------------------``
   | ``Deleted forward DNS zone "zone.test."``
   | ``----------------------------``



dnsforwardzone-enable
^^^^^^^^^^^^^^^^^^^^^

will enable specified forward zone(s) NOTE: Forward zones are enabled by
default.

::

   | ``dnsforwardzone-enable zone.test. ``
   | ``----------------------------``
   | ``Enabled forward DNS zone "zone.test."``
   | ``----------------------------``



dnsforwardzone-disable
^^^^^^^^^^^^^^^^^^^^^^

will disable specified forward zone(s)

::

   | ``dnsforwardzone-disable zone.test. ``
   | ``----------------------------``
   | ``Disabled forward DNS zone "zone.test."``
   | ``----------------------------``



dnsforwardzone-add-permission
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

will add system permission

| ``dnsforwardzone-add-permission zone.test.``
| ``---------------------------------------------------------``
| ``Added system permission "Manage DNS zone zone.test."``
| ``---------------------------------------------------------``
| ``  Manage DNS zone zone.test.``



dnsforwardzone-remove-permission
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

will remove system permission

| ``dnsforwardzone-remove-permission zone.test.``
| ``---------------------------------------------------------``
| ``Removed system permission "Manage DNS zone zone.test."``
| ``---------------------------------------------------------``
| ``  Manage DNS zone zone.test.``



Updates and Upgrades
--------------------

-  idnsForwardZone objectclass is already supported by bind-dyndb-ldap
   >= 3.5. This covers also RHEL/CentOS >= 7.0 so upgrades involving
   only RHEL 7.x machines are seamless.

-  Unfortunatelly, we did not realize that interaction with RHEL/CentOS
   < 7.0 && RHEL/CentOS >= 7.1 in the same topology will not be
   seamless. (See `bug
   1175318 <https://bugzilla.redhat.com/show_bug.cgi?id=1175318>`__.)

   -  RHEL 7.1 ships bind-dyndb-ldap >= 6.0 which relies on new object
      semantics which is not supported by bind-dyndb-ldap 2.3 shipped
      with RHEL 6.6. As a result, forward zones will stop working on old
      replicas as soon as RHEL 7.1 upgrade procedure is executed.
   -  Solution is to patch bind-dyndb-ldap in RHEL 6.6.z and add support
      for idnsForwardZone object class to it (see `bug
      1176129 <https://bugzilla.redhat.com/show_bug.cgi?id=1176129>`__).
      It will allow RHEL 6.6.z+ replicas to continue to work after RHEL
      7.1+ is joined to the topology.
   -  Assumption is that from a moment of upgrade to RHEL 7.1+ on all
      forward zones are managed from RHEL 7.1+ replicas (so the data are
      stored in the new format).

-  add idnsForwardZone objectclass to LDAP schema

-  All zones with configured forwarders and forward-policy not equal to
   none, will be moved to idnsForwardZone objectclass, and idnsZone
   class will be removed. First, the zones will be exported to LDIF as
   backup in **/var/lib/ipa/backup/** directory, named as
   **dns-forward-zones-backup-%Y-%m-%d-%H-%M-%S.ldif**

.. table:: Migration table

   ============== ====== ======= ======= =======
   forward-policy none   first   only    
   ============== ====== ======= ======= =======
   forwarders     master forward forward forward
   no forwarders  master master  master  master
   ============== ====== ======= ======= =======

-  Transformation to forward zones, is executed only once, by one
   replica only, and only if ipa version is lower than 4.0. This is
   ensured by detection: if 'idnsforwardzone' objectclass is presented
   in schema before schema upgrade, then no transformation is required,
   else transform master zone to forward zone using rules above.



How to Test
-----------



Basic configuration
----------------------------------------------------------------------------------------------

#. install *IPA server* with DNS, do not set up forwarders
#. Set up an *external DNS server* (IP: 192.0.2.200)
#. Configure zone *example.test.* on *external DNS server*
#. Add A record *host.example.test. IN A 192.0.2.111* into zone
   *example.test.* on *external DNS server*



Test a forward zone with forwarding only policy
----------------------------------------------------------------------------------------------

#. use the basic configuration above
#. test zone *example.test* using dig: **$ dig @ A host.example.test.**
#. expected result: NXDOMAIN
#. add forward zone on *IPA server*: **$ ipa dnsforwardzone-add
   example.test. --forward-policy=only --forwarder=192.0.2.200**
#. test zone *example.test* using dig: **$ dig @ A host.example.test.**
#. expected result: *host.example.test. IN A 192.0.2.111* record in the
   answer, AUTHORITY SECTION is pointing to *external DNS server*



Test a forward zone with forwarding none policy
----------------------------------------------------------------------------------------------

#. use the basic configuration above
#. test zone *example.test* using dig: **$ dig @ A host.example.test.**
#. expected result: NXDOMAIN
#. add global forwarder (external DNS server): **ipa dnsconfig-mod
   --forwarder=192.168.2.200**
#. test zone *example.test* using dig: **$ dig @ A host.example.test.**
#. expected result: *host.example.test. IN A 192.0.2.111* record in the
   answer, AUTHORITY SECTION is pointing to *external DNS server*
#. add forward zone with none policy: **$ ipa dnsforward-zone
   example.test. --forward-policy=none**
#. test zone *example.test* using dig: **$ dig @ A host.example.test.**
#. expected result: NXDOMAIN



Test Plan
---------



Unit tests
----------------------------------------------------------------------------------------------

-  Create forward zone:

   -  **dnsforwardzone-add fw-zone**

      -  Expectation: missing forwarders, ValidationError

   -  **dnsforwardzone-add fw-zone --forward-policy=only**

      -  Expectation: missing forwarders, ValidationError

   -  **dnsforwardzone-add fw-zone --forward-policy=none**

      -  Expectation: add fw-zone with policy none, no forwarders

   -  **dnsforwardzone-add fw-zone --forwarder=172.16.15.1**

      -  Expectation: add fw-zone with policy first, forwarder
         172.16.15.1

   -  **dnsforwardzone-add fw-zone --forwarder=172.16.15.1
      --forward-policy=only**

      -  Expectation: add fw-zone with policy only, forwarder
         172.16.15.1

   -  **Try to add duplicated zone**

      -  Expectation: DuplicationError

-  Modify forward zone

   -  **dnsforwardzone-mod fw-zone-without-forwarders
      --forward-policy=only**

      -  Expectation: missing forwarders, ValidationError

   -  **dnsforwardzone-mod fw-zone-without-forwarders
      --forward-policy=first**

      -  Expectation: missing forwarders, ValidationError

   -  **dnsforwardzone-mod fw-zone-policy-none
      --forwarder={172.16.15.1,172.16.15.2}**

      -  Expectation: zone policy=none, forwarders: 172.16.15.1,
         172.16.15.2

   -  **dnsforwardzone-mod fw-zone-with-forwarders
      --forward-policy=first**

      -  Expectation: zone policy=first, forwarders=

   -  **dnsforwardzone-mod fw-zone-with-forwarders
      --forward-policy=only**

      -  Expectation: zone policy=only, forwarders=

-  Show forward zone

   -  **dnsforwardzone-show fw-zone**

      -  Expectation: retrieve zone

-  Find forward zone

   -  **dnsforwardzone-find**

      -  Expectation: show all forward zones matching expression

-  Disable/Enable forward zone

   -  **dnsforwardzone-enable fw-zone**

      -  Expectation: fw-zone becomes enabled

   -  **dnsforwardzone-disable fw-zone**

      -  Expectation: fw-zone becomes disabled

-  Add/Remove per-zone permission

   -  **dnsforwardzone-add-permision fw-zone**

      -  Expectation: create system permission for fw-zone

   -  **dnsforwardzone-remove-permission fw-zone**

      -  Expectation: remove system permission for fw-zone

-  Delete forward zone

   -  dnszone-del fw-zone

      -  Expectation: Zone is removed

-  Mutual exclusion with master zones (\*-add)

   -  **dnszone-add zone-exists-as-forward**

      -  Expectation: DuplicateError

   -  **dnsforwardzone-add zone-exists-as-master**

      -  Expectation: DuplicateError

-  Mutual exclusion with master zones (\*-find)

   -  **dnszone-find**

      -  Expectation: Lists ONLY master zones

   -  **dnsforwardzone-find**

      -  Expectation: LIsts ONLY forward zones

-  Mutual exclusion with master zones (others)

   -  **dnszone-\* forward-zone**

      -  Expectation: NotFound Error

   -  **dnsforwardzone-\* master-zone**

      -  Expectation: NotFound Error

-  Prevent dnsrecord-\* commands work with forwardzone

   -  **dnsrecord-\* forward-zone**

      -  Expectation: ValidationError: only master zones can contain
         records



RFE Author
----------

`mbasti <User:Mbasti>`__

`Category:FreeIPA V4 Test Plan <Category:FreeIPA_V4_Test_Plan>`__
`Category:FreeIPA Test Plan <Category:FreeIPA_Test_Plan>`__