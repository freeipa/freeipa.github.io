Overview
--------

Allow Active Directory users to use FreeIPA command line interface (CLI)
to manage FreeIPA resources



Use Cases
---------

A forest trust is established between FreeIPA and Active Directory, most
of the users and groups are defined in Active Directory. Allow Active
Directory users to gain access to IPA CLI and manage resources defined
in FreeIPA with the help of IPA CLI.

Design
------

FreeIPA management framework authenticates users with Kerberos or user
name/password pair. For the latter case it obtains a ticket using the
user name and password and further uses Kerberos authentication against
other FreeIPA resources. Primary resource accessed with Kerberos
credentials by FreeIPA management framework is LDAP server. Kerberos
credentials aren't forwarded to FreeIPA management framework; instead,
it uses Kerberos S4U2Proxy mechanism to request Kerberos ticket to LDAP
server on behalf of the user.

-  FreeIPA management framework uses Kerberos credentials to
   authenticate to LDAP server.
-  FreeIPA LDAP server uses SASL mapping configuration to translate
   successfully authenticated Kerberos principal to an object in LDAP
-  Object in LDAP that corresponds to the Kerberos principal as result
   of SASL mapping process is then subjected to LDAP ACIs when
   identifying actual access rights to specific attributes of LDAP
   objects accessed.

Thus, in order to allow Active Directory users to authenticate to
FreeIPA framework and to FreeIPA LDAP server, one needs to make sure
Active Directory users are mapped to \*some\* LDAP objects that can be
used to define access in LDAP ACIs.

FreeIPA provides a mechanism called \*ID Overrides\* to associate
certain POSIX attributes with Active Directory users. ID override for
the user is a normal LDAP object which can be used to map a Kerberos
identity of an Active Directory user to itself. This LDAP object then
can be added as a member to other FreeIPA groups that define permissions
and roles in the FreeIPA framework.

Implementation
--------------

FreeIPA LDAP server supports multiple SASL maps. Each map is supposed to
produce a unique match between SASL identity and an LDAP object. If
there is a match, a matched LDAP object's DN is used as a resulted DN of
LDAP BIND operation.

FreeIPA ID Views support multiple views for overriding Active Directory
users and groups. Since a unique match is required, a 'Default Trust
View' ID view is used to provide a mapping. This is implemented with a
SASL mapping introduced in `this git
commit <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=b506fd178edbf1553ca581c44ac6697f88ead125>`__.
SASL mapping is not replicated because it is part of **cn=config** tree
in LDAP, thus every single FreeIPA master needs to be updated in order
to enable access by Active Directory users over LDAP.

ID override for an Active Directory user in 'Default Trust View' can be
added to any LDAP group to allow it to be a member of that group.
However, FreeIPA includes a special plugin, called "Member Of plugin"
that propagates information about membership back to the LDAP object
which became a member of a group in LDAP. In order to do so, object
classes associated with the LDAP object must allow 'memberof' attribute.
If 'memberof' attribute is not allowed, the operation to add ID override
as a member of any LDAP group in FreeIPA LDAP server will fail. Thus, ID
override entry needs to allow 'memberof' attribute.

An easiest way to allow 'memberof' attribute is to allow it as part of
ipaUserOverride class:

``objectClasses: (2.16.840.1.113730.3.8.12.31 NAME 'ipaUserOverride' DESC 'Override for User Attributes' SUP ipaOverrideAnchor STRUCTURAL MAY ( uid $ uidNumber $ gidNumber $ homeDirectory $ loginShell $ gecos $ ipaOriginalUid $ userCertificate $ memberOf) X-ORIGIN 'IPA v4' )``

A change like this requires schema update.

Finally, **ipa group-add-member** command should be extended to allow
specifying Active Directory users in **--users** option using fully
qualified user name syntax. Following logic should be used as soon as
'@' character is detected in **--users** option value:

-  try to resolve the user to SID
-  if succeeded, see if ID override for this user exists in Default
   trust view
-  if no ID override exist, add it to Default trust view (no attributes
   to override is just fine)
-  if succeeded, add DN of the override to the group membership list



Feature Management
------------------

UI
~~

Currently the feature is aimed for use in CLI in the first stage.

CLI
~~~

Overview of the CLI commands. The only change is to modify existing
group management commands to allow specifying trusted domain users
through fully qualified names:

+---------------------+----------------------+----------------------+
| Command             | Options              | Comment              |
+=====================+======================+======================+
| group-add-member    | --users=adm          | Add trusted domain   |
|                     | inistrator@ad.domain | user to a group. If  |
|                     |                      | ID override for this |
|                     |                      | user is missing, add |
|                     |                      | it to 'Default Trust |
|                     |                      | View'                |
+---------------------+----------------------+----------------------+
| group-remove-member | --users=adm          | Remove trusted       |
|                     | inistrator@ad.domain | domain user to from  |
|                     |                      | a group.             |
+---------------------+----------------------+----------------------+
| group-show          |                      | Show members that    |
|                     |                      | are trusted domain   |
|                     |                      | users                |
+---------------------+----------------------+----------------------+

Configuration
~~~~~~~~~~~~~

Upgrade
-------

Any impact on upgrades? Remove this section if not applicable.



How to Use
----------

Easy to follow instructions how to use the new feature according to the
`use cases <#Use_Cases>`__ described above. FreeIPA user needs to be
able to follow the steps and demonstrate the new features.

The chapter may be divided in sub-sections per `Use
Case <#Use_Cases>`__.



Test Plan
---------

Test scenarios that will be transformed to test cases for FreeIPA
`Continuous Integration <V3/Integration_testing>`__ during
implementation or review phase. This can be also link to `source in
cgit <https://git.fedorahosted.org/cgit/freeipa.git/>`__ with the test,
if appropriate.
