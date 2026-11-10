Replicated_lockout
==================

Overview
--------

Accounts are locked or unlocked based on passwordpolicy, but the data
used to determine if an account has to be locked or unlocked are only
local (explicitely excluded from replication). So it is possible to
retry on an other server once an accout is locked. The purpose of this
design is to make the lockout global without introducing heavy
replication traffic.

NOTE: If only the final lockout state is replicated there still is the
opportunity to try passwords in a round robin method on all servers
before on on server the final lockout state is reached

NOTE: A side effect of this feature is that the lockout state can be
tracked on any server and a locked account can be unlocked on any server
(ticket 3863/2792)



Use Cases
---------

If the login failed count is exceeded on one server it will be
impossible to login on an other server until the account is unlocked by
an admin or if the lockout duration is passed If an account is locked,
it can be unlocked on any server (solves ticket
`#3863 <https://fedorahosted.org/freeipa/ticket/3863>`__)

Design
------

To achieve a global lockout without replicating the basic lockout
attributes: krbLastFailedAuth, krbLoginFailedCount an new attribut has
to be defined: krbGlobalLockoutState with the states “locked” and
“unlocked”. This is set whenever an account gets locked or unlocked and
is repplicated to all other servers. The decision if a login attempt is
accepted checks the presence this new attribute first before continuing
with the local checks.

There are two other attributes required to make the lockout policy
consistent.

If a lockout is not permanent but has a finite duration based on the
last failed login which triggered the lockout, the time of the lockout
start now als has to be available on all servers. Since we do not want
to replicate krbLastFailedAuth whenever it is set, there has to be a new
attribute which is set and replicated together with
krbGlobalLockoutState: krbGlobalLockoutStartTime. This is one operation
and no extra replication traffic introduced.

The other required attribute is related to unlocking. The replication of
krbGlobalLockoutState and krbGlobalLockoutStartTime only sets these
attributes, but does not change the other attributes in the entry, eg
krbLoginFailedCount. To achieve this the lockout plugin would have to
intercept each modify operation and check if it is a change of the
lockout attributes and act upon them. This is unnecessary overhead, the
logic can be applied in the ipalockout plugin only, but has to take
special care for the unlock scenario. If the plugin detects that the
global state is “unlocked” it has to reset the local
krbLoginFailedCount, but it has to be done only once, this can be
controled by a attribute which tracks local lockout state changes:
krbLocalLockoutState. If the global state is unlocked and the local
state is locked, the failed count is reset and the local state is set to
unlocked.

Implementation
--------------

The schema (60kerberos.ldif) needs to be extended to contain the new
attributes: krbGlobalLockoutState, krbLocalLockoutState,
krbGlobalLockoutStartTime

The logic in the pre and postp ipa-lockout plugin needs to be changed to
use and set global lockout state:
daemons/ipa-slapi-plugins/ipa-lockout/ipa_lockout.c

This applys also to the code in the ipa kdb driver:
./daemons/ipa-kdb/ipa_kdb_audit_as.c

The command line utilities : user-status, user-unlock also need to be
enhanced ./ipalib/plugins/user.py



Feature Management
------------------

UI

How the feature will be managed via the UI will be handled in ticket
2792

CLI

The feature affects the following commands: ipa user-unlock ipa
user-status

user-unlock can now be used to unlock a user on any replica

Replication
-----------

The feature is relying on replication to have a global account lockout
policy. The new attributes krbGlobalLockoutState and
krbGlobalLockoutStartTime need to be replicated, the attribute
krbLocalLockoutState has to be excluded from replication



Updates and Upgrades
--------------------

Any impact on updates and upgrades?

Dependencies
------------

N/A



External Impact
---------------

N/A



Backup and Restore
------------------

N/A



Test Plan
---------

Test scenarios that will be transformed to test cases. TBD