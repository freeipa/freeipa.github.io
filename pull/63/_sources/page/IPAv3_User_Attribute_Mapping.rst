IPAv3_User_Attribute_Mapping
============================

Overview
========

This document describes the user attribute mapping from IPA to Samba and
vice versa in various scenarios.



Mapping IPA User to Samba User
==============================



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



Mapping Samba User to IPA User
==============================



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