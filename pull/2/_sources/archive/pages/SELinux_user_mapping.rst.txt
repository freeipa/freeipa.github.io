Purpose
-------

Manage mapping of users stored in IPA to SELinux users.

Storage
-------

IPA will store one new type of entry and two new pieces of
configuration.

| ``attributeTypes: (2.16.840.1.113730.3.8.11.10``
| ``  NAME 'ipaSELinuxUser'``
| ``  DESC 'An SELinux user'``
| ``  EQUALITY caseIgnoreMatch``
| ``  ORDERING caseIgnoreOrderingMatch``
| ``  SUBSTR caseIgnoreSubstringsMatch``
| ``  SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| ``  SINGLE-VALUE``
| ``  X-ORIGIN 'IPA v3'``
| ``)``

| ``objectClasses: (2.16.840.1.113730.3.8.12.5``
| ``  NAME 'ipaSELinuxUserMap'``
| ``  SUP ipaAssociation STRUCTURAL``
| ``  MUST SELinuxUser``
| ``  MAY ( accessTime $ seeAlso )``
| ``  X-ORIGIN 'IPA v3'``
| ``)``

-  SELinuxUser is the SELinux user this rule maps to.
-  accessTime is a FutureFeature.
-  seeAlso is the dn of an HBAC rule defining the users and hosts for
   this map

We will store these rules in cn=selinux,$SUFFIX

Some examples:

| ``dn: ipauniqueid=d4d97a3a-1167-11e1-9dea-0050562c8d82,cn=selinux,dc=example,dc=com``
| ``cn: Staff on rawhide``
| ``ipaenabledflag: TRUE``
| ``memberhost: fqdn=rawhide.example.com,cn=computers,cn=accounts,dc=example,dc=com``
| ``memberuser: uid=joe.user,cn=users,cn=accounts,dc=example,dc=com``
| ``accessruletype: allow``
| ``ipaselinuxuser: staff_u:s0-s0:c0.c1023``
| ``ipauniqueid: d4d97a3a-1167-11e1-9dea-0050562c8d82``
| ``objectclass: ipaassociation``
| ``objectclass: ipaselinuxusermap``

| ``dn: ipauniqueid=d4d97a3a-1167-11e1-9dea-0050562c8d82,cn=selinux,dc=example,dc=com``
| ``cn: User using hbac``
| ``ipaenabledflag: TRUE``
| ``seeAlso: ipauniqueid=79a60542-1168-11e1-851d-0050562c8d82,cn=hbac,dc=example,dc=com``
| ``ipaselinuxuser: user_u:s0-s0:c0.c1023``
| ``ipauniqueid: d4d97a3a-1167-11e1-9dea-0050562c8d82``
| ``objectclass: ipaassociation``
| ``objectclass: ipaselinuxusermap``

The other two values will be stored in cn=ipaConfig.

The first value is the order list of SELinux users we can map to in
ascending order of priority. This list will use $ as a separator.

| ``attributeTypes: ( 2.16.840.1.113730.3.8.3.27``
| ``  NAME 'ipaSELinuxUserMapOrder'``
| ``  DESC 'Available SELinux user context ordering'``
| ``  EQUALITY caseIgnoreMatch``
| ``  ORDERING caseIgnoreMatch``
| ``  SUBSTR caseIgnoreSubstringsMatch``
| ``  SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| ``  SINGLE-VALUE``
| ``  X-ORIGIN 'IPA v3'``
| ``)``

This table will look like:

``guest_u:s0$xguest_u:s0$user_u:s0-s0:c0.c1023$staff_u:s0-s0:c0.c1023$unconfined_u:s0-s0:c0.c1023``

The second value is the default SELinux user, e.g. guest_u:s0

| ``attributeTypes: ( 2.16.840.1.113730.3.8.3.26``
| ``  NAME 'ipaSELinuxUserMapDefault'``
| ``  DESC 'Default SELinux user'``
| ``  EQUALITY caseIgnoreMatch``
| ``  ORDERING caseIgnoreMatch``
| ``  SUBSTR caseIgnoreSubstringsMatch``
| ``  SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| ``  SINGLE-VALUE``
| ``  X-ORIGIN 'IPA v3'``
| ``)``

An HBAC rule pointed to by an SELinux mapping may not be removed until
the mapping rule is removed. The HBAC rule does not store a pointer,
rather it will need to search the maps to see if it is being referenced.

An SELinux user may not be removed from the ordered list if it appears
in any of the mapping rules.

An SELinux user may not be removed from the order list if it is the
default.

The default value must always be a member of the list.

The default user context may be blank/empty. An empty member tells sssd
to use the system default context.

The default user context in IPA is unconfined_u.

A map must contain both a (user/group or usercat) and (host/hostgroup or
hostcat).

If it points to an HBAC rule then that rule must conform to the above.
Otherwise the mapping rule is ignored.

seeAlso may not be defined if a user, usercat, host or hostcat is
defined. The use must either link to an HBAC rule or define everything
in the SELinux map.

.. _selinux_user_syntax:

SELinux user syntax
-------------------

An SELinux user has 3 components: ``user:MLS:MCS``. ``user`` and ``MLS``
are mandatory, so ``user:MLS:MCS`` or only ``user:MLS`` are also valid
(`relevant
Bugzilla <https://bugzilla.redhat.com/show_bug.cgi?id=885181>`__).

User traditionally ends with \_u but this is not mandatory. It may
contain letters or underscores, but must start with a letter.

The MLS part can only be s[0-15] (a single level), or s[0-15]-s[0-15] (a
range of levels).

Then MCS could be c[0-1023] (a single category), c[0-1023].c[0-0123] (a
range of categories), or any number of these separated by commas.

For example, the following are valid:

| ``user_u:s0``
| ``user_u:s0-s1``
| ``user_u:s0-s15:c0.c1023``
| ``user_u:s0-s1:c0,c2,c15.c26``
| ``user_u:s0-s0:c0.c1023``

See `SELinux
documentation <http://docs.fedoraproject.org/en-US/Fedora/21/html/SELinux_Users_and_Administrators_Guide/index.html>`__
for more details on MCS/MLS levels. Note that IPA only does a
rudimentary sanity check, so it may allow illegal values in some cases.

Evaluation
----------

IPA stores the maps, the ordered list of SELinux contexts and the
default context.

SSSD evaluates the maps to determine the correct context based on the
user and machine.

.. _order_of_operation:

Order of operation
~~~~~~~~~~~~~~~~~~

The maps are a triple of (host, user, selinuxuser)

host can be a single host, a hostgroup, or hostcat=all user can be a
single user, a group, or usercat=all selinuxuser is a single value

Matching is done from most to least-specific. You can think of this as
levels where user > group > usercat=all and host > hostgroup >
hostcat=all.

The host is checked first, then user.

If two matches on the same level are found then the ordered list of
selinux users is used to determine the winner. The last (highest
priority) in the list wins.

If after all this no match is found the default selinux user is used.

Since we can potentially point to another association (HBAC) which has
its own enabled flag both will need to be evaluated. If either is
disabled then the rule is ignored.

.. _evaluating_rules:

Evaluating Rules
~~~~~~~~~~~~~~~~

When a user attempts to log in we'll know two things: the uid of the
user logging in and the name of the host we're on.

For determining which rules apply to the user you have to check two
things:

1. Pull the user's entry and find any memberof in the SELinux user map
container, it will look like:

memberof:
ipaUniqueID=f99b9a7c-19f2-11e1-94cc-0050562c8d82,cn=usermap,cn=selinux,dc=example,dc=com

2. Find all the SELinux rules with seeAlso set and see if that DN is in
the user's memberof.

Do the same thing with the host, combine the two sets and you have your
set of rules.

The next calculation is more complex as we decided that specificity wins
(e.g. a rule with a specific user has more weight than a group or \*).
To determine the correct user context, iterate through the unordered set
of candidate rules and comparing current state to the rule.

The initial context is the default SELinux user.

The first rule context gets applied to the user and we also track
specificity of host and user (could be an enum: direct, group,
wildcard).

In the next rule we look to see if host or user is more specific and if
so we get that context, otherwise we punt.

If it is equally specific we take the highest context as defined in the
context ordering.

Do this until the list is exhausted. The final state is the context to
set.

Examples
~~~~~~~~

These rules are in the form (host, user, mapping).

Ordering is: ``guest_u$staff_u$unconfined_u``

Our user, ``joe.user``, is a member of the groups ``admins`` and
``users``.

We have a hostgroup, ``webservers``, which contains the hosts
``web1.example.com`` and ``web2.example.com>/code>``

.. _example_1:

Example 1
^^^^^^^^^

| ``(client.example.com, *, staff_u)``
| ``(*, joe.user, guest_u)``

If ``joe.user`` logs in from client.example.com he will get ``staff_u``
because hosts are evaluated first.

If joe.user logs in from any other host he gets ``guest_u``

.. _example_2:

Example 2
^^^^^^^^^

| ``(webservers, joe.user, staff_u)``
| ``(webservers, admins, unconfined_u)``

If ``joe.user`` logs in from web2.example.com he will get ``staff_u``.

This is because the rule containing his uid is more specific than the
group rule.
