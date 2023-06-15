Overview
--------

This document specifies how backwards-incompatible changes may be
backported to maintenance branches of FreeIPA.



Use Cases
---------

The use case is fixing issues like `RH bug
1117300 <https://bugzilla.redhat.com/show_bug.cgi?id=1117300>`__, in
which the fix to `ticket
2886 <https://fedorahosted.org/freeipa/ticket/2886>`__ could not be
backported to IPA 3.0. Unfortunately, it is too late to backport this
particular fix.

The process described here, and related the changes in IPA, will make
such backports possible in the future.

== Design= =

IPA handles backwards-incompatible API changes by "capabilities", which
mark that a feature was introduced in a certain API version. This was
implemented for `V3/Messages <V3/Messages>`__.

when a client implements a capability, it unconitionally uses some new
API semantics in the request to the server. Servers use the client's API
version to determine whether to use new or old semantics. A client that
implements a capability cannot successfully connect to a server that
does not implement the capability.

The above rules work with FreeIPA's current API versioning scheme. This
design defines a way to enable backporting backwards-incompatible
features to maintenance branches, while respecting these rules.

.. _freeipa_master_branch:

FreeIPA master branch
~~~~~~~~~~~~~~~~~~~~~

The master branch uses a two-element API versioning scheme, 2.x, where
'x' is a number that increases with each API change.

API versions are compared using `LooseVersion
rules <http://epydoc.sourceforge.net/stdlib/distutils.version.LooseVersion-class.html>`__
of Python distutils.

Backwards-incompatible changes are recorded as \*capabilities*.

For example, the capability 'optional_uid_params' was added in API
version '2.54'. Every client of version '2.54' or greater uses the new
API. Every server of version '2.54' or greater uses the client's API
version, which is sent in each RPC call, to determine the API to use.

Clients of version '2.54' or greater cannot connect to servers older
than '2.54'.

================ =============== ===============
\                Client < '2.54' Client >= '2.54
Server < '2.54'  Old API\*       Cannot connect
Server >= '2.54' Old API         New API\*
================ =============== ===============

-  A client may not connect to a server of lower version.

.. _maintenance_api_branches:

Maintenance API branches
~~~~~~~~~~~~~~~~~~~~~~~~

Currently, the API of a maintenance branch must remain fully backwards
compatible with the version it was branched from. (Alternatively it may
implement the exact same API changes as the master branch, in the same
sequence, and update the API version accordingly.)

This design allows a single capability at a time to be backported. When
this hapens, a '+' and the name of the capability is appended to the API
version. Once branched this way, versioning of the branch is strictly
linear: additional capabilities may only be appended.

Only versions that already implement the 'extra_capabilities' capability
may be forked in this way. 'extra_capabilities' will be added when
support for the mechanism described here is added to FreeIPA.

Example
^^^^^^^

Let's say the master branch is at API version '2.500', and a maintenance
branch at '2.200'. The maintenance branch needs to backport the
capability 'b' which was added in version '2.400'. After implementing
the capability, the API version of the maintenance branch becomes
'2.200+b'.

After a while the capability 'a', introduced in master with API version
'2.300', needs to be backported. When this is done, the API version
becomes '2.200+b+a'.

All further versions of the '2.200' branch must start with '2.200+b+a';
the version '2.200+a' may never appear. .

A summary:

+-------+-------+---+-------+-------+-------+-------+-------+-------+
|       | sema  |   | C     | C     | C     | C     | C     | C     |
|       | ntics |   | lient | lient | lient | lient | lient | lient |
|       | under |   | <=    | =     | >=    | >=    | >=    | >=    |
|       | stood |   | '2    | '2.2  | '2    | '2.   | '2.   | 2.400 |
|       | by    |   | .200' | 00+b' | .200+ | 201', | 300', |       |
|       | s     |   |       |       | b+a', | <     | <     |       |
|       | erver |   |       |       | <     | 2.300 | 2.400 |       |
|       | ↓     |   |       |       | '2    |       |       |       |
|       |       |   |       |       | .201' |       |       |       |
+-------+-------+---+-------+-------+-------+-------+-------+-------+
| re    |       |   | old   | 'b'   | 'a',  | old   | 'a'   | 'a',  |
| quest |       |   |       |       | 'b'   |       |       | 'b'   |
| sema  |       |   |       |       |       |       |       |       |
| ntics |       |   |       |       |       |       |       |       |
| →     |       |   |       |       |       |       |       |       |
+-------+-------+---+-------+-------+-------+-------+-------+-------+
|       |       |   |       |       |       |       |       |       |
+-------+-------+---+-------+-------+-------+-------+-------+-------+
| S     | old   |   | Old   | C     | C     | C     | C     | C     |
| erver |       |   | API\* | annot | annot | annot | annot | annot |
| <=    |       |   |       | co    | co    | co    | co    | co    |
| '2    |       |   |       | nnect | nnect | nnect | nnect | nnect |
| .200' |       |   |       |       |       |       |       |       |
+-------+-------+---+-------+-------+-------+-------+-------+-------+
| S     | 'b'   |   | Old   | 'b'   | C     | C     | C     | C     |
| erver |       |   | API   | \*    | annot | annot | annot | annot |
| =     |       |   |       |       | co    | co    | co    | co    |
| '2.2  |       |   |       |       | nnect | nnect | nnect | nnect |
| 00+b' |       |   |       |       |       |       |       |       |
+-------+-------+---+-------+-------+-------+-------+-------+-------+
| S     | 'a',  |   | Old   | 'b'   | 'a',  | C     | C     | C     |
| erver | 'b'   |   | API   |       | 'b'   | annot | annot | annot |
| >=    |       |   |       |       | \*    | co    | co    | co    |
| '2    |       |   |       |       |       | nnect | nnect | nnect |
| .200+ |       |   |       |       |       |       |       |       |
| b+a', |       |   |       |       |       |       |       |       |
| <     |       |   |       |       |       |       |       |       |
| '2    |       |   |       |       |       |       |       |       |
| .201' |       |   |       |       |       |       |       |       |
+-------+-------+---+-------+-------+-------+-------+-------+-------+
| S     | old   |   | Old   | C     | C     | Old   | C     | C     |
| erver |       |   | API   | annot | annot | API   | annot | annot |
| >=    |       |   |       | co    | co    | \*    | co    | co    |
| '2.   |       |   |       | nnect | nnect |       | nnect | nnect |
| 201', |       |   |       | †     | †     |       |       |       |
| <     |       |   |       |       |       |       |       |       |
| 2.300 |       |   |       |       |       |       |       |       |
+-------+-------+---+-------+-------+-------+-------+-------+-------+
| S     | 'a'   |   | Old   | C     | C     | Old   | 'a'   | C     |
| erver |       |   | API   | annot | annot | API   | \*    | annot |
| >=    |       |   |       | co    | co    |       |       | co    |
| '2.   |       |   |       | nnect | nnect |       |       | nnect |
| 400', |       |   |       | †     | †     |       |       |       |
| <     |       |   |       |       |       |       |       |       |
| 2.400 |       |   |       |       |       |       |       |       |
+-------+-------+---+-------+-------+-------+-------+-------+-------+
| S     | 'a',  |   | Old   | 'b'   | 'a',  | Old   | 'a'   | 'a',  |
| erver | 'b'   |   | API   |       | 'b'   | API   |       | 'b'   |
| >=    |       |   |       |       |       |       |       | \*    |
| 2.400 |       |   |       |       |       |       |       |       |
+-------+-------+---+-------+-------+-------+-------+-------+-------+

-  A client may still not connect to a server of lower version.

A version is lower than the same version with extra capability suffix
attached (e.g. '2.10+xy' < '2.10+xy+zzy').

† See the "Optional suffix" section

.. _optional_suffix:

Optional suffix
^^^^^^^^^^^^^^^

To provide greater compatibility, the client may omit a suffix from the
API version it sends, if the semantics with and without that capability
are equivalent for that particular request. Only

For example, `bug
1117300 <https://bugzilla.redhat.com/show_bug.cgi?id=1117300>`__ only
affects a few commands, and only when the numeric user or group ID
option is set to a particular value. If a fix backported to version
'2.30', the client could send API version '2.30+optional_uid_params'
only in the case above, and otherwise keep sending '2.30'.

However, if the client version was
'2.30+optional_uid_params+major_overhaul', and the 'major_overhaul'
capability changes the semantics of the call, 'optional_uid_params' may
not be dropped.

The exact mechanics are not specified here or in the implementation that
will accompany this design, since they depend on the particular feature
being backported.

Limitations
~~~~~~~~~~~

This scheme only allows "official" branches, over which the core FreeIPA
team has full control. Third-party extensions are encouraged to adopt a
private versioning scheme, and use it in parallel to the IPA core API
version.

Implementation
--------------

No additional requirements or changes were discovered during the
implementation phase.



Feature Management
------------------

No user-visible features to manage



Major configuration options and enablement
------------------------------------------

No configuration options, no way to disable the feature.

Replication
-----------

No impact on replication.



Updates and Upgrades
--------------------

No impact on updates and upgrades.

Dependencies
------------

No new package and library dependencies.



External Impact
---------------

No impact on other development teams and components.



Backup and Restore
------------------

No impact on B&R.



Test Plan
---------

Will be tested by FreeIPA's testsuite, until a concrete use case arises.
The test will check all cases in the table in the "Example" section.
