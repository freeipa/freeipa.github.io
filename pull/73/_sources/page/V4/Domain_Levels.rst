Domain_Levels
=============

Overview
--------

As we add more and more features and we change the way some operations
are done or add dependencies on new components we face some challenges
with mixing different versions of FreeIPA, and migrating from one
version to another.

A solution is needed to make it possible for 2 different versions of
FreeIPA to interact properly.



Use Cases
---------

-  Migration from one version of FreeIPA to another.
-  Coexistence of two different versions of FreeIPA servers in the same
   Domain.
-  Potentially, in future, identification of the features supported by a
   trusted FreeIPA Domain.

Design
------

The simplest way to deal with domain level compatibility issues is to
have a Domain-wide "Level" or "Version" associated with the Domain.

This Domain Version is a number written in the public shared tree and is
maintained by the administrator, which can decide to increase it to
activate new functionality after a migration from an older to a newer
version of FreeIPA an all the participating domain controllers.



Domain Level Compliance
----------------------------------------------------------------------------------------------

Checking if the Domain Level can be upgraded relies on the ability to
check if all servers in the Domain satisfy minimum requirements that
allow them to fully participate in a higher Domain Level.

Each server will expose a range of Supported Levels, which will be the
basis to determine if the Domain Level can be raised. However additional
checks maybe needed to determine if a domain level can be raised. For
example a Domain Level may depend on a specific component to be
configured and running on at least one server, but not all of them. And
although all servers are capable of running the specific component, the
server version alone won't be able to convey the fact that at least one
server in fact does run such a component.

When the need will arise we'll have to determine how to properly check
(and "warn" if not) if these services are available.

Older FreeIPA versions that do not maintain a Server Version attribute
will be considered at level "0" which will be defined as something
compatible with version 3.3.0/4.1.0 of FreeIPA as currently released by
various distributions.



Raising the Domain Level
----------------------------------------------------------------------------------------------

A CLI/UI command will be provided to allow raising the Domain Level.
This command will embed the knowledge necessary to determine if an
upgrade is possible. It will check various minimum version numbers and
obligatory component presence across the domain by consulting the public
shared tree.

If all the constraints are met then the Domain Level is raised to the
level specified.

The Domain Level cannot be lowered as raising the Domain Level can cause
changes to the tree (new schema, changes in behavior and data) that
cannot be easily undone.



Effects of a raise in level on the Domain
----------------------------------------------------------------------------------------------

All servers MUST monitor the Domain Level attribute. Upon replication if
the Domain Level is raised then all server features that are affected by
the change must be notified (or must be listening on the same change as
plugins) and accordingly alter behavior.

It is HIGHLY desirable that no restart be required for a Domain Level
change to take effect. If such will be needed in future, then we'll need
to develop a privileged component that can be activated by the level
change and can restart the affected components as needed.

An effort will need to be made each time a feature is change or a new
feature is added to understand if a new Domain Level is required, and if
so how a domain level change to enable the new feature will affect the
domain. In particular, races due to the delayed activation (replication
may take some time) must be taken in account when designing new features
that will require a domain level change.



Storing Domain levels
----------------------------------------------------------------------------------------------

Domain levels supported by the server should be stored in the server
record and automatically updated during upgrade. The level is stored in
``cn=HOSTNAME,cn=masters,cn=ipa,cn=etc,SUFFIX``. New attributeType and
objectClass will be needed for storing the domain level range.

Selected Domain level shall be stored in
``cn=Domain Level,cn=ipa,cn=etc,SUFFIX``



Feature Management
------------------

UI

Add page for setting the domain level to Server Configuration tab.

CLI

There should be a command to raise *global* domain level. Server
supported domain level(s) is automated, based on server version.

Configuration
----------------------------------------------------------------------------------------------

Presence of domain level is mandatory for all servers since the initial
version.

Upgrade
-------

Server supported domain level is automatically raised during upgrade to
next version.



How to Test
-----------

TBD.