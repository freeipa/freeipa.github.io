Samba_3_Schema
==============



Samba Schema
============

This schema is defined in /etc/dirsrv/schema/60samba.ldif.



Attribute Types
---------------

================ ===========================================
Name             Description
================ ===========================================
lmPassword       LanManager Password
ntPassword       NT Password
acctFlags        Account Flags
pwdLastSet       NT Password Last Set
logonTime        NT Logon Time
logoffTime       NT Logoff Time
kickoffTime      NT Kickoff Time
pwdCanChange     NT Password Can Change
pwdMustChange    NT Password Must Change
homeDrive        NT Home Drive
scriptPath       NT Script Path
profilePath      NT Profile Path
userWorkstations User Workstations
smbHome          Samba Home
domain           Windows NT domain to which the user belongs
rid              NT RID
primaryGroupID   NT Primary Group ID
================ ===========================================



Object Classes
--------------

============ =======================
Name         Description
============ =======================
sambaAccount Samba Auxiliary Account
============ =======================



Samba 3 Schema
==============

This schema is defined in /etc/dirsrv/schema/60samba3.ldif.



Attribute Types
---------------

+-------------------------+-------------------------------------------+
| Name                    | Description                               |
+=========================+===========================================+
| sambaLMPassword         | LanManager Password                       |
+-------------------------+-------------------------------------------+
| sambaNTPassword         | MD4 hash of the unicode password          |
+-------------------------+-------------------------------------------+
| sambaAcctFlags          | Account Flags                             |
+-------------------------+-------------------------------------------+
| sambaPwdLastSet         | Timestamp of the last password update     |
+-------------------------+-------------------------------------------+
| sambaPwdCanChange       | Timestamp of when the user is allowed to  |
|                         | update the password                       |
+-------------------------+-------------------------------------------+
| sambaPwdMustChange      | Timestamp of when the password will       |
|                         | expire                                    |
+-------------------------+-------------------------------------------+
| sambaLogonTime          | Timestamp of last logon                   |
+-------------------------+-------------------------------------------+
| sambaLogoffTime         | Timestamp of last logoff                  |
+-------------------------+-------------------------------------------+
| sambaKickoffTime        | Timestamp of when the user will be logged |
|                         | off automatically                         |
+-------------------------+-------------------------------------------+
| sambaBadPasswordCount   | Bad password attempt count                |
+-------------------------+-------------------------------------------+
| sambaBadPasswordTime    | Time of the last bad password attempt     |
+-------------------------+-------------------------------------------+
| sambaLogonHours         | Logon Hours                               |
+-------------------------+-------------------------------------------+
| sambaHomeDrive          | Driver letter of home directory mapping   |
+-------------------------+-------------------------------------------+
| sambaLogonScript        | Logon script path                         |
+-------------------------+-------------------------------------------+
| sambaProfilePath        | Roaming profile path                      |
+-------------------------+-------------------------------------------+
| sambaUserWorkstations   | List of user workstations the user is     |
|                         | allowed to logon to                       |
+-------------------------+-------------------------------------------+
| sambaHomePath           | Home directory UNC path                   |
+-------------------------+-------------------------------------------+
| sambaDomainName         | Windows NT domain to which the user       |
|                         | belongs                                   |
+-------------------------+-------------------------------------------+
| sambaMungedDial         |                                           |
+-------------------------+-------------------------------------------+
| sambaPasswordHistory    | Concatenated MD4 hashes of the unicode    |
|                         | passwords used on this account            |
+-------------------------+-------------------------------------------+
| sambaSID                | Security ID                               |
+-------------------------+-------------------------------------------+
| sambaPrimaryGroupSID    | Primary Group Security ID                 |
+-------------------------+-------------------------------------------+
| sambaSIDList            | Security ID List                          |
+-------------------------+-------------------------------------------+
| sambaGroupType          | NT Group Type                             |
+-------------------------+-------------------------------------------+
| sambaNextUserRid        | Next NT rid to give our for users         |
+-------------------------+-------------------------------------------+
| sambaNextGroupRid       | Next NT rid to give out for groups        |
+-------------------------+-------------------------------------------+
| sambaNextRid            | Next NT rid to give out for anything      |
+-------------------------+-------------------------------------------+
| sambaAlgorithmicRidBase | Base at which the samba RID generation    |
|                         | algorithm should operate                  |
+-------------------------+-------------------------------------------+
| sambaShareName          | Share Name                                |
+-------------------------+-------------------------------------------+
| sambaOptionName         | Option Name                               |
+-------------------------+-------------------------------------------+
| sambaBoolOption         | A boolean option                          |
+-------------------------+-------------------------------------------+
| sambaIntegerOption      | An integer option                         |
+-------------------------+-------------------------------------------+
| sambaStringOption       | A string option                           |
+-------------------------+-------------------------------------------+
| sambaStringListOption   | A string list option                      |
+-------------------------+-------------------------------------------+
| sambaPrivName           |                                           |
+-------------------------+-------------------------------------------+
| sambaPrivilegeList      | Privileges List                           |
+-------------------------+-------------------------------------------+
| sambaTrustFlags         | Trust Password Flags                      |
+-------------------------+-------------------------------------------+



Object Classes
--------------

================== ==================================
Name               Description
================== ==================================
sambaSamAccount    Samba 3.0 Auxilary SAM Account
sambaGroupMapping  Samba Group Mapping
sambaTrustPassword Samba Trust Password
sambaDomain        Samba Domain Information
sambaUnixIdPool    Pool for allocating UNIX uids/gids
sambaIdmapEntry    Mapping from a SID to an ID
sambaSidEntry      Structural Class for a SID
sambaConfig        Samba Configuration Section
sambaShare         Samba Share Section
sambaConfigOption  Samba Configuration Option
sambaPrivilege     Samba Privilege
================== ==================================

`Category:Obsolete <Category:Obsolete>`__