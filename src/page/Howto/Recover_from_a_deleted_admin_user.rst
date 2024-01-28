Recover from a deleted admin user
=================================

Description
-----------

This page explains how to recover when the admin user has been deleted.

In older IPA versions, it was possible to delete the admin user,
provided an alternate user was member of the admins group.

The deletion of the admin user caused multiple issues (for instance
breaking the upgrade as seen in 
`ticket 9500 <https://pagure.io/freeipa/issue/9500>`__).

A mechanism preventing this deletion has been implemented with the fix for
`ticket 8878 <https://pagure.io/freeipa/issue/8878>`__. The fix is available
in versions 4.9.13+ and 4.11.0+, but if your deployment has lost its admin
user, you need to recreate a new admin user following the procedure detailed
below.

Recovery procedure
------------------

In order to recreate the admin user, the following ``admin-user.update`` file
needs to be customized and provided to ``ipa-ldap-updater`` tool.

.. code-block:: text

   [root@server ~]# cat /tmp/admin-user.update
   dn: uid=admin,cn=users,cn=accounts,$SUFFIX
   default: objectClass: top
   default: objectClass: person
   default: objectClass: posixaccount
   default: objectClass: krbprincipalaux
   default: objectClass: krbticketpolicyaux
   default: objectClass: inetuser
   default: objectClass: ipaobject
   default: objectClass: ipasshuser
   default: objectClass: ipaNTUserAttrs
   default: uid: admin
   default: krbPrincipalName: admin@$REALM
   default: cn: Administrator
   default: sn: Administrator
   default: uidNumber: VALUE_UID
   default: gidNumber: VALUE_UID
   default: homeDirectory: /home/admin
   default: loginShell: /bin/bash
   default: gecos: Administrator
   default: nsAccountLock: FALSE
   default: ipaUniqueID: autogenerate
   default: ipaNTSecurityIdentifier: VALUESID-500

In this file, the values VALUE_UID and VALUE_SID must be replaced with
correct values obtained with these commands (let's consider that your
alternate admin user is ``otheradmin``):

.. code-block:: text

   [root@server ~] kinit otheradmin
   Password for otheradmin@IPA.TEST: 
   [root@server ~] ipa idrange-find --type ipa-local
   ---------------
   1 range matched
   ---------------
     Range name: IPA.TEST_id_range
     First Posix ID of the range: 1206200000
     Number of IDs in the range: 200000
     First RID of the corresponding RID range: 1000
     First RID of the secondary RID range: 100000000
     Range type: local domain range
   ----------------------------
   Number of entries returned 1
   ----------------------------
   [root@server ~]# ipa trustconfig-show
     Domain: ipa.test
     Security Identifier: S-1-5-21-3674471173-2442480195-1112681658
     NetBIOS name: IPA
     Domain GUID: 773b0192-6402-40a7-9026-e6b557f6daac
     Fallback primary group: Default SMB Group

Carefully note the value corresponding to ``First Posix ID of the range``
and substitute this value to ``VALUE_UID`` in the ``admin-user.update`` file
(there are 2 occurrences to replace, one for ``uidNumber`` and the other for
``gidNumber``).

Carefully note the value corresponding to ``Security Identifier`` and
substitute this value to ``VALUE_SID`` in the ``admin-user.update`` file (do not
remove the trailing ``-500`` part as it corresponds to the RID for the admin
user). If your deployment does not display any value for
``ipa trustconfig-show``, you can simply remove the lines containing
``ipaNTSecurityIdentifier`` and ``ipaNTUserAttrs`` from the
``admin-user.update`` file.

The resulting file should look like the following:

.. code-block:: text

   [root@server ~]# cat /tmp/admin-user.update
   dn: uid=admin,cn=users,cn=accounts,$SUFFIX
   default: objectClass: top
   default: objectClass: person
   default: objectClass: posixaccount
   default: objectClass: krbprincipalaux
   default: objectClass: krbticketpolicyaux
   default: objectClass: inetuser
   default: objectClass: ipaobject
   default: objectClass: ipasshuser
   default: objectClass: ipaNTUserAttrs
   default: uid: admin
   default: krbPrincipalName: admin@$REALM
   default: cn: Administrator
   default: sn: Administrator
   default: uidNumber: 1206200000
   default: gidNumber: 1206200000
   default: homeDirectory: /home/admin
   default: loginShell: /bin/bash
   default: gecos: Administrator
   default: nsAccountLock: FALSE
   default: ipaUniqueID: autogenerate
   default: ipaNTSecurityIdentifier: S-1-5-21-3674471173-2442480195-1112681658-500

The tool ``ipa-ldap-updater`` can now be used to create the admin user:

.. code-block:: text

   [root@server ~]# ipa-ldap-updater /tmp/admin-user.update
   Update complete
   The ipa-ldap-updater command was successful

After this step, you can add the admin user to the admins group:

.. code-block:: text

   [root@server ~]# ipa group-add-member admins --users admin
     Group name: admins
     Description: Account administrators group
     GID: 1206200000
     Member users: otheradmin, admin
   -------------------------
   Number of members added 1
   -------------------------

If you had SIDs for your domain, re-run the sid generation task and verify
that the admins group has a SID ending with -512 as before:

.. code-block:: text

   [root@server ~]# ipa config-mod --add-sids --enable-sid
   [root@server ~]# ipa group-show admins --all
     dn: cn=admins,cn=groups,cn=accounts,dc=ipa,dc=test
     Group name: admins
     Description: Account administrators group
     GID: 1206200000
     Member users: otheradmin, admin
     ipantsecurityidentifier: S-1-5-21-3674471173-2442480195-1112681658-512
     ipauniqueid: 53f23254-ab15-11ee-bdf6-fa163ee87a63
     objectclass: top, groupofnames, posixgroup, ipausergroup, ipaobject,
                  nestedGroup, ipaNTGroupAttrs


If you do not want to use the admin user, you can disable the account using:

.. code-block:: text

   [root@server ~]# ipa user-disable admin
   -----------------------------
   Disabled user account "admin"
   -----------------------------

