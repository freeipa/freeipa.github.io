Winsync_Migration_Tool
======================

Overview
--------

FreeIPA has an ability to create IPA-AD trusts, which are the preferable
way of providing access to the IPA domain for the users from AD.
However, the previous supported mechanism, which is already deployed in
many environments was the winsync replication agreement, where the users
from AD are replicated into the IPA tree and assigned UIDs and GIDs.

To migrate the deployment from winsync synchronization to cross-realm
trust, one needs to preserve the UIDs and GIDs that were generated for
the replicated users.

IPA 4.1 brings the ability to create user ID overrides for the users
from the AD domain which is precisely the mechanism we need for the
seamless migration, without changing the data on the AD side.

This tool aims to automate the migration process.



Use Cases
---------

The scope of this tool is to migrate all winsync'ed users from a given
AD forest.

Design
------

Requirements
----------------------------------------------------------------------------------------------

-  An established trust with the AD domain in question



The detection of the realm to use
----------------------------------------------------------------------------------------------

Realm name of the AD users will be provided by the user.



Detection of the applicable users
----------------------------------------------------------------------------------------------

The winsynced users will be detected using the
"(&(objectclass=ntuser)(ntUserDomainId=*))" LDAP filter on the
cn=users,$SUFFIX subtree.



Migration procedure for one user
----------------------------------------------------------------------------------------------

For each user, we will create an user ID override with the following
attributes from the original winsynced entry:

-  uid
-  uidnumber
-  gidnumber
-  homedirectory
-  gecos

The original user winsynced entry will be removed if the creation of the
user override was successful.



Feature Management
------------------

CLI

=================== ===============
Command             Options
=================== ===============
ipa-winsync-migrate --ad-realm
\                   --unattended -U
=================== ===============



How to Test
-----------

A continuous integration test will be created to test the migration
procedure.

Author
------

`Tomas Babej <User:Tbabej>`__