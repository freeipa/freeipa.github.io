Overview
========

Change Log Monitor listens to IPA's Retro Change Log subtree and invokes
the Sync Agent to propagate IPA changes into Samba.

IPA operations that need to be synchronized to Samba are:

-  Adding IPA User
-  Modifying IPA User
-  Deleting IPA User
-  Locking IPA User
-  Unlocking IPA User
-  Adding IPA Group
-  Modifying IPA Group
-  Deleting IPA Group
-  Adding IPA Host
-  Modifying IPA Host
-  Deleting IPA Host

Other operations will be ignored.

.. _adding_ipa_user:

Adding IPA User
===============

Consider the following example:

::

   % ipa-adduser -f "Endi" -l "Dewata" -c "Endi S. Dewata" -d "/home/edewata" \
   -s "/bin/sh" -p "Secret123" edewata

The above command will generate 6 change log records.

.. _adding_user_object:

Adding User Object
------------------

::

   dn: changenumber=1333,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1333
   targetDn: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   changeTime: 20090813172845Z
   changeType: add
   changes:: ...

The content of the changes attribute is:

::

   uid: edewata
   objectClass: top
   objectClass: person
   objectClass: organizationalPerson
   objectClass: inetOrgPerson
   objectClass: inetUser
   objectClass: posixAccount
   objectClass: krbPrincipalAux
   objectClass: radiusprofile
   loginShell: /bin/sh
   gidNumber: 1002
   gecos: Endi S. Dewata
   sn: Dewata
   homeDirectory: /home/edewata
   krbPrincipalName: edewata@DOMAIN1.COM
   givenName: Endi
   cn: Endi Dewata
   creatorsName: uid=admin,cn=users,cn=accounts,dc=domain1,dc=com
   modifiersName: uid=admin,cn=users,cn=accounts,dc=domain1,dc=com
   createTimestamp: 20090813172845Z
   modifyTimestamp: 20090813172845Z
   uidNumber: 1155

A new user will be added to Samba.

.. _adding_memberof_attribute:

Adding MemberOf Attribute
-------------------------

::

   dn: changenumber=1334,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1334
   targetDn: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   changeTime: 20090813172845Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   replace: memberOf
   memberOf: cn=ipausers,cn=groups,cn=accounts,dc=domain1,dc=com
   -
   replace: modifiersname
   modifiersname: cn=ipa-memberof,cn=plugins,cn=config
   -
   replace: modifytimestamp
   modifytimestamp: 20090813172845Z
   -

Changes by cn=ipa-memberof,cn=plugins,cn=config will be skipped.

.. _adding_member_attribute:

Adding Member Attribute
-----------------------

::

   dn: changenumber=1335,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1335
   targetDn: cn=ipausers,cn=groups,cn=accounts,dc=domain1,dc=com
   changeTime: 20090813172845Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   add: member
   member: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   -
   replace: modifiersname
   modifiersname: uid=admin,cn=users,cn=accounts,dc=domain1,dc=com
   -
   replace: modifytimestamp
   modifytimestamp: 20090813172845Z
   -

Changes to IPA group will be synchronized to Samba.

.. _change_password_grace_user_time:

Change Password Grace User Time
-------------------------------

::

   dn: changenumber=1337,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1337
   targetDn: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   changeTime: 20090813172845Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   replace: passwordgraceusertime
   passwordgraceusertime: 0
   -

Need to investigate.

.. _set_password:

Set Password
------------

::

   dn: changenumber=1338,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1338
   targetDn: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   changeTime: 20090813172845Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   replace: krbPrincipalKey
   krbPrincipalKey:: MIICTKADAgEBoQMCAQGiAwIBAaMDAgEApIICNDCCAjAwaqAdMBugAwIBAKE
    UBBJET01BSU4xLkNPTWVkZXdhdGGhSTBHoAMCARKhQAQ+IADqr85DPunaQeWhHM6x6nN9FWsxcYp
    EN4TILRUd6U955/QIO0rUNZOvOrtSGiF2sYBTRjU5aXYA+hHQg/8wWqAdMBugAwIBAKEUBBJET01
    BSU4xLkNPTWVkZXdhdGGhOTA3oAMCARGhMAQuEAAGXi3a7V7sBM7Aj58MvUCWzWCTrLxVppt/Zh3
    3+YK44kD4gBV1SSuC1gFItDBioB0wG6ADAgEAoRQEEkRPTUFJTjEuQ09NZWRld2F0YaFBMD+gAwI
    BEKE4BDYYAG9kUoG5ZI54SzveRoFuGpimYqOCobvTmI9EaGO4GbdRa3OluhUoyhBMGYbiQnF90/n
    wJpowWqAdMBugAwIBAKEUBBJET01BSU4xLkNPTWVkZXdhdGGhOTA3oAMCARehMAQuEADdXVu9l0l
    08s0LZL8g9t8pC+t4jJc8u3mfd4u5YMnv7SpT4k4m2zD0kEu7cDBSoB0wG6ADAgEAoRQEEkRPTUF
    JTjEuQ09NZWRld2F0YaExMC+gAwIBCKEoBCYIABT5wjT581SgPZ+ipDin+uLPWPuQr8QLuAP19uM
    PcWj2rMAhLzBSoB0wG6ADAgEAoRQEEkRPTUFJTjEuQ09NZWRld2F0YaExMC+gAwIBA6EoBCYIACf
    41I/k1eNguNoVrTY5fhC0O2l3urNamJ+E+msmkEuMYU6kRg==
   -
   replace: krbLastPwdChange
   krbLastPwdChange: 20090813172846Z
   -
   replace: krbPasswordExpiration
   krbPasswordExpiration: 20090813172846Z
   -
   replace: userPassword
   userPassword: {SSHA}nyibJqXkuNU6bf+kfERx5SO9a0QRMBDFPvDabw==
   -
   replace: modifiersname
   modifiersname: cn=ipa_pwd_extop,cn=plugins,cn=config
   -
   replace: modifytimestamp
   modifytimestamp: 20090813172845Z
   -
   replace: unhashed#user#password
   unhashed#user#password: Secret123
   -

User password will be synchronized to Samba.

.. _update_modifiers_info:

Update Modifiers Info
---------------------

::

   dn: changenumber=1339,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1339
   targetDn: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   changeTime: 20090813172847Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   replace: modifiersName
   modifiersName: cn=ipa-memberof,cn=plugins,cn=config
   -
   replace: modifyTimestamp
   modifyTimestamp: 20090813172848Z
   -

Need to investigate.

.. _modifying_ipa_user:

Modifying IPA User
==================

Consider the following example:

::

   % ipa-moduser -s "/bin/bash" edewata

This operation will generate 1 change log record.

::

   dn: changenumber=1363,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1363
   targetDn: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   changeTime: 20090818231246Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   delete: loginShell
   loginShell: /bin/sh
   -
   add: loginShell
   loginShell: /bin/bash
   -
   replace: modifiersname
   modifiersname: uid=admin,cn=users,cn=accounts,dc=domain1,dc=com
   -
   replace: modifytimestamp
   modifytimestamp: 20090818231246Z
   -

The corresponding Samba user will be modified.

.. _deleting_ipa_user:

Deleting IPA User
=================

Consider the following example:

::

   % ipa-deluser edewata

This operation will generate 3 change log records.

.. _deleting_group_member:

Deleting Group Member
---------------------

::

   dn: changenumber=1342,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1342
   targetDn: cn=ipausers,cn=groups,cn=accounts,dc=domain1,dc=com
   changeTime: 20090813225919Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   delete: member
   member: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   -
   replace: modifiersname
   modifiersname: cn=ipa-memberof,cn=plugins,cn=config
   -
   replace: modifytimestamp
   modifytimestamp: 20090813225919Z
   -

The corresponding Samba group will be modified.

.. _deleting_group_member_1:

Deleting Group Member
---------------------

::

   dn: changenumber=1343,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1343
   targetDn: cn=ipausers,cn=groups,cn=accounts,dc=domain1,dc=com
   changeTime: 20090813225923Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   delete: member
   member: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   -
   replace: modifiersName
   modifiersName: cn=ipa-memberof,cn=plugins,cn=config
   -
   replace: modifyTimestamp
   modifyTimestamp: 20090813225920Z
   -

Need to investigate.

.. _deleting_user_object:

Deleting User Object
--------------------

::

   dn: changenumber=1344,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1344
   targetDn: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   changeTime: 20090813225923Z
   changeType: delete

The corresponding Samba user will be deleted.

.. _adding_ipa_group:

Adding IPA Group
================

Consider the following example:

::

   % ipa-addgroup -d "Developers" developers

The above command will generate 1 change log record:

::

   dn: changenumber=1347,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1347
   targetDn: cn=developers,cn=groups,cn=accounts,dc=domain1,dc=com
   changeTime: 20090818060422Z
   changeType: add
   changes:: ...

The content of the changes attribute is:

::

   objectClass: top
   objectClass: groupofnames
   objectClass: posixGroup
   objectClass: inetUser
   cn: developers
   description: Developers
   creatorsName: uid=admin,cn=users,cn=accounts,dc=domain1,dc=com
   modifiersName: uid=admin,cn=users,cn=accounts,dc=domain1,dc=com
   createTimestamp: 20090818060421Z
   modifyTimestamp: 20090818060421Z
   gidNumber: 1241

A new group will be added to Samba.

.. _deleting_ipa_group:

Deleting IPA Group
==================

Consider the following example:

::

   % ipa-delgroup developers

The above command will generate 1 change log record.

::

   dn: changenumber=1348,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1348
   targetDn: cn=developers,cn=groups,cn=accounts,dc=domain1,dc=com
   changeTime: 20090818061104Z
   changeType: delete

The corresponding Samba group will be deleted.

.. _adding_group_member:

Adding Group Member
===================

Consider the following example:

::

   % ipa-modgroup -a edewata developers

The above command will generate 2 change log records.

.. _modifying_user_membership:

Modifying User Membership
-------------------------

::

   dn: changenumber=1355,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1355
   targetDn: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   changeTime: 20090818063212Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   replace: memberOf
   memberOf: cn=ipausers,cn=groups,cn=accounts,dc=domain1,dc=com
   memberOf: cn=developers,cn=groups,cn=accounts,dc=domain1,dc=com
   -
   replace: modifiersname
   modifiersname: cn=ipa-memberof,cn=plugins,cn=config
   -
   replace: modifytimestamp
   modifytimestamp: 20090818063212Z
   -

Changes by cn=ipa-memberof,cn=plugins,cn=config will be skipped.

.. _adding_group_member_1:

Adding Group Member
-------------------

::

   dn: changenumber=1356,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1356
   targetDn: cn=developers,cn=groups,cn=accounts,dc=domain1,dc=com
   changeTime: 20090818063212Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   add: member
   member: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   -
   replace: modifiersname
   modifiersname: uid=admin,cn=users,cn=accounts,dc=domain1,dc=com
   -
   replace: modifytimestamp
   modifytimestamp: 20090818063212Z
   -

The corresponding Samba group will be modified.

.. _deleting_group_member_2:

Deleting Group Member
=====================

Consider the following example:

::

   % ipa-modgroup -r edewata developers

The above command will generate 2 change log records.

.. _deleting_user_membership:

Deleting User Membership
------------------------

::

   dn: changenumber=1357,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1357
   targetDn: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   changeTime: 20090818064118Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   replace: memberOf
   memberOf: cn=ipausers,cn=groups,cn=accounts,dc=domain1,dc=com
   -
   replace: modifiersname
   modifiersname: cn=ipa-memberof,cn=plugins,cn=config
   -
   replace: modifytimestamp
   modifytimestamp: 20090818064118Z
   -

Changes by cn=ipa-memberof,cn=plugins,cn=config will be skipped.

.. _deleting_group_member_3:

Deleting Group Member
---------------------

::

   dn: changenumber=1358,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1358
   targetDn: cn=developers,cn=groups,cn=accounts,dc=domain1,dc=com
   changeTime: 20090818064118Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   delete: member
   member: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   -
   replace: modifiersname
   modifiersname: uid=admin,cn=users,cn=accounts,dc=domain1,dc=com
   -
   replace: modifytimestamp
   modifytimestamp: 20090818064118Z
   -

The corresponding Samba group will be modified.

.. _locking_ipa_user:

Locking IPA User
================

Consider the following example:

::

   % ipa-lockuser edewata

The above command will generate 2 change log records.

.. _modifying_user_membership_1:

Modifying User Membership
-------------------------

::

   dn: changenumber=1359,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1359
   targetDn: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   changeTime: 20090818065114Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   replace: memberOf
   memberOf: cn=ipausers,cn=groups,cn=accounts,dc=domain1,dc=com
   memberOf: cn=inactivated,cn=account inactivation,cn=accounts,dc=domain1,dc=com
   -
   replace: modifiersname
   modifiersname: cn=ipa-memberof,cn=plugins,cn=config
   -
   replace: modifytimestamp
   modifytimestamp: 20090818065114Z
   -

Changes by cn=ipa-memberof,cn=plugins,cn=config will be skipped.

.. _adding_user_into_inactivated_group:

Adding User Into Inactivated Group
----------------------------------

::

   dn: changenumber=1360,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1360
   targetDn: cn=inactivated,cn=account inactivation,cn=accounts,dc=domain1,dc=com
   changeTime: 20090818065114Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   add: member
   member: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   -
   replace: modifiersname
   modifiersname: uid=admin,cn=users,cn=accounts,dc=domain1,dc=com
   -
   replace: modifytimestamp
   modifytimestamp: 20090818065114Z
   -

The corresponding Samba user will be disabled. The corresponding Samba
group will be modified.

.. _unlocking_ipa_user:

Unlocking IPA User
==================

Consider the following example:

::

   % ipa-lockuser -u edewata

The above command will generate 2 change log records.

.. _modifying_user_membership_2:

Modifying User Membership
-------------------------

::

   dn: changenumber=1361,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1361
   targetDn: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   changeTime: 20090818071828Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   replace: memberOf
   memberOf: cn=ipausers,cn=groups,cn=accounts,dc=domain1,dc=com
   -
   replace: modifiersname
   modifiersname: cn=ipa-memberof,cn=plugins,cn=config
   -
   replace: modifytimestamp
   modifytimestamp: 20090818071828Z
   -

Changes by cn=ipa-memberof,cn=plugins,cn=config will be skipped.

.. _deleting_user_from_inactivated_group:

Deleting User From Inactivated Group
------------------------------------

::

   dn: changenumber=1362,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: 1362
   targetDn: cn=inactivated,cn=account inactivation,cn=accounts,dc=domain1,dc=com
   changeTime: 20090818071828Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   delete: member
   member: uid=edewata,cn=users,cn=accounts,dc=domain1,dc=com
   -
   replace: modifiersname
   modifiersname: uid=admin,cn=users,cn=accounts,dc=domain1,dc=com
   -
   replace: modifytimestamp
   modifytimestamp: 20090818071828Z
   -

The corresponding Samba user will be enabled. The corresponding Samba
group will be modified.

Replication
===========

If replication is enabled, it may generate some change log records like
the following:

::

   dn: changenumber=...,cn=changelog
   objectClass: top
   objectClass: changelogentry
   changeNumber: ...
   targetDn: dc=domain1,dc=com
   changeTime: 20090813172845Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   replace: nsds50ruv
   nsds50ruv: {replicageneration} 4a5b47dc000000040000
   nsds50ruv: {replica 4 ldap://ipa1.domain1.com:389} 4a5b5b97000000040000 4a844d
    4f000300040000
   nsds50ruv: {replica 5 ldap://ipa2.domain1.com:389} 4a5b5b9c000000050000 4a7372
    06000300050000
   nsds50ruv: {replica 3 ldap://ipa2.domain1.com:389} 4a5b47e2000000030000 4a5b47
    e4000400030000
   -
   replace: nsruvReplicaLastModified
   nsruvReplicaLastModified: {replica 4 ldap://ipa1.domain1.com:389} 4a844d4d
   nsruvReplicaLastModified: {replica 5 ldap://ipa2.domain1.com:389} 4a737204
   nsruvReplicaLastModified: {replica 3 ldap://ipa2.domain1.com:389} 00000000
   -

This will be ignored.

References
==========

`Category:Obsolete <Category:Obsolete>`__
