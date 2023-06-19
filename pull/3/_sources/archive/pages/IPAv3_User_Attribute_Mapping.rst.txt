Overview
========

This document describes the user attribute mapping from IPA to Samba and
vice versa in various scenarios.

.. _mapping_ipa_user_to_samba_user:

Mapping IPA User to Samba User
==============================

.. _ipa_user_doesnt_exist_in_samba:

IPA User Doesn't Exist in Samba
-------------------------------

A new user should be generated from IPA attributes and added to Samba:

================== =============================================
Samba              IPA
================== =============================================
dn                 CN=,CN=Users,DC=domain1,DC=com
objectClass        user, person, organizationalPerson
cn                 cn
sn                 sn
sAMAccountName     uid
homeDirectory      homeDirectory
accountExpires     convert krbPasswordExpiration to AD timestamp
pwdLastSet         convert krbLastPwdChange to AD timestamp
userAccountControl 512
================== =============================================

The IPA user should be updated with Samba attributes:

================== ================
IPA                Samba
================== ================
objectClass        extensibleObject
objectGUID         objectGUID
objectSid          objectSid
lastLogon          lastLogon
scriptPath         scriptPath
profilePath        profilePath
logonCount         logonCount
badPwdCount        badPwdCount
primaryGroupID     primaryGroupID
userAccountControl 512
================== ================

.. _ipa_user_exists_in_samba_but_not_linked:

IPA User Exists in Samba but Not Linked
---------------------------------------

The Samba user should be updated with IPA attributes:

============== =============================================
Samba          IPA
============== =============================================
sn             sn
homeDirectory  homeDirectory
accountExpires convert krbPasswordExpiration to AD timestamp
pwdLastSet     convert krbLastPwdChange to AD timestamp
============== =============================================

The IPA user should be updated with Samba attributes:

================== ==================
IPA                Samba
================== ==================
objectClass        extensibleObject
objectGUID         objectGUID
objectSid          objectSid
lastLogon          lastLogon
scriptPath         scriptPath
profilePath        profilePath
logonCount         logonCount
badPwdCount        badPwdCount
primaryGroupID     primaryGroupID
userAccountControl userAccountControl
================== ==================

.. _ipa_user_exists_in_samba_and_linked:

IPA User Exists in Samba and Linked
-----------------------------------

The Samba user should be updated with IPA attributes:

================== =============================================
Samba              IPA
================== =============================================
sn                 sn
homeDirectory      homeDirectory
accountExpires     convert krbPasswordExpiration to AD timestamp
pwdLastSet         convert krbLastPwdChange to AD timestamp
lastLogon          lastLogon
scriptPath         scriptPath
profilePath        profilePath
logonCount         logonCount
badPwdCount        badPwdCount
primaryGroupID     primaryGroupID
userAccountControl userAccountControl
================== =============================================

.. _mapping_samba_user_to_ipa_user:

Mapping Samba User to IPA User
==============================

.. _samba_user_doesnt_exist_in_ipa:

Samba User Doesn't Exist in IPA
-------------------------------

A new user is generated from Samba attributes and added to IPA:

+-----------------------+---------------------------------------------+
| IPA                   | Samba                                       |
+=======================+=============================================+
| dn                    | uid=,cn=users,cn=accounts,dc=domain1,dc=com |
+-----------------------+---------------------------------------------+
| objectClass           | inetOrgPerson, inetUser, krbPrincipalAux,   |
|                       | organizationalPerson, person, posixAccount, |
|                       | radiusProfile, extensibleObject             |
+-----------------------+---------------------------------------------+
| cn                    | cn                                          |
+-----------------------+---------------------------------------------+
| sn                    | sn or last word of cn                       |
+-----------------------+---------------------------------------------+
| uid                   | sAMAccountName                              |
+-----------------------+---------------------------------------------+
| homeDirectory         | homeDirectory or /tmp                       |
+-----------------------+---------------------------------------------+
| gidNumber             | 0                                           |
+-----------------------+---------------------------------------------+
| krbPrincipalName      | @domain1.com                                |
+-----------------------+---------------------------------------------+
| krbPasswordExpiration | convert accountExpires to IPA timestamp     |
+-----------------------+---------------------------------------------+
| krbLastPwdChange      | convert pwdLastSet to IPA timestamp         |
+-----------------------+---------------------------------------------+
| objectGUID            | objectGUID                                  |
+-----------------------+---------------------------------------------+
| objectSid             | objectSid                                   |
+-----------------------+---------------------------------------------+
| lastLogon             | lastLogon                                   |
+-----------------------+---------------------------------------------+
| scriptPath            | scriptPath                                  |
+-----------------------+---------------------------------------------+
| profilePath           | profilePath                                 |
+-----------------------+---------------------------------------------+
| logonCount            | logonCount                                  |
+-----------------------+---------------------------------------------+
| badPwdCount           | badPwdCount                                 |
+-----------------------+---------------------------------------------+
| primaryGroupID        | primaryGroupID                              |
+-----------------------+---------------------------------------------+
| userAccountControl    | userAccountControl                          |
+-----------------------+---------------------------------------------+

.. _samba_user_exists_in_ipa_but_not_linked:

Samba User Exists in IPA but Not Linked
---------------------------------------

A new user is generated from Samba attributes and added to IPA:

===================== =======================================
IPA                   Samba
===================== =======================================
objectClass           extensibleObject
krbPasswordExpiration convert accountExpires to IPA timestamp
krbLastPwdChange      convert pwdLastSet to IPA timestamp
objectGUID            objectGUID
objectSid             objectSid
lastLogon             lastLogon
scriptPath            scriptPath
profilePath           profilePath
logonCount            logonCount
badPwdCount           badPwdCount
primaryGroupID        primaryGroupID
userAccountControl    userAccountControl
===================== =======================================

The Samba user will be updated with IPA attributes:

============= =============
Samba         IPA
============= =============
cn            cn
sn            sn
homeDirectory homeDirectory
============= =============

.. _samba_user_exists_in_ipa_and_linked:

Samba User Exists in IPA and Linked
-----------------------------------

A new user is generated from Samba attributes and added to IPA:

===================== =======================================
IPA                   Samba
===================== =======================================
cn                    cn
sn                    sn or last word of cn
homeDirectory         homeDirectory or /tmp
krbPasswordExpiration convert accountExpires to IPA timestamp
krbLastPwdChange      convert pwdLastSet to IPA timestamp
lastLogon             lastLogon
scriptPath            scriptPath
profilePath           profilePath
logonCount            logonCount
badPwdCount           badPwdCount
primaryGroupID        primaryGroupID
userAccountControl    userAccountControl
===================== =======================================

`Category:Obsolete <Category:Obsolete>`__
