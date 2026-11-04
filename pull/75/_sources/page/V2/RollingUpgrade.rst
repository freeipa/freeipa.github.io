RollingUpgrade
==============

Introduction
------------

In a large authentication infrastructure upgrading can be difficult. Not
all servers can be brought down and upgraded at once. Instead a rolling
upgrade is preferred where the new version is installed incrementally
across the infrastructure, aka a rolling upgrade.

Therefore we need to be able to always provide smooth upgrades where a
few servers are upgraded in groups between point releases. All servers
should be brought into the same version as quickly as possible,
preferably hours or days and not weeks.

It may not be possible to upgrade any release to any other release.
Intermediate steps may be required (e.g. upgrade v2.1 to 2.3 to 3.0)



What is a server?
-----------------

We use the phrase "IPA server" a lot but in fact there is no IPA server.
IPA is a mechanism to get a lot of moving parts (389-ds, MIT Kerberos,
DNS, dogtag) to work well together. Careless upgrading of any of these
can cause problems.

On top of this sits the IPA management framework to manage identity
data.



Upgrade Strategy
----------------

IPA will use a so-called "split brain" strategy for upgrades. With this
the infrastructure will actually be running two different versions of
the software for a brief period while the upgrade is being rolled out.
Replication will be disabled to incompatible servers, preventing
upgraded data from flowing to non-upgraded servers.

This is the split condition. Data will not be replicated between
dissimilar servers until they are running the same version. Changes on
these servers will queue up until the servers either reconnect or the
changelog fills, potentially causing lost updates.

Version
----------------------------------------------------------------------------------------------

In this context Version means data version for the purposes of
determining if an upgrade is incompatible. Not every IPA update may
change the version.

The version will be compiled into the IPA binaries and will be stored in
two places:

-  ipalib/constants.py for the management framework
-  hardcoded in a new DS version plugin

This version is literally hardcoded into the binaries and lets us
determine if the installed version is compatible with other IPA
replicas. We have no way of preventing a server to be installed with a
particular set of binaries. We do have a way from preventing that
version from interfering with other servers.

The 389-ds replication plugin will add hooks to be able to register a
version. The IPA plugin will use those hooks to register the version.



Version format
----------------------------------------------------------------------------------------------

The version will be stored as a date which will make comparisons a lot
easier (e.g. 201003081732).



IPA management utilities
----------------------------------------------------------------------------------------------

The IPA management utilities will send their installed version when they
communicate to the IPA XML-RPC server. The server will use this version
to decide if it will work with the client (this is the purpose of
putting the IPA version into ``ipalib/constants.py``)



The Upgrade Process
-------------------

Each instance will be upgraded separately. The basic process is:

-  Stop IPA services (``/usr/sbin/ipactl stop``)
-  ``yum update ipa-*``

   -  we may want a %pretrans to stop the yum process if IPA is still
      running
   -  in %post we will call ``setup-ds -u`` to apply updates via ldif
      snippets

-  Start IPA service (``/usr/sbin/ipactl start``)

The process of upgrading a set of IPA servers does not have to happen at
once but should happen relatively quickly. While there are incompatible
versions of IPA in a network replication will not work between them.
Changes will be queued in the 389-ds changelog but the more changes
there are the more likelihood that MMR will need to determine winners
and losers.



Changes to Replica Installation Process
---------------------------------------

Early in the IPA replica installer we will need to do a version check to
ensure that a different version replica isn't added. We don't want old
servers to be added to an IPA domain. The initial replication population
would fail anyway due to the version checking.



Types of Incompatibilities
--------------------------

Here is a brief list of changes that would probably require the version
to increase:

-  New Directory Server plugin
-  Enhanced Directory Server plugin, not backwards compatible
-  New IPA management plugin
-  Enhanced IPA management plugin, not backwards compatible
-  Updated schema on existing entries
-  Updated DIT
-  Update to a component

Clients
-------

A client using the management framework will have its own local copy of
the server plugins to know what information to gather for a given
request. This too can become out-of-date. The ipa client will need to
send a version in each XML-RPC request and the server will decide if it
can honor the request.

There are three scenarios:

#. A newer client talking to an older server
#. An older client talking to a newer server
#. A client talking the same version

A newer client talking to an older server will need to be handled by
code within the framework. It is possible that the conversation can't
happen at all if the remote server is too old.

An older client talking with a newer server MAY be supported but will
need to be handled on a case-by-case basis. Chances are that not every
plugin will be updated on every update so some should work fine with
older clients. It is possible that some requests will be rejected as too
old a client.

Tasks
-----

-  DS add version interface to replication plugin
-  IPA implement a version plugin
-  IPA add version checks to replica installer
-  IPA enhance ipactl command
-  IPA add version to XML-RPC protocol
-  IPA management framework enforce version
-  IPA update rpm installation scriptlets to:

   -  Check for running services and abort install
   -  Automatically apply updates post-install

-  IPA use DS upgrade mechanism (setup-ds.pl -u)