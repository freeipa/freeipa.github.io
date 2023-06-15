Overview
--------

Corporate or big setups of Identity Management system often requires an
advanced user life-cycle management capabilities (like staging area for
new or deleted users) or standard LDAP interface for adding or removing
users that can be used by provisioning systems.



Use Cases
---------

.. _external_provisioning_system:

External Provisioning System
~~~~~~~~~~~~~~~~~~~~~~~~~~~~



User Story
^^^^^^^^^^

As an Administrator, I want to configure my employee provisioning system
to manipulate the employee records to FreeIPA via LDAP protocol using
standard LDAP objectclasses, so that I do not have to request FreeIPA
specific integration with my provisioning system vendor.

Description
^^^^^^^^^^^

The following supported workflows are expected:

#. *Create a user*: provisioning system adds user to staging tree are
   using standard LDAP add. Not all fields that standard FreeIPA user
   has need to be specified. Provisioning system **may or may not** use
   the Freeipa CLI to add the user.
#. *Activate user*: activate user by moving it from staging area to
   active user area. All attributes required by FreeIPA are generated if
   they are missing. All activations of a user accounts **must** be done
   on the same server, else it can leads to replication issues as
   attribute uniqueness can reject some operation.
#. *Assign group memberships*: add group membership to activated user.
   Membership can only be added in active users tree. Staged or deleted
   users have no membership information.
#. *Update user*: modify user attributes
#. *Change password*: administratively change user password
#. *Delete user*: move user from active user tree to staging tree to
   keep it's attributes (like UID) and settings in case the user returns
   back and need to be restored
#. *Restore deleted user*: move deleted user back to the tree of active
   users
#. *Lock user*: temporarily lock user so that it is not accessible
#. *Unlock user*: unlock temporarily locked user

.. _privilege_separation_for_employee_entry_and_activation:

Privilege Separation for Employee Entry and Activation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _user_story_1:

User Story
^^^^^^^^^^

As an Administrator, I want to allow Human Resources Administrators to
add future employees to FreeIPA and in the same time I do not want to
allow them activating them, so that I can delegate that privilege to my
Security Administrator after the employee credentials are verified on
the first day of employment.

Design
------

.. _user_status:

User status
~~~~~~~~~~~

In User Life Cycle, a user account is stored in a LDAP repository as an
entry in the DIT. A user account follows a workflow that changes the
status of the user. There are 3 differents status:

-  Stage

   -  This is an initial state where some of the user properties may not
      be already set
   -  In this state the user account can not be used to authenticate.
   -  The ldap entry of a user account is stored: cn=staged
      users,cn=accounts,cn=provisioning,$SUFFIX
   -  The staging container usually contains few user account that
      represent the few new comers in the organisation.

-  Active

   -  This is the normal state where all necessary properties are set
   -  In this state the user account can be used to authenticate (if
      user is *enabled*)
   -  The ldap entry of a user account is stored:
      cn=users,cn=accounts,$SUFFIX
   -  The active container contains hundreds/tousands user accounts that
      represent all the people currently in the organisation.

-  Delete

   -  This is the state of *deleted* active user account. Most of its
      properties have been preserved. Unpreserved properties are for
      example membership properties that have been reset
   -  The user account can not be used to authenticate
   -  The ldap entry of a user account is stored: cn=deleted
      users,cn=accounts,cn=provisioning,$SUFFIX
   -  The deleted container contains hundreds/tousands user accounts
      that represent all the people who left the organisation.
   -  The deleted container preserves former users to conform legacy,
      law and policies. It can help to recover user properties in case a
      former user returns back to the organization.

-  Active account

   -  in addition to active user, a special container contains all the
      active accounts. Accounts are not only users
   -  the ldap entry of active accounts is stored: *cn=accounts,$SUFFIX*

workflows
~~~~~~~~~

``The main workflow allows  moves: ``\ *``stage``*\ `` ``\ **``-->``**\ `` ``\ *``active``*\ `` ``\ **``<-->``**\ `` ``\ *``delete``*

-  stage -> active: The appropriate set of approvals are granted and a
   new comer is now fully operational in the organisation
-  active -> delete: The user leaves temporary the organisation
-  delete -> active: The user join back the organisation and recover
   some of his previous settings

The workflow allows to recreate a user if the entry gets corrupted:

-  delete -> stage: The entry got corrupted and was deleted. Moving it
   back to ' *Stage* ' the entry can later be activated again. The
   following FreeIPA attribute are preserved: ' *ipaUniqueID* ' , '
   *uidNumber* ' and ' *gidNumber* '.

The workflow allows to add user accounts in:

-  stage: the user needs some approval before being active
-  active: the user does not need any approval before being active

The workflow allows to delete user accounts from:

-  staging: The user account will never receive the approval in order to
   be active
-  delete: The user will never join back the organisation

Note: delete action will erase permanently the entry from the
repository. It is a LDAP DEL.

The workflow allows to modify user account from:

-  stage: modify the user account that remains in stage
-  active: modify the user account that remains in active

::

                                                       -- find ---+                     -- find --+
                                                       -- show ---+                     -- show --+
                                                       -- add-----+                               |
                                                       -- mod ----+                               |
                                                                  |                               |
                                                                  V                               V
                    -------------------                    ----------------                  ---------------
       -- find -->  |                 |                    |              |                 |              |
       --- mod -->  |                 |                    |              |                 |              |
       --- add  --> |                 |  --- activate -->  |              |  --- delete --> |              |
      <-- delete -- |      STAGE      |                    |     ACTIVE   | <-- undelete -- |    DELETE    | -- delete ->
                    |                 |                    |              |                 |              |
                    | <plg. stageuser>|                    |  <plg user>  |                 |  <plg. user> |
       -- show -->  |                 |                    |              |                 |              |
                    -------------------                    ----------------                 ---------------
                             ^                                                                      /
                             \                                                                    /
                               ----------------------- add (from-delete opt.) ---------------------

.. _stageuser_plugin:

stageuser plugin
^^^^^^^^^^^^^^^^

.. _add_a_stage_entry:

Add a stage entry
'''''''''''''''''

-  Support engineer can use the following command

   -  ipa stageuser-add <*user_identifier*> --first=<*first name*>
      --last=<*last name*>

      ::

         ipa stageuser-add tuser  --first=test --last=user

   -  if needed, command may specify more details about the user,
      including the password

      ::

         ipa stageuser-add  tuser --first=Test --last=User --random --manager=muser --phone 123456789

   -  ipa *stageuser-add* supports almost the same options as
      *user-add*, but compare to user-add here is the list of
      differences:

      -  noprivate: no supported as a stage entry has no private group
      -  manager: must be an active user

   -  Filled with
      `placeholders <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#Stage_placeholders>`__
      the entry will look like

::

   dn: uid=tuser,cn=staged users,cn=accounts,cn=provisioning,dc=example,dc=com
   objectClass: top
   objectClass: person
   objectClass: organizationalperson
   objectClass: posixAccount
   cn: Test User
   sn: User
   uid: tuser
   uidNumber: -1
   gidNumber: -1
   homeDirectory: /home/tuser
   loginShell: autogenerate

-  A stage user can also be created from a former user. There is still
   discussion (see
   `1 <https://www.redhat.com/archives/freeipa-devel/2015-July/msg00516.html>`__
   and
   `2 <https://www.redhat.com/archives/freeipa-devel/2015-August/msg00022.html>`__
   ) if the former user can be picked up from the 'Delete' container or
   from the 'Active' container or both.

Currently the proposed interface are

::

   stageuser-add <uid> --from-delete

   or

   user-undel <uid> --to-stage

   or

   user-unactivate <uid>

The drawback of the first CLI (stageuser-add) is that lastname/firstname
are required option, but when the 'uid' entry is taken from the 'Delete'
container lastname/firstname are useless.

.. _provision_stage_entry:

Provision stage entry
'''''''''''''''''''''

-  As described in the first `Use
   case <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#Use_Cases>`__,
   provisioning systems (external) create the vast majority of *Stage*
   entries (using or not the FreeIPA CLI).

   -  Provisioned entries *MUST* follow the following rules

      -  Provisioning system places a staged user entry to *cn=staged
         users,cn=accounts,cn=provisioning,SUFFIX*
      -  Entry RDN attribute is ' *uid* ' (see `Supported Staged
         entries <https://www.redhat.com/archives/freeipa-devel/2014-May/msg00407.html>`__)

   -  Entry may contain both data and
      `placeholders <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#Stage_placeholders>`__.
      Note that when the entry will become active, some of the
      data/placeholders may be changed.
   -  Provisoning system can create an entry with few constraints, that
      mean that the
      `activation <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#Activate_StageUser>`__
      of a stage entry must be done with care. So provisioning systems
      is the main justification why activation will be done using a
      `ADD-DEL <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#MODRDN_vs._ADD-DEL>`__
      approach
   -  A *Stage* entry does not need to have all attributes that standard
      FreeIPA user has. In order to allow the correct processing of User
      Life Cycle, staged users must have a minimal set of attributes

::

   dn: uid=tuser,cn=staged users,cn=accounts,cn=provisioning,dc=example,dc=com
   objectClass: top
   objectClass: inetOrgPerson
   cn: Test User
   sn: User
   uid: tuser

.. _activate_stageuser:

Activate StageUser
''''''''''''''''''

Activating a user is a major step in the User Life Cycle. It allows
FreeIPA to start managing the entry and the user to authenticate with
it. This action is only authorised to Support Engineer.

-  Support Engineer is using FreeIPA CLI: ipa stageuser-activate
   <*user_identifier*>

   ::

      ipa stageuser-activate tuser

-  The CLI supports only one account ID (no series of accounts can be
   activated in a row)

-  This operation 'moves' the entry, using LDAP ADD on the destination
   entry then DEL on the source entry

::

   Source:           cn=staged users,cn=accounts,cn=provisioning,SUFFIX
   Destination:     cn=users,cn=accounts,SUFFIX

-  Error handling

   -  ADD fails, the source entry is preserved and the CLI reports an
      error
   -  if DEL fails, the destination entry is removed and the CLI reports
      an error. If it fails to delete destination entry, both entries
      will remain. This is not a concern as the *Stage* entry will never
      be activated as long as the destination entry exists and have the
      same uid.

-  The destination *Active* LDAP entry is a *NEW* entry compare to the
   source *Stage* entry (see `MODRDN vs
   ADD-DEL <http://www.freeipa.org/page/V3/User_Life-Cycle_Management#MODRDN_vs._ADD-DEL>`__)

   -  It contains all FreeIPA required objectclasses/attributes
      (including structural objectclasses) *(comment: TBL in
      implementation)*
   -  Unsupported objectclasses/attributes present in the *Stage* entry
      have been removed

-  The destination *Active* LDAP entry is a *NEW* entry (LDAP ADD). The
   source *Stage* entry may contain *userPassword* in an hashed way (see
   `http://www.freeipa.org/page/V4/User_Life-Cycle_Management#Staging_entry
   stored
   password <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#Staging_entry_stored_password>`__).
   To allow the storing of pre-hashed password
   `ipa-pwd-extop:ipapwd_pre_add <https://www.redhat.com/archives/freeipa-devel/2014-June/msg00505.html>`__
   must relax its control for example if the krb keys already exists in
   the entry.
-  *Stage* container is out of the scope of uid uniqueness plugin, so
   the destination entry can be added even if the source entry still
   exists
-  *Active* and *Delete* containers are in scope of uid uniqueness
   plugin, so destination entry can not be added if it already exists an
   *Active* or *Delete* entry with the same RDN ('uid') value. For
   example, it exists *jdoe* *Active* or *Delete* entry that contains
   several *uid* values. The entry *jdoe* has been created by a
   provisioning system, in fact Regular Freeipa CLI do not create user
   entry with multiple *uid*. *foo* can not be *Activate* because *jdoe*
   already have the *uid: foo* value:

::

   dn: uid=foo,cn=staged users,cn=accounts,cn=provisioning,SUFFIX
   ...
   uid: foo

   dn: uid=jdoe,cn=users,cn=accounts,cn=provisioning,SUFFIX
   ...
   uid: jdoe
   uid: foo

or if it exists a *Delete* entry that already have the ''uid: foo value:

::

   dn: uid=foo,cn=staged users,cn=accounts,cn=provisioning,SUFFIX
   ...
   uid: foo

   dn: uid=jdoe,cn=deleted users,cn=accounts,cn=provisioning,SUFFIX
   ...
   uid: jdoe
   uid: foo

-  There are
   `ajustments <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#Adjustment_of_DN_syntax_attributes>`__
   of DN syntax attributes

.. _modrdn_vs._add_del:

MODRDN vs. ADD-DEL
                  

When a staged user is moved to active users tree or an active user is
moved to deleted users tree, there are 2 possible approaches -
*renaming* (LDAP MODRDN operation with defining ``newsuperior``
attribute) and *moving* the LDAP object (LDAP ADD and DEL operations).
In the end, the result will the same for outer world, but the operation
will affect DS internals and plugin function.

#. Renaming operation is a better approach from atomicity point of view
   (second, delete opration may fail and there would be 2 duplicate
   entries), but it may interfere with both internal DS plugin
   (referential integrity, memberOf, manged entry plugin) and 3rd party
   plugins. They may tend to keep all DN links to the entry unless they
   are modified not to do so when such entry is moved to *deleted* tree.
#. Moving operation seems cleaner approach as all these plugins
   (including current plugins) will understand the operation in the
   right semantics. However, it is more difficult to control (see
   related tickets) and ability to move a user from *staging* tree to
   *active* users tree would require ability to read and write all user
   attributes, including ``userPassword``.

Provisioning system does not guarantee to create the stage entry with
the appropriate set of `structural
objectclasses <https://www.redhat.com/archives/freeipa-devel/2014-May/msg00399.html>`__
(see `Activating staged
user <https://www.redhat.com/archives/freeipa-devel/2014-May/msg00471.html>`__.
The active entry requires all the FreeIPA structural
objectclasses/attributes. It can be addressed:

-  Adding **in** the stage entry each missing objectclasses/attributes -
   Remove the unwanted objectclasses/attributes
-  Create a new entry with all the required objectclasses/attributes and
   fill it from values taken in the stage entry

A stage entry is possibly partially initialized (especially with
provisioning systems), using MODRDN means that this incomplete entry
becomes active and one can bind with it (if credential are set). So some
MODs are needed to make it a valid entry, before we can issue the
MODRDN.

The *second approach is chosen* solution because:

-  it allows to (see this thread `'Supported Stage
   entries' <https://www.redhat.com/archives/freeipa-devel/2014-May/msg00411.html>`__
   )

   -  filter objectclasses/attributes set in the *Stage* entry but the
      admin do not want to see in *Active*. When filling the new entry
      we just need to skip the no wished values/attributes.
   -  add objectclasses/attributes necessary for *Active* entries. The
      new entry contains by default all required OC/Attribute, we need
      to pick values from the *Stage* entry

-  guaranty that `structural
   objectclasses <https://www.redhat.com/archives/freeipa-devel/2014-May/msg00399.html>`__
   are present
-  It is most simplest solution to implement.

.. _adjustment_of_dn_syntax_attributes:

Adjustment of DN syntax attributes
                                  

A *Stage* or *Delete* user entry may contains DN syntax attributes.
Referential integrity plugin is not checking the validity of the DN in
those containers, so they may contain invalid values. When an *Active*
entry is deleted (user-del), most of the its DN syntax attributes are
replaced with empty values except for *manager/managedby/secretary* that
are preserved.

A *Stage* entry may contain any values in its DN syntax attributes.

When an entry becomes *Active* (userstage-activate or user-undel) the
values of DN syntax attributes need to be checked. The value is
preserved if it is the DN of an *Active* entry, else it is replaced with
an empty value. (see `this
thread <https://www.redhat.com/archives/freeipa-devel/2014-June/msg00080.html>`__)

.. _update_stageuser:

Update StageUser
''''''''''''''''

#. Support engineer uses standard FreeIPA calls to modify user
   attributes (including password)

      ``ipa stageuser-mod tuser --phone=123456789``

#. *ipa stageuser-mod* supports the same options as *user-mod*, but more
   control are done when updating a stage user.

   #. **manager** must be an *active* user
   #. **nsaccountlock** can not be set. Its value is forced to be *True*
      by a **COS**

.. _delete_stageuser:

Delete StageUser
''''''''''''''''

#. Support engineer deletes the stage user using the following CLI

      ipa stageuser-del <*user identifier*>

#. ``stageuser-del`` triggers LDAP delete operation on LDAP stage entry
   that deletes it permanently

.. _restore_stageuser:

Restore StageUser
'''''''''''''''''

``Support engineer can restore a ``\ *``Delete``*\ `` user (under ``\ *``cn=deleted``\ ````\ ``users,cn=accounts,cn=provisioning,SUFFIX``*\ ``). ``\ *``Delete``*\ `` user contains properties of a former user. If some of the properties are corrupted, it may be not acceptable to ``\ ```restore``\ ````\ ``and``\ ````\ ``activate`` <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#Restore_Deleted_User>`__\ `` a ``\ *``Delete``*\ `` user. Stepping the entry into ``\ *``Stage``*\ `` allows to keep valid properties and update the corrupted ones before making the user ``\ *``Active``*\ ``.``

#. Support engineer restore the user using the following CLI

      ipa stageuser-add --from-preserved

.. _find_stageuser:

Find StageUser
''''''''''''''

#. authenticate user can find an *Stage* account with

      ipa stageuser-find [] []

#. *ipa stageuser-find* supports the same options as *user-find* except
   the flag *--preserved=<true|false* because stageuser-find only deal
   with *Stage* accounts (not *Delete* or *Active*)

::

   ipa stageuser-find
   ---------------
   2 users matched
   ---------------
     User login: kau1
     Home directory: /home/kau1
     UID: 181818
     GID: 181818
     Password: True
     Kerberos keys available: False

     User login: xy2
     First name: x
     Last name: y
     Home directory: /home/xy2
     Login shell: /bin/sh
     Email address: xy2@domain.com
     UID: -1
     GID: -1
     Password: False
     Kerberos keys available: False
   ----------------------------
   Number of entries returned 2
   ----------------------------

.. _show_stageuser:

Show StageUser
''''''''''''''

#. authenticate user can show an *Stage* account (entry
   *uid=LOGIN,cn=staged users,cn=accounts,cn=provisioning,$SUFFIX*)

      ipa stageuser-show <*LOGIN*>

.. _user_plugin:

user plugin
^^^^^^^^^^^

.. _add_an_active_user:

Add an active user
''''''''''''''''''

-  Support Engineer has permission to add active users using the
   following FreeIPA CLI : ipa user-add <*user_identifier*>
   --first=<*first name*> --last=<*last name*>

   ::

      ipa user-add tuser  --first=test --last=user

-  provisioning systems has not the permission to add active user (they
   are only allowed to add *Stage* entries)

-  if needed, command may specify more details about the user

   ::

      ipa user-add  tuser --first=Test --last=User  --manager=muser --phone 123456789

-  if password is specfies, to allow FreeIPA generate Kerberos keys, the
   following modifications occur

   -  add *krbprincipalaux* objectclass and *krbPrincipalName* attribute
   -  When the entry will be added FreeIPA plugin generates
      *krbPrincipalKey*, *krbLastPwdChange* and *krbPasswordExpiration*
      which will allow the user to authenticate by Kerberos

-  The active user entry looks like

::

   dn: uid=test_user, cn=staged users,cn=accounts,cn=provisioning,$SUFFIX
   objectClass: top
   objectClass: person
   objectClass: organizationalperson
   objectClass: inetorgperson
   objectClass: inetuser
   objectClass: posixaccount
   objectClass: krbprincipalaux
   objectClass: krbticketpolicyaux
   objectClass: ipaobject
   objectClass: ipasshuser
   objectClass: ipaSshGroupOfPubKeys
   homeDirectory: /home/tuser
   uidNumber: 646400009
   gidNumber: 646400009
   ipaUniqueID: 3f1b5cce-e1b8-11e3-86fe-001a4a104ecd
   nsAccountLock: yes
   uid: test_user
   cn: first last
   sn: last
   givenName: first
   gecos: first last
   displayName: first last
   loginShell: /bin/sh
   mail: test_user@domain.com
   krbPrincipalName: test_user@domain.com
   initials: tu

.. _assign_group_memberships_already_available:

Assign Group Memberships (already available)
''''''''''''''''''''''''''''''''''''''''''''

#. Support engineer uses standard FreeIPA calls to assign membership

      ``ipa group-add-member testgroup --user tuser``

#. This command is only available on 'Active' entries. When a 'Stage'
   entry containing *memberOf* attribute is
   `Activate <http://www.freeipa.org/page/V3/User_Life-Cycle_Management#Activate_User>`__,
   the *memberOf* attribute is not preserved and membershift is
   recomputed .

.. _update_user_already_available:

Update User (already available)
'''''''''''''''''''''''''''''''

#. Support engineer uses standard FreeIPA calls to modify user
   attributes (including password)

      ``ipa user-mod tuser --phone=123456789``

#. Regular user uses standard FreeIPA CLI calls to modify its own entry
   (including password)

#. Standard FreeIPA CLI does not allow to modify

   :\* ipaUniqueID

   :\* objectclass

#. When updating DN syntax attributes (like *--manager* option) , if the
   entry is not an *Active* entry the modification fails.

.. _change_password_already_available:

Change Password (already available)
'''''''''''''''''''''''''''''''''''

#. Support engineer uses standard FreeIPA calls to change user password:

      ``ipa passwd tuser``

#. Regular user uses same CLI to modify the password of its own entry

.. _delete_user:

Delete User
'''''''''''

Once activated a user is fully managed by freeIPA. If the user leaves
the company, his account can be *permanently* removed (deleting the ldap
entry) or moved (modrdn) to a 'Delete' container (cn=deleted
users,cn=accounts,cn=provisioning,$SUFFIX) in order to preserve it.

   ``ipa user-del tuser [[--no-preserve][--preserve]]``

Option *--no-preserve* and *--preserve* are mutually exclusive. The
default option is *--no-preserve*. A configuration attribute could
decide what is the default option (see `global
policy <http://post-office.corp.redhat.com/archives/ipa-and-samba-team-list/2015-March/msg00488.html>`__)\ **TBD**

If the entry was preserved (moved to *Delete* container), it can be
deleted (permanently) with the same command.

Both *Active*/*Delete* containers are under *uid* attribute uniqueness.
So for a give value *val*, it exists either
*uid=val,cn=accounts,$SUFFIX* or *uid=val,cn=deleted
users,cn=accounts,cn=provisioning,$SUFFIX*. So when the Support engineer
decides to delete a user, he does not need to specify if the user is in
*Delete* or *Active* state.

#. delete (Permanently) of an active entry

   #. This is done by a direct ldap delete of the entry.
   #. all references to the entry are
      `removed <https://www.redhat.com/archives/freeipa-devel/2014-June/msg00083.html>`__.
      That is done by `referential
      integrity <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#Referential_integrity>`__
      and
      `memberof <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#MemberOf_plugin>`__
      that scope *Active* containers.

#. delete (preserve) of an active entry

   #. removes `credential
      attributes <https://www.redhat.com/archives/freeipa-devel/2014-June/msg00427.html>`__
      ``userPassword`` and kerberors keys (krbPrincipalKey,
      krbLastPwdChange and krbPasswordExpiration) attributes to prevent
      any chance of using that entry to bind to LDAP server
   #. save *manager/managedby/secretary* that may get cleared when the
      entry will be moved to Delete container
   #. LDAP MODRDN operation on LDAP entry to move the entry (discussed
      `1 <https://www.redhat.com/archives/freeipa-devel/2014-June/msg00080.html>`__
      and
      `2 <http://post-office.corp.redhat.com/archives/ipa-and-samba-team-list/2015-March/msg00492.html>`__
      and
      `3 <http://post-office.corp.redhat.com/archives/ipa-and-samba-team-list/2015-March/msg00546.html>`__)

         Source: cn=users,cn=accounts,SUFFIX
         Destination: cn=deleted
         users,cn=accounts,cn=provisioning,SUFFIX

      #. all references to the entry must be
         `removed <https://www.redhat.com/archives/freeipa-devel/2014-June/msg00083.html>`__.
         That is done by `referential
         integrity <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#Referential_integrity>`__
         and
         `memberof <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#MemberOf_plugin>`__
         that only scope *Active* containers.

   #. Replace others existing DN syntax attributes with an *Empty*

      #. DN syntax attributes contains reference to entries. When the
         user entry is in *Delete* container, it is no longer in the
         scope of integrity plugin and then may contain invalid values
         (Note: is it usefull ? attribute covered by RI have been
         cleared on MODRDN, attribute not covered may be checked during
         activation rather than clearing them)
      #. removing the attribute may not be possible if it is required by
         the schema. So the attribute will be kept with an
         `Empty <https://www.redhat.com/archives/freeipa-devel/2014-June/msg00100.html>`__
         value.

   #. restore *manager/managedby/secretary* with the original values

      #. if later the entry is
         `activated <http://www.freeipa.org/page/V3/User_Life-Cycle_Management#Activate_StageUser>`__
         or
         `restored <http://www.freeipa.org/page/V3/User_Life-Cycle_Management#Restore_Deleted_User>`__,
         the value of those attributes will be checked.

   #. Some attributes are preserved, so that a corrupted active entry
      can get deleted/staged/activate and keep its previous settings

      #. ipaUniqueID
      #. uidNumber
      #. gidNumber
      #. `passwordHistory <https://www.redhat.com/archives/freeipa-devel/2014-June/msg00501.html>`__

#. delete (Permanently) of a *Delete* entry

   #. This is done by a direct ldap delete of the entry.

.. _restore_deleted_user:

Restore Deleted User
''''''''''''''''''''

Once activated a user is fully managed by freeIPA. If the user leaves
the company, his account is moved to a 'Delete' Status and is hold in a
separated container (cn=deleted
users,cn=accounts,cn=provisioning,$SUFFIX). Both Active/Delete
containers are under uid attribute uniqueness. So for a give value val,
if the entry was delelte (uid=val,cn=deleted
users,cn=accounts,cn=provisioning,$SUFFIX) it does not exists
uid=val,cn=accounts,$SUFFIX.

Once deleted a user entry still contains some properties that are
specific to the former user. The user account may becomes *Active*
again,

#. Support engineer call the following command

      ``ipa user-undel tuser``

#. user-undel triggers LDAP MODRDN operation on LDAP entry to move the
   entry (appropriate
   `aci <https://fedorahosted.org/389/ticket/47553>`__ allows only
   support engineer to move the entry)

      Source: cn=deleted users,cn=accounts,cn=provisioning,SUFFIX
      Destination: cn=users,cn=accounts,SUFFIX
      DS plugin will recompute the memberbship attributes and will add
      *nsAccountLock: True*

#. There is
   `ajustments <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#Adjustment_of_DN_syntax_attributes>`__
   of DN syntax attributes

.. _lock_user_already_available:

Lock User (already available)
'''''''''''''''''''''''''''''

#. Support engineer uses standard FreeIPA call:

      ``ipa user-disable tuser``

#. ``nsAccountLock`` operation attribute is put to ``TRUE``, user cannot
   bind to LDAP. Membership and password attributes are preserved
#. Only *Active* account can be locked/disabled

.. _unlock_user_already_available:

Unlock User (already available)
'''''''''''''''''''''''''''''''

#. Support engineer uses standard FreeIPA call:

      ``ipa user-enable tuser``

#. ``nsAccountLock`` operation attribute is put to ``FALSE``, user
   operation is restored
#. Only *Active* account can be unlocked/enabled

.. _find_user:

Find User
'''''''''

#. authenticate user can find an *Active* account (*Active* container
   *cn=users,cn=accounts,$SUFFIX*) with

      ipa user-find [] []

#. a new flag *--preserved=true* is used to find the entries from the
   *Delete* container (*cn=deleted
   users,cn=accounts,cn=provisioning,$SUFFIX*)
#. by default or if the flag is *--preserved=false* it only lookup into
   the *Active* container

In addition if the retrieved entry is *preserved* it displays a flag:
**Preserved user: True**

::

   prompt> ipa user-find --preserved=true
   --------------
   1 user matched
   --------------
     User login: xy2
     First name: x
     Last name: y
     Home directory: /home/xy2
     Login shell: /bin/sh
     Email address: xy2@domain.com
     UID: 1337000003
     GID: 1337000003
     Account disabled: True
     Preserved user: True
     Password: False
     Kerberos keys available: False
   ----------------------------
   Number of entries returned 1
   ----------------------------

.. _show_user:

Show User
'''''''''

#. authenticate user can show an *Active* account (entry
   *uid=LOGIN,cn=users,cn=accounts,$SUFFIX*)

      ipa user-show <*LOGIN*> []

#. This command can retrieve *Active* or *Deleted* account. If the
   returned account is *delete* account, it displays a flag: **Preserved
   user: True**

::

   prompt> ipa user-show xy2
     User login: xy2
     First name: x
     Last name: y
     Home directory: /home/xy2
     Login shell: /bin/sh
     Email address: xy2@domain.com
     UID: 1337000003
     GID: 1337000003
     Account disabled: True
     Preserved user: True
     Password: False
     Kerberos keys available: False

Placeholders
~~~~~~~~~~~~

When an entry is created using FreeIPA CLI
`user-add <http://www.freeipa.org/page/V3/User_Life-Cycle_Management#user-add>`__,
the value of the required attributes need to be specify somewhere.
Usually it is provided using the CLI options or if absent CLI uses some
default values.

A provisioning system when creating an entry needs to provide all
required attribute or to modify the CLI to adapt the default values.
That is not really flexible.

In addition with User Life Cycle, an entry will follow a workflow that
means that some require attribute value may be unknown at some point of
the flow and defined later. If a new plugin is developed to generate the
value, it requires to adapt CLI options/default value.

Instead of that we will use placeholders. A placeholder is added to the
entry and contains an initial value (waiting for its final value to be
set). A placeholder is couple attribute/value that defines the default
value of a given attribute. The placeholders definitions will be stored
under the FreeiPa configuration under the following entries:

::

   # placeholders for ADD entries
   dn: cn=placeholders,cn=ipaConfig,cn=etc,$SUFFIX
   cn: placeholders
   objectClass: top
   objectClass: extensibleObject

   # placeholders for ADDed entries in staging
   dn: cn=stage,cn=placeholders,cn=ipaConfig,cn=etc,$SUFFIX
   cn: stage
   objectClass: top
   objectClass: extensibleObject
   <attrname>: <value>
   ...

   #placeholders for ADDed entries in active
   dn: cn=active,cn=placeholders,cn=ipaConfig,cn=etc,$SUFFIX
   cn: active
   objectClass: top
   objectClass: extensibleObject
   <attrname>: <value>
   ...

Syntax
^^^^^^

attribute name is : ALPHA \*(ALPHA / DIGIT / HYPHEN)

The value of the placeholder may be used in different attribute types
with different syntaxes, they should follow the most restrictive syntax.
In case of LDAP, it should follow *Printable String* syntax (also see
RFC 4517):

::

         PrintableCharacter = ALPHA / DIGIT / SQUOTE / LPAREN / RPAREN /
                                         PLUS / COMMA / HYPHEN / DOT / EQUALS /
                                         SLASH / COLON / QUESTION / SPACE
         PrintableString    = 1*PrintableCharacter 

Mechanism
^^^^^^^^^

When creating an entry, the CLI picks up all definitions found in the
placeholders (of the container) and add them to the entry. There is no
modification of the value from what is in the placeholder definition.

An exception is if the placeholder value starts and finishes with '
**?** ', then CLI strip ' **?** ' from the value before adding it (i.e.
placeholder value is '?autogenerate?' -> added value is 'autogenerate').

Priority
^^^^^^^^

When an entry is created the attribute value is taken in the following
order:

-  CLI option
-  placeholder
-  CLI default value

That means that if the attribute value is defined as a CLI option it
selects it and the possible values in placeholder or default value are
ignored. Else if the placeholder attribute exists in the placeholders
entry it selects the placeholder value and the possible default value is
ignored. Else it selects the default values.

.. _limitation___future_enhancement:

Limitation - Future enhancement
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A placeholder allows an entry to conform the schema. In that purpose it
is sufficient to define a single value for a required attribute. If a
placeholder defines multiple values for an attribute, there is no
guaranty that all the values will be added in the entry.

As futur enhancement, we can imagine a placeholder being defined with:
*homeDirectory: /home/net/%{uid}*. In that case for user 'uid=tuser',
the added value for **homeDirectory** will be **/home/net/tuser**

.. _staging_container:

Staging container
~~~~~~~~~~~~~~~~~

.. _staging_tree:

Staging tree
^^^^^^^^^^^^

Core part of the feature is to allow provisioning system to add or
delete users with standard LDAP protocol without a need to understand or
use FreeIPA API. Standard tree of active FreeIPA users
(``cn=users,cn=accounts,$SUFFIX``) is not used to avoid collisions or
misinterpretation of active and staged users by other IdM systems.

Staging tree should have the following structure:

-  ``SUFFIX``

   -  ``cn=provisioning``

      -  ``cn=accounts``

         -  ``cn=staged users``: new staged users

.. _staging_users_in_a_special_database:

Staging Users in a Special Database
'''''''''''''''''''''''''''''''''''

*Stage* user container is separated from the *Active* container. A
question is to store the containers in separated database or keep them
in the same database protected by special ACIs. Both approaches have
pros and cons.

Pros
    

-  Is naturally separated from standard FreeIPA objects, no need to
   re-configure existing plugins (like attribute uniqueness or memberOf
   plugin) to ignore users in the staging area
-  Easier management in heterogeneous environment when some FreeIPA
   replicas have the feature and and some does not

Cons
    

-  Increased maintenance burden with multiple LDAP databases
-  Increased maintenance burden with replication agreements of the new
   database

.. _staging_users_in_a_normal_suffix_preferred:

Staging Users in a Normal Suffix (preferred)
''''''''''''''''''''''''''''''''''''''''''''

.. _pros_1:

Pros
    

-  IdM systems are kept idempotent - command to activate a user or
   moving it to deleted users tree can be done against any FreeIPA
   server.
-  No need to manage new databases or replication agreements

.. _cons_1:

Cons
    

-  Staging or deleted users container may interfere with normal users in
   the ``cn=users,cn=accounts,$SUFFIX`` in older FreeIPA replicas. Even
   though the tree is not visible by standard users due to ACIs, it is
   still visible by plugins.
-  during upgrade from a old instance, the Staging/deleted containers
   will be created if they do not already exists. The use of COS to
   disable potential already existing stage/deleted entry will prevent
   to authenticate with them.
-  Running in a topology with different versions, will require that the
   ACI, containers are replicated

.. _stage_placeholders:

Stage placeholders
^^^^^^^^^^^^^^^^^^

The placeholders for the staging container are:

::

   # placeholders for ADD entries
   dn: cn=placeholders,cn=ipaConfig,cn=etc,$SUFFIX
   cn: placeholders
   objectClass: top
   objectClass: extensibleObject

   # placeholders for ADDed entries in staging
   dn: cn=stage,cn=placeholders,cn=ipaConfig,cn=etc,$SUFFIX
   cn: stage
   objectClass: top
   objectClass: extensibleObject
   uidNumber: -1
   gidNumber: -1
   ipaUniqueId: ?autogenerate?
   nsAccountLock: yes

.. _stage_entry_requirements:

Stage entry requirements
^^^^^^^^^^^^^^^^^^^^^^^^

This container can contain entries coming from

-  external provisioning systems
-  stageuser CLI (stageuser-add)
-  *Delete* entries

The common requirements from these origins are

-  entry must conform the schema
-  Entry RDN attribute is ' uid ' (see `Supported Staged
   entries <https://www.redhat.com/archives/freeipa-devel/2014-May/msg00407.html>`__)
-  entry is disabled (i.e. contains operational attribute
   'nsAccountLock: True')
-  *ipaUniqueID* is set to *autogenerate*. This requirement is not
   enforced for provisioning systems (but for stageuser CLI) but if an
   entry have a different value its value will be reset during
   activation of the *Stage* entry (stageuser-activate). (see
   `ipaUniqueID
   reset <https://www.redhat.com/archives/freeipa-devel/2014-June/msg00344.html>`__)

.. _example_of_stage_entry_provisioning_origin:

Example of Stage entry (provisioning origin)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In addition to the common requirements above, the entry may contains any
objectclasses/attributes

.. _example_of_stage_entry_stageuser_add:

Example of Stage entry (stageuser-add)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A *staged* entry created with FreeIPA CLI is looking like the following:

::

   dn: uid=test_user, cn=staged users,cn=accounts,cn=provisioning,$SUFFIX
   objectClass: top
   objectClass: person
   objectClass: organizationalperson
   objectClass: inetorgperson
   objectClass: inetuser
   objectClass: posixaccount
   objectClass: krbprincipalaux
   objectClass: krbticketpolicyaux
   objectClass: ipaobject
   objectClass: ipasshuser
   objectClass: ipaSshGroupOfPubKeys
   homeDirectory: /home/tuser
   uidNumber: -1
   gidNumber: -1
   ipaUniqueID: autogenerate
   nsAccountLock: yes
   uid: test_user
   cn: first last
   sn: last
   givenName: first
   gecos: first last
   displayName: first last
   loginShell: /bin/sh
   mail: test_user@domain.com
   krbPrincipalName: test_user@domain.com
   initials: tu

In case CLI option/placeholder do not define them, the entries in
staging will contains those default values:

-  ``uid``: generated from first letter of ``cn`` and ``sn``
-  ``givenName``: generated from ``cn`` by removing the last part of the
   name
-  ``displayName``: copied from ``cn``
-  ``initials``: first letter of ``cn``, first letter of ``sn``
-  ``homeDirectory``: default value defined in FreeIPA configuration
-  ``gecos``: copied from ``cn``
-  ``loginShell``: default value defined in FreeIPA configuration
-  ``mail``: generated from uid and default domain defined in FreeIPA
   configuration
-  ``krbPrincipalName``: generate from uid and kerberos realm defined in
   FreeIPA configuration

The way those values are generated is hardcoded in the CLI core.

.. _example_of_stage_entry_stageuser_undel:

Example of Stage entry (stageuser-undel)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A *staged* entry that is created from a former *Delete* entry mainly
differs from an entry created by *stageuser-add* because the following
attributes:

-  uidNumber (with value different from **-1**)
-  gidNumber (with value different from **-1**)
-  ipaUniqueID (with value different from **autogenerate**)

::

   dn: uid=test_user, cn=staged users,cn=accounts,cn=provisioning,$SUFFIX
   objectClass: top
   objectClass: person
   objectClass: organizationalperson
   objectClass: inetorgperson
   objectClass: inetuser
   objectClass: posixaccount
   objectClass: krbprincipalaux
   objectClass: krbticketpolicyaux
   objectClass: ipaobject
   objectClass: ipasshuser
   objectClass: ipaSshGroupOfPubKeys
   homeDirectory: /home/tuser
   uidNumber: 646400009
   gidNumber: 646400009
   ipaUniqueID: 3f1b5cce-e1b8-11e3-86fe-001a4a104ecd
   nsAccountLock: yes
   uid: test_user
   cn: first last
   sn: last
   givenName: first
   gecos: first last
   displayName: first last
   loginShell: /bin/sh
   mail: test_user@domain.com
   krbPrincipalName: test_user@domain.com
   initials: tu

.. _active_container:

Active container
~~~~~~~~~~~~~~~~

.. _active_tree:

Active tree
^^^^^^^^^^^

Active entries are under *cn=users,cn=accounts,SUFFIX*

.. _active_placeholders:

Active Placeholders
^^^^^^^^^^^^^^^^^^^

The placeholders for the *Active* container are:

::

   # placeholders for ADD entries
   dn: cn=placeholders,cn=ipaConfig,cn=etc,$SUFFIX
   cn: placeholders
   objectClass: top
   objectClass: extensibleObject

   # placeholders for ADDed active entries
   dn: cn=active,cn=placeholders,cn=ipaConfig,cn=etc,$SUFFIX
   cn: active
   objectClass: top
   objectClass: extensibleObject
   uidNumber: -1
   gidNumber: -1
   ipaUniqueId: ?autogenerate?

-  ``uidNumber``: value generated by FreeIPA DNA plugin, based on the
   defined UID range of the server
-  ``gidNumber``: copied from uidNumber
-  ``ipaUniqueId``\ (final value generated by FreeIPA IPA UUID plugin)

.. _active_entry_requirements:

Active entry requirements
^^^^^^^^^^^^^^^^^^^^^^^^^

The requirements for active entries are

-  entry must conform the schema
-  entry RDN is uid
-  entry is enabled (*nsAccountLock: False* or absent)
-  entry is a *objectclass: posixAccount* and contains the following
   required attributes

   -  uidNumber (final value generated by FreeIPA DNA plugin)
   -  gidNumber (final value generated by FreeIPA DNA plugin)
   -  ipaUniqueId (final value generated by FreeIPA IPA UUID plugin)
   -  homeDirectory (generated from default FeeIPA configuration
      cn=ipaConfig,cn=etc,$SUFFIX)
   -  uid/cn (generated from the CLI)

.. _example_of_active_entry_user_add:

Example of Active entry (user-add)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The freeipa CLI creates an entry like:

::

   dn: uid=test_user, cn=users,cn=accounts,SUFFIX
   objectClass: top
   objectClass: person
   objectClass: organizationalperson
   objectClass: inetorgperson
   objectClass: inetuser
   objectClass: posixaccount
   objectClass: krbprincipalaux
   objectClass: krbticketpolicyaux
   objectClass: ipaobject
   objectClass: ipasshuser
   objectClass: ipaSshGroupOfPubKeys
   homeDirectory: /home/tuser
   uidNumber: 646400009
   gidNumber: 646400009
   ipaUniqueID: 3f1b5cce-e1b8-11e3-86fe-001a4a104ecd
   nsAccountLock: yes
   uid: test_user
   cn: first last
   sn: last
   givenName: first
   gecos: first last
   displayName: first last
   loginShell: /bin/sh
   mail: test_user@domain.com
   krbPrincipalName: test_user@domain.com
   initials: tu
   memberOf: cn=ipausers,cn=groups,cn=accounts,SUFFIX

.. _delete_container:

Delete container
~~~~~~~~~~~~~~~~

.. _delete_tree:

Delete tree
^^^^^^^^^^^

Delete entries are under *cn=deleted users,cn=accounts,cn=provisioning,
SUFFIX*

.. _delete_placeholders:

Delete placeholders
^^^^^^^^^^^^^^^^^^^

N/A No entry are added in the *Delete* container

.. _delete_entry_requirements:

Delete entry requirements
^^^^^^^^^^^^^^^^^^^^^^^^^

A *Delete* entry was create with
`user-del <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#Delete_User>`__
and conform the following requirements

-  entry must conform the schema
-  entry RDN is uid
-  entry is disabled (nsAccountLock: True)
-  entry is a objectclass: posixAccount and contains the at least the
   following required attributes

   -  uidNumber (value different from **-1**)
   -  gidNumber (value different from **-1**)
   -  ipaUniqueId (value different from **autogenerate**)
   -  homeDirectory
   -  uid
   -  cn

-  entry has no 'memberof' attribute

.. _example_of_delete_entry:

Example of Delete entry
^^^^^^^^^^^^^^^^^^^^^^^

A entry in Delete container is looking like

::

   dn: uid=test_user, cn=deleted users,cn=accounts,cn=provisioning,SUFFIX
   objectClass: top
   objectClass: person
   objectClass: organizationalperson
   objectClass: inetorgperson
   objectClass: inetuser
   objectClass: posixaccount
   objectClass: krbprincipalaux
   objectClass: krbticketpolicyaux
   objectClass: ipaobject
   objectClass: ipasshuser
   objectClass: ipaSshGroupOfPubKeys
   homeDirectory: /home/tuser
   uidNumber: 646400009
   gidNumber: 646400009
   ipaUniqueID: 3f1b5cce-e1b8-11e3-86fe-001a4a104ecd
   nsAccountLock: yes
   uid: test_user
   cn: first last
   sn: last
   givenName: first
   gecos: first last
   displayName: first last
   loginShell: /bin/sh
   mail: test_user@domain.com
   krbPrincipalName: test_user@domain.com
   initials: tu
   nsAccountLock: True

Authentication
~~~~~~~~~~~~~~

``Authentication is not allowed with ``\ *``Stage``*\ `` and ``\ *``Delete``*\ `` entries. Authentication can be done with simple bind or through an external mechanism (like GSSAPI).``

.. _staging_entry:

Staging entry
^^^^^^^^^^^^^

A first idea to prevent this authentication was to prevent credentials
(password / krb keys) creation in both stageuser-add/stageuser-mod. But
this is not enough because provisioning system can create entries with
`userPassword <https://www.redhat.com/archives/freeipa-devel/2014-June/msg00440.html>`__
and then allow simple bind. For kerberos authentication, it requires
kerberos keys that are generated from a plain text password.
*userPassword* being stored in an hashed way, we can not generate
kerberos keys from a stored password. kerberos keys need to be generated
when the *userPassword* is set (see
`https://www.redhat.com/archives/freeipa-devel/2014-June/msg00462.html
credential <https://www.redhat.com/archives/freeipa-devel/2014-June/msg00462.html_credential>`__).
So if a stage entry receives a *userPassword* then kerberos keys need to
be generated as well. That means:

-  stageuser-add / staguser-mod must support *--password* option
-  *ipa-pwd-extop* DS plugin must scope *Stage* container.

To prevent authentication from *Stage* entry, we can use two methods:

-  using a COS (likely a pointer COS), that *overwrite* the
   *nsAccountLock operational attribute.*\ nsAccountLock\ *is added to
   an*\ Active\ *entry when we need to disable (user-disable) it. It
   prevents simple bind as well as kerberos authentication. It is quite
   easy to implement (likely in the*.updates'' files). A drawback is to
   use a DS opertional attribute to do this.
-  using a *pre-op* DS plugin, that would reject *bind* (simple or SASL)
   on *Stage* and *Delete* containers. It is a bit more complex solution
   and requires a new deliverable.

The *COS* solution will be use because it seems good enough without
major drawback

.. _active_entry:

Active entry
^^^^^^^^^^^^

Authentication is allowed with an *Active* entry (as long as it owns
credential).

If this is a newly *activated* (*stageuser-activate*) entry it owns
credentials (userpassword/krb) at the condition *userPassword* was set
in the staging area. To *activate* such entry it requires a
`change <https://www.redhat.com/archives/freeipa-devel/2014-June/msg00505.html>`__
in ipa-pwd-extop to relax the setting of pre-hashed password.

If this is a newly *undelete* ("user-undelete") entry, credentials have
been removed. So it is not possible to authenticate with it before the
*userPassword* (user-mod --password) is set (and krb keys generated).

If this is a *True* new user (user-add), it is possible to authenticate
with it at the condition it has *userPassword* and kerberos keys

.. _delete_entry:

Delete entry
^^^^^^^^^^^^

Authentication is not allowed with *Delete* entry. *Delete* entries are
kept for legal or regulation reasons but they may become *Active*
(user-undelete) or *Stage* (stageuser-add) again. In that case we do not
want to preserve any credential because they are likely too old and we
want the user to assign again new credential.

So when deleting (user-delete) an *Active* entry, kerberos keys must be
cleared. Password attribute must be cleared as well at the exception of
passwordHistory. *passwordHistory* will prevent the user to reuse one of
password it used when the entry was *Active*.

.. _permissions_and_acis:

Permissions and ACIs
~~~~~~~~~~~~~~~~~~~~

New permissions should be created:

-  *Add Staged Users*
-  *Delete Staged Users*
-  *Modify Staged Users*
-  *Read Staged Users*
-  *Discard Deleted Users*
-  *Modify Deleted Users*
-  *Read Deleted Users*

*Add Deleted Users* permission is needed as this operation would be done
by new DS plugin.

New privilege should be created:

-  *Staged User Administrators*: should contain all permissions listed
   above
-  *Staged User Provisioning*: should contain *Add Staged Users*
   permission

Following roles should be updated:

-  *User Administrator*: should contain *Staged User Administrators*
   privilege

.. _allow_moving_staged_users_only:

Allow Moving Staged Users Only
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We cannot distinguish a situation when a new user is added and when a
user is moved (copied) from *staged* or *deleted* tree. Both require
*add* access control on
``cn=staged users, cn=accounts,cn=provisioning,SUFFIX`` tree. It is
therefore difficult to allow *helpdesk* people only moving an entry from
*staged* tree and not allowing creating a new random user.

The stageuser-activate command selects a stage user and "move" it to the
active container. The stage user attributes/values may be not
appropriated to because a true operational active user. The command
prepares a new user entry, taking the existing values from the stage
user and the check them, fills the required (for active user) values
that are missing from the stage entry. So the "helpdesk" people will not
move entry but always create new user from one the example stored in the
staging container.

User life cycle do not "move" entries (LDAP moddn) but create valid user
and delete stage user/preserved user.

.. _moving_users_from_staging_tree_automatically:

Moving Users from Staging Tree Automatically
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By default, new staged users are moved to active user area manually, by
running a specified ``user-add`` command. However, some deployments may
want to active users automatically. This operation should be done
periodically using a custom script run on one chosen server (to prevent
race unexpected conditions when more servers are activating staged
users).

.. _proposed_script_workflow:

Proposed Script Workflow
''''''''''''''''''''''''

-  *kinit* as a special user (with keytab)
-  With ``user-find``, search for all staged users
-  For each new staged user:

   -  Activate user with ``user-add``, log result

.. _affected_directory_server_plugins:

Affected Directory Server Plugins
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Active FreeIPA plugins that are active in any user-related operation
need to be checked or updated, to avoid interference with users in
*staged* or *deleted* tree, namely:

.. _dna_plugin:

DNA plugin
^^^^^^^^^^

This plugin is reponsible to update *uidNumber* and *gidNumber*
attribute. If those attributes have a predefined value (*dnaMagicRegen*:
-1), then the plugin generate a new value and replace *-1* value with
it. This plugin should not update values of *Stage* and *Delete*
entries. so DNA plugin should exclude *cn=provisioning,SUFFIX* (relies
on `47828 <https://fedorahosted.org/389/ticket/47828>`__) from its scope
set to:

::

   dn: cn=Posix IDs,cn=Distributed Numeric Assignment Plugin,cn=plugins,cn=config
   ...
   dnaScope: $SUFFIX
   dnaExcludeScope: cn=provisioning,$SUFFIX
   dnaMagicRegen: -1
   dnaFilter: (|(objectClass=posixAccount)(objectClass=posixGroup)(objectClass=ipaIDobject))

The following possibilities were evaluated but did not work:

-  *dnaScope: cn=accounts,$SUFFIX*. This is not possible because freeipa
   *trust* requires *cn=trusts,SUFFIX* (see `trusted
   domains <https://www.redhat.com/archives/freeipa-devel/2014-August/msg00186.html>`__)
-  *dnaFilter:
   (&(|(objectClass=posixAccount)(objectClass=posixGroup)(objectClass=ipaIDobject))(!(entrydn=*cn=provisioning*)))*.
   This is not possible because *entrydn* is not present in the added
   entry when the DNA preop-plugin is called (see `filter
   entrydn <https://www.redhat.com/archives/freeipa-devel/2014-August/msg00198.html>`__)
-  *trusts domain* is sharing the same DS config entry for *cn=Posix
   IDs,cn=Distributed Numeric Assignment Plugin,cn=plugins,cn=config*
   (because it needs the same IPA range). So having a different DNA
   config entry (i.e. "cn=Trust IDs,cn=Distributed Numeric Assignment
   Plugin,cn=plugins,cn=config'' is not an option)

.. _krbprincipalname_uniqueness:

krbPrincipalName uniqueness
^^^^^^^^^^^^^^^^^^^^^^^^^^^

This plugin is reponsible to enforce the uniqueness of an
*krbPrincipalName* value in Active and Delete containers. So that a
given value of *krbPrincipalName* is unique in both Delete and Active
containers *together*

This is important the *krbPrincipalName* remains unique as for
regulation, law or policy we need to be able to uniquely identify a user
(Active or not) with his *krbPrincipalName*

*Active* and *Delete* containers are under separated subtrees (see
`attribute
uniqueness <http://post-office.corp.redhat.com/archives/ldap-devel-list/2014-June/msg00158.html>`__).
In order to check simultaneous (requires
`47823 <https://fedorahosted.org/389/ticket/47823>`__) both container
the plugin configuration contains:

::

   dn: cn=krbPrincipalName uniqueness,cn=plugins,cn=config
   ...
   nsslapd-pluginAllSubtrees: on
   nsslapd-pluginAttributeName: krbPrincipalName
   nsslapd-pluginContainerScope: cn=accounts,SUFFIX
   nsslapd-pluginContainerScope: cn=deleted users,cn=accounts,cn=provisioning,SUFFIX
   uniqueness-across-all-subtrees: on

.. _krbcanonicalname_uniqueness:

krbCanonicalName uniqueness
^^^^^^^^^^^^^^^^^^^^^^^^^^^

For the same reasons as
`krbPrincipalName <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#krbPrincipalName_uniqueness>`__,
we apply the following configuration

::

   dn: cn=krbCanonicalName uniqueness,cn=plugins,cn=config
   ...
   nsslapd-pluginAllSubtrees: on
   nsslapd-pluginAttributeName: krbCanonicalName
   nsslapd-pluginContainerScope: cn=accounts,SUFFIX
   nsslapd-pluginContainerScope: cn=deleted users,cn=accounts,cn=provisioning,SUFFIX
   uniqueness-across-all-subtrees: on

.. _uid_uniqueness:

uid uniqueness
^^^^^^^^^^^^^^

For the same reasons as
`krbPrincipalName <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#krbPrincipalName_uniqueness>`__,
we apply the following configuration

::

   nsslapd-pluginAllSubtrees: on
   nsslapd-pluginAttributeName: uid
   nsslapd-pluginContainerScope: cn=accounts,SUFFIX
   nsslapd-pluginContainerScope: cn=deleted users,cn=accounts,cn=provisioning,SUFFIX
   uniqueness-across-all-subtrees: on

.. _ipauniqueid_uniquness:

ipaUniqueID uniquness
^^^^^^^^^^^^^^^^^^^^^

For the same reasons as
`krbPrincipalName <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#krbPrincipalName_uniqueness>`__,
we apply the following configuration

::

   dn: cn=ipaUniqueID uniqueness,cn=plugins,cn=config
   ...
   nsslapd-pluginAllSubtrees: on
   nsslapd-pluginAttributeName: ipaUniqueID
   nsslapd-pluginContainerScope: cn=accounts,SUFFIX
   nsslapd-pluginContainerScope: cn=deleted users,cn=accounts,cn=provisioning,SUFFIX
   uniqueness-across-all-subtrees: on

*Stage* container is not part of the scope because first provisioning
system can set any values and stageuser-add CLI creates entries with the
same value *autogenerate*

.. _referential_integrity:

Referential integrity
^^^^^^^^^^^^^^^^^^^^^

This plugin scopes *Active account* (nsslapd-pluginContainerScope and
nsslapd-pluginEntryScope). To correctly update the reference in entries
it requires that an *Active account* (**A**) refers only *Active
accounts* (**B**) (This is a requirement enforced during
stageuser-active, user-undel, user-mod, user-add). If it follow that
requirements then:

-  entry *B* is deleted (LDAP DEL) => entry *A* is updated removing *B*
   DN
-  entry *B* is moved (MODRDN) in *Active* container => entry *A* is
   updated with the new DN
-  entry *B* is moved (MODRDN) out of *Active* container (user-del) =>
   entry *A* is updated removing *B* DN

If an *Active account* does not follow this requirement, *Active
account* **A** refers an external entry **B** ( in *Stage* or *Delete*
or elsewhere), then the entry **A** may get corrupted

-  entry *B* is deleted (LDAP DEL) => entry *A* is **NOT** modified
-  entry *B* is moved (MODRDN) in *Active* container => entry *A* is
   updated with the new *B* DN
-  entry *B* is moved (MODRDN) out of *Active* container => entry *A* is
   **NOT** modified

The configuration of this plugin is :

::

   dn: cn=referential integrity postoperation,cn=plugins,cn=config
   ...
   nsslapd-pluginContainerScope: cn=accounts,SUFFIX
   nsslapd-pluginEntryScope: cn=accounts,SUFFIX

.. _memberof_plugin:

MemberOf plugin
^^^^^^^^^^^^^^^

MemberOf plugins excludes *Active* and *Delete* containers that are both
under *cn=provisioning,SUFFX* (config attribute
memberofentryscopeexcludesubtree). Like described in `referential
integrity <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#Referential_integrity>`__
if an *Active account* is member of a group and the account is deleted

-  RI remove the (*member*) account from the group
-  MemberOf plugin removes *memberof* from the account for the group

the configuration of this plugin is:

::

   dn: cn=MemberOf Plugin,cn=plugins,cn=config
   ..
   memberofentryscope: SUFFIX
   memberofentryscopeexcludesubtree: cn=provisioning,SUFFIX

.. _managed_entries_plugin:

Managed Entries plugin
^^^^^^^^^^^^^^^^^^^^^^

When a new *Active* entry is created (ADD or MODRDN), it creates a
managed entry and adds *mepManagedEntry* to the entry. When an *Active*
entry is deleted (MODRDN or DEL), the managed entry is removed and
*mepManagedEntry*

So configuration of that

::

   dn: cn=NGP Definition,cn=Definitions,cn=Managed Entries,cn=etc,SUFFIX
   ...
   originscope: cn=hostgroups,cn=accounts,SUFFIX

   dn: cn=UPG Definition,cn=Definitions,cn=Managed Entries,cn=etc,SUFFIX
    ...
   originscope: cn=users,cn=accounts,SUFFIX

When a *stage* entry is created/deleted it is not in under
*originscope*, so no managed entry is added/delete. This is the expected
behaviour. When a *stage* entry is activated, it is deleted from the
stage container and added into the *Active* container. When it is added
to the *Active* container, it falls under the scope of mep plugin and
its managed entry is created. Here again this is the expected behavior.

When an *Active* entry is deleted (permanently (del) or move (modrdn)
from the *Active* container), its associated managed entry is deleted.
This is the expected behavior.

In conclusion, the current configuration of the mep plugin does not need
to be change because of user life cycle.

.. _ipa_uuid_plugin:

IPA UUID plugin
^^^^^^^^^^^^^^^

This plugin excludes *Stage* and *Delete* entries (ipaUuidExcludeSubtee)
from generation of *ipaUniqueID* (if the current value is *autogenerate*
(*ipauuidmagicregen*)). *Delete* entries (user-del) will keep the value
of *ipaUniqueID*.

The configuration of that plugin is

::

   dn: cn=IPA Unique IDs,cn=IPA UUID,cn=plugins,cn=config
   ...
   ipaUuidExcludeSubtree: cn=provisioning,SUFFIX

.. _schema_compatibility_plugin:

Schema Compatibility plugin
^^^^^^^^^^^^^^^^^^^^^^^^^^^

should not be affected as it is already focused on
``cn=users, cn=accounts, SUFFIX`` tree only **TBC**

.. _ipa_modrdn_plugin:

IPA MODRDN plugin
^^^^^^^^^^^^^^^^^

This plugin make sure that a targetattribute (*krbPrincipaLName*) is
composed of a sourceattribute (RDN) (*uid*) and a given suffix. For
example *uid=foo,cn=accounts,SUFFIX* will have *krbPrincipalName: foo@*.

Scoping the plugin to the full SUFFIX mean that *Stage* and *Delete*
entries will be updated if the *sourceattribute* is modified. It is not
strictly necessary, but it makes sense that any provisioning entry (even
*Stage*) conforms the same rules as *Active*.

So this plugin scope will not be restricted to *Active* entries but to
the full SUFFIX.

.. _ipa_kdb:

ipa-kdb
^^^^^^^

*Stage* and *Delete* entries should not contain valid kerberos keys. Now
to be sure *Stage* or *Delete* are not used for kerberos authentication,
ipa-kdb should ignore users in *staged* or *deleted*

.. _ipa_pwd_extop:

ipa-pwd-extop
^^^^^^^^^^^^^

This plugin should scope *Stage* and *Active* containers. In fact both
*Stage* and *Active* container may receive *userpassword* (see
`authenticate <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#Authentication>`__)
This plugin should allow the storing of pre-hashed password
ipa-pwd-extop:ipapwd_pre_add must relax its control for example if the
krb keys already exists in the entry (see
`user-activate <http://www.freeipa.org/page/V4/User_Life-Cycle_Management#Activate_StageUser>`__)

.. _future_enhancements:

Future enhancements
~~~~~~~~~~~~~~~~~~~

.. _deletion_of_active_user:

Deletion of active user
^^^^^^^^^^^^^^^^^^^^^^^

When using the FreeIPA CLI
`user-del <http://www.freeipa.org/page/V3/User_Life-Cycle_Management#Delete_User>`__,
the entry is moved (MODRDN) from the *Active* container to the *Delete*
container. If the authorized person, issue LDAP DEL (outside of FreeIPA
framework) on an active user the entry will be deleted permanently from
the database and its properties lost. It could be interesting to
implement a new preop DS plugin, which would intercept the LDAP delete
operation and move the *Active* entry to the *Delete* container.

Implementation
--------------

`http://www.freeipa.org/page/V3/User_Life-Cycle_Management/Implementation
Implementation
details <http://www.freeipa.org/page/V3/User_Life-Cycle_Management/Implementation_Implementation_details>`__
still need to be defined. Consider this section as a work in progress.

.. _freeipa_tickets:

FreeIPA tickets
~~~~~~~~~~~~~~~

-  `#3911 <https://fedorahosted.org/freeipa/ticket/3911>`__: [RFE] Allow
   managing users add/modify/delete via LDAP client
-  `#3813 <https://fedorahosted.org/freeipa/ticket/3813>`__: [RFE]
   Provide user lifecycle managment capabilities
-  `#4675 <https://fedorahosted.org/freeipa/ticket/4675>`__: [RFE]
   prevent newly activated user to be immediatly in the configured
   automember groups

.. _directory_server_tickets:

389 Directory Server tickets
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _scoping_of_plugins:

Scoping of plugins
^^^^^^^^^^^^^^^^^^

-  `#47527 <https://fedorahosted.org/389/ticket/47527>`__: Allow
   referential integrity suffixes to be configurable (fixed *1.3.2.8*)
-  `#47621 <https://fedorahosted.org/389/ticket/47621>`__: make
   referential integrity configuration more flexible (exclude subtree)
   (fixed *1.3.2.9*)
-  `#47525 <https://fedorahosted.org/389/ticket/47525>`__: Allow
   memberOf to use an alternate config area (fixed *1.3.3*)
-  `#47526 <https://fedorahosted.org/389/ticket/47526>`__: Allow
   memberOf suffixes to be configurable (fixed *1.3.2.8*)
-  `#47829 <https://fedorahosted.org/389/ticket/47829>`__: memberof
   scope: allow to exclude subtrees (fixed *1.3.3*)
-  `#47828 <https://fedorahosted.org/389/ticket/47828>`__: DNA scope:
   allow to exlude some subtrees

others
^^^^^^

-  `#47529 <https://fedorahosted.org/389/ticket/47529>`__: Automember
   plug-in should treat MODRDN operations as ADD operations (fixed
   *1.3.3*)
-  `#47553 <https://fedorahosted.org/389/ticket/47553>`__: Enhance ACIs
   to have more control over MODRDN operations (fixed *1.3.3*)
-  `#47823 <https://fedorahosted.org/389/ticket/47823>`__: Enforce
   attribute uniqueness accross all the scoped subtrees (fixed *1.3.3*)



Feature Management
------------------

UI
~~

When user provisioning is enabled, add new tab to *Identity* section -
*Staged Users*. There should be 2 user sections:

-  List of *staged* users. Ability to make the staged user active
-  List of *deleted* users. Ability to make the deleted user active and
   move either to *staged* or *active* users area.

In *IPA Server* section, *Configuration* tab, there should be a new
section *User Life-Cycle Management* with configuration options
described in section `#Major configuration options and
enablement <#Major_configuration_options_and_enablement>`__.

CLI
~~~

.. _user_add:

user-add
^^^^^^^^

-  New option ``--from-staged=tuser``: activates user specified by it's
   primary key in the *staged* tree
-  New option ``--to-staged``: creates a new user in the staging tree
-  New option ``--staged-pkey=uid``: optional, defines alternative
   primary key of the staged user. ``uid`` is the default
-  New option ``--from-deleted=tuser``: specifies primary key of the
   deleted user to be activated

.. _user_find:

user-find
^^^^^^^^^

-  New option ``--staged``: lists staged users instead of active users
-  New option ``--deleted``: lists deleted users instead of active users

user_provisioning_is_enabled
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Returns TRUE if ``cn=provisioning`` exists. Will be used by Web UI to
find out if the feature is enabled or not.

.. _config_mod:

config-mod
^^^^^^^^^^

-  New config options described in section `#Major configuration options
   and enablement <#Major_configuration_options_and_enablement>`__



Major configuration options and enablement
------------------------------------------

Following global user configuration options should be implemented:

-  ``ipaStagedUserNumericMagic``: placeholder for generation of user
   value for numeric attribute types. Default is **-1**
-  ``ipaStageDeletedUser``: move deleted users to *deleted* users tree
   instead of deleting them permanently. Default is **FALSE**

Replication
-----------

``cn=provisioning`` subtree should be replicated.

Given number of affected plugin configuration described in section
`#Affected Directory Server
Plugins <#Affected_Directory_Server_Plugins>`__ we should consider
moving configuration of some of these plugins from ``cn=config`` to
replicated tree to enable easier changes in the configuration.

Attribute uniqueness will scope *Active* and *Delete* containers. In
order to prevent replication issue, **all** activation
(stageuser-activate) of user account should be done on the **same**
server.



Updates and Upgrades
--------------------

Feature should not be enabled by default, but after running a
configuration script ``ipa-advanced-provisioning-install``. The script
should:

#. Check if all replicas are of the current FreeIPA version. If not, it
   should end with error stating that the feature may have negative
   impact on the older FreeIPA versions. There should be a ``--force``
   option to overcome this limitation.
#. When check is successful or ``--force`` flag is used:

   #. Create ``cn=provisioning`` structure
   #. Add new schema and config options
   #. Add new ACIs, permissions, privileges and roles
   #. Add new Directory Server plugins
   #. Restart Directory Server

.. _heterogeneous_environment:

Heterogeneous Environment
~~~~~~~~~~~~~~~~~~~~~~~~~

If there is an environment with FreeIPA servers supporting the feature
and olded FreeIPA servers, there may be issues with the life-cycle
management, like:

-  When user delete operation is run against an older FreeIPA server, it
   is not copied to *deleted* users tree
-  Staged or deleted users in ``cn=provisioning`` tree may interfere
   with older FreeIPA and it's DS plugins.



External Impact
---------------

Changes in *389 Directory Server* and *slapi-nis* packages will be
required.

.. _how_to_test41:

How to Test
-----------

.. _external_provisioning_system_1:

External Provisioning System
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _adding_new_user:

Adding New User
^^^^^^^^^^^^^^^

In following example, we will simulate adding new *Stage User* by
provisioning system using the standard inetorgperson objectclass,
without using any FreeIPA specific attribute.

Add Stage User with ldapmodify:

::

   # ldapmodify -Y GSSAPI
   SASL/GSSAPI authentication started
   SASL username: admin@RHEL72
   SASL SSF: 56
   SASL data security layer installed.
   dn: uid=stageuser,cn=staged users,cn=accounts,cn=provisioning,dc=rhel72
   changetype: add
   objectClass: top
   objectClass: inetorgperson
   cn: Stage
   sn: User

   adding new entry "uid=stageuser,cn=staged users,cn=accounts,cn=provisioning,dc=rhel72"

Show it's all attributes (see that the account is explicitly disabled by
nsaccountlock attribute):

::

   # ipa stageuser-show stageuser --all --raw
     dn: uid=stageuser,cn=staged users,cn=accounts,cn=provisioning,dc=rhel72
     uid: stageuser
     sn: User
     cn: Stage
     has_password: FALSE
     has_keytab: FALSE
     nsaccountlock: TRUE
     objectClass: top
     objectClass: inetorgperson
     objectClass: organizationalPerson
     objectClass: person

Activate the Stage User (may be also done by other admin, with *System:
Add Users* permission):

::

   # ipa stageuser-activate stageuser
   ------------------------------
   Stage user stageuser activated
   ------------------------------
     User login: stageuser
     First name: Stage
     Last name: User
     Full name: Stage
     Home directory: /home/stageuser
     Login shell: /bin/sh
     Kerberos principal: stageuser@RHEL72
     UID: 626000004
     GID: 626000004

Show the resulting activated user which has all the FreeIPA specific
attributes generated:

::

   # ipa user-show stageuser --all --raw
     dn: uid=stageuser,cn=users,cn=accounts,dc=rhel72
     uid: stageuser
     givenname: Stage
     sn: User
     cn: Stage
     homedirectory: /home/stageuser
     loginshell: /bin/sh
     uidnumber: 626000004
     gidnumber: 626000004
     nsaccountlock: FALSE
     has_password: FALSE
     has_keytab: FALSE
     ipaUniqueID: 48a58be2-dc67-11e5-b93e-001a4a23140a
     krbPrincipalName: stageuser@RHEL72
     memberof: cn=ipausers,cn=groups,cn=accounts,dc=rhel72
     mepManagedEntry: cn=stageuser,cn=groups,cn=accounts,dc=rhel72
     objectClass: ipaobject
     objectClass: person
     objectClass: top
     objectClass: ipasshuser
     objectClass: inetorgperson
     objectClass: organizationalperson
     objectClass: krbticketpolicyaux
     objectClass: krbprincipalaux
     objectClass: inetuser
     objectClass: posixaccount
     objectClass: ipaSshGroupOfPubKeys
     objectClass: mepOriginEntry

.. _activating_users_automatically:

Activating Users Automatically
''''''''''''''''''''''''''''''

Alternatively, if the Provisioning System does not have any means of
calling the API operation and you do not want to do it manually, you can
also setup a cron call (`example for
Fedora <https://docs.fedoraproject.org/en-US/Fedora/22/html/System_Administrators_Guide/ch-Automating_System_Tasks.html#s2-configuring-cron-jobs>`__)
that would periodically *kinit* with a keytab of a service allowed to
*add users*, search for staged users and activating them.

Original situation:

::

   # ipa stageuser-find
   ---------------
   2 users matched
   ---------------
     User login: bar
     First name: bar
     Last name: bar
     Home directory: /home/bar
     Login shell: /bin/sh
     Email address: bar@rhel72.test
     UID: -1
     GID: -1
     Password: False
     Kerberos keys available: False

     User login: foo
     First name: foo
     Last name: bar
     Home directory: /home/foo
     Login shell: /bin/sh
     Email address: foo@rhel72.test
     UID: -1
     GID: -1
     Password: False
     Kerberos keys available: False
   ----------------------------
   Number of entries returned 2
   ----------------------------

The automated activation task example that can be used in cron script:

::

   # kinit -kt service.keytab activator/ipa.provisioning.system.rhel72.test
   # for x in `ipa stageuser-find | grep "User login:" | cut -d":" -f 2`; do
       ipa stageuser-activate $x;
   done
   ------------------------
   Stage user bar activated
   ------------------------
     User login: bar
     First name: bar
     Last name: bar
     Full name: bar bar
     Display name: bar bar
     Initials: bb
     Home directory: /home/bar
     GECOS: bar bar
     Login shell: /bin/sh
     Kerberos principal: bar@RHEL72
     Email address: bar@rhel72.test
     UID: 626000005
     GID: 626000005
   ------------------------
   Stage user foo activated
   ------------------------
     User login: foo
     First name: foo
     Last name: bar
     Full name: foo bar
     Display name: foo bar
     Initials: fb
     Home directory: /home/foo
     GECOS: foo bar
     Login shell: /bin/sh
     Kerberos principal: foo@RHEL72
     Email address: foo@rhel72.test
     UID: 626000006
     GID: 626000006

.. _deleting_a_user:

Deleting a User
^^^^^^^^^^^^^^^

When a user account is deleted permanently, Provisioning System should
simply issue LDAP DEL operation. When the user account is to be
*preserved*, it just needs to be moved to specific container.

Preserving a user can be done with CLI/API call:

::

   # ipa user-find
   ...
     User login: fbar
     First name: Foo
     Last name: Bar
     Home directory: /home/fbar
     Login shell: /bin/sh
     Email address: foo@rhel72.test
     UID: 626000001
     GID: 626000001
     Account disabled: False
     Password: True
     Kerberos keys available: True

     User login: stageuser
     First name: Stage
     Last name: User
     Home directory: /home/stageuser
     Login shell: /bin/sh
     UID: 626000004
     GID: 626000004
     Account disabled: False
     Password: False
     Kerberos keys available: False
   ...
   # ipa user-del fbar --preserve
   -------------------
   Deleted user "fbar"
   -------------------
   # ipa user-find --preserved=1
   --------------
   1 user matched
   --------------
     User login: fbar
     First name: Foo
     Last name: Bar
     Home directory: /home/fbar
     Login shell: /bin/sh
     Email address: foo@rhel72.test
     UID: 626000001
     GID: 626000001
     Account disabled: True
     Preserved user: True
     Password: False
     Kerberos keys available: False
   ----------------------------
   Number of entries returned 1
   ----------------------------

The preserved user is automatically disabled and can no longer
authenticate.

Second option is to preserve the user via LDAP MODRDN command directly:

::

   # ldapmodify -Y GSSAPI
   SASL/GSSAPI authentication started
   SASL username: admin@RHEL72
   SASL SSF: 56
   SASL data security layer installed.
   dn: uid=stageuser,cn=users,cn=accounts,dc=rhel72
   changetype: modrdn
   newrdn: uid=stageuser
   deleteoldrdn: 0
   newsuperior: cn=deleted users,cn=accounts,cn=provisioning,dc=rhel72

   modifying rdn of entry "uid=stageuser,cn=users,cn=accounts,dc=rhel72"

   # ipa user-find --preserved=1
   ---------------
   2 users matched
   ---------------
     User login: fbar
     First name: Foo
     Last name: Bar
     Home directory: /home/fbar
     Login shell: /bin/sh
     Email address: foo@rhel72.test
     UID: 626000001
     GID: 626000001
     Account disabled: True
     Preserved user: True
     Password: False
     Kerberos keys available: False

     User login: stageuser
     First name: Stage
     Last name: User
     Home directory: /home/stageuser
     Login shell: /bin/sh
     UID: 626000004
     GID: 626000004
     Account disabled: True
     Preserved user: True
     Password: False
     Kerberos keys available: False
   ----------------------------
   Number of entries returned 2
   ----------------------------

.. _privilege_separation_for_employee_entry_and_activation_1:

Privilege Separation for Employee Entry and Activation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In following example, we will simulate adding a user in 2 steps. First,
adding a *Stage User* by a Helpdesk administrator without a permission
to add active users and second activating the user by *Security
Administrator*.

First, Stage User is added (*System: Add Stage User* permission is
sufficient for that operation):

::

   # ipa stageuser-add barbar --first Bar --last Bar
   -------------------------
   Added stage user "barbar"
   -------------------------
     User login: barbar
     First name: Bar
     Last name: Bar
     Full name: Bar Bar
     Display name: Bar Bar
     Initials: BB
     Home directory: /home/barbar
     GECOS: Bar Bar
     Login shell: /bin/sh
     Kerberos principal: barbar@RHEL72
     Email address: barbar@rhel72.test
     UID: -1
     GID: -1
     Password: False
     Kerberos keys available: False

The user is already created with all FreeIPA specific attributes. UID
and GID are not generated yet:

::

   # ipa stageuser-show barbar --all --raw
     dn: uid=barbar,cn=staged users,cn=accounts,cn=provisioning,dc=rhel72
     uid: barbar
     givenname: Bar
     sn: Bar
     cn: Bar Bar
     initials: BB
     homedirectory: /home/barbar
     gecos: Bar Bar
     loginshell: /bin/sh
     mail: barbar@rhel72.test
     uidnumber: -1
     gidnumber: -1
     has_password: FALSE
     has_keytab: FALSE
     description: __no_upg__
     displayName: Bar Bar
     ipaUniqueID: autogenerate
     krbPrincipalName: barbar@RHEL72
     nsaccountlock: TRUE
     objectClass: ipaobject
     objectClass: person
     objectClass: top
     objectClass: ipasshuser
     objectClass: inetorgperson
     objectClass: organizationalperson
     objectClass: krbticketpolicyaux
     objectClass: krbprincipalaux
     objectClass: inetuser
     objectClass: posixaccount
     objectClass: ipaSshGroupOfPubKeys

Activate the Stage User (*System: Add Users* permission is required):

::

   # ipa stageuser-activate barbar
   ---------------------------
   Stage user barbar activated
   ---------------------------
     User login: barbar
     First name: Bar
     Last name: Bar
     Full name: Bar Bar
     Display name: Bar Bar
     Initials: BB
     Home directory: /home/barbar
     GECOS: Bar Bar
     Login shell: /bin/sh
     Kerberos principal: barbar@RHEL72
     Email address: barbar@rhel72.test
     UID: 626000003
     GID: 626000003

See that the user has all the FreeIPA attributes including UID and GID
generated:

::

   # ipa user-show barbar --all --raw
     dn: uid=barbar,cn=users,cn=accounts,dc=rhel72
     uid: barbar
     givenname: Bar
     sn: Bar
     cn: Bar Bar
     initials: BB
     homedirectory: /home/barbar
     gecos: Bar Bar
     loginshell: /bin/sh
     mail: barbar@rhel72.test
     uidnumber: 626000003
     gidnumber: 626000003
     nsaccountlock: FALSE
     has_password: FALSE
     has_keytab: FALSE
     displayName: Bar Bar
     ipaUniqueID: 9d5dddca-dc66-11e5-b542-001a4a23140a
     krbPrincipalName: barbar@RHEL72
     memberof: cn=ipausers,cn=groups,cn=accounts,dc=rhel72
     mepManagedEntry: cn=barbar,cn=groups,cn=accounts,dc=rhel72
     objectClass: ipasshgroupofpubkeys
     objectClass: ipaobject
     objectClass: person
     objectClass: top
     objectClass: ipasshuser
     objectClass: inetorgperson
     objectClass: organizationalperson
     objectClass: krbticketpolicyaux
     objectClass: krbprincipalaux
     objectClass: inetuser
     objectClass: posixaccount
     objectClass: mepOriginEntry



Test Plan
---------

See `Unit tests plan <V4/User_Life-Cycle_Management/tests>`__
