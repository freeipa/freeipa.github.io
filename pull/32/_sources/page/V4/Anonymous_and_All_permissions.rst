Anonymous_and_All_permissions
=============================

Overview
========

Allow IPA permissions to apply to Anonymous and All authenticated users.



Use Cases
=========

Note: This is not very practical for write, but until the global read
ACI is removed, examples need to use write permissions to be effective.

-  Allow all authenticated users to edit other users' login shell

``ipa permission-add 'Shell editable by all' --type=user --attrs=loginshell --bindtype=all --permissions=write``

-  Allow all users (even anonymous ones via LDAP) to edit other users'
   login shell

``ipa permission-add 'Shell editable by anonymous' --type=user --attrs=loginshell --bindtype=anonymous --permissions=write``

Design
======

The ``permission_{add,mod,find}`` commands will get a new ``--bindtype``
option (attribute name: ``ipapermbindruletype``) with these values:

-  "permission" (default) - Permission behaves as before -- grants
   access through privileges+roles
-  "all" - Permission applies to all authenticated users
   (```ldap:///all`` <ldap:///all>`__)
-  "anonymous" - Permission applies to all users, even unauthenticated
   ones (```ldap:///anyone`` <ldap:///anyone>`__)

Note: As IPA API always requires authentication, unauthenticated users
would need to use LDAP directly.

The ``permission_{add,mod,find,show}`` commands will output the
``ipapermbindruletype``.

Permissions with ``ipapermbindruletype`` other than "permission" may not
be added to privileges, and ``ipapermbindruletype`` other than
"permission" may not be set on permissions that are already members of a
privilege. (This will not be enforced on older servers. Adding such a
permission to a privilege will not have any effect.)

Implementation
==============

No additional requirements or changes discovered during the
implementation phase.



Feature Management
==================

UI

The UI will need a new field for the bind type.

Adding permissions with non-default ``bindtype`` set to privileges, and
setting non-default ``bindtype`` on permissions in privileges, should be
disabled in the UI.

CLI

See Design.



Major configuration options and enablement
==========================================

N/A

Replication
===========

ACIs are replicated.



Updates and Upgrades
====================

Permissions with non-default ``bindtype`` can only be created on new
servers, so thety will be `V2 permissions <V3/Permissions_V2>`__. This
means old servers will not read or modify their ACIs.

Old servers will be able to add all permissions to privileges, but
privilege membership will not have any effect unless
bindtype=permission. Removing any permission from privileges will be
possible on any server.

Dependencies
============

None



External Impact
===============

Will need tests and documentation



Backup and Restore
==================

ACIs are part of the DIT and so they are backed up.



Test Plan
=========



RFE Author
==========

`Petr Viktorin <User:Pviktorin>`__

`Category:FreeIPA V4 Test Plan <Category:FreeIPA_V4_Test_Plan>`__
`Category:FreeIPA Test Plan <Category:FreeIPA_Test_Plan>`__