Overview
========

The IPA framework will handle default permissions more robustly with
respect to IPA updates and user-made changes.

In the future, this allow us to create a system of fine-grained,
user-controllable permissions with good defaults and predictable
behavior over upgrades.



Use Cases
=========

Modifications to permissions that come with FreeIPA by default will be
restricted to changing the attribute lists and bind types.

For example, attempting to change a default "Modify Users" permission to
apply to Groups will fail:

| ``$ ipa permission-mod 'System: Modify Users' --type=group``
| ``ipa: ERROR: invalid 'ipapermlocation': not modifiable on managed permissions``

So will attempting to make it apply to Read operations:

| ``$ ipa permission-mod 'System: Modify Users' --right=read,``
| ``ipa: ERROR: invalid 'ipapermright': not modifiable on managed permissions``

On the other hand, making the permission not apply to the "gecos"
attribute is possible:

| ``$ ipa permission-mod 'System: Modify Users' --excludedattrs=gecos``
| ``------------------------------------------``
| ``Modified permission "System: Modify Users"``
| ``------------------------------------------``
| ``  Permission name: System: Modify Users``
| ``  Granted rights: write``
| ``  Effective attributes: businesscategory, carlicense, cn, description, displayname, employeetype, facsimiletelephonenumber, givenname, homephone, inetuserhttpurl, initials, l, labeleduri, loginshell, manager, mepmanagedentry, mobile, objectclass, ou, pager, postalcode,``
| ``                        preferredlanguage, roomnumber, secretary, seealso, sn, st, street, telephonenumber, title, userclass``
| ``  Excluded attributes: gecos``
| ``  Default attributes: telephonenumber, cn, labeleduri, manager, street, displayname, homephone, title, facsimiletelephonenumber, loginshell, employeetype, description, businesscategory, preferredlanguage, roomnumber, mepmanagedentry, carlicense, postalcode, givenname,``
| ``                      pager, seealso, objectclass, inetuserhttpurl, l, st, mobile, gecos, sn, ou, secretary, userclass, initials``
| ``  Bind rule type: permission``
| ``  Subtree: cn=users,cn=accounts,dc=idm,dc=lab,dc=eng,dc=brq,dc=redhat,dc=com``
| ``  Type: user``
| ``  Granted to Privilege: Modify Users and Reset passwords, User Administrators``
| ``  Indirect Member of roles: helpdesk, User Administrator``

So is changing the bind rule type, i.e. making a permission apply to all
authenticated users, anyone including anonymous users, or specific users
selected by privileges:

| ``$ ipa permission-mod 'System: Read User Membership' --bindtype=anonymous``
| ``--------------------------------------------------``
| ``Modified permission "System: Read User Membership"``
| ``--------------------------------------------------``
| ``  Permission name: System: Read User Membership``
| ``  Granted rights: read, compare, search``
| ``  Effective attributes: memberof``
| ``  Default attributes: memberof``
| ``  Bind rule type: anonymous``
| ``  Subtree: cn=users,cn=accounts,dc=idm,dc=lab,dc=eng,dc=brq,dc=redhat,dc=com``
| ``  Type: user``

.. _documentation_draft:

Documentation draft
===================

This section explains the behavior from the user's point of view:

"Managed" permissions are those permissions that come pre-installed with
IPA, and are updated in new releases to cover new functionality. They
behave like regular, user-created permissions, with these differences:

-  Their name, location and target can not be modified
-  They cannot be deleted (but note that setting default bindtype and
   removing from all privileges will effectively disable a permission)
-  They have three sets of attributes:

   -  *default* attributes, managed by IPA, cannot be changed by the
      user
   -  *included* attributes (--includedattrs in the CLI) are extra attrs
      that the user has added
   -  *excluded* attributes (--excludedattrs in the CLI) are those that
      the user has removed

The permission applies to all attributes that appear in either its
*default* or *included* sets, but not in its *excluded* set.

It is possible to control the effective attributes using
permission-mod's --attrs option, as with other permissions. The modifies
the *included* and *excluded* lists so that the effective list matches
the request.

(Note that attributes of user-created permissions work the same way, but
since the *default* and *excluded* sets are empty, only the *included*
set determines the effective attributes.)

Newer versions of IPA may add additional attributes to the *default*
set. Attributes added this way should be listed in the corresponding
Release Notes. If some of the new attributes are undesirable, add them
to the corresponding *excluded* list before the upgrade.

All managed permissions currently have the "System:" prefix in their
name, to prevent naming clashes between user-created permissions and
future IPA defaults.

In FreeIPA 4.0, most default permissions have been turned into Managed
ones. Some permissions still use the old "SYSTEM" scheme (i.e. they are
not modifiable, but may be only assigned to privileges). A list of these
permissions follows:

-  Add Automember Rebuild Membership Task
-  Add Replication Agreements
-  Certificate Remove Hold
-  Get Certificates status from the CA
-  Modify DNA Range
-  Modify Replication Agreements
-  Remove Replication Agreements
-  Request Certificate
-  Request Certificates from a different host
-  Retrieve Certificates from the CA
-  Revoke Certificate
-  Write IPA Configuration

.. _upgrade_considerations:

Upgrade considerations
----------------------

When upgrading to IPA 4.0, the process will convert old default
permissions, such as.

| ``ipa-3-3$ ipa permission-find 'Modify Groups'``
| ``--------------------``
| ``1 permission matched``
| ``--------------------``
| ``  Permission name: Modify Groups``
| ``  Permissions: write``
| ``  Attributes: cn, description, gidnumber, objectclass, mepmanagedby, ipauniqueid``
| ``  Type: group``
| ``  Granted to Privilege: Group Administrators``
| ``  Indirect Member of roles: User Administrator``
| ``----------------------------``
| ``Number of entries returned 1``
| ``----------------------------``

to managed ones, e.g.

| ``$ ipa permission-find 'System: Modify Groups'``
| ``--------------------``
| ``1 permission matched``
| ``--------------------``
| ``  Permission name: System: Modify Groups``
| ``  Granted rights: write``
| ``  Effective attributes: cn, description, gidnumber, ipauniqueid, mepmanagedby, objectclass``
| ``  Default attributes: cn, objectclass, mepmanagedby, gidnumber, ipauniqueid, description``
| ``  Bind rule type: permission``
| ``  Subtree: cn=groups,cn=accounts,dc=idm,dc=lab,dc=eng,dc=brq,dc=redhat,dc=com``
| ``  Type: group``
| ``  Granted to Privilege: Group Administrators``
| ``  Indirect Member of roles: User Administrator``
| ``----------------------------``
| ``Number of entries returned 1``
| ``----------------------------``

If the default permissions that come with IPA have been modified,
special care should be taken when upgrading.

.. _removed_default_permissions:

Removed default permissions
~~~~~~~~~~~~~~~~~~~~~~~~~~~

If a default permission was removed, the upgrade process will simply
create a new default permission. This is consistent with to how all IPA
upgrades work.

.. _changed_attribute_lists:

Changed attribute lists
~~~~~~~~~~~~~~~~~~~~~~~

If only the attribute list differs from a past default, the permission
updater will set the *included*/*excluded* lists of the new default
permission to match the modifications.

Note that the updater considers default values from all past IPA
versions. Be sure to check the result after updating.

.. _other_aci_changes:

Other ACI changes
~~~~~~~~~~~~~~~~~

If any other changes were made to a default permission, the updater
emits a warning and does not create the new permission. In this case,
there are two options:

1) Delete the old permission (e.g.
``ipa permission-del 'Modify Groups'``), then run
``ipa-ldap-updater -p``. This will create the new default permission.

2) Change the old permission to a new-style (V2) permission by issuing
e.g. ``ipa permission-mod 'Modify Groups'`` *on a server with IPA 4.0+*,
then run ``ipa-ldap-updater -p``. This will also create the new default
permission, but the old one will be preserved as a user-created
permission.



The problem
===========

Currently, updates to permissions that come with IPA are specified in
.update files.

This approach has the disadvantage that if the user modifies the
permission, the updater will not recognize it, so it will end up not
being updated. This may result in reduced functionality (if needed
attributes are not added), or security issues (if attributes are not
deleted).

Design
======

.. _managed_permissions:

Managed Permissions
-------------------

`V2 permissions <V3/Permissions_V2>`__ with the ``MANAGED`` flag set are
called Managed permissions.

These permissions grant access to a set of attributes defined by IPA and
kept up-to-date on upgrades, while allowing users to add or remove
specific attributes from the default list.

The user can also manage the bind rule and privilege membership of
Managed permissions.

The other aspects of Managed permissions (name, location, target) are
not modifiable by the user. The user cannot manually add new Managed
permissions, or delete existing ones (unless --force is applied; but
later we may restrict this via ACIs).

To ensure that installing low-version replicas or disabling plugins does
not revoke access to existing data, the default list of attributes will
be kept in LDAP as ``ipaPermDefaultAttr``. Users can not modify this
list via the framework. On updates, new attributes will *only* be added
to this list. (To remove attributes, we would need to write a separate
update plugin.)

There will be two attribute types for holding attributes the admin added
and removed: ``ipaPermIncludedAttr`` and ``ipaPermExcludedAttr``
respectively. (In user-created permissions, ``ipaPermIncludedAttr`` is
used for the same purpose as here, and excluded & default are empty.)
See `V3/Permissions V2 <V3/Permissions_V2>`__ for the schema definition.

When generating the ACI, the resulting attribute list will be computed
by taking the ``ipaPermDefaultAttr`` set, adding any
``ipaPermIncludedAttr``\ s, and then removing any
``ipaPermExcludedAttr``\ s.

For example, this permission:

| ``dn: cn=Read Users,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: Read Users``
| ``ipaPermDefaultAttr: cn``
| ``ipaPermDefaultAttr: sn``
| ``ipaPermDefaultAttr: givenName``
| ``ipaPermDefaultAttr: l``
| ``...``
| ``ipaPermIncludedAttr: favoriteColor``
| ``ipaPermExcludedAttr: givenName``
| ``objectClass: top``
| ``objectClass: groupOfNames``
| ``objectclass: ipaPermission``
| ``objectclass: ipaManagedPermission``
| ``ipaPermType: SYSTEM``
| ``ipaPermType: V2``
| ``ipaPermType: MANAGED``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: read``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermBindRuleType: permission``

would allow users to read all default user attributes except
``givenName``, plus additionally ``favoriteColor``.

.. _cli_api:

CLI & API
~~~~~~~~~

The ``permission-{mod,find}`` commands will gain two new options,
``--includedattrs`` (API: ``ipapermincludedattr``) and
``--excludedattrs`` (API: ``ipapermexcludedattr``). For
``permission-mod`` it is an error to use ``--excludedattrs`` with
non-managed permissions.

For a managed permission, the ``permission-{mod,find,show}`` commands
will output all three lists (``ipapermdefaultattr``,
``ipapermallowedattr``, ``ipapermexcludedattr``), as well as the
computed list of effective attributes.

For a non-managed permission, ``permission-{mod,find,show}`` will only
output the effective attributes (``attrs``). With ``--all``, the
included attributes will also be included. As any missing attribute
course excluded and default will not be output. With ``--raw``, only
``ipaPermIncludedAttr``, and not ``attrs``, wil be output.

It is an error to set the ``ipapermlocation``, ``ipapermtargetfilter``,
or ``ipapermtarget`` of a managed permission. (This means that it's an
error to ise the API options ``subtree``, ``extratargetfilter``,
``target``, ``memberof``, ``targetgroup``, or ``type`` with a managed
permission.)

.. _default_permission_updater:

Default Permission Updater
--------------------------

A server post-update plugin will walk through ipalib ``Object`` plugins
and create/update managed permissions pertaining to them.

Names of such default permissions are *required* to start with "System:
", so that default permissions added in future IPA releases do not
conflict with user-created permissions. The ":" character will not be
usable in ``permission-add``. (It will be usable in ``permission-mod``
and ``permission-del``, where managed permissions are subject to the
limitations stated above.)

The IPA Object plugins will gain a new Python attribute,
``managed_permissions``, which will hold a template for the permissions
that are to be added by default to manage that object.

This will allow plugins to be more self-contained; it will no longer be
necessary to modify IPA's update files to add the common cases of
plugin-specific permissions.

The format of the managed_permissions templates will be documented in
the ``update_managed_permissions`` server plugin
(`link <https://git.fedorahosted.org/cgit/freeipa.git/tree/ipaserver/install/plugins/update_managed_permissions.py>`__).

.. _replacing_legacy_default_permissions:

Replacing legacy default permissions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Another entry in the ``managed_permissions``\ template, ``replaces``,
will be used for replacing legacy permissions with new managed ones.
Example:

| ``   managed_permissions = {``
| ``       'ipa:Modify SUDO Rule': {``
| ``           'ipapermbindruletype': 'permission',``
| ``           'ipapermright': {'write'},``
| ``           'ipapermdefaultattr': {``
| ``               'description', 'ipaenabledflag', 'usercategory',``
| ``               'hostcategory', 'cmdcategory', 'ipasudorunasusercategory',``
| ``               'ipasudorunasgroupcategory', 'externaluser',``
| ``               'ipasudorunasextuser', 'ipasudorunasextgroup', 'memberdenycmd',``
| ``               'memberallowcmd', 'memberuser'``
| ``           },``
| ``           'replaces': [``
| ``               '(targetattr = "description || ipaenabledflag || usercategory || hostcategory || cmdcategory || ipasudorunasusercategory || ipasudorunasgroupcategory || externaluser || ipasudorunasextuser || ipasudorunasextgroup || memberdenycmd || memberallowcmd || memberuser")(target = "``\ ```ldap:///ipauniqueid=`` <ldap:///ipauniqueid=>`__\ ``*,cn=sudorules,cn=sudo,$SUFFIX")(version 3.0;acl "permission:Modify Sudo rule";allow (write) groupdn = "``\ ```ldap:///cn=Modify`` <ldap:///cn=Modify>`__\ `` Sudo rule,cn=permissions,cn=pbac,$SUFFIX";)',``
| ``           }``
| ``       },``
| ``       ...``

If the an existing *legacy* (i.e. non-v2) permission exists either
without an associated ACI or with an ACI that *exactly* matches the
information specified in the ``replaces`` list, the old permission is
removed after the new one is added.

This ensures that

-  the old permission is retained if the user has changed it
-  at no time are the ACIs revoked (briefly, there are two ACIs granting
   the same access).

If an existing legacy permission does match ``cn`` but *not* some other
attributes in the ``replaces`` dict, a warning is logged, the new
permission is added, and the old one is left in place.

The exception are attributes. If the ACI only differs in the list of
attributes, the permission is upgraded as notmal but with
``ipapermincludedattr`` and ``ipapermexcludedattr`` set to reflect the
difference between the old default and the pre-existing permission.

.. _removing_the_global_anonymous_read_aci:

Removing the global anonymous read ACI
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After the permission updater successfully runs, it will look for an ACI
named "Enable Anonymous access" in $SUFFIX, and remove it.

The ``update_anonymous_aci`` server update plugin will be removed.

ACI.txt
-------

To ensure that permission changes are properly reviewed, a summary file
similar to API.txt will be generated, and it will be checked on each
build.

It will contain a summary of the default managed permissions.

A ``makeaci`` script similar to ``makeapi`` will be provided and called
to check the file on each build.

Implementation
==============

No additional requirements or changes discovered during the
implementation phase.

.. _feature_managment:

Feature Managment
=================

UI
~~

The immutable aspects of Managed permissions are grayed out in the Web
UI.

CLI
~~~

See the CLI & API section in Design.



Major configuration options and enablement
==========================================

Access control is configured via the existing RBAC system.

Replication
===========

N/A, ACIs and permissions are replicated.



Updates and Upgrades
====================

Old servers will not be able to modify Managed permissions, except
adding/removing them to/from prigileges. Details are in `Permissions
V2 <V3/Permissions_V2>`__, which will be implemented in the same
release. Managed permissions use the MANAGED flag.

Installations with customized ACIs will need some extra care when
upgrading, as detailed in Upgrade considerations above. (But note that
IPA's upgrade behavior wrt modified default permissions has always been
underspecified and likely surprising.)

Dependencies
============

No new package and library dependencies.



External Impact
===============

Tests and documentation need to be written.



Backup and Restore
==================

ACIs, permissions, privileges and roles are already included in backup &
restore.



Test Plan
=========



RFE Author
==========

`Petr Viktorin <User:Pviktorin>`__

`Category:FreeIPA V4 Test Plan <Category:FreeIPA_V4_Test_Plan>`__
`Category:FreeIPA Test Plan <Category:FreeIPA_Test_Plan>`__
