Access_Control
==============



The problem
-----------

Fine-grained access to parts of the DIT

We need to be able to say WHO can do WHAT to WHOM

Writes
----------------------------------------------------------------------------------------------

The WHO can be:

-  a user or users
-  a group or groups

The WHAT can be:

-  modify one or more attributes on an entry (ex.)

   -  reset a password
   -  modify an address
   -  add a user to a group

-  add a new entry
-  remove an existing entry

The WHOM can be:

-  a user or all users
-  a group or groups
-  self

Reads
----------------------------------------------------------------------------------------------

Any user can read any other user or group. This is simply the unix way.
Some attributes such as password will always be protected. This was
extended to services in IPA v1 and will continue in v2 so one can see
what services are available.

We will not by default grant read access to:

-  Machines
-  Policy



Other rights
----------------------------------------------------------------------------------------------

The interesting rights that 389 DS provides that IPA would use are:
read, write, add, delete, search, and selfwrite.



Anonymous Access
----------------------------------------------------------------------------------------------

Currently there is read access to the entire subtree (minus a few
important attributes like userPassword) for anonymous users. In order to
limit access to services, machines, etc. I propose that we modify the
anonymous read access to include just users and groups. Basically just
enough so that nss_ldap will still work with an anonymous bind.



The Solution
------------

From a high level we will:

-  Create groups (task groups) that are allowed to perform tasks
-  Create an ACI that grants permission for a group to perform a
   specific task
-  Create groups (role groups) that will be be members of task groups,
   granting the role group the underlying permission
-  Assign users and groups to role groups



Task Groups and ACIs
----------------------------------------------------------------------------------------------

For the purposes of this discussion one task == one ACI "group". These
groups will be stored outside of cn=accounts so they don't interfere
with standard groups. I propose cn=taskgroup, dc=SUFFIX. The structural
objectclass will be groupOfNames.

1 task == 1 ACI

There may be multiple ACIs for a given high level task but it will be
implemented by combining low level tasks.

Take adding a user as an example. When a user is added we also
automatically add that user to the default IPA group (ipausers). For the
"Add a user" task we will need 3 ACIs:

-  one to allow creating users and one to allow adding a user to a group
   (access to the member attribute).
-  one to allow writing the member attribute of a group
-  allow the a password reset

A 3rd task group may be created, "Add an IPA User", that is a member of
these 3 task groups, combining them into a logical task. This will help
limit the total number of ACIs in the system, particularly with
duplication, because a task may be a member of several groups.

The ACIs will be stored in cn=accounts, the same way as in v1.



Role Groups
----------------------------------------------------------------------------------------------

We will also have role groups. A role is a collection of tasks or other
roles. These will be things like helpdesk, user admin, group admin,
replica admin, etc. These will also be stored outside of the cn=accounts
tree. These groups are for data management only and not for system
management. Perhaps in cn=delegationgroup, dc=SUFFIX. The structural
objectclass will be groupOfNames.

A role here is not to be confused with RBAC or native 389 DS/RHDS roles.
It is simply a term I'm using to designate the permissions of an
administrative group.

Rights are assigned by adding tasks as a member to these role groups. So
a helpdesk user may be able to see users, groups, and systems but only
have write access on user passwords. We would have several ACIs/task
groups to handle this:

-  reset password
-  view users
-  view groups
-  view systems

An IPA install will include a default set of canned tasks, including
things like:

-  Add/Delete user
-  Add/Remove user to group
-  View Machines
-  Change password
-  etc

Each one of these is a discrete ACI granting the appropriate permission
to a unique group (like add_user group, add user_to_group, etc). The
task group name itself is unimportant, it just needs to be unique and
hopefully meaningful.

We will continue to have the admin user and the admins group. This is
the super-user and group of users that can do whatever they would like
in the IPA world.



The bottom line
----------------------------------------------------------------------------------------------

-  An ACI grants permissions to a task group. There is a single ACI for
   each task group. One may group ACIs together by adding their task
   groups the same group.
-  A role is the member of one or more task groups
-  A user/group/role is a member of one or more roles



ACI Details
----------------------------------------------------------------------------------------------

An ACI is made up 3 major components that we're interested in:

-  source/bind rule (WHO is being granted access)
-  rights (read, write, etc) (WHAT is being granted)
-  target (WHO you are granting rights to)

Source
^^^^^^

This will always be a Task group.

Rights
^^^^^^

Will be: read, write, add, delete, search, and/or selfwrite

All ACIs will GRANT access, not deny it.

Target
^^^^^^

The target may be a set of attributes, a portion of the subtree a filter
or a combination of these.

Examples
^^^^^^^^

Note that in these examples I'm not using the new location to store the
task groups. You can apply these to a v1 IPA server to see how it works.



Create a new user
'''''''''''''''''

``aci: (target="``\ ```ldap:///uid=`` <ldap:///uid=>`__\ ``*,cn=users,cn=accounts,dc=example,dc=com")(version 3.0;acl "add_user";allow (add) groupdn="``\ ```ldap:///cn=add_user,cn=taskgroups,dc=example,dc=com`` <ldap:///cn=add_user,cn=taskgroups,dc=example,dc=com>`__\ ``";)``

But this isn't enough. We also add the new user to the default IPA
group. Here is an ACI which allows that, specifically limiting the write
operation to the default group. This would be difficult to keep in sync
in reality but illustrates how tight we can make things.

``aci: (targetattr=member)(target="``\ ```ldap:///cn=ipausers,cn=groups,cn=accounts,dc=example,dc=com`` <ldap:///cn=ipausers,cn=groups,cn=accounts,dc=example,dc=com>`__\ ``")(version 3.0;acl "add_user_to_default_group";allow (write) groupdn="``\ ```ldap:///cn=add_user_to_default_group,cn=taskgroups,dc=example,dc=com`` <ldap:///cn=add_user_to_default_group,cn=taskgroups,dc=example,dc=com>`__\ ``";)``

And still this isn't enough. We also try to set the password. Rather
than doing this by setting the userPassword attribute we do an LDAP
password change. So we need to grant permission to change passwords as
well (see Reset password).

So now we have 3 task groups to add a user. If we wanted we could create
a 4th task group which combines these as a shortcut. This shows that
good descriptions will be required so that people making delegations can
understand what each task does. So this combined task should be named
something like create_ipa_user.

The task entry for the add_user ACI will look like:

::

   dn: cn=add_user,cn=taskgroups,dc=example,dc=com
   objectclass: top
   objectclass: groupofnames
   cn: add_user
   description: Allowed to add new users
   member: uid=tuser,cn=users,cn=accounts,dc=example,dc=com

The task entry for create_ipa_user will look like:

::

   dn: cn=create_ipa_user,cn=taskgroups,dc=example,dc=com
   objectclass: top
   objectclass: groupofnames
   cn: create_ipa_user
   description: Allowed to create IPA users
   member: cn=add_user,cn=taskgroups,dc=example,dc=com
   member: cn=add_user_to_default_group,cn=taskgroups,dc=example,dc=com
   member: cn=change_password,cn=taskgroups,dc=example,dc=com



Reset password
''''''''''''''

Crafting some rules may require a fairly detailed knowledge of LDAP and
the IPA implementation, as demonstrated with this long list of
attributes that may be written when resetting a password.

``aci: (targetattr = "userPassword || krbPrincipalKey || sambaLMPassword || sambaNTPassword || passwordHistory")(version 3.0; acl "change_password"; allow (write) groupdn = "``\ ```ldap:///cn=change_password,cn=taskgroups,dc=example,dc=com`` <ldap:///cn=change_password,cn=taskgroups,dc=example,dc=com>`__\ ``";)``



Remove user
'''''''''''

Once the basic structure of the ACIs is found then granting specific
rights becomes easier and easier. This is the same ACI as add_user
simply with a different right. One would also need to be a member of the
"modify group membership" group so that the membership may be modified.

``aci: (target="``\ ```ldap:///uid=`` <ldap:///uid=>`__\ ``*,cn=users,cn=accounts,dc=example,dc=com")(version 3.0;acl "delete_user";allow (delete) groupdn="``\ ```ldap:///cn=delete_user,cn=taskgroups,dc=example,dc=com`` <ldap:///cn=delete_user,cn=taskgroups,dc=example,dc=com>`__\ ``";)``



Side effects
''''''''''''

We may need to add in specific ACIs that prevent the deletion of
specific users and groups. admin comes to mind.

Roles
'''''

These ACIs will be rolled up into a set of Roles, a set of which will be
pre-defined when IPA is shipped. These roles can then be customized by
IPA administrators to fit the site needs.

Helpdesk
        

Helpdesk users can typically reset passwords.

So we start with a helpdesk role:

::

   dn: cn=helpdesk,cn=rolegroups,dc=example,dc=com
   objectclass: top
   objectclass: groupofnames
   cn: helpdesk
   description: Helpdesk
   member: uid=tuser,cn=users,cn=accounts,dc=example,dc=com

And add that role to the task for changing passwords:

::

   dn: cn=change_password,cn=taskgroups,dc=example,dc=com
   objectclass: top
   objectclass: groupofnames
   cn: create_ipa_user
   description: Allowed to change passwords
   member: cn=helpdesk,cn=rolegroups,dc=example,dc=com



Use Cases
----------------------------------------------------------------------------------------------



Separate admins for separate containers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Currently all users are in cn=users,cn=accounts. If we allow users to be
created in another part of the tree (aka another user container) then we
can create per-container admins, granting full access to this container
to that admin.

Alternatively we can grant access based on the value of an attribute in
a record that isn't part of the DN using ``targetfilter`` to set the
target based on the value of an attribute:

``(targetfilter = "(|(ou=accounting)(ou=engineering))")``



Limit self-service changes by attribute
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Add ability to limit what attributes can be modified on the self-service
page



Flexibility in attributes that may be delegated
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The list of attributes that one can grant write access to needs to be
configurable



Delegate entry (user, group, whatever) creation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In v1 we only delegate attribute writes, not add or delete permissions.
Some granularity can be obtained by granting access only to users
(cn=users), groups (cn=groups), etc. along with attributes.



Additional UI Capabilities Needed
---------------------------------

-  Means to select entries by container (if supported)
-  Means to select one or more entries (could be users or groups or
   both)
-  Means to manage list of attributes that may be delegated
-  Means to manage add/list/delete delegations



Delegate anything
----------------------------------------------------------------------------------------------

-  add users/groups/systems/other
-  delete users/groups/systems/other
-  allow arbitrary attributes (potential for abuse, breakage?)
-  An admin is a special kind of delegation, need a way to recognize
   this in the UI



New ACI Parser
----------------------------------------------------------------------------------------------

-  a fuller ACI class that can handle more complex syntax

   -  Needs to understand LDAP target
   -  Ability to set source to targetattr, targetfilter and/or target
   -  Set rights as a list
   -  Validate ACIs before they are written



UI Requirements
----------------------------------------------------------------------------------------------

It is difficult to select an individual ACI over LDAP. What we will do
instead is slurp in all of them and prove that to the UI to display.
This should be refreshed between operations to avoid concurrency issues.

Once an ACI is written to LDAP it is immediately in effect.

The following operations are needed:

-  CRUD for managing Task groups
-  CRUD for managing ACIs
-  CRUD for managing Role groups

ACI
^^^

An ACI has 4 attributes:

-  name - a description of the ACI
-  source/bind rule - who is being authorized. This will generally be a
   task group\*
-  rights - read, write, add, delete, search, and selfwrite (may be more
   than one)
-  target - May specify whether = or != one or more of the following:

   -  target - an LDAP uri pointing at a specific entry or a subtree
   -  targetattr - one or more attributes
   -  targetfilter - an LDAP filter

There are a couple of special LDAP bind rules:

-  userdn = "ldap:///self"
-  userdn = "ldap:///anyone"

self is used when defining an ACI for self-service. These are things
that you can do in your own record.

anyone is any bind, including an anonymous one.



Task Groups
^^^^^^^^^^^

A task group has 3 attributes:

-  cn (the group name)
-  description
-  member

A member is a role group(s)

The membership of task groups will be read only. This will be managed
from the Role Groups. Otherwise may seem a bit backwards. What we are
doing with Role groups is defining what tasks a role may execute. To do
that we add the Role to the task group.



How to create a new Task
''''''''''''''''''''''''

#. Create a new task group for the task
#. Create an ACI and assign it to the task group you just created



Role Groups
^^^^^^^^^^^

A role group has 3 attributes:

-  cn (the group name)
-  description
-  member

A member can either be another role group, a group or a user.

The Add/Update operations need to provide the ability to manage
membership of the task group. This defines the users/groups/roles that
may do the tasks associated with the role.

It also needs to provide the ability to manage which tasks a role may
operate on. By adding a task to a role the role gets added as a group
member of the task.



Additional Possible Capabilities
--------------------------------

These would be for v3 or beyond.



Limit Bind Rules
----------------------------------------------------------------------------------------------

We can add on additional bind rules for making changes if desired by:

-  IP
-  time of day
-  IP
-  hostname



Current State of Affairs
------------------------

Currently all ACIs are put into cn=accounts,dc=example,dc=com and can
grant the ability to write a fixed set of attributes from one group of
users to another.

| ``aci: (targetattr="title")(targetfilter="(memberOf=cn=bar,cn=groups,cn=accounts``
| ``,dc=example,dc=com)")(version 3.0;acl "foobar";allow (write) groupdn="``\ ```ldap://`` <ldap://>`__
| ``/cn=foo,cn=groups,cn=accounts,dc=example,dc=com";)``



Who writes the ACIs, tasks and roles?
-------------------------------------

Plugin authors, who know best what access may be granted for their given
operations, will create a list of ACIs for the plugin. This will likely
revolve around the CRUD operations to grant create, read, update and
delete access. There are special cases too, such as granting write
access to specific attributes in the case of passwords.

A predefined set of Roles will be created as well. The initial list will
be:

-  Helpdesk
-  User admin
-  Group admin
-  Replica admin
-  Host admin
-  Service admin
-  CA admin
-  Netgroup admin
-  automount admin
-  netgroups admin



How will ACIs be created?
-------------------------

There will be no web UI for creating new ACIs.



How will ACIs be modified?
--------------------------

The only modifications to ACIs will be in the list of attributes that
they cover and will only be available from the CLI.

For example, the edit User ACI may need to be expanded to include more
attributes than we grant by default. This CLI capability will let an
admin select from a set of attributes those whihc may be written.

Diagram
-------

A picture for your viewing pleasure.

Remember:

-  An ACI grants access to an operation to a single Task group
-  A Role is a member of one or more Task groups
-  A user, group or Role is a member of one or more Roles

.. figure:: Delegation.png
   :alt: Picture

   Picture