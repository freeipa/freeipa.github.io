\__NOTOC_\_

Introduction
------------

This represents a large re-design of the `Access
Control <FreeIPAv2:Access_Control>`__ system.

acis, taskgroups and rolegroups are being replaced by permissions,
privileges and roles.

Roughly:

-  acis and taskgroups are combined into permissions.
-  rolegroups are renamed to privileges.
-  a new group type is created, roles. A role may not be nested.

The way we are using 389-ds ACIs is relatively unchanged. We create an
ACI that grants permission to a group (in this case a permission). A
permission is a member of a privilege and a privilege is a member of a
role.

Permissions
-----------

The permissions object is a simple group, no nesting. It is the target
of an aci. The permissions plugin will make calls to the existing,
internal only aci plugin to create acis targeting a given permission.
The linkage between the two is the description of the group and the name
of the aci. By implication there is one aci allowed per permission.

A permission is a groupofnames.

Commands:

-  permission-add
-  permission-del
-  permission-find
-  permission-mod
-  permission-show

Privileges
----------

A privilege is a way to logically combine permissions. There are some
tasks, such as as adding a user, that require multiple permissions to
perform.

A privilege is a nestedgroup (needs memberOf). We don't provide
facilities to make a privilege a member of another privilege.

Commands:

-  privilege-add
-  privilege-del
-  privilege-find
-  privilege-mod
-  privilege-show
-  privilege-add-permission
-  privilege-remove-permission

Role
----

A role is a group of privileges. Users, groups, etc. will be assigned as
members of a role and inherit privileges and permissions via memberOf.

A role is a nestedgroup. We don't provide facilities to make a role a
member of another role.

Commands:

-  role-add
-  role-del
-  role-find
-  role-mod
-  role-show
-  role-add-member
-  role-remove-member
-  role-add-privilege
-  role-remove-privilege

Allowed member types are: users, groups, hosts and hostgroups.

.. _differences_to_old_system:

Differences to old system
-------------------------

One of the more obvious differences to the old system is in the user
interface. The membership will be made to be more logical, so rather
than adding a rolegroup to a taskgroup you add a permission to a
privilege and a privilege to a role.

Two additional plugins will be required to provide some other
capabilities of the aci plugin:

-  selfserve will manage self-service plugins for various objects (most
   likely just users and hosts)
-  delegation will manage v1-style group to group delegation (group A
   can write these attributes of the users in group B).

APPENDIX
--------

Here is a breakdown of what the various objects look like.

.. _permission_entry:

permission entry
~~~~~~~~~~~~~~~~

A permission entry and an aci pointing at that permission. The
permission links back to the ACI though the permission description. The
ACI is stored in ``dc=example,dc=com``.

::

   dn: cn=addusers,cn=permissions,cn=accounts,dc=example,dc=com
   objectClass: top
   objectClass: groupofnames
   cn: addusers
   description: Add Users
   member: cn=useradmin,cn=privileges,cn=accounts,dc=example,dc=com

::

   aci: (target = "ldap:///uid=*,cn=users,cn=accounts,dc=example,dc=com")(version
     3.0;acl "Add Users";allow (add) groupdn = "ldap:///cn=addusers,cn=permission
    s,cn=accounts,dc=example,dc=com";)

.. _privilege_entry:

privilege entry
~~~~~~~~~~~~~~~

A privilege entry using the example permission. Note the other
permissions that make up this privilege. It is a member of the heldesk
role.

::

   dn: cn=useradmin,cn=privileges,cn=accounts,dc=example,dc=com
   objectClass: top
   objectClass: groupofnames
   objectClass: nestedgroup
   cn: useradmin
   description: User Administrators
   memberOf: cn=addusers,cn=permissions,cn=accounts,dc=example,dc=com
   memberOf: cn=change_password,cn=permissions,cn=accounts,dc=example,dc=com
   memberOf: cn=add_user_to_default_group,cn=permissions,cn=accounts,dc=example,d
    c=com
   memberOf: cn=removeusers,cn=permissions,cn=accounts,dc=example,dc=com
   memberOf: cn=modifyusers,cn=permissions,cn=accounts,dc=example,dc=com
   member: cn=helpdesk,cn=roles,cn=accounts,dc=example,dc=com

.. _role_entry:

role entry
~~~~~~~~~~

The helpdesk role. Note that the memberOf permissions have carried
forward. This role has no current members of its own.

::

   dn: cn=helpdesk,cn=roles,cn=accounts,dc=example,dc=com
   objectClass: top
   objectClass: groupofnames
   objectClass: nestedgroup
   cn: helpdesk
   description: Helpdesk
   memberOf: cn=useradmin,cn=privileges,cn=accounts,dc=example,dc=com
   memberOf: cn=addusers,cn=permissions,cn=accounts,dc=example,dc=com
   memberOf: cn=change_password,cn=permissions,cn=accounts,dc=example,dc=com
   memberOf: cn=add_user_to_default_group,cn=permissions,cn=accounts,dc=example,d
    c=com
   memberOf: cn=removeusers,cn=permissions,cn=accounts,dc=example,dc=com
   memberOf: cn=modifyusers,cn=permissions,cn=accounts,dc=example,dc=com
