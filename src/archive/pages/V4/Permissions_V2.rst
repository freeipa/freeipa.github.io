Permissions_V2
==============

Overview
--------

The internal workings of Permissions and their relationship to ACIs are
overhauled.

-  Permission flags are extended to ensure backwards compatibility
-  Permission ACIs may be located in any container, not just $SUFFIX
-  Permission entries now contain all data needed to (re-)generate (or
   check) the ACI
-  It is now possible to grant read rights



Use Cases
---------

This feature brings no major user-visible changes, but will enable
development of `Managed Read
permissions <V3/Managed_Read_permissions>`__.

The UI is re-organized and the CLI now allows some previously disabled
combinations of options -- see the respective sections.

Design
------



V2 permissions
----------------------------------------------------------------------------------------------

This document describes the semantics of "V2" permissions. These are
permissions with a "V2" flag.

V2 permissions cannot be modified by older versions of IPA. An old-style
permission will be converted to V2 whenever it is modified.

Assigning permissions to privileges will still be possible on older IPA
servers.



Permission flags
----------------------------------------------------------------------------------------------

Permission objects can have flags in their "ipaPermissionType"
attribute. Currently we use one such flag, SYSTEM, which means that the
underlying ACI is not managed by the Permission object. The code does
not even assume an underlying ACI exists for SYSTEM permissions.

The permission flags will be repurposed to allow adding new
functionality while retaining a degree of backwards compatibility.

New flags will be added for additional features. Permissions that have
an unknown flag will be assumed to have unknown semantics implemented in
future IPA versions, and the underlying ACI will not be manipulated (as
with current SYSTEM flags).

For compatibility with old versions of IPA, all new flags must be
accompanied by SYSTEM. Permissions with *only* the SYSTEM flag retain
the old semantics.



Data in the permission entry
----------------------------------------------------------------------------------------------

The new permission objects will contain all data needed to generate (or
check) the ACI. This means IPA code will no longer need to parse ACIs
(except when upgrading old permissions). Any change to a Permission will
result in removing any old ACI and adding a new one.

The new schema (including schema from other related designs) is as
follows:

| ``attributeTypes: (2.16.840.1.113730.3.8.11.42 NAME 'ipaPermDefaultAttr' DESC 'IPA permission default attribute' EQUALITY caseIgnoreMatch ORDERING caseIgnoreOrderingMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 X-ORIGIN 'IPA v3' )``
| ``attributeTypes: (2.16.840.1.113730.3.8.11.43 NAME 'ipaPermIncludedAttr' DESC 'IPA permission explicitly included attribute' EQUALITY caseIgnoreMatch ORDERING caseIgnoreOrderingMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 X-ORIGIN 'IPA v3' )``
| ``attributeTypes: (2.16.840.1.113730.3.8.11.44 NAME 'ipaPermExcludedAttr' DESC 'IPA permission explicitly excluded attribute' EQUALITY caseIgnoreMatch ORDERING caseIgnoreOrderingMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 X-ORIGIN 'IPA v3' )``
| ``attributeTypes: (2.16.840.1.113730.3.8.11.45 NAME 'ipaPermBindRuleType' DESC 'IPA permission bind rule type' EQUALITY caseExactMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 SINGLE-VALUE X-ORIGIN 'IPA v3' )``
| ``attributeTypes: (2.16.840.1.113730.3.8.11.46 NAME 'ipaPermLocation' DESC 'Location of IPA permission ACI' EQUALITY distinguishedNameMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 SINGLE-VALUE X-ORIGIN 'IPA v3' )``
| ``attributeTypes: (2.16.840.1.113730.3.8.11.47 NAME 'ipaPermRight' DESC 'IPA permission rights' EQUALITY caseIgnoreMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 X-ORIGIN 'IPA v3' )``
| ``attributeTypes: (2.16.840.1.113730.3.8.11.48 NAME 'ipaPermTargetFilter' DESC 'IPA permission target filter' EQUALITY caseIgnoreMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 X-ORIGIN 'IPA v3' )``
| ``attributeTypes: (2.16.840.1.113730.3.8.11.49 NAME 'ipaPermTarget' DESC 'IPA permission target' EQUALITY caseIgnoreMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 X-ORIGIN 'IPA v3' )``
| ``objectClasses: (2.16.840.1.113730.3.8.12.21 NAME 'ipaPermissionV2' DESC 'IPA Permission objectclass, version 2' SUP ipaPermission AUXILIARY MUST ( ipaPermissionType $ ipaPermBindRuleType $ ipaPermRight $ ipaPermLocation ) MAY ( ipaPermDefaultAttr $ ipaPermIncludedAttr $ ipaPermExcludedAttr $ ipaPermTargetFilter $ ipaPermTarget ) X-ORIGIN 'IPA v3' )``

(Note: ipaPermIncludedAttr was known as ipaPermAllowedAttr in the first
versions of this document, and in the patches that implement it. It has
been renamed for `V3/Managed Read
permissions <V3/Managed_Read_permissions>`__.)

-  ipaPermIncludedAttr lists the attributes of the ACI; for example:

| ``   ipaPermIncludedAttr: cn``
| ``   ipaPermIncludedAttr: sn``

-  the other ipaPerm*Attr are for Managed permissions, will be explained
   in `V3/Managed Read permissions <V3/Managed_Read_permissions>`__
-  ipaPermBindRuleType is for general access ACIs, will be explained in
   `V3/Anonymous and Any
   permissions <V3/Anonymous_and_Any_permissions>`__; default:

``   ipaPermBindRuleType: permission``

-  ipaPermLocation is the location of the ACI in the tree.

``   ipaPermLocation: cn=users,cn=accounts,dc=idm,dc=lab,dc=eng,dc=brq,dc=redhat,dc=com``

-  ipaPermRight is the right(s) the permission grants; for example:

| ``   ipaPermRight: add``
| ``   ipaPermRight: write``

-  ipaPermTargetFilter is the target filter part of the ACI, for
   example:

``   ipaPermTargetFilter: (memberOf=cn=ipausers,cn=groups,cn=accounts,dc=idm,dc=lab,dc=eng,dc=brq,dc=redhat,dc=com)``

-  ipaPermTarget is the target part of the ACI, as a DN with possible
   wildcards; for example:

``   ipaPermTarget: uid=*,cn=users,cn=accounts,dc=idm,dc=lab,dc=eng,dc=brq,dc=redhat,dc=com``



Moving ACIs out of the root
----------------------------------------------------------------------------------------------

ACIs will be created on appropriate containers, rather than in the
$SUFFIX. This will increase performance of LDAP operations, as fewer
ACIs need to be checked for an entry.

The location of the ACI will be determined by ``ipaPermLocation``.



Option/Attribute mapping
----------------------------------------------------------------------------------------------

Due to technical reasons, API parameters need to be named the same as
LDAP attributes. (This assumption runs rather deep in the framework, and
makes adding new LDAP attributes for previously existing options
somewhat complicated.)

In each of the following pairs of options, the two are mutually
exclusive; if the second is present it acts as the first one. The second
option will only be available when called with a lower API version than
the one where V2 permissions are introduced.

| ``  API name            CLI name``
| ``  -------------------------------``
| ``/ ipapermright        permissions``
| ``\ permissions         (no_cli)``
| ``/ ipapermincludedattr  attrs``
| ``\ attrs               (no_cli)``
| ``/ ipapermtargetfilter filter``
| ``\ filter              (no_cli)``
| ``/ ipapermlocation     subtree``
| ``\ subtree             (no_cli)``

A new no-cli option will be added: ``ipapermtarget``.

IPA will validate the filter before setting it by performing a search.

Finally, 3 existing options will be available to set the above:

``memberof`` will set target filter to ``(memberOf={group})``, after
checking that the group exists. ``memberof`` and ``filter`` are mutually
exclusive.

``targetgroup`` will set target to
```ldap:///{group}`` <ldap:///%7Bgroup%7D>`__, after checking that the
group exists. ``targetgroup`` and ``target`` are mutually exclusive.

``type`` will set target and subtree to one of pre-defined values
correspondint to IPA object types. ``type``, ``subtree`` and ``target``
are mutually exclusive.

These values will be output if the filter/target/location values match
the respective patterns.

Note that this introduces an incompatibility: when a permission "type"
is set, updating it to "memberof" or "targetgroup" will NOT reset all
aspects of "type", as was the case before. (This change is part of
`ticket 2355 <https://fedorahosted.org/freeipa/ticket/2355>`__ - Allow
filter and subtree to be added in same permission)

Also Note that ``subtree`` (ipapermlocation) must now always refer to an
existing entry; wildcards or non-existent DNs are not allowed.



add_noaci command
----------------------------------------------------------------------------------------------

There is no expectation of backwards compatibility for
``permission_add_noaci``. This command will be marked as internal.

Examples
----------------------------------------------------------------------------------------------

See the Test Plan below.



Compatibility with old clients
----------------------------------------------------------------------------------------------

For clients which report an API version lower than the one of this
feature, the output will be modified to keep basic compatibility. A new
"permissions2" capability will be added to track the version.

Attributes ipapermright, ipapermincludedattr, ipapermtargetfilter,
ipapermlocation will be output under their old names: permissions,
attrs, filter, subtree.

The ``type``, ``targetgroup``, and ``memberof`` will be output as single
values rather than single-element lists.

For ``subtree`` (ipapermlocation), the entry will also be made
single-valued, and a 'ldap:///' prefix will be added to it.



Modifying and Upgrading Permissions
----------------------------------------------------------------------------------------------

A permission object can not be modified if:

-  it has any unknown flags (``ipaPermissionType``), or
-  it has \*only\* the SYSTEM flag

The process for modifying a permission (and its ACI) is as follows:

1. Before a permission object is changed, location of its corresponding
ACI is found using the old permission name and ipaPermLocation (which
defaults to $SUFFIX).

2. If the permission object does not have any flags (i.e. it is an
old-style permission), the ACI is parsed and appropriate attributes are
set on the Python representation of the entry, the SYSTEM and V2 flags
are added, and the ipaPermissionV2 objectclass is added. The new ACI
string is calculated from this.

3. If the location (subtree) did \*not\* change, the old ACI is replaced
by the new ACI.

4. If the location (subtree) changed, the old ACI is removed.

5. The permission entry is updated in LDAP.

6. If the location (subtree) changed, the new ACI is added.



Adding and Deleting Permissions
----------------------------------------------------------------------------------------------

When adding, the ACI is inserted after the permission entry.

When deleting, the ACI is deleted first, then the permission entry.



Mass update
----------------------------------------------------------------------------------------------

Old permisisons can be updated to V2 by running the process in
"Modifying and Upgrading Permissions" on all permissions without the
SYSTEM flag. We can run this process on upgrade (for example in the
release where we introduce an audit tool), or just let users upgrade
manually.

The decision on when/if to do this has been postponed.

(The upgrade-on-modify mechanism, and permission-{find,show} for
old-style permissions, need to be in place in any case, because
old-style permisisons can be created on old servers.)



Find/show for old permissions
----------------------------------------------------------------------------------------------

Old-style permissions will continue be recognized by
permission-{find,show} commands.

The show command will upgrade the entry (in memory, without commiting to
LDAP), before outputting data.

This "output only" upgrade is the same as step 2 in "Modifying and
Upgrading Permissions", except it will not touch the flags, i.e. "V2"
and "SYSTEM" will not be added, and the 'ipaPermissionV2' objectclass is
not added.

The find command will iterate through all old-stryle permissions, do the
"output-only" upgrade on each one, and filter the result based on given
options.



Read rights
----------------------------------------------------------------------------------------------

The "read", "search", and "compare" rights are added to "write", "add",
"delete", and "all" in the list of rights that can be granted
(ipaPermRight).

UI

The UI will need to be updated to use the attribute names in the API,
see the "Option/Attribute mapping" section.



Feature Management
------------------



UI

As part of the related `ticket
2355 <https://fedorahosted.org/freeipa/ticket/2355>`__, The UI is
reorganized to allow type, filter, subtree, targetgroup to be specified
independently.

CLI

-  As part of the related `ticket
   2355 <https://fedorahosted.org/freeipa/ticket/2355>`__, the type,
   filter, subtree, targetgroup options are no longer mutually exclusive
-  The type is a convenience option that sets rawfilter and subtree on
   input, and is computed from them on output



Test Plan
---------

See `dedicated Test Plan page <V3/Permissions_V2/tests>`__