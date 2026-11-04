Filtering_groups_by_type
========================

\__NOTOC_\_

Overview
========

There is no method how to filter groups by type. Group type is basically
a variation of object classes. Prerequisite for #3333.



Use Cases
=========

-  Web UI ticket #3333 needs this functionality in order to offer a
   combobox with only posix groups for setting Default Fallback Group in
   Trust config.
-  CLI user can use it for getting only POSIX groups, external groups
   and non-POSIX groups.

Design
======

There are three types of public groups: non-POSIX, POSIX and external.
Private groups are considered a special case and are not part of this
design.

-  POSIX group is a group with object class *posixGroup*
-  external group is a group with object class *ipaExternalGroup*
-  non-POSIX group is a group which is not posix nor external: it
   doesn't have *posixGroup* nor *ipaExternalGroup* object class.

New flags were added to group find command:

-  --posix
-  --nonposix
-  --external

When a flag is set, filtering is done according to mapping described
above.

Implementation
==============

No additional requirements or changes were discovered during the
implementation phase.



Feature Management
==================

UI

A prerequisite for other feature. No other direct impact.

CLI

Variations with following additional option variations of *group-find*
command can be now executed:

::

   group-find --external
   group-find --posix
   group-find --nonposix



Major configuration options and enablement
==========================================

N/A

Replication
===========

N/A



Updates and Upgrades
====================

N/A

Dependencies
============

N/A



External Impact
===============

N/A



RFE author
==========

`Pvoborni <User:Pvoborni>`__