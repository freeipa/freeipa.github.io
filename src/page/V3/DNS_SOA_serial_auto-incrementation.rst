DNS_SOA_serial_auto-incrementation
==================================

Overview
========

*Short overview of the problem set and any back ground material or
references one would need to understand the details.*

Each DNS zone has *serial number*, i.e. version number of the zone. Nice
explanation can be found e.g. in `Zytrax book about DNS, chapter
8 <http://www.zytrax.com/books/dns/ch8/soa.html>`__.

Quotations from `Zytrax DNS book <http://www.zytrax.com/books/dns/>`__:

   sn = serial number

   This value MUST increment when any resource record in the zone file
   is updated.

   A slave (Secondary) DNS server will read the master DNS SOA record
   periodically, either on expiry of refresh (defined below) or when it
   receives a NOTIFY and compares arithmetically its current value of sn
   with that received from the master DNS. If the sn value from the
   master is arithmetically HIGHER than that currently stored by the
   slave then a zone transfer (AXFR/IXFR) is initiated. If the value of
   sn from the master DNS SOA is the same or LOWER then no zone transfer
   is initiated.

..

   Note: the arithmetic used by the serial number is defined in `RFC
   1982 <http://tools.ietf.org/html/rfc1982>`__.



Structure of DNS sub-tree in LDAP
---------------------------------

Please get familiar with `the way how DNS data are stored in
LDAP <https://fedorahosted.org/bind-dyndb-ldap/wiki/DatabaseStructure>`__.



Use Cases
=========

Correct serial number is a requirement for functional zone tranfers.
Corrent zone transfers from IPA to non-IPA DNS server are impossible if
zone serial number is not incremented after each change.



Design in IPA 3.0
=================

Design was discussed on freeipa-devel mailing list: At the beginning (in
`e-mail thread from April
2012 <http://www.redhat.com/archives/freeipa-devel/2012-April/msg00222.html>`__)
various options were discussed. Maintaining single global SOA serial for
all servers was found as hard problem. Attributes like
``modifyTimestamp`` can't be used because they can go back in time
(because of replication).

`In May
2012 <http://www.redhat.com/archives/freeipa-devel/2012-May/msg00047.html>`__
different approach was proposed: Maintain SOA serial separately for each
IPA server and do not synchronize serials between IPA servers at all.

This design is significantly simpler and was implemented in IPA 3.0.

Advantages
----------

-  It is relatively simple to implement.
-  No DS plugin is required, all the logic is part of bind-dyndb-ldap
   plugin for BIND.

Disadvantages
-------------

Slave DNS servers cannot fall-back to other masters, because of SOA
serial inconsistency.



Very basic implementation
-------------------------

#. Do not replicate ``idnsSOASerial`` attribute between IPA servers: Use
   `nsDS5ReplicatedAttributeList <https://access.redhat.com/knowledge/docs/en-US/Red_Hat_Directory_Server/9.0/html/Administration_Guide/fractional-repl-total.html>`__
   attribute to exclude ``idnsSOASerial`` from replication.
#. Use persistent search to watch for incoming changes.
#. After each change increment "local" SOA serial number (and write it
   to LDAP - to survive DNS server restart).
#. Always increment stored SOA serial number after DNS server start.
   Data in LDAP could be changed when bind-dyndb-ldap didn't listen on
   persistent search.

Implementation was done in the scope of `IPA ticket
#2554 <https://fedorahosted.org/freeipa/ticket/2554>`__ and
`bind-dyndb-ldap ticket
#67 <https://fedorahosted.org/bind-dyndb-ldap/ticket/67>`__.



Problems found after implementation to IPA 3.0
----------------------------------------------

Problem described in following text appears only in environments with at
least one IPA replica.

IPA command stores newly added zone to one chosen LDAP server, including
some initial value of ``idnsSOAserial`` attribute. The new zone is
replicated to other LDAP servers, except ``idnsSOAserial`` attribute.
This behaviour creates an inconsistent state, where mandatory attribute
``idnsSOAserial`` is not present in some LDAP databases.

IPA tools and bind-dyndb-ldap are not ready for state where mandatory
``idnsSOAserial`` attribute is missing.

-  bind-dyndb-ldap: `Red Hat bug
   895083 <https://bugzilla.redhat.com/show_bug.cgi?id=895083>`__,
   `bind-dyndb-ldap ticket
   #103 <https://fedorahosted.org/bind-dyndb-ldap/ticket/103>`__,
   `hot-fix <http://git.fedorahosted.org/cgit/bind-dyndb-ldap.git/commit/?id=5fcfb292ca07d0aa3a0d1a87baf2f6b35336dba2>`__
-  IPA: `Red Hat bug
   894143 <https://bugzilla.redhat.com/show_bug.cgi?id=894143>`__
   `FreeIPA ticket
   #3341 <https://fedorahosted.org/freeipa/ticket/3341>`__, `Red Hat bug
   894131 <https://bugzilla.redhat.com/show_bug.cgi?id=894131>`__
   `FreeIPA ticket
   #3340 <https://fedorahosted.org/freeipa/ticket/3340>`__,
   `hot-fix <https://fedorahosted.org/freeipa/changeset/55bace6546095d78760be413896c824efe9c2f20/>`__

According to `conversation on
389-users-list <http://lists.fedoraproject.org/pipermail/389-users/2013-January/015436.html>`__
389 DS is not able to replicate data during ADD operations and don't
replicate data during MOD operations. Rich Megginson proposed another
solution:

   So ``idnsSOAserial`` is a "local only" attribute that lives in a
   "global" entry.

   I agree that this is an attribute that should be managed internally
   by the directory server, like the entryusn attribute.



Feature Managment
-----------------



Major configuration options and enablement
----------------------------------------------------------------------------------------------

SOA serial is managed internally by bind-dyndb-ldap plugin. New boolean
option ``serial_autoincrement`` was added to ``/etc/named.conf``. Value
``yes`` enables this feature.

There is no command for enabling/disabling this feature.

WebUI~

WebUI doesn't manage this option.

CLI

New option for ``ipa-server-install``:

| ``   --no-serial-autoincrement``
| ``                       Do not enable SOA serial autoincrement``

This feature is enabled by default. Persistent search is required for
SOA serial auto-increment feature, so ``--no-persistent-search`` can't
be used without ``--no-serial-autoincrement``.

``ipa dnszone`` commands are unchanged:

-  It is possible to show SOA serial value. Shown value is read from
   single LDAP server. This value can be different on other LDAP
   servers.
-  Any modification via ``ipa dnszone-mod`` affects only single LDAP
   server (i.e. server which was selected by IPA CLI for particular
   operation).

Replication
-----------

Zone attribute ``idnsSOASerial`` is not replicated between IPA servers.
``idnsSOASerial`` is added to the ``nsDS5ReplicatedAttributeList``
attribute inside each replication agreement.

Each write to ``idnsSOAserial`` can potentially trigger same problem as
described in `IPA ticket
#2534 <https://fedorahosted.org/freeipa/ticket/2534>`__.



Updates and Upgrades
--------------------

-  Option ``serial_autoincrement yes`` has to be added to
   ``/etc/named.conf``.
-  Persistent search is required for SOA serial auto-increment feature,
   so ``psearch`` option has to be switched to ``yes``.

Dependencies
------------

-  bind-dyndb-ldap version >= 2.0 is required.



External Impact
---------------

(Hopefully) none.



Design in IPA 3.1
=================

Move SOA serial maintenance from bind-dyndb-ldap to (new) 389 DS plugin:
`IPA ticket #3347 <https://fedorahosted.org/freeipa/ticket/3347>`__.

DS plugin watches ``cn=dns`` sub-tree for changes.

Any change in DNS record in this subtree will increment
``idnsSOAserial`` attribute in record's parent zone.



Basic idea
----------

| ``if objectClass is idnsZone``
| ``    increment idnsSOAserial in the same object``
| ``else if objectClass is idnsRecord``
| ``    increment idnsSOAserial in object's immediate parent``
| ``    e.g. change in idnsName=test, idnsName=example.com, cn=dns will increment idnsSOAserial in object idnsName=example.com, cn=dns``
| ``    if parent's objectClass is not idnsZone``
| ``         log an error (This should never happen :-))``
| `` else``
| ``    do nothing``



SOA serial incrementation algorithm
-----------------------------------

| ``OLDSerial = actual idnsSOAserial value``
| ``timestamp = actual UNIX timestamp``
| ``if (OLDSerial < timestamp)``
| ``    newSerial = timestamp``
| ``else``
| ``   newSerial = OLDSerial + 1``
| ``Write newSerial value to particular idnsSOAserial attribute``



Interaction with BIND serial update mechanism
---------------------------------------------

BIND does direct SOA serial update (not trigerred by serial
autoincrement feature) after any dynamic update. We have to catch those
attempts and ignore them:

-  A plugin can intercept any modify and manipulate it, including
   suppressing changes to SOA Serial.
-  It should be possible to catch & discard SOA serial modifications
   inside BIND. This will save some load from LDAP server.



Possible optimization
---------------------

Increment serial value at most once per second.

**Problem**: How to solve LDAP server crash?

Problematic scenario:

::

   (numbers represent time in "second.millisecond" format)

   1.000 : new_serial = time() + 1
   1.100 : record test.example.com. updated
   1.100 : zone serial overwritten with new_serial
   1.500 : zone transfer started
   1.500 : search result for all records and zone serial stored
   1.500 : search result is transferred to slaves
   1.700 : record test2.example.com. updated
   1.700 : zone serial overwritten with new_serial
   <no changes from now>

Result: Zones on master and it's slave servers have serial =
"new_serial" but the zone content is different (records under
test2.example.com. are not equal).



Variant with delayed serial write
----------------------------------------------------------------------------------------------

| ``When updating zone serial:``
| ``if (old serial value < time())``
| ``   cancel scheduled serial write (if exists)``
| ``   write zone serial = time()``
| ``else``
| ``   schedule serial bump after 1 second``
| ``   (do nothing if bump is scheduled already)``

| ``When starting Directory Server:``
| ``Bump each serial by one.``



Variant with modified search operation
----------------------------------------------------------------------------------------------

Modify search operation for zone serial to return:

| ``if (serial value == time())``
| ``   return (serial value - 1)``
| ``else``
| ``   return (serial value)``

::

   Scenario:
   1.000 : new_serial = time()
   1.100 : record test.example.com. updated
   1.100 : zone serial overwritten with new_serial
   1.500 : zone transfer started
   1.500 : search result for all records and zone serial stored
   1.500 : zone serial in search result is (new_serial - 1)
   1.500 : snapshot is transferred to slaves
   1.700 : record test2.example.com. updated
   1.700 : zone serial overwritten with new_serial
   <no changes from now>
   2.000 : now the search for serial value returns new_serial, i.e. slaves see value incremented by one from last zone transfer (1.500)
   9.000 : serial value is unchanged from last search

**Expected result:** Zone data can be inconsistent between master and
slaves for only one second. Data will be consistent if directory server
crashed at 1.701 - new zone transfer can be initiated after server
restart.

**Requirement:** DS plugin have to modify serial value during reads.

**Problem:** It is hard to intercept and modify search operation.

Implementation
--------------

Any additional requirements or changes discovered during the
implementation phase.



Feature Management
------------------

-  Add new option like ``serial_remote`` to ``/etc/named.conf``. This
   option should be mutually exclusive with ``serial_autoincrement``
   option from IPA 3.0.
-  Do not create UI for enabling/disabling this feature. We can provide
   some boolean directly in plugin configuration, but nothing else.



Replication
-----------

No change from IPA 3.0.



Updates and Upgrades
--------------------

Replace ``serial_autoincrement`` option in ``/etc/named.conf`` with
``serial_remote`` option.



Dependencies
------------

New version of bind-dyndb-ldap + the new 389 DS plugin.



External Impact
---------------

Hopefully none.



Impact on testing
-----------------

Zone serial should be incremented after each change. Delay between
record change and serial change should be at most 1 second.

In IPA 3.0 there was some delay, but prediction for 3.0 is harder.