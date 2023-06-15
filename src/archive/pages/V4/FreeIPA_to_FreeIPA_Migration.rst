Overview
--------

Purpose of this feature is to provide means of migration between two
FreeIPA deployments that would be easy to use and allowed migration of
as large a portion of FreeIPA content as possible.

Existing migration feature provided by *ipa migrate-ds* command is
originally designed to migrate contents from a Directory Server. When
used for migration from another FreeIPA deployment, user has to specify
number of options in order to secure successfull migration. More
importantly, this feature only provides migration for users and groups
only.

New migration feature will provide means to migrate not only user and
groups, but also certificates, sudo rules, HBAC rules etc. Also since
the structure of source data is predefined, the command will require
significantly less options to run properly.



Use Cases
---------

.. _user_stories:

User stories
~~~~~~~~~~~~

As a company manager, I want to modernize name of the company and change
domain accordingly to maintain uniformity. In such a case, I want to be
able to migrate at least the most important contents of company's
FreeIPA deployment to ensure as little disruption of operations as
possible.

As a businessman, I want to include newly acquired company into the
organization structure of my existing company. If both companies use
FreeIPA, such a transition will be less costly and more secure with a
dedicated migration tool.

As an administrator, I want to be able to use a tool to extract contents
of FreeIPA in case our infrastructure is damaged. Such a tool would
enable us to salvage as much data as possible by transferring it to
newly built infrastructure.

Design
------

All contents in FreeIPA deployment are saved in LDAP directory tree.
Migration from one deployment to another is thus best handled via LDAP
operations on source and target deployments - find entries for migration
on source deployment and insert them to target deployment.

This approach requires changes to be made for entries:

-  *domain name* - every attribute that contains domain name (even as
   part of DN) needs to be changes to refer to domain name of target
   deployment (e.g. DN, Kerberos principal, certificate subject,
   references to other entries (e.g. user/manager relationship), ...)
-  *Kerberos key* - so far there is no way to migrate Kerberos master
   key, which makes migration of Kerberos passwords and related
   attributes irrelevant

The migration cannot be performed for some entries, e.g. migration of
topology and trusts is not possible - these need to be newly created in
target deployment.

Types of entries that can be migrated are:

-  users, groups;
-  sudo rules, sudo commands, sudo command groups;
-  hosts, services;
-  HBAC;
-  automount;
-  automember;
-  ID views;
-  ID ranges;
-  role;
-  SELinux;
-  CA ACL;
-  certificates;
-  delegation;
-  dns;
-  netgroups;
-  OTP tokens;
-  permissions;
-  privilefes;
-  password policies;
-  RADIUS;
-  selfservice;
-  vault;

Migration is designed to be available for users with "admin" privileges
in target FreeIPA deployment. In source deployment the user can
authenticate as any user even without admin privileges, and only the
data this user has access to will be migrated.

Implementation
--------------

TODO

Entries to be migrated from original FreeIPA deployment will be
retrieved in two steps: at first, only DNs of the entries will be
retrieved via ldapsearch. Then, for each stored DN, a new ldapsearch
will be performed that will retrieve all data for the entry. This
approach is chosen to eliminate issues occuring with migrate-ds command,
which fails when migrating deployments with huge amount of entries.



Feature Management
------------------

UI
~~

Web UI management for this feature is not yet considered.

CLI
~~~

TODO

======= ======= ===========
Command Options Description
======= ======= ===========
\               
======= ======= ===========

Configuration
~~~~~~~~~~~~~

FreeIPA must be installed and migration enabled on target server:

::

   $ kinit admin
   $ ipa config-mod --enable-migration=True

By default, migration is disabled when compat plugin is enabled. Either
disable compat plugin and restart Directory Server, or use
*--with-compat* option.
