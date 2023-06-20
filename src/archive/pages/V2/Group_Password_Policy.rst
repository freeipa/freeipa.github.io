Group_Password_Policy
=====================

Introduction
------------

Password Policy in IPA v2 is still limited to the password policy
provided by the KDC. This means that we check the following:

-  Minimum Password Lifetime (krbMinPwdLife): The minimum period of
   time, in hours, that a user's password must be in effect before the
   user can change it. The default value is one hour.
-  Maximum Password Lifetime (krbMaxPwdLife): The maximum period of
   time, in days, that a user's password can be in effect before it must
   be changed.
-  Minimum Number of Character Classes (krbPwdMinDiffChars): The minimum
   number of different classes, or types, of character that must exist
   in a password before it is considered valid. The default value is
   zero (0). The following character classes are supported:

   -  Upper-case characters
   -  Lower-case characters
   -  Digits
   -  Special characters (for example, punctuation)

-  Minimum Length of Password (krbPwdMinLength): The minimum number of
   characters that must exist in a password before it is considered
   valid.
-  Password History Size (krbPwdHistoryLength): The number of previous
   passwords that IPA stores, and which a user is prevented from using.
   The default value is zero (0) (disable password history).



Policy types
------------

A default so-called "global" policy is created when IPA is installed.
This policy affects all users. To change this policy use the
``ipa pwpolicy-mod`` command.

It is possible to create per-group policies as well. This group may be
either a user or a posix group, it makes no difference. Use the
``ipa pwpolicy-add GROUP`` command to add a per-group policy. The bottom
line is that if you don't include a group as the first argument you are
operating on the global policy. If you include a group as an argument
you are working on a group.

A group policy takes a special argument, ``priority``, that sets the
priority for this policy. If a user is in several groups that have a
password policy this resolves which policy "wins." The policy with the
lowest priority wins. Two policies with the same priority is undefined,
the directory server will arbitrarily decide.

For group policy it is not required to set all policy attributes. If an
attribute is not defined it does NOT fall back to the global policy, it
merely enforces no policy for it. For example if you do not set a
minimum password length then there isn't one.

The group you are adding policy for must exist prior to creating the
policy. When a group is removed its policy is removed as well.

Examples
--------

Add a new group policy for group **g2**:

% ipa pwpolicy-add g2 --maxlife=90 --minlife=8 --history=15
--minclasses=3 --minlength=6 --priority=20

| `` Group: g2``
| `` Max lifetime (days): 90``
| `` Min lifetime (hours): 8``
| `` History size: 15``
| `` Character classes: 3``
| `` Min length: 6``
| `` Priority: 20``

Modify a group policy:

% ipa pwpolicy-mod --minlength=9 g2

| `` Group: g2``
| `` Max lifetime (days): 90``
| `` Min lifetime (hours): 8``
| `` History size: 15``
| `` Character classes: 3``
| `` Min length: 9``

I have a user tuser1 and I'm not sure what policy applies to him:

% ipa pwpolicy-show --user=tuser1

| `` Group: g2``
| `` Max lifetime (days): 90``
| `` Min lifetime (hours): 8``
| `` History size: 15``
| `` Character classes: 3``
| `` Min length: 9``

Show the global policy:

% ipa pwpolicy-show

| `` Group: GLOBAL``
| `` Max lifetime (days): 90``
| `` Min lifetime (hours): 1``
| `` History size: 0``
| `` Character classes: 0``
| `` Min length: 8``



Schema details
--------------

Group policy is implemented using the Class of Service plugin, using it
in a slightly different way than usual. This difference is due to
limitations in the krb5-ldap-server plugin to handle DNs. It is
extremely picky about the format of policy DNs.

The policy itself gets stored in
``cn=REALM,cn=kerberos,dc=example,dc=com``:

| ``dn: cn=group1,cn=EXAMPLE.COM,cn=kerberos,dc=example,dc=com``
| ``objectClass: top``
| ``objectClass: nscontainer``
| ``objectClass: krbpwdpolicy``
| ``krbMinPwdLife: 7200``
| ``cn: group1``

The CoS entry is what contains the reference to this policy. It looks
like:

| ``dn:cn="cn=group1,cn=groups,cn=accounts,dc=example,dc=com",cn=cosTemplates,cn=acco``
| ``unts,dc=example,dc=com``
| ``objectClass: top``
| ``objectClass: costemplate``
| ``objectClass: extensibleobject``
| ``objectClass: krbcontainer``
| ``krbPwdPolicyReference: cn=group1,cn=EXAMPLE.COM,cn=kerberos,dc=example,dc=com``
| ``cosPriority: 10``
| ``cn: "cn=group1,cn=groups,cn=accounts,dc=example,dc=com"``

The DN of the CoS entry contains the DN of the group, as is usual. What
is a bit unusual is the DN of the krbPwdPolicyReference. Ideally this
would be the DN of the group but this causes the KDC to not be able to
find the entry so we use just the CN of the group. It is also completely
intolerant to spaces in the DN so great care needs to be taken in
normalizing it.

So the way that password policy is resolved is this:

-  CoS provides the attribute krbPwdPolicyReference for members of
   groups
-  If the password plugin or KDC finds a krbPwdPolicyReference in the
   entry it uses that for password policy
-  If not it uses the global policy

When a group is deleted any password policy associated with it is also
removed.