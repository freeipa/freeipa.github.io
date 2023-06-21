SUDO_Schema_Design
==================



Alternative SUDO Schema Proposal
================================

Abstract
--------

This page describes a proposal for the LDAP schema objects to support
centrally managed SUDO rules.



Design Considerations
---------------------

The SUDO rules can already be loaded into an LDAP server and managed
centrally. The schema for the solutions is described here:
`1 <http://www.sudo.ws/sudo/sudoers.ldap.man.html>`__. The SUDO client
can use this schema to query central server and download rules. The
schema can be loaded into IPA server too, however, it is disjoint from
other objects IPA manages. The goal of the this document is to define
and alternative schema that can be used by IPA and would be tighter
integrated with other objects IPA manages. This would provide a better
manageability of the access control rules and would reduce a risk of a
mistake or misconfiguration of SUDO. The mail considerations for the
design are:

-  The design should take into the objects that are currently managed by
   IPA:

   -  Users
   -  Groups
   -  Hosts
   -  Host groups
   -  Netgoups

and integrate tightly with those objects providing easy cross
referencing of the entries.

-  The designed schema should be able to express all the core use cases
   including

   -  Defaults
   -  Rules that boil down to Identity A on Host B can execute set C as
      identity D. Where:

      -  Identities A and D can be expressed as a:

         -  User(s)
         -  Group(s)
         -  Netgroup(s)

      -  Host B can be expressed as a:

         -  Host(s)
         -  Host Group(s)
         -  Netgroup(s)

      -  Set C is a set of commands (may be a combination of individual
         commands and groups of commands)

-  The schema should be sufficient to express the rules. It is not
   required to follow the exact same syntax as the SUDO schema provides
   as long as the rules are powerful to express the access control
   policy and a rule can be translated into standard SUDO schema entry
   by the compatibility plugin.
-  The ambiguities of the order of the LDAP entries should be
   considered. The new schema should allow deterministic result
   regardless of the order the records returned.
-  The new schema should allow disabling rules without deleting them.



Proposed Schema
---------------

Sudo rules should be located in some part of the tree. Assume that
**SUDOers** will be used as a name of the container. Then the full DN
will look like this:

``cn=SUDOers,dc=example,dc=com.``

Rules
----------------------------------------------------------------------------------------------

The rule object will be similar to the one defined by the standard SUDO
schema but with additional attributes and will use DNs instead of string
attributes to point to users or groups.

Here is the standard SUDO schema for reference in OpenLDAP format.

| ``attributetype ( 1.3.6.1.4.1.15953.9.1.1``
| ``   NAME 'sudoUser'``
| ``   DESC 'User(s) who may  run sudo'``
| ``   EQUALITY caseExactIA5Match``
| ``   SUBSTR caseExactIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )``
| ``attributetype ( 1.3.6.1.4.1.15953.9.1.2``
| ``   NAME 'sudoHost'``
| ``   DESC 'Host(s) who may run sudo'``
| ``   EQUALITY caseExactIA5Match``
| ``   SUBSTR caseExactIA5SubstringsMatch``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )``
| ``attributetype ( 1.3.6.1.4.1.15953.9.1.3``
| ``   NAME 'sudoCommand'``
| ``   DESC 'Command(s) to be executed by sudo'``
| ``   EQUALITY caseExactIA5Match``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )``
| ``attributetype ( 1.3.6.1.4.1.15953.9.1.4``
| ``   NAME 'sudoRunAs'``
| ``   DESC 'User(s) impersonated by sudo'``
| ``   EQUALITY caseExactIA5Match``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )``
| ``attributetype ( 1.3.6.1.4.1.15953.9.1.5``
| ``   NAME 'sudoOption'``
| ``   DESC 'Options(s) followed by sudo'``
| ``   EQUALITY caseExactIA5Match``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )``
| ``attributetype ( 1.3.6.1.4.1.15953.9.1.6``
| ``   NAME 'sudoRunAsUser'``
| ``   DESC 'User(s) impersonated by sudo'``
| ``   EQUALITY caseExactIA5Match``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )``
| ``attributetype ( 1.3.6.1.4.1.15953.9.1.7``
| ``   NAME 'sudoRunAsGroup'``
| ``   DESC 'Group(s) impersonated by sudo'``
| ``   EQUALITY caseExactIA5Match``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )``
| ``objectclass ( 1.3.6.1.4.1.15953.9.2.1 NAME 'sudoRole' SUP top STRUCTURAL``
| ``   DESC 'Sudoer Entries'``
| ``   MUST ( cn )``
| ``   MAY ( sudoUser $ sudoHost $ sudoCommand $ sudoRunAs $ sudoRunAsUser $``
| ``         sudoRunAsGroup $ sudoOption $ description )``
| ``   )``

Let us look at each of the attributes and its use more closely.

-  **sudoUser** (required per spec)

   -  A user name
   -  uid
   -  Unix group
   -  User netgroup

   As we can see the SUDO user can be easily expressed by the attribute
   that would point to a DN of the existing user, group or netgroup
   object. The combination of the userCategory and memberUser attributes
   well described in the definition of the association object can also
   express special cases that we might want to handle in future. One of
   the examples will be ALL users and another will be External trusted
   users. This will become relevant when we get the domain trusts
   implemented in a later version. For the sake of the current version
   it makes sense to include userCategory attribute but make the
   software not use it. We would not be able to take advantage of the
   userCategory capabilities until either the SUDO client is tough to
   support special values or the SSSD is implemented and an intermediary
   between SUDO client and new server schema discussed here.
   In addition to the users known to IPA for the sake of SUDO rules it
   will be beneficial to allow configuring SUDO rules that apply to the
   external to IPA users. The best examples are standard local users
   like "adm", "oracle", "apache" etc. To allow handling such accounts
   we will introduce a new attribute:

| ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD``
| ``                 NAME 'externalUser' ``
| ``                 DESC 'Multivalue string attribute that allows storing user names.' ``
| ``                 EQUALITY caseIgnoreMatch ``
| ``                 ORDERING caseIgnoreMatch ``
| ``                 SUBSTR caseIgnoreSubstringsMatch ``
| ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 ``
| ``                 X-ORIGIN 'IPA v2' )``

   For the sake of simplicity we will support only accounts specified by
   name and not by uid.

-  **sudoHost** (required per spec)

   -  A host name
   -  IP address
   -  IP network
   -  Host netgroup
   -  ALL will match any host.

   For the host the memberHost and hostCategory attributes can handle
   most of these cases. If memberHost is a DN of a hist, host group, or
   host netgroup we cover half. The value "All" in the hostCategory will
   be equivalent to the special value ALL used by SUDO. There is a need
   to express IP network. I see three different ways of doing it:

   #. Not support it at all - is this an option? The point is that the
      hosts in the same subnet should probably have a group anyways so
      instead of giving IP mask a group or netgroup can be referenced.
   #. Create a special attribute that will hold the value as a text
      string
   #. Use hostCategory attribute which is in some way a special category

   The answer very much depends on the feedback from the community and
   our preference.
   Using a separate attribute would probably be the right thing to do
   just for the sake of the clean design and maintainability.
   The attribute then can look like this:

::

   | ``attributeTypes: ( 2.16.840.1.113730.3.8.7.TBD ``
   | ``                  NAME 'hostMask' ``
   | ``                  DESC 'IP mask to identify a subnet.' ``
   | ``                  EQUALITY caseIgnoreMatch``
   | ``                  SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 ``
   | ``                  ORDERING caseIgnoreMatch ``
   | ``                  SUBSTR caseIgnoreSubstringsMatch ``
   | ``                  X-ORIGIN 'IPA v2' )``

   The values it will hold may look like IPv4 or IPv6 addresses or
   expressed using the CIDR notation for example:

| ``128.138.243.0``
| ``128.138.204.0/24``
| ``128.138.242.0``
| ``ffff:ffff:ffff:ffff::``

   Instead of defining a new attribute we can also consider existing
   attribute **ipNetmaskNumber** but this attribute is defines as single
   value attribute which would create a limitation on specifying
   multiple masks in one entry.

::

   | ``attributeTypes: ( nisSchema.1.21 ``
   | ``                  NAME 'ipNetmaskNumber'``
   | ``                  DESC 'IP netmask as a dotted decimal, eg. 255.255.255.0, omitting leading zeros'``
   | ``                  EQUALITY caseIgnoreIA5Match``
   | ``                  SYNTAX 'IA5String{128}' SINGLE-VALUE )``

   Or we reuse an attribute already defined in the schema for the
   external (unmanaged) hosts. This can also be a good option since we
   need to also support hosts that run SUDO but are not a part of the
   IPA universe and thus must be directly listed in the rule. However in
   this case we would have to have a special prefixing inside the
   attribute value to distinguish the two.

::

   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD``
   | ``                 NAME 'externalHost' ``
   | ``                 DESC 'Multivalue string attribute that allows storing host names.' ``
   | ``                 EQUALITY caseIgnoreMatch ``
   | ``                 ORDERING caseIgnoreMatch ``
   | ``                 SUBSTR caseIgnoreSubstringsMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 ``
   | ``                 X-ORIGIN 'IPA v2' )``

   After a brief evaluation I suggest including both attributes. The
   **externalHost** for external names not otherwise managed by the
   system and the **hostMask** for the net mask or subnet specification
   as define by SUDO syntax. The management plugin should implement
   strict syntax checking rules to make sure that this string has the
   right format and matches the expectation. When synthesizing entries
   the compat plugin will take the value of this attribute verbatim, add
   a prefix and stick into the outgoing attribute. No syntax checking
   will be performed.

-  **sudoCommand** (required per spec)

   According to SODO manual this is: a Unix command with optional
   command line arguments, potentially including globbing characters
   (aka wild cards). The special value ALL will match any command. If a
   command is prefixed with an exclamation point '!', the user will be
   prohibited from running that command.
   There are several important ideas that worth discussing regarding the
   commands in a rule.

-  

   -  We can use the commands in the same way as SUDO uses this
      attribute, however this does not to seem to be the most efficient
      way.
   -  We can create a special object class to store commands and a
      special object class to store groups of commands. This would allow
      defining a set of the commands once, grouping them in a logical
      way and making a rule reference a DN of a group of commands as
      well as individual commands directly if needed. It can be a mixed
      bag of both. It also leads to a more controversial idea of not
      allowing negation of the commands on per command basis but rather
      a negation of the whole rule. Unfortunately this does not work
      since SUDO utility has an issue with matching multiple records.
      The problem is that if there is an allow and deny rule that can be
      matched there is no guarantee which one would come first to the
      client. SUDO does not take this into account and does not check
      deny rules first. To account for this complication we would have
      to allow deny and allow commands in one rule. For this we will
      have two similar attributes. One will be the pointer to the
      commands or groups of commands that are allowed by the rule and
      other attribute will be the pointer to the denied commands or
      groups of commands.

      It seems that if the rules are defined following this paradigm the
      conversion of the proposed schema into a legacy schema via compat
      plugin would still produce a set of rules that old clients will be
      able to deal with. The proposed approach much better structures
      the access control policies for the advantage of the administrator
      (he can easier see who can do what) and future use for the times
      when SUDO is enhanced to offload the decision making to a plugin
      that will be capable of directly or indirectly (most likely via
      SSSD) access the new schema and take advantage of its structure.
      However it does not make much sense to allow nested groups of the
      commands at least originally. The nested group support comes with
      cost. It is not clear if there is or will be a requirement to
      support nested groups of commands in SUDO rules. So for the first
      implementation we will assume that the nested groups support for
      commands is not required.
      Command and command group objects will have ipaUniqueID attribute
      to allow easy changing of the commands or group names to avoid
      costly subtree renames. However we will use the cn too as the UI
      displays the names rather than IDs.
      Category of the commands will be added to denote classes of the
      commands. For the first implementation only "all" will be
      supported. The logic of handling the member command attributes and
      category attributes should be the following:

         If no memberAllowCmd, memberDenyCmd or cmdCategory attribute is
         specified - no command is allowed
         If cmdCategory is specified (the only supported value so far is
         "all")

            The memberAllowCmd is ignored
            If memberDenyCmd is specified it defines commands or groups
            of the commands that are not allowed while all the rest are
            allowed by the category attribute.

         If cmdCategory is not specified

            If memberAllowCmd is specified it defines commands or groups
            of the commands that are allowed
            If memberDenyCmd is specified it defines commands or groups
            of the commands that are not allowed

      The SUDO commands will be stored in the cn=SUDOcmd,dc=...
      container while the sudo groups will be stored in the
      cn=SUDOcmdgrp,dc=... container.

::

   | ``objectClasses: (2.16.840.1.113730.3.8.8.TBD ``
   | ``                NAME 'ipaSudoCmd' ``
   | ``                DESC 'IPA object class for SUDO command'``
   | ``                STRUCTURAL ``
   | ``                MUST ( ipaUniqueID $ sudoCmd ) ``
   | ``                MAY  ( memberOf $ description ) ``
   | ``                X-ORIGIN 'IPA v2' )``
   | ``objectClasses: (2.16.840.1.113730.3.8.8.TBD ``
   | ``                NAME 'ipaSudoCmdGrp' ``
   | ``                DESC 'IPA object class to store groups of SUDO commands' ``
   | ``                SUP groupOfNames ``
   | ``                MUST ( ipaUniqueID )``
   | ``                STRUCTURAL``
   | ``                X-ORIGIN 'IPA v2' )``
   | `` ``
   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD``
   | ``                 NAME 'sudoCmd'``
   | ``                 DESC 'Command(s) to be executed by sudo'``
   | ``                 EQUALITY caseExactMatch ``
   | ``                 ORDERING caseExactMatch ``
   | ``                 SUBSTR caseExactSubstringsMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
   | ``                 X-ORIGIN 'IPA v2' )``

   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD ``
   | ``                 NAME 'memberAllowCmd' ``
   | ``                 DESC 'Reference to a command or group of the commands.' ``
   | ``                 SUP distinguishedName ``
   | ``                 EQUALITY distinguishedNameMatch ``
   | ``                 ORDERING distinguishedNameMatch ``
   | ``                 SUBSTR distinguishedNameMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 ``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD ``
   | ``                 NAME 'memberDenyCmd' ``
   | ``                 DESC 'Reference to a command or group of the commands.' ``
   | ``                 SUP distinguishedName ``
   | ``                 EQUALITY distinguishedNameMatch ``
   | ``                 ORDERING distinguishedNameMatch ``
   | ``                 SUBSTR distinguishedNameMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 ``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD ``
   | ``                 NAME 'cmdCategory' ``
   | ``                 DESC 'Additional classification for commands' ``
   | ``                 EQUALITY caseIgnoreMatch ``
   | ``                 ORDERING caseIgnoreMatch ``
   | ``                 SUBSTR caseIgnoreSubstringsMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 X-ORIGIN 'IPA v2' )``

-  **sudoOption** (optional per spec)

   This attribute is used for two purposes - first to define the default
   options that apply to all SUDO rules and secondarily to be able to
   override specific options in the specific rules. It does not make
   sense to change something in comparison to the standard SUDO schema
   for this attribute so we will define and analogous attribute of the
   same type.

::

   | ``attributetype ( 2.16.840.1.113730.3.8.7.TBD``
   | ``                NAME 'ipaSudoOpt'``
   | ``                DESC 'Options(s) followed by sudo'``
   | ``                EQUALITY caseExactIA5Match``
   | ``                SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )``

-  **sudoRunAs** - is deprecated

-  **sudoRunAsUser** & **sudoRunAsGroup** (optional per spec)

   -  User

      -  A user name or uid that commands may be run as
      -  Unix group that contains a list of users that commands may be
         run as
      -  User netgroup that contains a list of users that commands may
         be run as.
      -  The special value ALL will match any user.

   -  Group (defines the gid of the group the command will be run as)

      -  A Unix group or gid that commands may be run as.
      -  The special value ALL will match any group.

   The run as functionality is very complex requires several multiple
   attributes to do it cleanly. First of all there should be a way to
   point to and existing IPA managed users, groups or netgroups that
   aggregate uses the command can be run as. To point to those objects
   we need a DN style attribute.

::

   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD ``
   | ``                 NAME 'ipaSudoRunAs' ``
   | ``                 DESC 'Reference to a user or group that the commands can be run as.' ``
   | ``                 SUP memberUser``
   | ``                 X-ORIGIN 'IPA v2' )``

   Secondarily we need to allow the sudo commands to be run as users
   that are not managed.

::

   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD ``
   | ``                 NAME 'ipaSudoRunAsExtUser' ``
   | ``                 DESC 'Multivalue string attribute that allows storing user name the command can be run as' ``
   | ``                 EQUALITY caseIgnoreMatch ``
   | ``                 ORDERING caseIgnoreMatch ``
   | ``                 SUBSTR caseIgnoreSubstringsMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 ``
   | ``                 X-ORIGIN 'IPA v2' )``

   We will not support referencing external users by the uid only by
   login name.

   Lastly we need to support an option to run as any user. This can be
   accomplished by using a special value "ALL" in the
   "ipaSudoRunAsExtUser" attribute. The draback of this solution is that
   it potentially creates a naming collision between a local user named
   "all" and this spacial value. it also introduces special processing
   and handling of the attribute.
   Alternatively we can create a special attribute similar to the
   userCategory attribute in the association object to express notion of
   "all" users or all "external users" or "all trusted users" etc.
   Though it is a very corner case and this approach seems a bit an
   overkill it allows a cleaner and consistent logic across the board of
   how we handle user entries in the system as a whole.


::
   
   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD``
   | ``                 NAME 'ipaSudoRunAsUserCategory' ``
   | ``                 DESC 'Additional classification for users' ``
   | ``                 SUP userCategory``
   | ``                 X-ORIGIN 'IPA v2' )``

   The only value that will be supported so far is "all".

   For the run as group we will need to have very similar handling.

::

   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD ``
   | ``                 NAME 'ipaSudoRunAsGroup' ``
   | ``                 DESC 'Reference to group that the commands can be run as.' ``
   | ``                 SUP memberUser``
   | ``                 X-ORIGIN 'IPA v2' )``

   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD ``
   | ``                 NAME 'ipaSudoRunAsExtGroup' ``
   | ``                 DESC 'Multivalue string attribute that allows storing group name the command can be run as' ``
   | ``                 EQUALITY caseIgnoreMatch ``
   | ``                 ORDERING caseIgnoreMatch ``
   | ``                 SUBSTR caseIgnoreSubstringsMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 ``
   | ``                 X-ORIGIN 'IPA v2' )``

   We will not support referencing external groups by the gid only by
   group name.

::

   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD``
   | ``                 NAME 'ipaSudoRunAsGroupCategory' ``
   | ``                 DESC 'Additional classification for groups' ``
   | ``                 SUP userCategory``
   | ``                 X-ORIGIN 'IPA v2' )``



SUDO rules and HBAC rules
----------------------------------------------------------------------------------------------

When a user invokes SUDO he needs to authenticate. On the managed hosts
the SSSD will do the access control enforcement for those
authentications using HBAC rules. If the authentication is not allowed
the SUDO command will fail with the authentication error. This need to
be avoided. Different proposals have been considered. Some were colling
for some kind of the automatic (using a DS managed entry plugin) or less
automatic (using a special management plugin) solution. Both of these
solutions might cause many unwanted HBAC entries to be created in the
system significantly reducing its manageability. After a thorough
evaluation we came to the conclusion that the best approach would be to
add several special preloaded entries that will help to over come the
SUDO authentication problem.

-  First we will create a special service group named "SUDO".
-  This service group will have two services "sudo" and "sudo-i"
-  We will add a disabled allow HBAC rule for all users and on all hosts
   referencing this service group. It will be to administrator to enable
   it if he is planning to manage SUDO with IPA. Alternatively the
   administrator will be able to add other more granular access rules at
   his discretion.

The pre configured data template will look like this:

::

   | ``dn: cn=SUDO,cn=hbacservicegroups,cn=accounts,$SUFFIX``
   | ``changetype: add``
   | ``objectClass: ipaobject``
   | ``objectClass: ipahbacservicegroup``
   | ``objectClass: nestedGroup``
   | ``objectClass: groupOfNames``
   | ``objectClass: top``
   | ``cn: SUDO``
   | ``description: Default group of SUDO related services``
   | ``dn: cn=sudo,cn=hbacservices,cn=accounts,$SUFFIX``
   | ``changetype: add``
   | ``objectClass: ipaobject``
   | ``objectClass: ipahbacservice``
   | ``cn: sudo``
   | ``memberOf:'cn=SUDO,cn=hbacservicegroups,cn=accounts,$SUFFIX'``
   | ``description: Login service for sudo``
   | ``dn: cn=sudo-i,cn=hbacservices,cn=accounts,$SUFFIX``
   | ``changetype: add``
   | ``objectClass: ipaobject``
   | ``objectClass: ipahbacservice``
   | ``cn: sudo-i``
   | ``memberOf:'cn=SUDO,cn=hbacservicegroups,cn=accounts,$SUFFIX'``
   | ``description: Login service for sudo-i``

   | ``dn: cn=SUDO Login,cn=hbac,cn=accounts,$SUFFIX``
   | ``changetype add``
   | ``objectClass: top``
   | ``objectClass: ipaAssociation``
   | ``objectClass: ipaHBACRule``
   | ``cn: SUDO Login``
   | ``description: Default HBAC rule to allow authentication via SUDO commands.``
   | ``ipaEnabledFlag: false``
   | ``accessRuleType: allow``
   | ``userCategory: all``
   | ``hostCategory: all``
   | ``sourceHostCategory: all``
   | ``memberService: 'cn=SUDO,cn=hbacservicegroups,cn=accounts,$SUFFIX'``

If we realize that we need a more tight coupling between the SUDO and
HBAC rules we will implement them later based on the feedback from the
community.

Defaults
----------------------------------------------------------------------------------------------

As in the standard SUDO schema the "default" options will be represented
by the same rule object but with a special name: cn=defaults. This
allows to maintain consistency in the lookups between old and new
schema.

Summary
----------------------------------------------------------------------------------------------

To summarize the schema for the new SUDO rule object will look like
this:

Existing objects already defined in the IPA schema:

::

   | ``attributeTypes: (2.16.840.1.113730.3.8.3.1 ``
   | ``                 NAME 'ipaUniqueID' ``
   | ``                 DESC 'Unique identifier' ``
   | ``                 EQUALITY caseIgnoreMatch ``
   | ``                 ORDERING caseIgnoreMatch ``
   | ``                 SUBSTR caseIgnoreSubstringsMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 ``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.3.5 ``
   | ``                 NAME 'memberUser' ``
   | ``                 DESC 'Reference to a principal that performs an action (usually user).' ``
   | ``                 SUP distinguishedName ``
   | ``                 EQUALITY distinguishedNameMatch ``
   | ``                 ORDERING distinguishedNameMatch ``
   | ``                 SUBSTR distinguishedNameMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 ``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.3.6 ``
   | ``                 NAME 'userCategory' ``
   | ``                 DESC 'Additional classification for users' ``
   | ``                 EQUALITY caseIgnoreMatch ``
   | ``                 ORDERING caseIgnoreMatch ``
   | ``                 SUBSTR caseIgnoreSubstringsMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 ``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.3.7``
   | ``                 NAME 'memberHost' ``
   | ``                 DESC 'Reference to a device where the operation takes place (usually host).' ``
   | ``                 SUP distinguishedName ``
   | ``                 EQUALITY distinguishedNameMatch ``
   | ``                 ORDERING distinguishedNameMatch ``
   | ``                 SUBSTR distinguishedNameMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.3.8 ``
   | ``                 NAME 'hostCategory' ``
   | ``                 DESC 'Additional classification for hosts' ``
   | ``                 EQUALITY caseIgnoreMatch ``
   | ``                 ORDERING caseIgnoreMatch ``
   | ``                 SUBSTR caseIgnoreSubstringsMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 ``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.3.9``
   | ``                 NAME 'ipaEnabledFlag' ``
   | ``                 DESC 'The flag to show if the association is active or should be ignored' ``
   | ``                 EQUALITY booleanMatch ``
   | ``                 ORDERING booleanMatch ``
   | ``                 SUBSTR booleanMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.7 SINGLE-VALUE ``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``objectClasses: (2.16.840.1.113730.3.8.4.6 ``
   | ``                NAME 'ipaAssociation' ``
   | ``                ABSTRACT ``
   | ``                MUST ( ipaUniqueID    $ cn ) ``
   | ``                MAY  ( memberUser     $ userCategory $ ``
   | ``                       memberHost     $ hostCategory $ ``
   | ``                       ipaEnabledFlag $ description ) ``
   | ``                X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.3.11``
   | ``                 NAME 'externalHost' ``
   | ``                 DESC 'Multivalue string attribute that allows storing host names.' ``
   | ``                 EQUALITY caseIgnoreMatch ``
   | ``                 ORDERING caseIgnoreMatch ``
   | ``                 SUBSTR caseIgnoreSubstringsMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 ``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``objectClasses: (2.16.840.1.113730.3.8.4.3 ``
   | ``                NAME 'nestedGroup' ``
   | ``                DESC 'Group that supports nesting' ``
   | ``                SUP groupOfNames ``
   | ``                STRUCTURAL ``
   | ``                MAY memberOf ``
   | ``                X-ORIGIN 'IPA v2' )``
   | ``attributeTypes ( 2.16.840.1.113730.3.8.3.13 ``
   | ``                 NAME 'accessRuleType' ``
   | ``                 DESC 'The flag to represent if it is allow or deny rule.' ``
   | ``                 EQUALITY caseIgnoreMatch ``
   | ``                 ORDERING caseIgnoreMatch ``
   | ``                 SUBSTR caseIgnoreSubstringsMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
   | ``                 X-ORIGIN 'IPA v2')``
   | ``Note: valid values for accessRuleType are "allow" or "deny"``

New attributes and objects added by this design:

::

   | ``objectClasses: (2.16.840.1.113730.3.8.8.TBD ``
   | ``                NAME 'ipaSudoCmd' ``
   | ``                DESC 'IPA object class for SUDO command'``
   | ``                STRUCTURAL ``
   | ``                MUST ( ipaUniqueID $ sudoCmd ) ``
   | ``                MAY  ( memberOf $ description ) ``
   | ``                X-ORIGIN 'IPA v2' )``
   | ``objectClasses: (2.16.840.1.113730.3.8.8.TBD ``
   | ``                NAME 'ipaSudoCmdGrp' ``
   | ``                DESC 'IPA object class to store groups of SUDO commands' ``
   | ``                SUP groupOfNames ``
   | ``                MUST ( ipaUniqueID )``
   | ``                STRUCTURAL``
   | ``                X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD``
   | ``                 NAME 'sudoCmd'``
   | ``                 DESC 'Command(s) to be executed by sudo'``
   | ``                 EQUALITY caseExactMatch ``
   | ``                 ORDERING caseExactMatch ``
   | ``                 SUBSTR caseExactSubstringsMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD ``
   | ``                 NAME 'memberAllowCmd' ``
   | ``                 DESC 'Reference to a command or group of the commands that are allowed by the rule.' ``
   | ``                 SUP distinguishedName ``
   | ``                 EQUALITY distinguishedNameMatch ``
   | ``                 ORDERING distinguishedNameMatch ``
   | ``                 SUBSTR distinguishedNameMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 ``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD ``
   | ``                 NAME 'memberDenyCmd' ``
   | ``                 DESC 'Reference to a command or group of the commands that are denied by the rule.' ``
   | ``                 SUP distinguishedName ``
   | ``                 EQUALITY distinguishedNameMatch ``
   | ``                 ORDERING distinguishedNameMatch ``
   | ``                 SUBSTR distinguishedNameMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 ``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD ``
   | ``                 NAME 'cmdCategory' ``
   | ``                 DESC 'Additional classification for commands' ``
   | ``                 EQUALITY caseIgnoreMatch ``
   | ``                 ORDERING caseIgnoreMatch ``
   | ``                 SUBSTR caseIgnoreSubstringsMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 X-ORIGIN 'IPA v2' )``
   | ``attributetypes: (2.16.840.1.113730.3.8.7.TBD``
   | ``                 NAME 'externalUser' ``
   | ``                 DESC 'Multivalue string attribute that allows storing user names.' ``
   | ``                 EQUALITY caseIgnoreMatch ``
   | ``                 ORDERING caseIgnoreMatch ``
   | ``                 SUBSTR caseIgnoreSubstringsMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 ``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``attributetypes: (2.16.840.1.113730.3.8.7.TBD``
   | ``                 NAME 'ipaSudoOpt'``
   | ``                 DESC 'Options(s) followed by sudo'``
   | ``                 EQUALITY caseExactIA5Match``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD ``
   | ``                 NAME 'ipaSudoRunAs' ``
   | ``                 DESC 'Reference to a user or group that the commands can be run as.' ``
   | ``                 SUP memberUser``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD ``
   | ``                 NAME 'ipaSudoRunAsExtUser' ``
   | ``                 DESC 'Multivalue string attribute that allows storing user name the command can be run as' ``
   | ``                 EQUALITY caseIgnoreMatch ``
   | ``                 ORDERING caseIgnoreMatch ``
   | ``                 SUBSTR caseIgnoreSubstringsMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 ``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD``
   | ``                 NAME 'ipaSudoRunAsUserCategory' ``
   | ``                 DESC 'Additional classification for users' ``
   | ``                 SUP userCategory``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD ``
   | ``                 NAME 'ipaSudoRunAsGroup' ``
   | ``                 DESC 'Reference to group that the commands can be run as.' ``
   | ``                 SUP memberUser``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD ``
   | ``                 NAME 'ipaSudoRunAsExtGroup' ``
   | ``                 DESC 'Multivalue string attribute that allows storing group name the command can be run as' ``
   | ``                 EQUALITY caseIgnoreMatch ``
   | ``                 ORDERING caseIgnoreMatch ``
   | ``                 SUBSTR caseIgnoreSubstringsMatch ``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 ``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD``
   | ``                 NAME 'ipaSudoRunAsGroupCategory' ``
   | ``                 DESC 'Additional classification for groups' ``
   | ``                 SUP userCategory``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``attributeTypes: (2.16.840.1.113730.3.8.7.TBD ``
   | ``                 NAME 'hostMask' ``
   | ``                 DESC 'IP mask to identify a subnet.' ``
   | ``                 EQUALITY caseIgnoreMatch``
   | ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 ``
   | ``                 ORDERING caseIgnoreMatch ``
   | ``                 SUBSTR caseIgnoreSubstringsMatch ``
   | ``                 X-ORIGIN 'IPA v2' )``
   | ``objectClasses: (2.16.840.1.113730.3.8.8.TBD ``
   | ``                NAME 'ipaSudoRule' ``
   | ``                SUP ipaAssociation ``
   | ``                STRUCTURAL ``
   | ``                MAY ( externalUser $ ``
   | ``                      externalHost $ hostMask $ ``
   | ``                      memberAllowCmd $ memberDenyCmd $ cmdCategory $``
   | ``                      ipaSudoOpt $``
   | ``                      ipaSudoRunAs $ ipaSudoRunAsExtUser $ ipaSudoRunAsUserCategory $``
   | ``                      ipaSudoRunAsGroup $ ipaSudoRunAsExtGroup $ ipaSudoRunAsGroupCategory ) ``
   | ``                X-ORIGIN 'IPA v2' )``

Examples
----------------------------------------------------------------------------------------------

Default rule

| `` dn: ipaUniqueID=d4453480-cc53-11dd-ad8b-0800200c9a66,cn=SUDOers...``
| `` objectclass: top``
| `` objectclass: ipaAssociation``
| `` objectclass: ipaSudoRule``
| `` ipaUniqueID: d4453480-cc53-11dd-ad8b-0800200c9a66``
| `` cn: defaults``
| `` ipaSudoOpt: env_keep+=SSH_AUTH_SOCK``
| `` ipaSudoOpt: ...``
| `` ipaSudoOpt: ...``
| `` compatVisible: true``

A rule that denies specified users on the given machines to run su
command as a local root on centrally managed "superuser" account.

| `` dn: ipaUniqueID=d4453480-cc53-11dd-ad8b-0800200c9a66,cn=SUDOers...``
| `` objectclass: top``
| `` objectclass: ipaAssociation``
| `` objectclass: ipaSudoRule``
| `` ipaUniqueID: d4453480-cc53-11dd-ad8b-0800200c9a66``
| `` cn: defaults for virtual lab``
| `` compatVisible: true``
| `` memberHost: cn=VirtGuests,cn=hostgroups,cn=accounts,...``
| `` memberHost: fqdn=myhost.lab.com,cn=computers,cn=accounts,...``
| `` externalHost: lobby.workstation.external.com  ``
| `` hostMask: 128.138.204.0/24``
| `` memberUser: cn=sss,cn=users,cn=accounts,...``
| `` memberUser: cn=dpal,cn=users,cn=accounts,...``
| `` memberUser: cn=Engineering,cn=groups,cn=accounts,...``
| `` memberDenyCmd: f4453480-cc53-11dd-ad8b-0abc200c9a67,cn=SUDOcmd...``
| `` ipaSudoRunAsExtUser: root``
| `` ipaSudoRunAs: cn=superuser,cn=users,cn=accounts,...``
| `` dn: ipaUniqueID=f4453480-cc53-11dd-ad8b-0abc200c9a67,cn=SUDOcmd...``
| `` objectclass: top``
| `` objectclass: ipaSudoCmd``
| `` ipaUniqueID: f4453480-cc53-11dd-ad8b-0abc200c9a67``
| `` sudoCmd: /bin/su``



Why we must support netgroups in the SUDO rules?
----------------------------------------------------------------------------------------------

Current SUDO client when needs to evaluate whether user is allowed to
execute the command or not works the following way:

-  It downloads all LDAP rules that are applicable to this current user
-  Filters out the rules that do not apply to the host
-  Filters out the rules that do not apply to the command in question.

For the sake of the argument we are interested in step 2). The
deployments that centrally manage SUDO via LDAP do not put individual
hosts into each SUDO rule. Instead they create a netgroup consisting of
only hosts and reference it in the SUDO rule. Putting individual hosts
into the SUDO rules will be unmanageable. We can't do anything with the
SUDO client on all the platforms the customers are using SUDO on. We can
eventually solve it for Linux but not for existing Solaris, HP-UX, AIX
and other boxes. So the IPA server should be capable of:

-  Serving SUDO rules in the standard SUDO format since the client
   expects it this way (will be done via the compat plugin)
-  Allowing SUDO rules to have a netgroup name as a value in the
   synthesized sudoHost attribute
-  Serving netgroup information in the standard netgroup format defined
   by RFC 2307 (already done by the compat plugin)

Our original plan was to allow SUDO rule to point to the netgroup DN via
memberHost attribute. However later we realized that for easier
migration and compatibility it would be better to create a managed
netgroup entry for every host group automatically. So now we decided not
allow SUDO rule to point to the netgroup directly. Instead the compat
plugin will detect that whether host group has a shadow netgroup entry.
If it does it will use its name in the synthesized compatible SUDO rule,
otherwise it will expand the host group and stick member hosts directly
into the entry. The IPA server will automatically create netgoups for
host groups for years to come until the need need for the netgroups is
completely eliminated and admins would be able to turn it off. By that
time there will be no more need for the SUDO compat configuration.



Open questions
----------------------------------------------------------------------------------------------

-  Is it Ok to not allow specifying external users and groups by uid and
   gid?

   **Current plan is to not allow specifying users by uid and gid.**

-  Can we not support netgroups with memberUser attribute?

   **We will not support netgroups for users.**

-  What should we do about hostMask? Can we defer it?

   **We will defer it at least in the UI.**