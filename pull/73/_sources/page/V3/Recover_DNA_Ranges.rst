Recover_DNA_Ranges
==================

\__NOTOC_\_

Overview
========

IPA uses the 389-ds Distributed Numeric Assignment (DNA) plugin to
automatically manage POSIX uid/gid assignment. This plugin manages the
range of available IDs cross all masters.
https://access.redhat.com/knowledge/docs/en-US/Red_Hat_Directory_Server/8.1/html/Configuration_and_Command_Reference/dna-attributes.html

When a master is deleted then its range is not recovered, potentially
leaving a huge hole in the number of available entries. For example, the
first master gets half the range. The second gets the other half. If
either of these is deleted, or replaced, then half the available range
is gone as well.

A master does not receive a range until the DNA plugin is first used, so
it is perfectly possible for a master to have no defined range.



Use Cases
=========



A master has run out of IDs
---------------------------

The simplest case is that a master has exhaused its range and its
request for more was not fulfilled.

The admin is going to need to evaluate the current range usage and
re-adjust as necessary. This may involve manually splitting an existing
range or extending past the initial 200k entries we configure.



3-way IPA MMR, delete one, lose half the range
----------------------------------------------

Install initial server with --idstart=100000 --idmax=102000 giving a
total range of 2k users (and groups). Install two additional masters
from this master (so A->B and A->C) and create a user on B and C.

Start with two masters:

master A:

-  dnaNextValue: 100000
-  dnaMaxValue: 101000

master B:

-  dnaNextValue: 101002
-  dnaMaxValue: 102000

Note that master A original had the full range. After master B is added
the max went to 101000).

master C:

-  dnaNextValue: 100502
-  dnaMaxValue: 101000

master A dnaMaxValue goes to 100500.

If master B is deleted we lose half the range which will quickly lead to
the previous use case.



Dead master, lose the range
---------------------------

A master has died for some reason and it will be deleted. We perhaps
can't know the range, or can't connect to the server to detect the
range. In this case the range will be lost and a tool will be required
to manually recover the range.

Design
======

This solution will consist of two parts:

#. Attempt to recover any lost ranges into the "on-deck" range of
   another master
#. Provide a tool to manage the current and on-base ranges of masters

The on-deck range is used internally by DNA when a range transfer takes
place. If a replica hits the configured low-water mark (the
"dnaThreshold" attribute), it will request a range transfer from another
replica. It might still have some active range values left to use, so it
puts the transferred range on-deck and swaps it to the active range once
the active range is fully exhausted. When you are trying to choose a
server to reassign the deleted replica range to, you should make sure it
doesn't already have another range on-deck.

The on-deck range is defined by the "dnaNextRange" attribute, which
takes a value in the form "-" ("500-1000" would be an example).

The tool will be an extension to ipa-replica-manage to list and modify
the on-deck range. I don't believe we want to allow modifying the active
range itself.



Additional ACI Permissions
--------------------------

We will need to write a new ACI to allow admins to write a subset of DNA
attributes, including dnaNextRange, dnaNextValue and dnaMaxValue. This
will look like:

::

   dn: cn=Posix IDs,cn=Distributed Numeric Assignment Plugin,cn=plugins,cn=config
   changetype: modify
   add: aci
   aci: (targetattr=dnaNextRange || dnaNextValue || dnaMaxValue)(version 3.0;acl "permission:Modify DNA Range";allow (write) groupdn = "ldap:///cn=Modify DNA Range,cn=permissions,cn=pbac,$SUFFIX";)

A new permission will need to be added and included in the privilege
'Replication Administrators'.



Setting the on-deck range
-------------------------

In the del operation of ipa-replica-manage we will query the values of
dnaNextValue and dnaMaxValue from the server to be removed. We will then
walk the rest of the masters. The first one with no dnaNextRange value
will have it added in the form of dnaNextValue-dnaMaxValue.

If no servers have an empty dnaNextRange we will WARN about the range of
values that will be lost and inform the user of the command to manage
it.

We will NOT try to modify the current DNA range configuration.



Determining if a replica has a range
------------------------------------

Identifying the on-deck range is easy, use dnaNextRange.

To see if a range has been assigned, the replication agreement is by
default pre-configured with an exhausted range with dnaNextValue = 1101
and the dnaMaxValue = 1100.

Note too that one can also look in the dnaSharedCfgDN value for the
replica, something like:

``dnaHostname=dart.example.com+dnaPortNum=389,cn=posix-ids,cn=dna,cn=ipa,cn=etc,dc=example,dc=com``

dnaRemainingValues will be 0.



Enhancing ipa-replica-manage
----------------------------

Four new commands will be added, in sets of two: dnarange-show and
dnarange-set and dnanextrange-show and dnanextrange-set.

::

   # ipa-replica-manage dnarange-show
   masterA.example.com: 100-500
   masterB.example.com: 500-1000
   masterC.example.com: No range set

::

   # ipa-replica-manage dnarange-show masterA.example.com
    masterA.example.com: 100-500

::

   # ipa-replica-manage dnarange-set masterA.example.com 250-499

::

   # ipa-replica-manage dnanextrange-show
   masterA.example.com: 1001-1500
   masterB.example.com: No on-deck range set
   masterC.example.com: No on-deck range set

::

    # ipa-replica-manage dnanextrange-show masterA.example.com
    masterA.example.com: 1001-1500

::

   # ipa-replica-manage dnanextrange-set masterB.example.com 1001-5000

A show on no specific host will show them all. A show on a specific host
will show only that host.

The list of masters comes from cn=masters.

When any range is set we will need to make sure that it does not overlap
with existing DNA ranges AND any existing on-deck ranges.

Setting a range of 0-0 in an on-deck range deletes the range (removes
the attribute altogether). We do not allow removing the main DNA range.

NOTE: We will need to be clear that this range has nothing to do with
Trust ranges.

That doesn't remove our responsibility to not test for overlaps in the
idranges though. We will need to verify that the manual configuration
changes do:

-  not overlap with ranges from trusted domains
   (objectclass=ipatrustedaddomainrange) in cn=ranges,cn=etc,$SUFFIX, or
   reject the change otherwise.
-  overlap completely with ranges from the local IPA domain
   (objectclass=ipaDomainIDRange) in cn=ranges,cn=etc,$SUFFIX, or give a
   warning otherwise which asks the user to add a new suitable idrange."

Codewise the logic could be:

#. check if the new range is a subset of a local idrange(s), if yes, all
   is fine
#. if not, check if it overlaps with an idrange of a trusted domain, if
   yes, reject
#. if not, reject and ask to add a new idrange for the local domain

The overall logic for deleting a master and saving the range(s):

#. Connect to the remote master
#. If the connection fails, report this and ask to continue (and lose
   the range(s)). If so, skip the rest and delete the master.
#. Put the remote master into read-only and force a sync
#. Retrieve the DNA range and on-deck values (if any)
#. Check for overlap (just in case)
#. If overlap, report the overlap and skip the range
#. Search through list of masters, excluding the one we're deleting to
   find one without an on-deck
#. Set first any valid DNA range on the first available master with an
   on-deck
#. Search for another available master if the deleted master has an
   on-deck and set that
#. Report errors as needed



Hosts that are down
----------------------------------------------------------------------------------------------

We need specially handling for hosts that are not up. This could be
either a temporary or a permanent issue. I think that when modifying a
range we need to prompt that an overlap can occur if they continue.

The --force flag will be used to avoid the prompt. The default answer to
the Proceed question is No.

Implementation
==============

-  No special handling was needed to deal with hosts that are down
   because the --force flag is required to get very far at all. A
   message was added that any range on the host would be lost, but no
   additional prompts were added.



Feature Managment
=================

UI

It will not have a UI component.

CLI

The ipa-replica-manage tool.



Major configuration options and enablement
==========================================

None.

Replication
===========

Indirectly. It can affect the available range(s) to a replica. If a
replica runs out and not enough values are left then the DNA plugin will
give up:

::

   # ipa user-add --first=tim --last=user tuser4
   ipa: ERROR: Operations error: Allocation of a new value for range cn=posix ids,cn=distributed numeric assignment plugin,cn=plugins,cn=config failed! Unable to proceed.

Managing ranges can be dangerous. If there are overlapping ranges you
run the risk of 2 different masters assigning the same value. This will
then cause grief when the entries are replicated.



Updates and Upgrades
====================

No

Dependencies
============

No



External Impact
===============

No