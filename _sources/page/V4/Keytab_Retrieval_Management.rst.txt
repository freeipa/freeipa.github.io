Keytab_Retrieval_Management
===========================

Overview
--------

`Keytab Retrieval feature <V4/Keytab_Retrieval>`__ defines keytab
retrieval mechanism and new LDAP schema for management of the feature.
This part utilizes the existing schema a builds a management interface,
both CLI and Web UI, on top of it.



Use Cases
---------

Administrator allows users, groups, hosts or hostgroups to retrieve or
create a keytab of a service or a host.

Design
------

The rights for retrieval or creation of a keytab are handled by
extending the target entry with **ipaAllowedOperations** object class
and adding a DN of target user or group into **ipaAllowedToPerform**
attribute with a subtype which matches the operation.

Retrieval is associated with **read_keys** subtype, creation with
**write_keys**.

**host** and **service** plugins will get new methods:
**allow_retrieve_keytab**, **disallow_retrieve_keytab**,
**allow_create_keytab** and **disallow_create_keytab**. Each command
expects principal and a list of user or groups which should be allowed
/disallowed to retrieve/create keytab of that principal.



Access Control
----------------------------------------------------------------------------------------------

New system permissions to allow non-admin users to manage keytab rights:

-  System: Manage Host Keytab Permissions
-  System: Manage Service Keytab Permissions

Implementation
--------------

``ipalib`` is not written with subtypes in mind. There is an assumption
that param names, which match the attribute names in LDAP also have to
be a valid python identifier. Therefore transformations in pre and post
callback were needed to circumvent this limitation. Other possibility
was to loosen the regex for checking the param name. But this approach
has a global effect which is potentially dangerous to do in a end phase
of 4-1 development.



Feature Management
------------------

UI

User and Host details facets will be extended with association widgets.
One for each combination of operation and entity:

-  Allowed to retrieve keytab:

   -  Users
   -  User Groups

-  Allowed to create keytab

   -  Users
   -  User Groups

CLI

| `` ipa host-allow-retrieve-keytab HOSTNAME --users=STR --groups=STR --hosts=STR --hostgroups=STR``
| `` ipa host-disallow-retrieve-keytab HOSTNAME --users=STR --groups=STR --hosts=STR --hostgroups=STR``
| `` ipa host-allow-create-keytab HOSTNAME --users=STR --groups=STR --hosts=STR  --hostgroups=STR``
| `` ipa host-disallow-create-keytab HOSTNAME --users=STR --groups=STR --hosts=STR  --hostgroups=STR``

| `` ipa service-allow-retrieve-keytab PRINCIPAL --users=STR --groups=STR --hosts=STR  --hostgroups=STR``
| `` ipa service-disallow-retrieve-keytab PRINCIPAL --users=STR --groups=STR --hosts=STR  --hostgroups=STR``
| `` ipa service-allow-create-keytab PRINCIPAL --users=STR --groups=STR --hosts=STR  --hostgroups=STR``
| `` ipa service-disallow-create-keytab PRINCIPAL --users=STR --groups=STR --hosts=STR  --hostgroups=STR``



How to Test
-----------

See `Keytab Retrieval feature <V4/Keytab_Retrieval>`__. Only use the CLI
or Web UI interface instead of ldapmodify to delegate the keytab
manipulation privileges.



Test Plan
---------

Basic manipulation will be handled by RPC-tests.

Should be combined with tests for `Keytab Retrieval
feature <V4/Keytab_Retrieval>`__. First set the permission here and then
check the actual functionality using **ipa-getkeytab**.



RFE Author
----------

`Petr Vobornik <User:Pvoborni>`__