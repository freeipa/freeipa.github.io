\__NOTOC_\_

Overview
========

When a provisioning system like Foreman, Spacewalk or other creates a
host or a user provisioning system provisions a user it should be able
to automatically drive the placement of user or host into the right
groups and creation of the right policies. This integration can be done
by making the provisioning system aware of the specific group structure
and policy management in IPA however in reality this is impractical. It
would be better if the provisioning system can mark (put a tag) on the
entry it creates and then IPA (though its DS plugins) would do the rest
based on the specific placement policies defined in the deployment.

The idea is that a single entry should be created and a special
attribute would be set at the creation time by the provisioning system.
IPA would be configured to look at this attribute and build the
appropriate groups structure using automembership plugin. In future
other plugins can be created to look at this attribute and perform other
operations as needed.

.. _use_cases10a:

Use Cases
=========

Provisioning system calls:

::

   ipa host-add ... --some-attribute=somevalue

or

::

   ipa user-add ... --some-attribute=somevalue

and the host or user is placed int other the right groups based on the
automembership plugin configuration. The configuration of the
automembership plugin as well as the creation of the specific group
structure is outside of the scope of the current design and should be
treated as deployment specific. However procedures of making the
solution work from end to are welcome and can be added to `How To
section <HowTos>`__.

Design
======

.. _new_schema:

New schema
----------

There is already a standard ``userClass`` attribute that can be used
exactly for this purpose:

::

   attributeTypes: ( 0.9.2342.19200300.100.1.8 NAME 'userClass'  EQUALITY caseIgn
    oreMatch SUBSTR caseIgnoreSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.1
    5{256} X-ORIGIN 'RFC 4524' )

Add the attribute to MAY list of ``ipaHost`` object class available on
all hosts.

Implementation
==============

-  Schema will be added to the initial deployment
-  Schema will be augmented at upgrade time



Feature Management
==================

UI
~~

There is no UI exposure other than userClass attribute should be visible
and selectable in the automember management UI. (not done in initial
implementation, `link to
ticket <https://fedorahosted.org/freeipa/ticket/3590>`__)

CLI
~~~

Host-add and user-add (not done in initial implementation, `link to
ticket <https://fedorahosted.org/freeipa/ticket/3588>`__) commands would
accept a new optional attribute ``--class``.



Major configuration options and enablement
==========================================

Any configuration options? *No*

Any commands to enable/disable the feature or turn on/off its parts?
*No*

Replication
===========

Any impact on replication? *No*



Updates and Upgrades
====================

Any impact on updates and upgrades? *Yes, new schema should be added on
upgrades*

Dependencies
============

Any new package and library dependencies. *No*



External Impact
===============

Impact on other development teams and components. *Yes, this feature
allows a better integration with the external provisioning systems*



RFE Author
==========

`Dpal <User:Dpal>`__.
