Performance_Improvements
========================

Overview
--------

FreeIPA server has performance issue under certain circumstances. This
design page covers the improvements(topics) which will be done in scope
of 4.4 release. Each individual topic should have its own use cases,
design, implementation and feature management section.



Slow user-add
-------------

Adding of higher number of users is slowing with the numbers of users
added.

Related ticket(s):
`#5448 <https://fedorahosted.org/freeipa/ticket/5448>`__,
`#5788 <https://fedorahosted.org/freeipa/ticket/5788>`__,
`#5787 <https://fedorahosted.org/freeipa/ticket/5787>`__ ,
`5916 <https://fedorahosted.org/freeipa/ticket/5916>`__



Use Cases
----------------------------------------------------------------------------------------------

#. We have already a big number of users. With new year it is required
   to add couple thousands of users within a short period of time.
#. We are migrating from old solution to FreeIPA and need to add more
   than 40K users.

Design
----------------------------------------------------------------------------------------------

Adding user is part of the ongoing administration of a freeipa
deployment. Few users can be added at a time but administrator may also
provision many users within a short period of administrative period
(typical a week end). Adding few users is usually done with CLIs like
*user-add*, *user-activate* or *user-undelete*. The problem of
performance with those CLI is that the CLI gets slower and slower as
there are more users in the deployment. Addressing this slow down is the
purpose of `5448 <https://fedorahosted.org/freeipa/ticket/5448>`__.

On the other side administrator may want to fit into a given period, the
provisioning of large number of users. This is an administrative task
where it can be acceptable that during that period freeipa is not fully
operational. Some controls can be skipped or updates (like group
membership) can be postponed to the end of the provisioning. RFE should
be created for these improvements.



Performance slow down of CLIs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There is slight difference between CLIs like *user-add*, *user-activate*
or *user-undelete* but regarding the performance they are all doing
similar phases:

-  authenticate
-  do some controls with several searches on LDAP
-  add a user entry and make it member of groups (by default *ipausers*
   group)
-  (optional: delete stage user or delete preserved user)

In Freeipa 4.2 for example, adding a single user can last around 3sec
for the firsts users but if it already exist thousand of them it can
last for example 10sec with 50K users like described in this
`update <https://fedorahosted.org/freeipa/ticket/5448#comment:10>`__. In
the rest of this paragraph, the base duration of *ipa user-add* is 10s
and duration of individual ldap request are given in percentage of 10s.

Regarding the duration (10s) of a *user-add* CLI with 50K users the
`layout <https://fedorahosted.org/freeipa/ticket/5448#comment:10>`__
(initial durations) of the ldap operations given below. It shows that
'~90%' of "user-add" duration is spent in 'DS':

``Details:``

| ``kerberos authentication: 30%``
| ``ldap add:                28%  (sum 58%)``
| ``update group membership: 15%  (sum 73%)``
| ``ldap bind:               10%  (sum 83%)``
| ``user membership lookup:   8%  (sum 91%)``

authenticate
''''''''''''

Authentication is done on the LDAP server using the GSSAPI external
mechanism and then being bound with the entry mapping the kerberos
principal. GSSAPI Kerberos authentication is also using LDAP server so
the complete authentication is looking like (note the delay between the
connection open and the BIND:


::

   ``Details:``

   | ``[11/Jan/2016:``\ **``14:35:24``**\ `` +0100] conn=86 fd=107 slot=107 connection from xxx to yyy``
   | ``...``
   | ``[11/Jan/2016:``\ **``14:35:27``**\ `` +0100] conn=86 op=0 BIND dn="" method=sasl version=3 mech=GSSAPI``
   | ``[11/Jan/2016:14:35:27 +0100] conn=4 op=376 RESULT err=0 tag=101 nentries=1 etime=0.004000``
   | ``[11/Jan/2016:14:35:27 +0100] conn=86 op=0 RESULT err=14 tag=97 nentries=0 etime=0.015000, SASL bind in progress``
   | ``[11/Jan/2016:14:35:28 +0100] conn=86 op=1 BIND dn="" method=sasl version=3 mech=GSSAPI``
   | ``[11/Jan/2016:14:35:28 +0100] conn=86 op=1 RESULT err=14 tag=97 nentries=0 etime=0.006000, SASL bind in progress``
   | ``[11/Jan/2016:14:35:28 +0100] conn=86 op=2 BIND dn="" method=sasl version=3 mech=GSSAPI``
   | ``[11/Jan/2016:14:35:28 +0100] conn=86 op=2 RESULT err=0 tag=97 nentries=0 etime=0.002000 dn="uid=admin,cn=users,cn=accounts,``\ ``"``
   | ``.... ``
   | ``.... ``
   | ``<MOD(s) to add the users into the related groups>``
   | ``.... ``



kerberos authentication cost
                            

The reason of the duration (3s in above example) taken by kerberos
authentication is that kerberos is doing several LDAP lookup and some of
them are quite expensive:

``Details:``

| ``[11/Jan/2016:14:35:24 +0100] conn=4 op=367 SRCH base="``\ ``" scope=2 filter="(&(|(objectClass=krbprincipalaux)(objectClass=krbprincipal)(objectClass=ipakrbprincipal))(|(ipaKrbPrincipalAlias=krbtgt/``\ ``@``\ ``)(krbPrincipalName=krbtgt/``\ ``@``\ ``)))"``
| ``[11/Jan/2016:14:35:24 +0100] conn=4 op=367 RESULT err=0 tag=101 nentries=1 etime=0.648000``
| ``...``
| ``[11/Jan/2016:14:35:24 +0100] conn=4 op=369 SRCH base="``\ ``" scope=2 filter="(&(|(objectClass=krbprincipalaux)(objectClass=krbprincipal)(objectClass=ipakrbprincipal))(|(ipaKrbPrincipalAlias=ldap/``\ ``.``\ ``@``\ ``)(krbPrincipalName=ldap/``\ ``.``\ ``@``\ ``)))"``
| ``[11/Jan/2016:14:35:25 +0100] conn=4 op=369 RESULT err=0 tag=101 nentries=1 etime=0.646000``
| ``...``
| ``[11/Jan/2016:14:35:25 +0100] conn=4 op=371 SRCH base="``\ ``" scope=2 filter="(&(|(objectClass=krbprincipalaux)(objectClass=krbprincipal))(krbPrincipalName=HTTP/``\ ``.``\ ``@``\ ``))"``
| ``[11/Jan/2016:14:35:26 +0100] conn=4 op=371 RESULT err=0 tag=101 nentries=1 etime=0.530000``
| ``...``
| ``[11/Jan/2016:14:35:26 +0100] conn=4 op=373 SRCH base="``\ ``" scope=2 filter="(&(objectClass=ipaKrb5DelegationACL)(memberPrincipal=HTTP/``\ ``.``\ ``@``\ ``))"``
| ``[11/Jan/2016:14:35:26 +0100] conn=4 op=373 RESULT err=0 tag=101 nentries=1 etime=0.424000``
| ``...``
| ``[11/Jan/2016:14:35:26 +0100] conn=4 op=374 SRCH base="``\ ``" scope=2 filter="(&(|(objectClass=krbprincipalaux)(objectClass=krbprincipal))(krbPrincipalName=admin@``\ ``))"``
| ``[11/Jan/2016:14:35:27 +0100] conn=4 op=374 RESULT err=0 tag=101 nentries=1 etime=0.551000``

The SRCH requests are costly because of the base search, each of them
triggers an internal lookup in the schema compat plugin map. The more
entries (users) are present in the map, the more expensive the lookup
is. For example, the same searches without schema compat lookup are 35
times faster.

There are several possibilities to avoid this extra cost:

-  change the base search to that it does not cover the *cn=compat,*.
   But krb principals are either in *cn=kerberos* and *cn=accounts*.
   Changing the the single search into two searches on each branch was
   too complex and this idea was dropped
-  Add a new ldap control supported by schema compat, so that a ldap
   client could request schema compat to avoid lookup into the map. Two
   tickets were opened for
   `client <https://fedorahosted.org/freeipa/ticket/5599>`__ and `server
   side <https://fedorahosted.org/freeipa/ticket/5597>`__.
-  Kerberos is looking of real users, not for compat users. The idea is
   to make schema compat aware the request comes from kerberos
   application and so avoid lookup in the map. Kerberos access ldap
   server using *ldapi* interface and authenticate as *cn=directory
   manager*. A simple fix on schema compat plugin side, is to ignore any
   requests coming *ldapi/root*.

The solution implemented to address the kerberos authentication cost was
fixing **schema compat** because it is an easy fix. 389-ds server,
*assuming* that a local agent (*ldapi* interface) bound as *root* (like
kerberos) is not interested by the schema compat mapped entries.



ldap bind cost
              

The ldap BIND itself is not expensive. In the above example, it lasts
around 0.012s that is not significant (0.1%) regarding the complete
user-add duration (take a base time of 10s). Looking at the top
consumption of DS plugins, none of plugin involved in BIND op appears in
top consumer.

For this reason we did not do specific improvement on LDAP BIND



Control and LDAP searches
'''''''''''''''''''''''''

Adding a freeipa user mainly consist in add user entry and update the
group(s) the user entry belongs to. Before and after each of those two
steps, there are several LDAP searchs like: reading the config
(*cn=ipaconfig,cn=etc,*), checking that the user does not already exist
(active or preserved or private group), checking credential, and group
membership.

The total number of searches is typically 25 but only one is expensive
the search looking for group membership of the added user (see
`update <https://fedorahosted.org/freeipa/ticket/5448#comment:10>`__).

Some optimization could likely be done on the 24 others. For example 13
out of the 24 are identical and are reading the config
(*cn=ipaconfig,cn=etc,*). The total of those search account for ~0.04s
that is not significant (0.4% req duration) but would likely increase
more response time because of the multiple requests to send/wait/decode.
The caching of the ipaconfig has been fixed in
`5463 <https://fedorahosted.org/freeipa/ticket/5463>`__. With this fix,
only one lookup of ipaconfig is done.

The request that is expensive is :

| ``[05/Apr/2016:13:57:33 +0200] conn=75540 op=17 SRCH base="``\ ``" scope=2 filter="(|(member=uid=tb51420,cn=users,cn=accounts,``\ ``)(memberUser=uid=tb51420,cn=users,cn=accounts,``\ ``)(memberHost=uid=tb51420,cn=users,cn=accounts,``\ ``))" attrs=""``
| ``[05/Apr/2016:13:57:33 +0200] conn=75540 op=17 RESULT err=0 tag=101 nentries=0 etime=0.275000``



Add user
''''''''

The add of the user account is looking like

| ``[05/Apr/2016:13:57:31 +0200] conn=75540 op=13 ADD dn="uid=tb51420,cn=users,cn=accounts,``\ ``"``
| ``[05/Apr/2016:13:57:33 +0200] conn=75540 op=13 RESULT err=0 tag=105 nentries=0 etime=1.850000``

The ldap ADD accounts for nearly 20% of the total CLI. But
`90% <https://fedorahosted.org/freeipa/ticket/5448#comment:6>`__ of the
time spent in the ADD is spent in 6 lookup in schema compat map. Those
lookup are **internal searches** done by DNA, uniqueness
(krbPrincipalName, krbCanonicalName, ipaUniqueID, uid) and schema compat
itself.

``Details:``

| ``2 identical internal search done by 'DNA'``
| ``SRCH base="``\ ``" scope=2 filter="(&(|(objectClass=posixAccount)(objectClass=posixGroup)(objectClass=ipaIDobject))(|(uidNumber=1677038171)(gidNumber=1677038171)))" attrs="dn"``
| ``3 searches done by 'uniqueness'``
| ``SRCH base="``\ ``" scope=2 filter="(&(objectClass=posixAccount)(|(uid=tb38189)))" attrs="dn"``
| ``SRCH base="``\ ``" scope=2 filter="(|(ipaUniqueID=8549a6d6-a969-11e5-bfb1-001a4a231292))" attrs="dn"``
| ``SRCH base="``\ ``" scope=2 filter="(|(krbPrincipalName=tb38189@``\ ``))" attrs="dn"``
| ``1 search done by 'schema compat'. note this one dumps ipausers group``
| ``SRCH base="cn=groups,cn=accounts,``\ ``" scope=1 filter="(member=uid=tb38189,cn=users,cn=accounts,``\ ``)" attrs=ALL``

There are two options to reduce the impact of those internal searches:

-  modify DNA and uniqueness plugins configuration like described
   `here <https://fedorahosted.org/freeipa/ticket/5448#comment:7>`__. It
   does not fix the last internal search triggered by 'schema compat'
   itself. Those change improves the performance of LDAP ADD by 10.
-  Fixing schema compat plugin so that it does not trigger map lookup on
   **internal operations**. This fix has a large impact as it applies
   for any use case not only user-add. The gain is in the same range ADD
   drops from 2.7s to 0.3s (see
   `update <https://fedorahosted.org/freeipa/ticket/5448#comment:10>`__)

Because of the fix in schema compat being very simple (skip internal
operation), major gain (even for other use case). This is the one that
was implement.



Update of the group membership
''''''''''''''''''''''''''''''

When a user is added it is by default added to the group
''cn=ipausers,cn=groups,cn=accounts,". This updates last around 15% of
the duration of the CLI.
`Half <https://fedorahosted.org/freeipa/ticket/5448#comment:8>`__ of the
duration of group update is spent in schema compat plugin handling
**internal operation**. Those operations where triggered by others
plugins:

-  memberof
-  mep
-  check-range
-  uuid
-  password-retry

Except for *mep* plugins, changing the plugin configuration in order to
avoid schema compat divides by 2 the duration of the update of the
group.

There are two options to reduce the impact of those internal searches:

-  modify the configuration of the above plugins like it is described
   `here <https://fedorahosted.org/freeipa/ticket/5448#comment:8>`__.
   Improvement for mep plugin can not be achieve that way. The gains is
   to divide by 2 the update
-  Fixing schema compat plugin so that it does not trigger map lookup on
   **internal operations**. This fix has a large impact as it applies
   for any use case not only MOD of groups. The gain is higher, MOD
   drops from 1.56s to 0.46s
   `update <https://fedorahosted.org/freeipa/ticket/5448#comment:10>`__

Because the fix in **schema compat** being very simple (skip internal
operation), **major gain** (even for other use case). This is the one
that was implemented.



broken SchemaCache
''''''''''''''''''

Due `#5787 <https://fedorahosted.org/freeipa/ticket/5787>`__ every IPA
command call downloads the LDAP schema first without any caching. It
took 40-60% of time of user-add command without groups.

::

   ``Profiler output:``

   | ``170386 function calls (170213 primitive calls) in ``\ **``0.680``\ ````\ ``seconds``**
   | ``Ordered by: cumulative time``
   | `` ``
   | ``ncalls  tottime  percall  cumtime  percall filename:lineno(function)``
   | ``...``
   | ``206    0.000    0.000    0.470    0.002 /usr/lib/python2.7/site-packages/ipapython/ipaldap.py:731(_get_schema)``
   | ``  1    0.000    0.000    0.470    0.470 /usr/lib/python2.7/site-packages/ipapython/ipaldap.py:113(get_schema)``
   | ``  1    0.000    0.000    ``\ **``0.470``**\ ``    0.470 /usr/lib/python2.7/site-packages/ipapython/ipaldap.py:140(_retrieve_schema_from_server)``
   | `` 32    0.000    0.000    0.364    0.011 /usr/lib64/python2.7/site-packages/ldap/ldapobject.py:87(_ldap_call)``
   | ``...``

This performance issue will be resolved by fixing
`#5787 <https://fedorahosted.org/freeipa/ticket/5787>`__.



option --noprivate is not efficient
'''''''''''''''''''''''''''''''''''

Related ticket(s):
`#5788 <https://fedorahosted.org/freeipa/ticket/5788>`__

With option --noprivate postcallback of user_add command executes
user-mod command for simple value change. This is ineffective and
internal ldap mod call should be executed.



CLI framework
'''''''''''''

The following
`implementation <http://www.freeipa.org/page/V4/Performance_Improvements#Directory_Server>`__
drop the CLI duration from 10s to 3s. However, looking at the time spent
in those 3s, it appears that remaining ldap requests are only accounting
for 0.5s, so it remains more than 2s spent in CLI framework. The
following ticket `5916 <https://fedorahosted.org/freeipa/ticket/5916>`__
is to track this remaining part

Implementation
----------------------------------------------------------------------------------------------



User-add CLI
^^^^^^^^^^^^

The improvement described in `Control and LDAP
searches <http://www.freeipa.org/page/V4/Performance_Improvements#Control_and_LDAP_searches>`__
was implemented since **4.3.4** with the ticket
`5463 <https://fedorahosted.org/freeipa/ticket/5463>`__ and
`commit <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=7f0d018c66da1fe2adedd45aa9f5a63c913e4527>`__



Directory Server
^^^^^^^^^^^^^^^^

The improvement seen in
`authenticate <http://www.freeipa.org/page/V4/Performance_Improvements#authenticate>`__
was implemented in slapi-nis plugin.

The improvements seen in ldap
`ADD <http://www.freeipa.org/page/V4/Performance_Improvements#Add_user>`__
and
`MOD <http://www.freeipa.org/page/V4/Performance_Improvements#Update_of_the_group_membership>`__
were implemented in slapi-nis plugin `slapi-nis: process requests only
when initialization
completed <https://git.fedorahosted.org/cgit/slapi-nis.git/diff/src/back-sch.c?id=594fcb2320033d01cfe2b8121793d431d1017987>`__.
Actually the subject of the commit does not reflect those changes in
that file, where the perf improvement are

| ``+  if (slapi_op_internal(pb) || (slapi_is_ldapi_conn(pb) && isroot)) {``
| ``+      /* The plugin should not engage in internal searches of other``
| ``+       * plugins or ldapi+cn=DM */``
| ``+      return 0;``
| ``+  }``

Those improvements are available since **Release 0.55**



Feature Management
----------------------------------------------------------------------------------------------

UI
^^

CLI
^^^



Slow user-find
--------------

High number of users stored in LDAP causes slowdown of the IPA command.

Related ticket(s):
`#5281 <https://fedorahosted.org/freeipa/ticket/5281>`__,
`#5282 <https://fedorahosted.org/freeipa/ticket/5282>`__,
`#3376 <https://fedorahosted.org/freeipa/ticket/3376>`__,
`#4995 <https://fedorahosted.org/freeipa/ticket/4995>`__



Use Cases
----------------------------------------------------------------------------------------------

#. Increase the usability of user-find command because with many users
   searches in LDAP take too long and may result into timeout.



Design
----------------------------------------------------------------------------------------------



Don't do extra search for ipasshpubkey attribute
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Related ticket(s):
`#3376 <https://fedorahosted.org/freeipa/ticket/3376>`__,
`#5281 <https://fedorahosted.org/freeipa/ticket/5281>`__

*ipasshpubkey* can be fetched together with user entry, there is no need
for an extra search operation.

``User-find with 2000 entries with sshpubkey``

| ``6310241 function calls (6200125 primitive calls) in ``\ **``16.453``**\ `` seconds``
| ``   Ordered by: cumulative time``
| ``   ncalls  tottime  percall  cumtime  percall filename:lineno(function)``
| ``....``
| ``        1    0.027    0.027   16.449   16.449 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:2015(execute)``
| ``     6002    0.256    0.000   12.501    0.002 /usr/lib/python2.7/site-packages/ipapython/ipaldap.py:1272(find_entries)``
| ``        1    0.008    0.008    9.519    9.519 /usr/lib/python2.7/site-packages/ipalib/plugins/user.py:801(post_callback)``
| ``        1    0.041    0.041    9.392    9.392 /usr/lib/python2.7/site-packages/ipalib/plugins/baseuser.py:618(post_common_callback)``
| ``    16009    0.120    0.000    6.697    0.000 /usr/lib64/python2.7/site-packages/ldap/ldapobject.py:87(_ldap_call)``
| ``    10006    0.024    0.000    6.348    0.001 /usr/lib64/python2.7/site-packages/ldap/ldapobject.py:472(result3)``
| ``    10006    0.057    0.000    6.324    0.001 /usr/lib64/python2.7/site-packages/ldap/ldapobject.py:480(result4)``
| ``    10006    6.114    0.001    6.114    0.001 {built-in method result4}``
| ``     2000    0.053    0.000    5.341    0.003 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:733(get_password_attributes)``
| ``        1    0.000    0.000    4.283    4.283 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:1145(wrapped)``
| ``     2000    0.043    0.000    ``\ **``3.787``**\ ``    0.002 /usr/lib/python2.7/site-packages/ipalib/util.py:293(``\ **``convert_sshpubkey_post``**\ ``)``
| ``    10004    0.095    0.000    3.147    0.000 /usr/lib/python2.7/site-packages/ipapython/ipaldap.py:895(_convert_result)``
| ``.....``

As profiling output shows approximately **23%** of time was spent on
processing *ipasshpubkey* attribute because for each user it was
downloaded separately

ldap access log contains

| ``[15/Apr/2016:12:59:11 +0200] conn=30 op=5624 SRCH base="uid=user1871,cn=users,cn=accounts,dc=example,dc=com" scope=0 filter="(objectClass=*)" attrs="ipaSshPubKey"``
| ``[15/Apr/2016:12:59:11 +0200] conn=30 op=5624 RESULT err=0 tag=101 nentries=1 etime=0``

for each user (2000 times for this case)

Fetching *ipsshpubkey* together with all attributes in one search will
increase speed rapidly.



Remove userPassword, krbPrincipalKey attributes from search results
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Related ticket(s):
`#5281 <https://fedorahosted.org/freeipa/ticket/5281>`__

*userPassword* and *krbPrincipalKey* attributes require extra search.
These attribute should be removed from user-find command to get better
performance.

``user-find with 2000 users:``

| ``6310241 function calls (6200125 primitive calls) in ``\ **``16.453``**\ `` seconds``
| ``   Ordered by: cumulative time``
| ``   ncalls  tottime  percall  cumtime  percall filename:lineno(function)``
| ``....``
| ``        1    0.027    0.027   16.449   16.449 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:2015(execute)``
| ``     6002    0.256    0.000   12.501    0.002 /usr/lib/python2.7/site-packages/ipapython/ipaldap.py:1272(find_entries)``
| ``        1    0.008    0.008    9.519    9.519 /usr/lib/python2.7/site-packages/ipalib/plugins/user.py:801(post_callback)``
| ``        1    0.041    0.041    9.392    9.392 /usr/lib/python2.7/site-packages/ipalib/plugins/baseuser.py:618(post_common_callback)``
| ``    16009    0.120    0.000    6.697    0.000 /usr/lib64/python2.7/site-packages/ldap/ldapobject.py:87(_ldap_call)``
| ``    10006    0.024    0.000    6.348    0.001 /usr/lib64/python2.7/site-packages/ldap/ldapobject.py:472(result3)``
| ``    10006    0.057    0.000    6.324    0.001 /usr/lib64/python2.7/site-packages/ldap/ldapobject.py:480(result4)``
| ``    10006    6.114    0.001    6.114    0.001 {built-in method result4}``
| ``     2000    0.053    0.000    ``\ **``5.341``**\ ``    0.003 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:733(``\ **``get_password_attributes``**\ ``)``
| ``        1    0.000    0.000    4.283    4.283 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:1145(wrapped)``
| ``....``

Getting and processing password attributes took approximately **32%** of
time.

The ldap access log contains

| ``[15/Apr/2016:12:59:12 +0200] conn=30 op=5764 SRCH base="uid=user1918,cn=users,cn=accounts,dc=example,dc=com" scope=0 filter="(krbPrincipalKey=*)" attrs="krbPrincipalKey"``
| ``[15/Apr/2016:12:59:12 +0200] conn=30 op=5764 RESULT err=0 tag=101 nentries=0 etime=0``

for each user (2000 times for this case)

Note: this change causes that the output of user-find is not backward
compatible.



processing members
^^^^^^^^^^^^^^^^^^

user-find does not process members (groups, roles, sudorules, hbacrules,
...) by default.

However with option --all

| ``$ ipa user-find --all``
| ``ipa: ERROR: cannot connect to '``\ ```https://ipa.example.com/ipa/json`` <https://ipa.example.com/ipa/json>`__\ ``': Gateway Timeout``

This testcase contains 2000 users with 110 direct and indirect
memberships.

Fro more details please read `\*-find
section <http://www.freeipa.org/page/V4/Performance_Improvements#.2A-find>`__



Implementation
----------------------------------------------------------------------------------------------



Feature Management
----------------------------------------------------------------------------------------------



UI
^^

WebUI is not affected, because it uses user-show heavily instead of
user-find. From user find it requires only list of primary keys.

user-find --pkey-only with 2000 users

``708478 function calls (694369 primitive calls) in 1.889 seconds``



CLI
^^^

Configuration
^^^^^^^^^^^^^

N/A

Upgrade
----------------------------------------------------------------------------------------------

N/A



Slow host-find
--------------

High number of hosts stored in LDAP causes slowdown of the IPA command.

Issue here are similar to user-find issues.



Use Cases
----------------------------------------------------------------------------------------------

#. Increase the usability of host-find command because with many host
   searches in LDAP take too long and may result into timeout.



Design
----------------------------------------------------------------------------------------------



Don't do extra search for ipasshpubkey attribute
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

See
`user-find <http://www.freeipa.org/page/V4/Performance_Improvements#Slow_user-find>`__



Remove userPassword, krbPrincipalKey attributes from search results
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

See
`user-find <http://www.freeipa.org/page/V4/Performance_Improvements#Slow_user-find>`__



processing members
^^^^^^^^^^^^^^^^^^

| ``$ ipa host-find``
| ``ipa: ERROR: cannot connect to '``\ ```https://ipa.example.com/ipa/json`` <https://ipa.example.com/ipa/json>`__\ ``': Gateway Timeout``

This testcase contains 2000 hostss with 110 direct and indirect
memberships.

For more details please read `\*-find
section <http://www.freeipa.org/page/V4/Performance_Improvements#.2A-find>`__



Implementation
----------------------------------------------------------------------------------------------



Feature Management
----------------------------------------------------------------------------------------------



UI
^^



CLI
^^^



Configuration
^^^^^^^^^^^^^

N/A



Upgrade
----------------------------------------------------------------------------------------------

N/A



Improvements of other commands
------------------------------

Side effects/benefits from user commands related changes to other IPA
commands



typical provisioning: ldapadd entries, migrate-ds...
----------------------------------------------------------------------------------------------



Use case
^^^^^^^^

-  We are migrating (see `this
   RFE <http://www.freeipa.org/page/V4/FreeIPA_to_FreeIPA_Migration>`__)
   from old solution to FreeIPA and need to add **entries**
   (users/groups/hosts/rules...) withing a short period of time

Freeipa LDAP entries are typically:

-  read from a **source instance** into a **ldif** format
-  entries are possibly modified according to business/admin
   requirements (for example during migration scenario)
-  added/imported into a **target instance**

This chapter is related to the performance problem that can occur during
**add/import**

A provisioning tool
`create-test-data.py <https://github.com/freeipa/freeipa-tools/blob/master/create-test-data.py>`__
is used to create a ldif file to import. Such tool/file can be used to
identify bottleneck and possible performance improvement and later used
to detect performance regression.

The entries are added synchronously and in sequence:

-  users
-  hosts
-  user groups (nested)
-  host groups (nested)
-  sudo rules
-  hbac rules

The specification of the data are:

-  users - default 50K - each user is member of 10 user groups
-  hosts - default 40K - each host is member of 5 hostgroups
-  user group - default 1K - each group contains 1000 users
-  host group - default 1K - each group contains 400 hosts
-  sudo rule - default 200
-  hbac rules - default 200
-  each user will be direct member of random 5 unique hbac rules and 5
   unique sudo rules
-  create a structure of nested groups and add users to these groups so
   that users will be indirect member of more than 50 hbac rules and 50
   sudo rulesthe same with host and hostgroups
-  so we can achieve results of user and host entries being direct and
   indirect member of more than 100 groups/sudo rules/hbac rules

Related opened tickets

-  `5861 <https://fedorahosted.org/freeipa/ticket/5861>`__: failing
   internal MOD when adding empty host group
-  `5802 <https://fedorahosted.org/freeipa/ticket/5802>`__: perf: adding
   a group with 1000 users/hosts lasts long (up to 12s)
-  `48812 <https://fedorahosted.org/389/ticket/48812>`__: exclude
   backends from plugin operation
-  `5914 <https://fedorahosted.org/freeipa/ticket/5914>`__: invalid
   setting of DS lock table size
-  `48856 <https://fedorahosted.org/389/ticket/48856>`__: Memberof
   plugins compute 'memberof' using internal searches that can be costly
-  `48861 <https://fedorahosted.org/389/ticket/48861>`__: Memberof
   plugins can update several times the same entry to set the same
   values
-  `48868 <https://fedorahosted.org/389/ticket/48868>`__: Checking of
   cache tuning is too strict and make DS unusable
-  `48812 <https://fedorahosted.org/389/ticket/48812>`__: Exclude
   Backends From Plugin Operations



Provisioning throughput and DS tuning
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^



Entry cache tuning
''''''''''''''''''

The following table shows the duration of import depending of the
**entry cache** size (domain). Tests have been done with different size
(10Mb, 50Mb, 100Mb) of **db cache**, it had almost no impact on the
duration.

The import was done with **memberof: enabled**. (slapi-nis and retroCL
disabled).

============== ==== ===== =====
Cache size     10Mb 100Mb 200Mb
Duration       4h00 2h30  1h40
Entries cached 4%   45%   100%
============== ==== ===== =====

While the tests was running the number of entries in the **entry
caches** was monitored. When the cache was too small to fit all entries
(100Mb), monitoring shows that when adding sudorules and hbacrules
significantly reduce the number of entries in the cache. That means
added entries are **large static groups** like hbac having 2200 members.
The consequence of large static groups is that it moves out of the entry
cache the members entries that memberof will update. So memberof updates
will be slowed down because members entries need to be **reloaded in
entry cache** for the updates.

In conclusion:

-  If provisioning contains large static group, it is better to have an
   entry cache that can fit all entries (groups and members)
-  having entry cache larger than 400Mb is likely not a good idea
   because it would also create a large memory footprint without giving
   much benefit
-  the benefit of caching all entries is in the range of **2-3 times**

If the machine has enough memory, the **entry cache could range from
100Mb to 400Mb**. This tuning should leave enough free memory for the
file system cache.



database cache tuning
'''''''''''''''''''''

Tuning of this attribute usually requires some iterating tests. In fact
having a large cache allows to cache more DB pages but can be a problem
during checkpointing. On the other side, db pages are also file pages.
So before going into the DB cache those pages, even evicted from DB
cache, usually remain into the **file system** cache and are easily
reloaded.

Relying on file system cache is a good approach to keep as much DB page
as possible. But on the other side having a too small DB cache can
create constant reload.

If the machine has enough memory, the **db cache could range from 200Mb
to 500Mb**. This tuning should leave enough free memory for the file
system cache.

In my tests tuning of db cache has no noticeable impact. So if we need
to save memory (for file system cache), it would be recommended to give
the priority to entry cache



database locks
''''''''''''''

During tests it appears that the default number of database locks was
too low. This can be monitored with

| ``ldapsearch -LLL -o ldif-wrap=no -D "cn=directory manager" -w Secret123 -b "cn=database,cn=monitor,cn=ldbm database,cn=plugins,cn=config" nsslapd-db-configured-locks nsslapd-db-current-locks nsslapd-db-max-locks``
| ``dn: cn=database,cn=monitor,cn=ldbm database,cn=plugins,cn=config``
| ``nsslapd-db-configured-locks: 100000``
| ``nsslapd-db-current-locks: 8980``
| ``nsslapd-db-max-locks: 42675``

``One rule of thumb, for large provisioning, is to set database lock to the half of number of provisioned users and hosts.``



Provisioning throughput and DS plugins
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^



Small DB (10K entries)
''''''''''''''''''''''

The dataset is:

-  5K users - each user is member of 10 users group
-  4K hosts - each host is member of 5 hosts group
-  100 users groups with 1000 users (+nested)
-  100 hosts group with 400 hosts (+nested)
-  100 sudorules with 2200 users/hosts (direct/indirect)
-  100 hbacrules

   -  20 with 2200 users/hosts (direct)
   -  46 with 1400-1800 users/hosts (nested)
   -  23 with 400-800 users/hosts (nested)
   -  1 with no member

The following table present the provisioning duration and number of
operations (vast majority of them are internal) depending which plugins
are enabled:

+-------------+-------------+-------------+-------------+-------------+
| Plugin      | P           | ADD         | MOD         | SRCH        |
| enabled     | rovisioning |             |             |             |
|             | Duration    |             |             |             |
|             | (**)        |             |             |             |
+-------------+-------------+-------------+-------------+-------------+
| memberof    | slapi-nis   | retroCL     | style="     | style="     |
|             |             |             | width:100px | width:100px |
|             |             |             | style="     | style="tex  |
|             |             |             | text-align: | t-align:cen |
|             |             |             | center;" Nb | ter;"  Cumul|
|             |             |             |             | srch        |
|             |             |             |             | duration    |
+-------------+-------------+-------------+-------------+-------------+
| Y           | Y           | Y           | 4h36min     | | 580K      |
|             |             |             |             | | (95%      |
|             |             |             |             |   retroCL)  |
+-------------+-------------+-------------+-------------+-------------+
| Y           | Y           | *no*        | 5h28min     | 15K         |
+-------------+-------------+-------------+-------------+-------------+
| Y           | *no*        | *no*        | 4h04min     | 15K         |
+-------------+-------------+-------------+-------------+-------------+
| *no*        | Y           | Y           | 12min(*)    | 39K         |
+-------------+-------------+-------------+-------------+-------------+
| *no*        | Y           | *no*        | 11min(*)    | 15K         |
+-------------+-------------+-------------+-------------+-------------+
| *no*        | *no*        | *no*        | 9min(*)     | 15K         |
+-------------+-------------+-------------+-------------+-------------+

(**\***) If **memberof** plugin is disabled during provisioning, the
memberof attribute in the entries is not updated. So at the end of the
provisioning, we need to run fixup tasks to rebuild this attribute in
the entries. These duration are including fixup routines duration that
last 5m30 and trigger 9K MOD/0.4M SRCH. Note that to run fixup routines,
memberof plugin needs to be enabled.

(**\*\***) Some tests were not done the same day. Performance of the VM
over the days is not that stable. Strict comparison of duration are not
valid. The duration just gives a rough idea how long lasts the
provisioning.

(**\**\***) 80% of the SRCH are below 1ms and 99.5% are below 2ms. To
estimate the duration of the all SRCHs we take the hypothesis that each
individual SRCH costs 1ms.

Regarding the response time of the **hbacrules** that are the longest
ADD operations. There is no correlation between the duration of the ADD
operation and the number of members.

+-------------+-------------+-------------+-------------+-------------+
| HBAC rule   | | Empty     | Small grp   | Medium grp  | Large grp   |
|             | | group     | (400-800)   | (           | (2200)      |
|             |             |             | 1400-1800)> |             |
+-------------+-------------+-------------+-------------+-------------+
| min.        | max.        | min.        | max.        | min.        |
+-------------+-------------+-------------+-------------+-------------+
| Duration    | 58s         | 61s         | 136s        | 33s         |
+-------------+-------------+-------------+-------------+-------------+



Medium DB (100K entries)
''''''''''''''''''''''''

The dataset is:

-  50K users
-  40K hosts
-  x users groups with x users (+nested)
-  x hosts group with x hosts (+nested)
-  100 sudorules with 22500 users/hosts (direct/indirect)
-  100 hbacrules

The following table shows value of provisioning of a medium DB in two
steps: provisioning without memberof and fixup of memberof.

.. figure:: performance_improvements.png
   :alt: performance_improvements.png

   performance_improvements.png



Memberof plugin
'''''''''''''''

According to the measurements (see
`table <http://www.freeipa.org/page/V4/Performance_Improvements#Small_DB_.2810K_entries.29>`__),
the major bootleneck is the memberof plugin. Disabling memberof during
provisioning allows to make the full (provisioning+fixup) provisioning
**20 times faster** (13min instead of 4h14).

**Accelarate provisioning worth restarting DS**. The
`discussion <https://www.redhat.com/archives/freeipa-devel/2016-May/msg00226.html>`__
on freeipa-devel concluded that it is acceptable to restart DS in order
to accelerate provisioning.

**Replication will slowly converge**. In a replicated topology, it would
be very difficult on **all** DS instances to disable memberof, wait for
provisioned entries to be replicated and finally run the fixup. It is
decided to disable/fixup only on the server where the provisioning
occurs. The user experience of provisioning will be better than now. On
replica, the replicated updates will be slow because of memberof being
enabled but it will not be worse than now.



Schema compat plugin
''''''''''''''''''''

According to the measurements (see
`table <http://www.freeipa.org/page/V4/Performance_Improvements#Small_DB_.2810K_entries.29>`__),
the schema compat plugin **is not** a performance bottleneck. However,
when memberof is disabled, it **reduces** the number of SRCH by an extra
**90%** and the overall **duration** by an extra **10%**.

LDAP client is supposed to not access DS during provisioning so
disabling Schema Compat during this period has no impact and the later
restart will allow to reenable Schema Compat.

In conlusion, it gives an extra throughput benefice to disable Schema
Compat during provisioning and to reenable it later. Preferably is to
reenable it after the fixup, but then it will require one more restart.



RetroCL plugin
''''''''''''''

According to the measurements (see
`table <http://www.freeipa.org/page/V4/Performance_Improvements#Small_DB_.2810K_entries.29>`__),
the Retro CL plugin **is not** a performance bottleneck. However,
disabling retroCL reduces by **2*(#user + #hosts)** the number of ADD.

The benefit is an extra reduction of **10%** of the duration of the ADD.
The drawback is that is that the server will no longer be able to
syncrepl the provisioned entries.

This improvement is not that significant and if support of **syncrepl is
a requirement**, it is ok to keep **RetroCL enabled**.

The ticket `48812 <https://fedorahosted.org/389/ticket/48812>`__ does
not provide a measurable performance gain:

| ``DBcache: 100Mb``
| ``Entrycache: 110Mb``
| ``DNcache: 60Mb``
| ``Memberof:     disabled``
| ``slapi-nis:     disabled``
| ``RetroCL:     enabled``
| ``Content:     enabled``

=============================================================== ========
DS Version                                                      Duration
Provisioning                                                    Fixup
1.3.4.9                                                         3 min 58
1.3.5.6+\ `48812 <https://fedorahosted.org/389/ticket/48812>`__ 4 min 03
=============================================================== ========

Conclusions
'''''''''''

-  **Disable** memberof and run fixup. **memberof** plugin has a major
   impact on the throughput and duration of the provisioning. Even
   taking into account the provisioning and fixup tasks duration, the
   overall procedure is much faster. The expected benefit is in a range
   **20 times faster**. The
   `discussion <https://www.redhat.com/archives/freeipa-devel/2016-May/msg00226.html>`__
   on freeipa-devel concluded that it is acceptable to restart DS in
   order to accelerate provisioning
-  **Disable** Schema compat during provisioning and fixup. A possible
   option to *save* a restart is to enable *Schema compa* at the fixup
   time.
-  **Keep enabled** RetroCL, because the expected benefit does not worth
   loosing the ability to use syncrepl
-  accelerate provisioning gives a much better user experience of
   provisioning
-  slow replication of provisioned data existed before, so the situation
   after improving provision is not worse than before.



Proposed improvements
^^^^^^^^^^^^^^^^^^^^^

Algorithm
'''''''''

The CLI that will do the provisioning of a given ldif file will:

-  Retrieve "cn=directory manager" credential. Using DM is required to
   tune DS during provisioning and avoid ACL cost.
-  Parse ldif file to check that each provisioned entry matches one of
   the condition:

::

   | ``(objectClass=inetorgperson)``
   | ``(objectClass=ipausergroup)``
   | ``(objectClass=ipahost)``
   | ``(objectClass=ipahostgroup)``
   | ``(objectClass=ipasudorule)``
   | ``(objectClass=ipahbacrule)``

-  Compute and set the appropriate `db cache
   <http://www.freeipa.org/page/V4/Performance_Improvements#database_cache_tuning>`__
   size and `db locks <http://www.freeipa.org/page/V4/Performance_Improvements#database_locks>`__

::

   | ``dn: cn=config,cn=ldbm database,cn=plugins,cn=config``
   | ``changetype: modify``
   | ``replace: nsslapd-dbcachesize``
   | ``nsslapd-dbcachesize: ``
   | ``-``
   | ``replace: nsslapd-db-locks``
   | ``nsslapd-db-locks: ``

-  Compute and set the appropriate *domain* `entry cache <http://www.freeipa.org/page/V4/Performance_Improvements#Entry_cache_tuning>`__ size

::

   | ``dn: cn=userRoot,cn=ldbm database,cn=plugins,cn=config``
   | ``changetype: modify``
   | ``replace: nsslapd-cachememsize``
   | ``nsslapd-cachememsize: ``

-  Disable memberof

::

   | ``dn: cn=MemberOf Plugin,cn=plugins,cn=config``
   | ``changetype: modify``
   | ``replace: nsslapd-pluginEnabled``
   | ``nsslapd-pluginEnabled: off``

-  Disable Schema Compat

::

   | ``dn: cn=Schema Compatibility,cn=plugins,cn=config``
   | ``changetype: modify``
   | ``replace: nsslapd-pluginEnabled``
   | ``nsslapd-pluginEnabled: off``

-  stop ipa (that will stop DS)
-  **start DS**
-  ldapadd -D "xxx" -y -f
-  Enable memberof

| ``dn: cn=MemberOf Plugin,cn=plugins,cn=config``
| ``changetype: modify``
| ``replace: nsslapd-pluginEnabled``
| ``nsslapd-pluginEnabled: on``

-  **restart DS**
-  Run fixup (and monitor completion) for each of the following filters
   (if it existed entries in the ldif file matching the filter).

| ``fixup-memberof.pl  -D "cn=directory manager" -j ``\ `` -Z ``\ *``server-id``*\ `` -b "``\ *``suffix``*\ ``" -f "(objectClass=inetorgperson)" -P LDAP``
| ``fixup-memberof.pl  -D "cn=directory manager" -j ``\ `` -Z ``\ *``server-id``*\ `` -b "``\ *``suffix``*\ ``" -f "(objectClass=ipausergroup)" -P LDAP``
| ``fixup-memberof.pl  -D "cn=directory manager" -j ``\ `` -Z ``\ *``server-id``*\ `` -b "``\ *``suffix``*\ ``" -f "(objectClass=ipahost)" -P LDAP``
| ``fixup-memberof.pl  -D "cn=directory manager" -j ``\ `` -Z ``\ *``server-id``*\ `` -b "``\ *``suffix``*\ ``" -f "(objectClass=ipahostgroup)" -P LDAP``
| ``fixup-memberof.pl  -D "cn=directory manager" -j ``\ `` -Z ``\ *``server-id``*\ `` -b "``\ *``suffix``*\ ``" -f "(objectClass=ipasudorule)" -P LDAP``
| ``fixup-memberof.pl  -D "cn=directory manager" -j ``\ `` -Z ``\ *``server-id``*\ `` -b "``\ *``suffix``*\ ``" -f "(objectClass=ipahbacrule)" -P LDAP``

-  Enable Schema Compat

| ``dn: cn=Schema Compatibility,cn=plugins,cn=config``
| ``changetype: modify``
| ``replace: nsslapd-pluginEnabled``
| ``nsslapd-pluginEnabled: on``

-  **stop DS**
-  **start ipa**



Provisioning constraints
''''''''''''''''''''''''



Provisioning server is offline
                              

Provisioning is done on a server where the memberof plugin is disabled.
That means **memberof** attribute is **invalid** on that server until
provisioning/fixup is completed.

That means that the server is considered to be
`offline <https://www.redhat.com/archives/freeipa-devel/2016-May/msg00424.html>`__
because ldap client accessing it may receive invalid data.

An other
`option <https://www.redhat.com/archives/freeipa-devel/2016-May/msg00416.html>`__
would be to run the provisioning on the IPA master and provision on
**ldapi**. The advantages would be to

-  use autobind without the need of DM password.
-  disable ldap ports so that we are sure no ldap client can receive
   invalid data

   -  Note that the replication to the IPA master will be stopped



Replication being late
                      

Disabling memberof during provisioning allows a *faster* provisioning.
Actually much faster than the same update on a replica where memberof is
enabled.

If we are doing provisioning in a topology with single instance this is
not an issue. But if there are replicas, replication will send added
entries and on replicas the *replicated provisioning* will be processed
much slower.

The consequence is that replicas will be **very late** (and possibly may
require some tuning of the **flow control** of the replication)

For example provisioning of a `medium size
DB <http://www.freeipa.org/page/V4/Performance_Improvements#Medium_DB_.28100K_entries.29>`__
can put replicas **days behind** the provisioned replica. In such case a
provision rule (hbac, sudo,...) can exist on the provisioned replica but
will not exist for a long time on the others. If that rule grants some
rights it can create security issue.

in conlusion:

-  it is recommended to not use *fast* provisioning on a replicated
   topology unless it is planed to reinitialize all replicas from the
   provisioned one.



Fixup procedure
               

Fixup is a procedure to compute the **memberof** attribute for a **set
of entries**. This set is selected with a filter so if for example we
added *host* entries, we can run the fixup command using the
*"(objectclass=ipaHost)"*.

A difficulty is to fixup **all** the provisioned entries so it is
important to identify the filters that will cover all the provisioned
entries. For example if we provision
*user/usergroup/host/hostgroup/sudorules/hbacrules* the following set of
filters will fixup all the them

| ``(objectClass=inetorgperson)``
| ``(objectClass=ipausergroup)``
| ``(objectClass=ipahost)``
| ``(objectClass=ipahostgroup)``
| ``(objectClass=ipasudorule)``
| ``(objectClass=ipahbacrule)``

A second difficulty is to have filters that do not overlap. Else we will
fixup several times the same entries. For example adding
*usergroup/hostgroup* the following set of filters overlaps because
*hostgroup* also match the first filter.

| ``(objectClass=groupofnames)``
| ``(objectClass=ipahostgroup)``

A third difficulty is if provisioning is adding entries (e.g. user) in a
server where it already exists others users. In that case the filter
*(objectClass=inetorgperson)* will fixup the provisioned entries (that
need to be fixup) as well as already existing ones (that do not need
fixup).



provisioning command
''''''''''''''''''''

The administrator who wants to do a bulk load of a set of LDAP entries
that are contained in a ldif-file can use the command:

-  ipa provision *ldif_entries_file* [--password-file *password_file*]

*ldif_entries_file* contains the entries in a ldif format

*password_file* is a readable file that contains the *directory manager*
password



Detailed descriptions of each provisioning costs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The objectif is to determine what makes memberof plugin so expensive
compare to memberof fixup. The following paragraphs are a summary of the
tests/results. No design or improvements are described in those
paragraphs.



summary of the test
'''''''''''''''''''

The provisioning adds in the following order users, groups of users,
hosts, groups of hosts, sudorules and hbacrules. The specifications
entries are:

-  100 users
-  20 users groups

   -  10 empty groups
   -  10 groups with 100 users + 1 nested group

-  80 hosts
-  20 hosts groups

   -  10 empty groups
   -  10 groups with 40 hosts + 1 nested group

-  100 sudorules

   -  20 with 25 users and 20 hosts
   -  80 with 1 host group

-  100 hbacrules

   -  20 with 25 users and 20 hosts
   -  80 with 1 host group

The overall time spent to provision all these objects

============ ===============
Objects      memberof plugin
enabled      disabled
add obj.     fixup
Users        3sec
Users groups 7sec
Hosts        1sec
Hosts groups 5sec
Sudorules    16sec
Hbacrules    38sec
\            70 seconds
============ ===============

Note these values are taken for quite *small* groups. So the ratio
with/without memberof is only **6 times**. The ratio found in with
`larger <http://www.freeipa.org/page/V4/Performance_Improvements#improvement_of_the_throughput_with_admin_period>`__
groups (5000) raise up to **20 times**. It is likely that with very
large groups (100K and above), the ratio would be **much higher**.

The comparison of the **ADD** when the memberof plugin is enabled vs.
disabled is **15 times less** and is presented in the table below

''Note the values are only for non empty groups (user/host)"

============ ===============
Objects      memberof plugin
enabled      disabled
Users        6
Users groups 105
Hosts        2
Hosts groups 90
Sudorules    47
Hbacrules    47
\            297
============ ===============

The comparison of the **MOD** when the memberof plugin is enabled vs.
disabled is **35 times less** presented in the table below

''Note the values are only for non empty groups (user/host)"

============ ===============
Objects      memberof plugin
enabled      disabled
Users        4
Users groups 104
Hosts        0
Hosts groups 88
Sudorules    45
Hbacrules    45
\            286
============ ===============

The comparison of the **SRCH** when the memberof plugin is enabled vs.
disabled is **3.3 times less** presented in the table below

''Note the values are only for non empty groups (user/host)"

============ ===============
Objects      memberof plugin
enabled      disabled
Users        22
Users groups 1342
Hosts        7
Hosts groups 718
Sudorules    918
Hbacrules    1313
\            4320
============ ===============



provisioning with memberof plugin
'''''''''''''''''''''''''''''''''



add users
         

The add of **one** user triggers the following operations (1 direct, 31
internals): 6 ADDs, 4 MODs, 22SRCHs

``Details:``

| ``ADD a user``
| ``   22 SRCHs``
| ``       5 for uniqueness (ipaUniqueID, krbPrincipalName, uid, uidNumber, gidNumber)``
| ``       3 for DNA config update (2 identicals (*))``
| ``       2 for DNA shared config (2 identicals (*))``
| ``       4 for group membership of the added user  (2 identicals (*))``
| ``       4 for group membership of the added private group  (2 identicals (*))``
| ``       2 for group membership``
| ``       2 for updating the added user with its private group``
| ``    4 MODs``
| ``       1 for DNA config``
| ``       1 for DNA shared config``
| ``       2 for updating the added user with its private group/entryusn (curiously the first update fails with LDAP_TYPE_OR_VALUE_EXISTS)``
| ``    6 ADD``
| ``       user ADD``
| ``       private group ADD``
| ``       retroCL log of ADD user ``
| ``       retroCL log of MOD of DNA share config``
| ``       retroCL log of ADD private group``
| ``       retroCL log of MOD user (adding its private group)``
| ``(*) Searches are identicals``



add a usergroup
               

The add of **one** user group triggers the following operations:

-  If the group is empty (1 direct, 31 internals): 3 ADDs, 2 MODs,
   15SRCHs

``Details:``

| ``ADD an empty usergroup``
| ``   15 SRCHs``
| ``       3 for uniqueness (ipaUniqueID, uidNumber, gidNumber)``
| ``       3 for DNA config update (3 identicals (*))``
| ``       2 for DNA shared config (2 identicals (*))``
| ``       1 for ?? (lookup objectclass=ipantdomainattrs)``
| ``       2 for group members (2 identicals (*))``
| ``       4 for group membership of the added user group  (2 identicals (*))``
| ``    2 MODs``
| ``       1 for DNA config``
| ``       1 for DNA shared config``
| ``    3 ADD``
| ``       user group``
| ``       retroCL log of ADD user ``
| ``       retroCL log of MOD of DNA share config``

-  If the group contains 102 members (100+2nested) (1 direct, 105ADD,
   104 MOD, 1342 SRCH)

``Details:``

ADD usergroup with 100 user member and 2 nested groups

| ``   1342 SRCHs``
| ``       3 for uniqueness (ipaUniqueID, uidNumber, gidNumber)``
| ``       3 for DNA config update (3 identicals (*))``
| ``       2 for DNA shared config (2 identicals (*))``
| ``       1 for ?? (lookup objectclass=ipantdomainattrs)``
| ``       1 for group members``
| ``       202 = 2 identical searchs per direct members  (retrieve all attribute including member that are lookup below)``
| ``       101 = searchs for members of each direct member [435]``
| ``         2 = 2 indentical search per indirect members (retrieve all attribute including member that are lookup below)``
| ``         1 = searchs for members of each indirect member``
| ``       102 = search for 'uid' of each direct/indirect members [643]``
| ``       1 for group members [847]``
| ``       202 = 2 identical searchs per direct members  (retrieve all attribute including member that are lookup below)``
| ``       101 = searchs for members of each direct member [1254] (slapi-nis ?)``
| ``         2 = 2 indentical search per indirect members (retrieve all attribute including member that are lookup below)``
| ``         1 = searchs for members of each indirect member``
| ``       ``
| ``       1 for group members [1459]``
| ``       103 = search for members direct/indirect of the group 'ipaexternalmember' (slapi-nis ?)``
| ``       4 search for group memberships [1665]``
| ``       for each member (total srch = 510 (102*5), 102 ADD, 102 MOD)``
| ``           1 search "member memberUser memberHost"``
| ``           1 search group owner of the member``
| ``           1 search group owner of the usergroup (done at each iteration)``
| ``           1 MOD + 1 ADD (see MOD/ADD)``
| ``           2 search of the member (2 identical)``
| ``   104 MODs``
| ``       1 for DNA config``
| ``       1 for DNA shared config``
| ``       for each member (102)``
| ``           MOD users to add 'memberof'``
| ``       ``
| ``   105 ADDs``
| ``       user group``
| ``       for each member (102)``
| ``               RetroCL log for above MODs (MOD member to add 'memberof')``



add host
        

The add of **one** host triggers : 2 ADD, 7 SRCHs

``Details:``

| ``ADD a host``
| ``   7 SRCH``
| ``       2 search (uniqueness ipaUniqueID, krbPrincipalName)``
| ``       4 membership search (2 identical)``
| ``       1 search for group from 'ipantdomainattrs'``
| ``   2 ADD``
| ``       add host``
| ``       RetroCL add``



add a hostgroup
               

The add of **one** host group triggers the following operations:

-  If the group is empty (1 direct, 39 internals): 5 ADDs, 3 MODs,
   32SRCHs

``Details:``

| ``ADD empty hostgroup``
| ``   32 SEARCHES``
| ``       1 search (uniqueness ipaUniqueID)``
| ``       4 membership search (2 identical)``
| ``       5 search of the alt networkgroup (3 for 'member', 1 for 'memberuser', 1 for 'memberhost')``
| ``       6 searches of added hostgroup (2 for ALL, 1 for 'memberuser', 1 for 'memberhost, 1 for 'fqdn', 1 for "member memberUser  memberHost")``
| ``       8 searches to find groups owning alt networkgroup``
| ``       2 searches to find groups owning hostgroup``
| ``       4 search of add hostgroup (4 identical) related to MODs``
| ``       1 search for group from 'ipantdomainattrs'``
| ``           ``
| ``   3 MOD``
| ``       1 update hostgroup to 'memberof' alt networkgroup (memberof plugin)``
| ``       1 update hostgroup to 'mepManagedEntry' alt networkgroup (mep plugin) ((curiously the first update fails with LDAP_TYPE_OR_VALUE_EXISTS)``
| ``   5 ADD``
| ``       add hostgroup``
| ``       add hostgroup alt networkgroup (slapi-nis)``
| ``       3 retroCL``

-  If the hostgroup contains 42 members (40 direct, 2 nested) (1 direct,
   895 internals): 90 ADDs, 88 MODs, 718 SRCHs


::

   ``Details:``

   | ``ADD hostgroup with 42 members (nested)``
   | ``   718 SRCH``
   | ``       1 search (uniqueness ipaUniqueID)``
   | ``       4 membership search (2 identical)``
   | ``       5 search of the alt networkgroup (3 for 'member', 1 for 'memberuser', 1 for 'memberhost')``
   | ``       for each member (42): total = 84srch``
   | ``               2 search of the member entry (identical BUG)``
   | ``    ``
   | ``       for each member (42): total = 84``
   | ``               1 search of 'member' ``
   | ``               1 search of 'fqdn'``
   | ``       10 search to find groups owning hostgroup (4 identical )``
   | ``       for each member (42): total = 252srch [405->1125]``
   | ``           /* related to the MOD 'memberof' of the member */``
   | ``           1 search to find the member "member memberUser memberHost"``
   | ``           1 search to find groups owning member``
   | ``           2 search to find groups owning hostgroup (identical BUG + same search for each member)``
   | ``           2 search member during MOD (identical BUG ?)``
   | ``       for each member (42): total = 252srch [1125->1760]``
   | ``           /* related to the second "BUGGY" MOD 'memberof' of the member */``
   | ``           1 search to find the member "member memberUser memberHost"``
   | ``           1 search to find groups owning member``
   | ``           2 search to find groups owning hostgroup (identical BUG + same search for each member)``
   | ``           2 search member during MOD (identical BUG ?)``
   | ``    87 MOD``
   | ``       for each host in hostgroup [418]``
   | ``           update 'memberof' for hostgroup and alt networkgroup``
   | ``       for each host in hostgroup (Yes this is done twice ! BUG) [1122]``
   | ``           update 'memberof' for hostgroup and alt networkgroup``
   | ``       update hostgroup for 'mepmanageentry'``
   | ``        ``
   | ``    90 ADD``
   | ``      add hostgroup``
   | ``      add alt networkgroup``
   | ``      88 RetroCL add due to MODs``



add sudorules
             

Adding **one** sudorule with 25 users/20 hosts, triggers the following
internal operations 47 ADDs, 45 MODs and 918 SRCH

::

   ``Details:``

   | ``ADD sudorules 25 users/20 hosts``
   | ``   918 SRCH``
   | ``       1 search (uniqueness ipaUniqueID)``
   | ``           /* Follow comes slapi-nis 'cn=sudoers,cn=Schema Compatibility' */``
   | ``               for each memberHost (20): 40``
   | ``                   2 search host (2 identical BUG - objectclass=ipaHostGroup)(!(objectclass=mepOriginEntry))``
   | ``                   ``
   | ``               for each memberuser (25): 25``
   | ``                   1 search 'cn'``
   | ``               for each memberHost (20): 20``
   | ``                   1 search host ((objectclass=ipaHostGroup)(objectclass=mepOriginEntry))``
   | ``               for each memberUser (25): 25 ``
   | ``                   1 search 'uid'``
   | ``               for each memberHost (20): 20``
   | ``                   1 search host (ipaNisNetgroup)``
   | ``               for each memberHost (20): 20``
   | ``                   1 search host (objectclass=ipaHost)``
   | ``               for each memberUser (25): 50``
   | ``                   2 search host (2 identical BUG - (objectclass=ipaUserGroup)(!(objectclass=posixGroup))``
   | ``               for each memberUser (25):  25``
   | ``                   1 search user (objectclass=ipaNisNetgroup)``
   | ``       10 searchs to find if add sudorules belong to a group``
   | ``       For each memberUser (25):``
   | ``           /* search all groups it can belong to */``
   | ``           10 search based on member 'memberof'``
   | ``   45 MOD``
   | ``       for each users:``
   | ``           update memberof attribute to add the 'ipaUniqueID=xxx,cn=sudorules,cn=sudo,``\ ``' value``
   | ``       for each host:``
   | ``           update memberof attribute to add the 'ipaUniqueID=xxx,cn=sudorules,cn=sudo,``\ ``' value``
   | ``   47 ADD``
   | ``       add sudorule``
   | ``       RetroCL add sudorule + 45 updates of memberof (MODs)``



add hbacrules
             

Adding **one** sudorule with 25 users/20 hosts, triggers the following
internal operations 47 ADDs, 45 MODs and 1313 SRCH

``Details:``

| ``ADD hbacrule 25 users/20 hosts``
| ``   1313 SRCH``
| ``       For each memberUser 25: ``
| ``           search the groups it belongs to (17)``
| ``       For each memberHost 20: ``
| ``           search the groups it belongs to (40)``
| `` ``
| ``   45 MOD``
| ``       for each users:``
| ``           update memberof attribute to add the 'ipaUniqueID=xxx,cn=hbacrules,cn=hbac,``\ ``' value``
| ``       for each host:``
| ``           update memberof attribute to add the 'ipaUniqueID=xxx,cn=hbacrules,cn=hbac,``\ ``' value``
| ``   47 ADD``
| ``       add hbacrule``
| ``       RetroCL add hbacrule + 45 updates of memberof (MODs)``



provisioning without memberof plugin
''''''''''''''''''''''''''''''''''''



add user
        

The add of **one** user gives same results as `add user with memberof
plugin <http://www.freeipa.org/page/V4/Performance_Improvements#add_users>`__



add usergroup (no memberof)
                           

The add of **one** user group triggers the following operations:

-  If the group is empty (1 direct, 19 internals): 3 ADDs, 2 MODs,
   15SRCHs - this is identical results vs add an empty user group `with
   memberof <http://www.freeipa.org/page/V4/Performance_Improvements#add_a_usergroup>`__
-  If the group contains 102 members (100+2nested) (1 direct, 3ADD, 2
   MOD, 813 SRCH)

``Details:``

| ``   813 SRCHs``
| ``       3 for uniqueness (ipaUniqueID, uidNumber, gidNumber)``
| ``       3 for DNA config update (3 identicals (*))``
| ``       2 for DNA shared config (2 identicals (*))``
| ``       1 for ?? (lookup objectclass=ipantdomainattrs)``
| ``       A) for each group members (102): (total 204)``
| ``           2 identical base search of the member all_attr (BUG)``
| ``       B) for each group members (102): (total 102)``
| ``           base search of the member 'member' (BUG it could reuse the A)``
| ``       C) for each group members (102): (total 102)``
| ``           base search of the member 'uid' (BUG it could reuse the A)``
| ``       D) identical to A (total 102)``
| ``       E) identical to B (total 102)``
| ``       F) for each group members (102): (total 102)``
| ``           base search of the member 'ipaexternalmember' (BUG it could reuse the A)``
| ``   2 MODs                                                                                                                          ``
| ``       1 for DNA config``
| ``       1 for DNA shared config``
| ``   3 ADDs``
| ``       user group``
| ``       RetroCL for user_group and MOD DNA``



add host
        

The add of **one** host gives same results as `add host with memberof
plugin <http://www.freeipa.org/page/V4/Performance_Improvements#add_host>`__



add hostgroup
             

The add of **one** hostgroup triggers the following operations:

-  If the hostgroup is empty (1 direct, 34 internals): 5 ADDs, 3 MODs,
   27SRCHs

It gives results **almost** identical to `add an empty hostgroup with
memberof
plugin <http://www.freeipa.org/page/V4/Performance_Improvements#add_a_hostgroup>`__.
But memberof plugin triggers 5 more internal searches (2 membership and
3 on the added hostgroup), so running without memberof plugin **saves 5
SRCHs**.

-  if the hostgroup contains 42 members (40 direct, 2 nested) (1 direct,
   895 internals): 5 ADDs, 2 MODs, 201 SRCHs

``Details:``

| ``ADD hostgroup with 42 members (nested)``
| ``   201 SRCH``
| ``       1 search (uniqueness ipaUniqueID)``
| ``       4 membership search on netgroup``
| ``       4 membership search on groups``
| ``       5 search on add hostgroup (2 ALL, 1 'member', 1 'fqdn', 1 'memberHost' , 1 'member')``
| ``       for each member (42): total = 84srch``
| ``               2 search of the member entry (identical BUG)``
| ``    ``
| ``       for each member (42): total = 84``
| ``               1 search of 'member' ``
| ``               1 search of 'fqdn'``
| ``       9 searches to find groups (ng, users, groups, computers, hostgoups) owning the added hostgroup``
| ``                                                                                                                                   ``
| ``    2 MOD``
| ``       2 update hostgroup to 'mepManagedEntry' alt networkgroup (mep plugin) (the first MOD fails with LDAP_TYPE_OR_VALUE_EXISTS)``
| ``       ``
| ``    5 ADD``
| ``      add hostgroup``
| ``      add alt networkgroup``
| ``       RetroCL add due to MODs``



add sudorule
            

Adding **one** sudorule with 25 users/20 hosts, triggers the following
internal operations: 2 ADD, 0 MOD, 243 SRCH


::

   ``Details:``

   | ``ADD sudorules 25 users/20 hosts``
   | ``   243 SRCH``
   | ``       1 search (uniqueness ipaUniqueID)``
   | ``               for each memberHost (20): 40``
   | ``                   2 search host all_attrs (2 identical BUG - objectclass=ipaHostGroup)(!(objectclass=mepOriginEntry))``
   | ``                   ``
   | ``               for each memberuser (25): 25``
   | ``                   1 search 'cn'``
   | ``               for each memberHost (20): 20``
   | ``                   1 search host 'cn' ((objectclass=ipaHostGroup)(objectclass=mepOriginEntry))``
   | ``               for each memberUser (25): 25 ``
   | ``                   1 search 'uid'((objectclass=posixAccount))``
   | ``               for each memberHost (20): 20``
   | ``                   1 search host 'cn' ((objectclass=ipaNisNetgroup))``
   | ``               for each memberHost (20): 20``
   | ``                   1 search host 'fqdn' (objectclass=ipaHost)``
   | ``               for each memberUser (25): 50``
   | ``                   2 search host (2 identical BUG - (objectclass=ipaUserGroup)(!(objectclass=posixGroup))``
   | ``               for each memberUser (25):  25``
   | ``                   1 search user (objectclass=ipaNisNetgroup)``
   | ``       10 searchs to find if added sudorules belong to a group (user/ng/hostgroups/grous/computers)``
   | `` ``
   | ``       For each memberUser (25):``
   | ``           /* search all groups it can belong to */``
   | ``           10 search based on member 'memberof'``
   | `` ``
   | ``   0 MOD``
   | `` ``
   | ``   2 ADD``
   | ``       add sudorule``
   | ``       RetroCL add sudorule``



add hbacrules
             

Adding **one** sudorule with 25 users/20 hosts, triggers the following
internal operations 2ADD, 0 MOD, 13 SRCH

::

   ``Details:``

   | ``ADD hbacrule 25 users/20 hosts``
   | ``   13 SRCH``
   | ``       1 search (uniqueness ipaUniqueID)``
   | ``       10 searchs to find if added hbacrules belong to a group (user/ng/hostgroups/grous/computers)``
   | ``       1 unindexed search in sudorules if one of them owns the added hbacrule``
   | `` ``
   | ``           (&(&(objectclass=ipaSudoRule)``
   | ``               (!(compatVisible=FALSE))``
   | ``               (!(ipaEnabledFlag=FALSE)))``
   | ``             (|(memberUser=ipauniqueid=22f91e42-0d34-11e6-9927-001a4a2314dc,cn=hbac,``\ ``)``
   | ``               (memberHost=ipauniqueid=22f91e42-0d34-11e6-9927-001a4a2314dc,cn=hbac,``\ ``)``
   | ``               (ipaSudoRunAsGroup=ipauniqueid=22f91e42-0d34-11e6-9927-001a4a2314dc,cn=hbac,``\ ``)``
   | ``               (memberAllowCmd=ipauniqueid=22f91e42-0d34-11e6-9927-001a4a2314dc,cn=hbac,``\ ``)                                  ``
   | ``               (ipaSudoRunAs=ipauniqueid=22f91e42-0d34-11e6-9927-001a4a2314dc,cn=hbac,``\ ``)``
   | ``               (memberDenyCmd=ipauniqueid=22f91e42-0d34-11e6-9927-001a4a2314dc,cn=hbac,``\ ``))``
   | ``           )``
   | `` ``
   | ``   0 MOD``
   | `` ``
   | ``   2 ADD``
   | ``       add hbacrule``
   | ``       RetroCL add hbacrule``



memberof fixup
              

============= =========
filter OC     Operation
ADD           MOD
inetorgperson 100
============= =========

Hypothese
^^^^^^^^^

The preliminary tests of `memberof
fixup <http://www.freeipa.org/index.php?title=V4/Performance_Improvements&action=submit#improvement_of_the_throughput_with_admin_period>`__,
shows that both procedures are equivalent in terms of final results but
much faster (fixup) in term of throughput.

A possible explanation is that each time we add a group with members, it
triggers the recomputation of the 'memberof' attribute. It is time
consuming (internal search) because if an entry is member of N groups
(direct or nested) and those N groups are composed of M entries. When
the entry is added to a new goup, memberof plugin recomputes 'memberof'
attribute and needs to lookup each of the M entries to know if they are
themself groups.

There is a waste of time if a group/member was evaluated when adding an
entry and need to be evaluated again when adding a second entry.

With fixup we do this evaluation only **once**

Note the 389-ds memberof `RFE
47963 <https://fedorahosted.org/389/ticket/47963>`__ has no impact on
performace with the current use case. In fact, freeipa uses nested group
but perf hit is not due to nested groups.



all commands
----------------------------------------------------------------------------------------------

Caching issue described in
`1 <http://www.freeipa.org/page/V4/Performance_Improvements#broken_SchemaCache>`__
affects all IPA commands.



all commands working with members and indirect members
----------------------------------------------------------------------------------------------

Related ticket(s):
`#4995 <https://fedorahosted.org/freeipa/ticket/4995>`__

Get member and indirect members is resource consuming operation and
usually user don't want all membership details. IPA already has hidden
option *--no-members* that can be public visible.

Summary: option *--no-members* is publicly visible for all commands

\*-find
----------------------------------------------------------------------------------------------



members and indirect members processing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Related ticket(s):
`#4995 <https://fedorahosted.org/freeipa/ticket/4995>`__

``host-find (2000 hosts):``

| ``76640658 function calls (75069144 primitive calls) in ``\ **``227.351``**\ `` seconds``
| `` ``
| ``   Ordered by: cumulative time``
| `` ``
| ``   ncalls  tottime  percall  cumtime  percall filename:lineno(function)``
| `` ....``
| ``        1    0.103    0.103  227.348  227.348 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:2015(execute)``
| ``73967/73966    3.240    0.000  186.341    0.003 /usr/lib/python2.7/site-packages/ipapython/ipaldap.py:1272(find_entries)``
| ``   247887    1.882    0.000  131.877    0.001 /usr/lib64/python2.7/site-packages/ldap/ldapobject.py:87(_ldap_call)``
| ``   173920    0.392    0.000  127.617    0.001 /usr/lib64/python2.7/site-packages/ldap/ldapobject.py:472(result3)``
| ``   173920    0.953    0.000  127.225    0.001 /usr/lib64/python2.7/site-packages/ldap/ldapobject.py:480(result4)``
| ``   173920  123.784    0.001  123.784    0.001 {built-in method result4}``
| ``     2000    2.283    0.001  ``\ **``111.509``**\ ``    0.056 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:637(``\ **``convert_attribute_members``**\ ``)``
| ``     2000    0.014    0.000  ``\ **``104.078``**\ ``    0.052 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:672(``\ **``get_indirect_members``**\ ``)``
| ``     2000    0.249    0.000  104.064    0.052 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:706(get_memberofindirect)``
| ``    77961    0.571    0.000   ``\ **``85.341``**\ ``    0.001 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:598(``\ **``get_primary_key_from_dn``**\ ``)``
| ``    67965    0.323    0.000   79.816    0.001 /usr/lib/python2.7/site-packages/ipapython/ipaldap.py:1415(get_entry)``
| ``   173919    1.286    0.000   23.806    0.000 /usr/lib/python2.7/site-packages/ipapython/ipaldap.py:895(_convert_result)``
| ``   283906    0.407    0.000   16.624    0.000 /usr/lib/python2.7/site-packages/ipapython/dn.py:1265(endswith)``
| ``   283906    0.996    0.000   16.077    0.000 /usr/lib/python2.7/site-packages/ipapython/dn.py:1280(_tailmatch)``
| `` ....``

As is show in output of profiler, the most time consuming operations are
**convert_attribute_members**, **get_indirect_members**,
**get_primary_key_from_dn**

Possible solutions:



Do not fetch members by default
'''''''''''''''''''''''''''''''

This change is related to all \*-find commands. Fetching members and
indirect members is expensive operation for find commands. By default
\*-find commands will not do members processing. To get members in
\*-find command option *--all* should be used.

Note: this changes makes output of \*-find commands backward
incompatible.

Note: due API backward compatibility option *--no-members* must be still
present even if it has no effect on \*-find commands. This option can be
hidden in CLI for \*-find commands

Note: user-find already does not return members in result without --all
option



Temporal caching of members during \*-find command
''''''''''''''''''''''''''''''''''''''''''''''''''

**This has not been implemented in 4.4, due technical issues with cache.
Prototype of the cache does not cover corner cases, so time was not
reduced as much as listed here. There was only minor enhancement and was
decided to postpone this**

Caching may heavily reduce amount of ldapsearches and internal framework
operations.

Test with cache only for **convert_attribute_members** method reduces
total time of operation from 227.351 (111.509) to 113.474 (3.892)
seconds

| `` 16803443 function calls (16602409 primitive calls) in ``\ **``113.474``**\ `` seconds``

| ``   Ordered by: cumulative time``
| `` ``
| ``   ncalls  tottime  percall  cumtime  percall filename:lineno(function)``
| ``        1    0.031    0.031  113.471  113.471 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:2015(execute)``
| ``8137/8136    0.512    0.000  103.554    0.013 /usr/lib/python2.7/site-packages/ipapython/ipaldap.py:1272(find_entries)``
| ``     2000    0.013    0.000   98.526    0.049 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:672(get_indirect_members)``
| ``     2000    0.254    0.000   98.513    0.049 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:706(get_memberofindirect)``
| ``    50397    0.342    0.000   93.376    0.002 /usr/lib64/python2.7/site-packages/ldap/ldapobject.py:87(_ldap_call)``
| ``....``
| ``    44123    0.874    0.000    4.029    0.000 /usr/lib64/python2.7/site-packages/ldap/dn.py:56(dn2str)``
| ``     2000    0.321    0.000    ``\ **``3.892``**\ ``    0.002 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:2120(``\ **``convert_attribute_members``**\ ``)``
| ``     2000    0.039    0.000    3.204    0.002 /usr/lib/python2.7/site-packages/ipalib/util.py:293(convert_sshpubkey_post)``
| ``   469301    1.701    0.000    2.919    0.000 /usr/lib64/python2.7/site-packages/ldap/dn.py:20(escape_dn_chars)``
| `` ....``
| ``     2161    0.012    0.000    ``\ **``0.233``**\ ``    0.000 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:598(``\ **``get_primary_key_from_dn``**\ ``)``
| `` ....``

For case when

``number of groups/sudorules/hostgroups/hbacrules/roles ``\ **``<<``**\ `` number of users/host``

the cache is very effective. In other way cache can cause small slowdown
but it should not be very noticeable.

The cache must be invalidated after each \*-find call. There is no need
for having outdated copy of ldap data.

**Indirect members**

Now the most time consumig operation is getting indirect members:

| ``     2000    0.013    0.000   98.526    0.049 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:672(get_indirect_members)``
| ``     2000    0.254    0.000   98.513    0.049 /usr/lib/python2.7/site-packages/ipalib/plugins/baseldap.py:706(get_memberofindirect)``

For indirect members, each entry currently requires 2 LDAP searches.
Implemented search are very effective, but results are not usable for
caching (because each search returns entries specific for the current
entry). The code might be rewritten to get nested entries per
group/hostgroup and store it in cache to be able reuse results. However
this change is not trivial with lot of caveats and might not bring too
much performance. For now we can keep conversion of indirect members as
it is.

Other possibilities are:

-  just do direct membership and add option to enable
   indirect-membership
-  don't do indirect membership at all
-  try to implement cache for indirect membership



Test Plan
---------

`Performance Improvements V4.4 test
plan <V4/Performance_Improvements/Test_Plan>`__