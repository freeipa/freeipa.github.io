Overview
========

Samba Change Log Monitor listens to Samba's Retro Change Log subtree and
invokes the Sync Agent to propagate changes from Samba into IPA.

Samba operations that need to be synchronized to IPA are:

-  Adding Samba User
-  Modifying Samba User
-  Deleting Samba User
-  Locking Samba User
-  Unlocking Samba User
-  Adding Samba Group
-  Modifying Samba Group
-  Deleting Samba Group
-  Adding Samba Host
-  Modifying Samba Host
-  Deleting Samba Host

Other operations will be ignored.

.. _adding_samba_user:

Adding Samba User
=================

Consider the following example:

::

   % ldapadd -x -D "cn=Administrator,cn=Users,dc=samba,dc=example,dc=com" -W
   dn: cn=Test User,cn=Users,dc=samba,dc=example,dc=com
   objectClass: user
   userPassword: secret
   userAccountControl: 640

This operation will generate 2 change log records.

.. _updating_dna_plugin:

Updating DNA Plugin
-------------------

::

   dn: changenumber=17,cn=changelog
   objectClass: top
   objectClass: changelogentry
   objectClass: extensibleObject
   changeNumber: 17
   targetDn: dnaHostname=localdc1.samba.example.com+dnaPortNum=0,cn=Samba SIDs,ou
    =Ranges,CN=Samba
   changeTime: 20091208030510Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   replace: dnaRemainingValues
   dnaRemainingValues: 1844674407370955
   -
   replace: modifiersname
   modifiersname: cn=Distributed Numeric Assignment Plugin,cn=plugins,cn=config
   -
   replace: modifytimestamp
   modifytimestamp: 20091208030510Z
   -

.. _adding_user_object:

Adding User Object
------------------

::

   dn: changenumber=18,cn=changelog
   objectClass: top
   objectClass: changelogentry
   objectClass: extensibleObject
   changeNumber: 18
   targetDn: CN=Test User,CN=Users,DC=samba,DC=example,DC=com
   changeTime: 20091208030510Z
   changeType: add
   changes:: ...

The content of the changes attribute is:

::

   userAccountControl: 640
   name: Test User
   cn: Test User
   objectClass: top
   objectClass: person
   objectClass: organizationalPerson
   objectClass: user
   objectClass: extensibleObject
   objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=samba,DC=example,DC=co
    m
   nTSecurityDescriptor:: ...
   sambaBadPasswordCount: 0
   codePage: 0
   countryCode: 0
   badPasswordTime: 0
   sambaLogoffTime: 0
   sambaLogonTime: 0
   primaryGroupID: 513
   accountExpires: 9223372036854775807
   logonCount: 0
   sAMAccountName: $537438A8-9F2CD2C571881C7C
   sAMAccountType: 805306368
   unicodePwd:: h42AFGBs2ilnekTvoTU/xw==
   dBCSPwd:: VSkCAxvt6e+q07Q1tRQE7g==
   ntPwdHistory:: h42AFGBs2ilnekTvoTU/xw==
   lmPwdHistory:: VSkCAxvt6e+q07Q1tRQE7g==
   supplementalCredentials:: ...
   sambaPwdLastSet: 129047151100000000
   msDS-KeyVersionNumber: 1
   instanceType: 4
   creatorsName: cn=samba-admin,cn=samba
   modifiersName: cn=samba-admin,cn=samba
   createTimestamp: 20091208030510Z
   modifyTimestamp: 20091208030510Z
   sambaSID: S-1-5-21-1463069339-4227668456-4007226777-1004

.. _modifying_samba_user:

Modifying Samba User
====================

Consider the following example:

::

   % ldapmodify -x -D "cn=Administrator,cn=Users,dc=samba,dc=example,dc=com" -W
   dn: cn=Test User,cn=Users,dc=samba,dc=example,dc=com
   changetype: modify
   replace: userPassword
   userPassword: secret

This operation example will generate 1 change log record.

.. _modifying_user_object:

Modifying User Object
---------------------

::

   dn: changenumber=19,cn=changelog
   objectClass: top
   objectClass: changelogentry
   objectClass: extensibleObject
   changeNumber: 19
   targetDn: cn=Test User,cn=Users,dc=samba,dc=example,dc=com
   changeTime: 20091208030951Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   replace: unicodePwd
   unicodePwd:: h42AFGBs2ilnekTvoTU/xw==
   -
   replace: dBCSPwd
   dBCSPwd:: VSkCAxvt6e+q07Q1tRQE7g==
   -
   replace: ntPwdHistory
   ntPwdHistory:: h42AFGBs2ilnekTvoTU/xw==
   -
   replace: lmPwdHistory
   lmPwdHistory:: VSkCAxvt6e+q07Q1tRQE7g==
   -
   replace: supplementalCredentials
   supplementalCredentials:: ...
   -
   replace: sambaPwdLastSet
   sambaPwdLastSet: 129047153920000000
   -
   replace: msDS-KeyVersionNumber
   msDS-KeyVersionNumber: 2
   -
   replace: modifiersname
   modifiersname: cn=samba-admin,cn=samba
   -
   replace: modifytimestamp
   modifytimestamp: 20091208030951Z
   -

.. _deleting_samba_user:

Deleting Samba User
===================

Consider the following example:

::

   % ldapdelete -x -D "cn=Administrator,cn=Users,dc=samba,dc=example,dc=com" -W
   cn=Test User,cn=Users,dc=samba,dc=example,dc=com

This operation will generate 1 change log record.

.. _deleting_user_object:

Deleting User Object
--------------------

::

   dn: changenumber=20,cn=changelog
   objectClass: top
   objectClass: changelogentry
   objectClass: extensibleObject
   changeNumber: 20
   targetDn: cn=Test User,cn=Users,dc=samba,dc=example,dc=com
   changeTime: 20091208031201Z
   changeType: delete

.. _locking_samba_user:

Locking Samba User
==================

.. _unlocking_samba_user:

Unlocking Samba User
====================

.. _adding_samba_group:

Adding Samba Group
==================

Consider the following example:

::

   % ldapadd -x -D "cn=Administrator,cn=Users,dc=samba,dc=example,dc=com" -W
   dn: cn=Test Group,cn=Users,dc=samba,dc=example,dc=com
   objectClass: top
   objectClass: group
   cn: Test Group
   member: cn=Test User,cn=Users,dc=samba,dc=example,dc=com

This operation will generate 2 change log records.

.. _updating_dna_plugin_1:

Updating DNA Plugin
-------------------

::

   dn: changenumber=26,cn=changelog
   objectClass: top
   objectClass: changelogentry
   objectClass: extensibleObject
   changeNumber: 26
   targetDn: dnaHostname=localdc1.samba.example.com+dnaPortNum=0,cn=Samba SIDs,ou
    =Ranges,CN=Samba
   changeTime: 20091208080540Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   replace: dnaRemainingValues
   dnaRemainingValues: 1844674407370955
   -
   replace: modifiersname
   modifiersname: cn=Distributed Numeric Assignment Plugin,cn=plugins,cn=config
   -
   replace: modifytimestamp
   modifytimestamp: 20091208080540Z
   -

.. _adding_group_object:

Adding Group Object
-------------------

::

   dn: changenumber=28,cn=changelog
   objectClass: top
   objectClass: changelogentry
   objectClass: extensibleObject
   changeNumber: 28
   targetDn: CN=Test Group,CN=Users,DC=samba,DC=example,DC=com
   changeTime: 20091208080540Z
   changeType: add
   changes:: ...

The content of the changes attribute is:

::

   cn: Test Group
   member: CN=Test User,CN=Users,DC=samba,DC=example,DC=com
   name: Test Group
   objectClass: top
   objectClass: group
   objectClass: extensibleObject
   objectCategory: CN=Group,CN=Schema,CN=Configuration,DC=samba,DC=example,DC=com
   nTSecurityDescriptor:: ...
   sambaGroupType: -2147483646
   sAMAccountName: $12A614C5-E9463803ADAC2566
   sAMAccountType: 268435456
   instanceType: 4
   creatorsName: cn=samba-admin,cn=samba
   modifiersName: cn=samba-admin,cn=samba
   createTimestamp: 20091208080540Z
   modifyTimestamp: 20091208080540Z
   sambaSID: S-1-5-21-1463069339-4227668456-4007226777-1005

.. _adding_group_member:

Adding Group Member
-------------------

::

   dn: changenumber=27,cn=changelog
   objectClass: top
   objectClass: changelogentry
   objectClass: extensibleObject
   changeNumber: 27
   targetDn: CN=Test User,CN=Users,DC=samba,DC=example,DC=com
   changeTime: 20091208080540Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   add: memberOf
   memberOf: cn=test group,cn=users,dc=samba,dc=example,dc=com
   -
   replace: modifiersname
   modifiersname: cn=Linked Attributes,cn=plugins,cn=config
   -
   replace: modifytimestamp
   modifytimestamp: 20091208080540Z
   -

.. _modifying_samba_group:

Modifying Samba Group
=====================

Consider the following example:

::

   ldapmodify -x -D "cn=Administrator,cn=Users,dc=samba,dc=example,dc=com" -W
   dn: cn=Test Group,cn=Users,dc=samba,dc=example,dc=com
   changetype: modify
   add: member
   member: cn=Test User,cn=Users,dc=samba,dc=example,dc=com

This operation will generate 2 change log records.

.. _modifying_user_object_1:

Modifying User Object
---------------------

::

   dn: changenumber=34,cn=changelog
   objectClass: top
   objectClass: changelogentry
   objectClass: extensibleObject
   changeNumber: 34
   targetDn: CN=Test User,CN=Users,DC=samba,DC=example,DC=com
   changeTime: 20091208083534Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   add: memberOf
   memberOf: cn=test group,cn=users,dc=samba,dc=example,dc=com
   -
   replace: modifiersname
   modifiersname: cn=Linked Attributes,cn=plugins,cn=config
   -
   replace: modifytimestamp
   modifytimestamp: 20091208083534Z
   -

.. _modifying_group_object:

Modifying Group Object
----------------------

::

   dn: changenumber=35,cn=changelog
   objectClass: top
   objectClass: changelogentry
   objectClass: extensibleObject
   changeNumber: 35
   targetDn: cn=Test Group,cn=Users,dc=samba,dc=example,dc=com
   changeTime: 20091208083534Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   add: member
   member: CN=Test User,CN=Users,DC=samba,DC=example,DC=com
   -
   replace: modifiersname
   modifiersname: cn=samba-admin,cn=samba
   -
   replace: modifytimestamp
   modifytimestamp: 20091208083534Z
   -

.. _deleting_samba_group:

Deleting Samba Group
====================

Consider the following example:

::

   % ldapdelete -x -D "cn=Administrator,cn=Users,dc=samba,dc=example,dc=com" -W
   cn=Test Group,cn=Users,dc=samba,dc=example,dc=com

This operation generates 2 change log records.

.. _modifying_user_object_2:

Modifying User Object
---------------------

::

   dn: changenumber=36,cn=changelog
   objectClass: top
   objectClass: changelogentry
   objectClass: extensibleObject
   changeNumber: 36
   targetDn: CN=Test User,CN=Users,DC=samba,DC=example,DC=com
   changeTime: 20091208084310Z
   changeType: modify
   changes:: ...

The content of the changes attribute is:

::

   delete: memberOf
   memberOf: cn=test group,cn=users,dc=samba,dc=example,dc=com
   -
   replace: modifiersname
   modifiersname: cn=Linked Attributes,cn=plugins,cn=config
   -
   replace: modifytimestamp
   modifytimestamp: 20091208084310Z
   -

.. _deleting_group_object:

Deleting Group Object
---------------------

::

   dn: changenumber=37,cn=changelog
   objectClass: top
   objectClass: changelogentry
   objectClass: extensibleObject
   changeNumber: 37
   targetDn: cn=Test Group,cn=Users,dc=samba,dc=example,dc=com
   changeTime: 20091208084310Z
   changeType: delete

.. _adding_samba_host:

Adding Samba Host
=================

.. _modifying_samba_host:

Modifying Samba Host
====================

.. _deleting_samba_host:

Deleting Samba Host
===================

`Category:Obsolete <Category:Obsolete>`__
