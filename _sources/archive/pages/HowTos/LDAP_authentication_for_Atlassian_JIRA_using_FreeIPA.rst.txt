LDAP_authentication_for_Atlassian_JIRA_using_FreeIPA
====================================================

Draft / Work in Progress on formatting

Introduction
------------

This page describes how to integrate JIRA to a FreeIPA LDAP server. Very
likely the same approach can be taken to integrate other Atlassian
products such as Confluence to FreeIPA.

JIRA offers a number of different ways to integrate to LDAP.
Unfortunately FreeIPA integration is not natively supported, but once
you know how, it is not that difficult. There are probably other
solutions that work as well, but this is the one that worked for us.

At the time of integration we were running FreeIPA Server 4.1.0 and JIRA
6.4.7

See:

https://confluence.atlassian.com/display/JIRA/Connecting+to+an+LDAP+Directory
https://confluence.atlassian.com/display/DEV/How+to+write+LDAP+search+filters



Integration Path
----------------

JIRA can be integrated to an LDAP by a number of different paths
including full bi-directional sync.

We chose **Internal Directory with LDAP Authentication**, which means
that FreeIPA users and groups are copied to the JIRA internal directory
when a FreeIPA user logs in to JIRA. i.e.

-  FreeIPA is used for authentication
-  FreeIPA users, and optionally groups + group membership are
   replicated one-way to JIRA on user login.

In particular this means that only a subset of the FreeIPA users will be
replicated to JIRA - only those that actually log in to JIRA.

See
https://confluence.atlassian.com/display/JIRA/Connecting+to+an+Internal+Directory+with+LDAP+Authentication

Much of the config described below could probably be used for the other
integration paths.

Toolset
-------

We used the following tools to gain insight into the structure of the
FreeIPA LDAP directory, and to understand and simulate the queries that
JIRA (might) be making against FreeIPA

-  **Apache Directory Studio:** This or any similar visual LDAP browser
   is invaluable to getting into the guts of an LDAP, seeing what is
   where, which attributes are available (and even making changes in
   extremis). See https://directory.apache.org/studio/

-  **ldapsearch:** This command line tool is your friend! Use it to
   simulate queries that JIRA might be making, fine tune filters, and
   see what FreeIPA returns. This approach helped us to see that we were
   initially getting results from both the compat and accounts trees -
   which confused JIRA, and then how to prevent this.

-  **FreeIPA and JIRA logs:** What queries is JIRA actually making?
   Although you can guesstimate based on your config, it is great to see
   the actual queries.

   -  FreeIPA Server: ``/var/log/dirsrv/slapd-*/access``
   -  JIRA: See
      https://confluence.atlassian.com/display/JIRA/Logging+and+Profiling
      We added the package
      com.atlassian.crowd.directory.SpringLDAPConnector, and got partial
      results.



Key Challenges
--------------



LDAP Adapter Type
----------------------------------------------------------------------------------------------

While JIRA offers a wide range of LDAP Adapters, it does not (yet) offer
a FreeIPA Adapter out of the box. We chose the Generic LDAP adapter, and
were able to configure this for FreeIPA.



Other Candidate Adapters
^^^^^^^^^^^^^^^^^^^^^^^^

-  **FedoraDS:** We did NOT use the FedoraDS adapter (even though
   FedoraDS is an ancestor of FreeIPA), as this uses the compat tree due
   to the ``(objectclass=posixAccount)`` filter. However other freeIPA
   users have reported success with this adapter:
   https://www.redhat.com/archives/freeipa-users/2015-June/msg00200.html
-  **Other Adapters:** It is possible that other adapters can also be
   persuaded to work, with more or less additional configuration.



RFC Schemas + FreeIPA Trees
----------------------------------------------------------------------------------------------

There are several different LDAP RFC Schemas. FreeIPA uses the RFC
2307bis schema (with users stored under ``cn=accounts, cn=users``), but
also offers publishes an alternative "compat" tree
``cn=users,cn=compat,dc=example,dc=com`` with users in a RFC 2307
schema.

The chosen adapter, and the configuration applied to it, plays an
important role here in determining which of the trees returns data. Here
using ``ldapsearch`` was invaluable. For example it proved that in one
of our first attempt we were getting returns from both trees, and that
JIRA was using only the first return.

Making the query with the correct filter gave us data from the desired
tree ``(cn=accounts, cn=users)``. The ldap.user.filter
``(objectclass=inetorgperson)`` ensures that replies DO NOT come from
the compat tree (which among other things does not have the mail
attribute).

See https://www.freeipa.org/page/Directory_Server and
https://www.redhat.com/archives/freeipa-users/2015-June/msg00547.html



E-Mail Attribute and Bind Type
----------------------------------------------------------------------------------------------

We managed to get users synced from FreeIPA, and able to authenticate
and thus log in to JIRA fairly easily. However the email field was
empty, which means that JIRA mail notification didn't work, rendering
JIRA about as useful as a chocolate teapot. While the email field could
be updated in the JIRA user management, it always emptied every time the
user logged back on.

Once again ``ldapsearch`` proved what was going on: We had configured
the LDAP Adapter without a user: i.e. anonymous bind. Performing the
same query via simple bind: i.e. with an LDAP user, returned additional
attributes, including the all important mail attribute. We therefore
reconfigured the LDAP Adapter to use a FreeIPA user and password, and
bingo! JIRA received the mail attribute!

As the password of the bind user is stored in plaintext in the jira
database, make sure the user configured is a limited user (member of the
default ipa-users group is sufficient). e.g. don't use the Directory
Manager user!



Replicating Users and Groups
----------------------------------------------------------------------------------------------

Users
^^^^^

It is possible to replicate only the users to JIRA, with groups managed
locally (you can configure default groups to which user are added
automatically). We did this as a first step.To get this to work change:

-  **User Schema Settings / Additional User DN** to ``cn=accounts``
-  **User Schema Settings / User Unique ID Attribute** from
   ``entryUUID`` to ``uid``

Groups
^^^^^^

Additionally groups, and group memberships can be replicated to JIRA. To
get this to work change:

-  **Group Schema Settings / Additional Group DN** to ``cn=accounts``
-  **Group Schema Settings / Group Object Class** from
   ``groupOfUniqueNames`` to ``groupOfNames``
-  **Group Schema Settings / Group Object Filter** from
   ``(objectclass=groupOfUniqueNames)`` to
   ``(objectclass=groupOfNames)``
-  **Member Schema Settings / Group Members Attribute** from
   ``uniqueMember`` to ``Member``



The Final Configuration
-----------------------

Below is the config direct from the Jira database (of course we made the
config changes via the Jira admin GUI, which has a nifty Test function).

Note: you will need to change some values to reflect your domain,
hostname etc

mysql> select attribute_name, attribute_value from
cwd_directory_attribute where directory_id = xxxx;

+----------------------------------+----------------------------------+
| **Attribute Name**               | **Attribute Value**              |
+----------------------------------+----------------------------------+
| autoAddGroups                    | jira-users                       |
+----------------------------------+----------------------------------+
| crowd.dele                       | true                             |
| gated.directory.auto.create.user |                                  |
+----------------------------------+----------------------------------+
| crowd.dele                       | true                             |
| gated.directory.auto.update.user |                                  |
+----------------------------------+----------------------------------+
| crowd.                           | true                             |
| delegated.directory.importGroups |                                  |
+----------------------------------+----------------------------------+
| crowd.delegated.directory.type   | com.atlas                        |
|                                  | sian.crowd.directory.GenericLDAP |
+----------------------------------+----------------------------------+
| ldap.basedn                      | dc=my,dc=silly,dc=example,dc=com |
+----------------------------------+----------------------------------+
| ldap.external.id                 | uid                              |
+----------------------------------+----------------------------------+
| ldap.group.description           | description                      |
+----------------------------------+----------------------------------+
| ldap.group.dn                    | cn=accounts                      |
+----------------------------------+----------------------------------+
| ldap.group.filter                | (objectclass=groupOfNames)       |
+----------------------------------+----------------------------------+
| ldap.group.name                  | cn                               |
+----------------------------------+----------------------------------+
| ldap.group.objectclass           | groupOfNames                     |
+----------------------------------+----------------------------------+
| ldap.group.usernames             | Member                           |
+----------------------------------+----------------------------------+
| ldap.nestedgroups.disabled       | true                             |
+----------------------------------+----------------------------------+
| ldap.pagedresults                | false                            |
+----------------------------------+----------------------------------+
| ldap.pagedresults.size           | 1000                             |
+----------------------------------+----------------------------------+
| ldap.password                    |                                  |
+----------------------------------+----------------------------------+
| ldap.referral                    | false                            |
+----------------------------------+----------------------------------+
| ldap.url                         | ldap                             |
|                                  | :/\ */*.my.silly.example.com:389 |
+----------------------------------+----------------------------------+
| ldap.user.displayname            | displayName                      |
+----------------------------------+----------------------------------+
| ldap.user.dn                     | cn=accounts                      |
+----------------------------------+----------------------------------+
| ldap.user.email                  | mail                             |
+----------------------------------+----------------------------------+
| ldap.user.filter                 | (objectclass=inetorgperson)      |
+----------------------------------+----------------------------------+
| ldap.user.firstname              | givenName                        |
+----------------------------------+----------------------------------+
| ldap.user.group                  | memberOf                         |
+----------------------------------+----------------------------------+
| ldap.user.lastname               | sn                               |
+----------------------------------+----------------------------------+
| ldap.user.objectclass            | inetorgperson                    |
+----------------------------------+----------------------------------+
| ldap.user.username               | uid                              |
+----------------------------------+----------------------------------+
| ldap.user.username.rdn           | uid                              |
+----------------------------------+----------------------------------+
| ldap.userdn                      | uid=,cn=users,cn=accounts,       |
|                                  | dc=my,dc=silly,dc=example,dc=com |
+----------------------------------+----------------------------------+
| ldap.usermembership.use          | false                            |
+----------------------------------+----------------------------------+
| ld                               | false                            |
| ap.usermembership.use.for.groups |                                  |
+----------------------------------+----------------------------------+