Overview
--------

IPA uses the Kerberos
`S4U2Proxy <http://k5wiki.kerberos.org/wiki/Projects/Services4User>`__
feature to allow the web server framework to obtain an ldap service
ticket on the user's behalf. Similarly, it is also used by the trust
system to obtain a cifs principal. This eliminates the need for the user
to delegate their full TGT.

The access control for these rules is controlled by several LDAP entries
contained in cn=s4u2proxy,cn=etc,$SUFFIX. Rules define what principals
can obtain service tickets for which other services, and optionally for
which subset of users. Targets define the list of services that may be
delegated.



Use Cases
---------

Delegation of obtaining an ldap service principal for the HTTP service.
Two entries are required:

The first is a rule which defines the ACL:

| ``dn: cn=ipa-http-delegation,...``
| ``objectClass: ipaKrb5DelegationACL``
| ``objectClass: groupOfPrincipals``
| ``cn: ipa-http-delegation``
| ``memberPrincipal: HTTP/ipaserver.example.com@EXAMPLE.COM``
| ``ipaAllowedTarget: cn=ipa-ldap-delegation-targets,...``

The second is a target of this rule which defines which principals may
be obtained:

| ``dn: cn=ipa-ldap-delegation-targets,...``
| ``objectClass: groupOfPrincipals``
| ``cn: ipa-ldap-delegation-targets``
| ``memberPrincipal: ldap/ipaserver.example.com@EXAMPLE.COM``

Both types of entries contain members in the form of memberPrincipal. In
the case of a rule these are the members that the rule applies to. In
the case of a target the members are the targets of the delegation. In
this case the rule has a member of HTTP/ and a rule with a member of
ldap/ which means that the HTTP principal can obtain an ldap service
ticket on behalf of the bound user.

Design
------

The tricky bit here is that all entries are stored in the same container
and the only real distinguishing feature between a rule and a target is
the objectClass ipaKrb5DelegationACL.

A rule will have these objectClasses: top, groupOfPrincipals,
ipaKrb5DelegationACL.

A target will have these objectClasses: top, groupOfPrincipals.

This is only an issue when doing a find for targets because it will also
find rules. A new filter will need to be prepared to exclude
ipaKrb5DelegationACL.

Several entries may not be removed but need to be user-modifiable. This
could still result in an unusable system. The entries are
ipa-http-delegation, ipa-ldap-delegation-targets and
ipa-cifs-delegation-targets.

Implementation
--------------

Some major methods of the standard LDAPAddMember and LDAPRemoveMember
classes need to be overridden since memberPrincipal is not a DN, nor
should it be. One reason is that the principal may not necessarily be
local so there may not be a DN to reference. There are also performance
reasons within the kdb code.

These built-in add/remove methods assume, and require via asserts, that
all members be a DN. get_member_dns() will need to ignore any
memberPrincipal values and return only the DN-based values when adding
targets to rules to let the standard mechanics of LDAP*Member do their
work.

In order to handle the memberPrinicpal values a post_callback() is
required. This also means that there will be at worst two writes per
membership update. Given that this feature is not expected to be
frequently used then speed and efficiency are not a factor.

Similarly, we need to enforce that only a target is a member of a rule,
and not another rule. That would be an undefined relationship. To do
this each member will need to be retrieved and evaluated before adding
as a member.

A referential integrity rule will be needed for ipaallowedtarget.

A referential integrity rule is needed, but is not possible, for
memberPrincipal. It is not possible because it is a DN. This adds the
potential to have dangling pointers.

The ACL system also provides a means of limiting which user's a ticket
may be obtained for using the ipaAllowToImpersonate attribute. This is
not implemented.



Feature Management
------------------

UI
~~

TBD https://fedorahosted.org/freeipa/ticket/5044

CLI
~~~

Overview of the CLI commands.

============================================ ===========================
Command                                      Options
============================================ ===========================
servicedelegationrule-add RULE               
servicedelegationrule-find                   
servicedelegationrule-del RULE               
servicedelegationrule-add-member             --principals={PRINC,PRINC2}
servicedelegationrule-remove-member          --principals={PRINC,PRINC2}
servicedelegationrule-add-target             --targets={TARGET,TARGET2}
servicedelegationrule-remove-target          --targets= {TARGET,TARGET2}
servicedelegationtarget-add TARGET           
servicedelegationtarget-find                 
servicedelegationtarget-del TARGET           
servicedelegationtarget-add_member TARGET    --principals={PRINC,PRINC2}
servicedelegationtarget-remove_member TARGET --principals={PRINC,PRINC2}
\                                            
============================================ ===========================

Configuration
~~~~~~~~~~~~~

N/A

Upgrade
-------

N/A

.. _how_to_test37:

How to Test
-----------

Simple test to create a couple of principals, a constraint rule, then
fetch a ticket on behalf of a user for one of the principals.

**NOTE**: This uses kvno which requires S4U2Self to operate for some
reason, hence having to use +ok_to_auth_as_delegate. Whatever you do,
**DON'T** do this in production.

Become admin:

``# kinit admin``

Create the service for the rule and allow it to impersonate users:
[**NOTE**: DO NOT DO THIS IN PRODUCTION, this allows the 'test' service
to impersonate \*any\* user to itself and then by proxy to the target
services]

| ``# ipa service-add test/ipa.example.com --force``
| ``# kadmin.local``
| ``kadmin.local: modprinc +ok_to_auth_as_delegate test/ipa.example.com``

Create the second service:

``# ipa service-add test2/ipa.example.com --force``

Get keytabs for these services:

| ``# ipa-getkeytab -s ipa.example.com -k /tmp/test.keytab -p test/ipa.example.com``
| ``# ipa-getkeytab -s ipa.example.com -k /tmp/test2.keytab -p test2/ipa.example.com``

Show that we can't do delegation yet:

| ``# kdestroy -A``
| ``# kinit -kt /tmp/test.keytab  test/ipa.example.com``
| ``# kvno -k /tmp/test.keytab -U admin -P test/ipa.example.com test2/ipa.example.com``
| ``kvno: KDC returned error string: NOT_ALLOWED_TO_DELEGATE test2/ipa.example.com@EXAMPLE.COM: constrained delegation failed``

Add the service constraint delegation:

| ``# kdestroy -A``
| ``# kinit admin``
| ``# ipa servicedelegationrule-add test``
| ``# ipa servicedelegationtarget-add target-test``
| ``# ipa servicedelegationrule-add-target --servicedelegationtargets=target-test test``
| ``# ipa servicedelegationrule-add-member --principals test/ipa.example.com test``
| ``# ipa servicedelegationtarget-add-member --principals=test2/ipa.example.com target-test``

Now try again:

| ``# kdestroy -A``
| ``# kinit -kt /tmp/test.keytab  test/ipa.example.com``
| ``# kvno -k /tmp/test.keytab -U admin -P test/ipa.example.com test2/ipa.example.com``
| ``test/ipa.example.com@EXAMPLE.COM: kvno = 2, keytab entry valid``
| ``test2/ipa.example.com@EXAMPLE.COM: kvno = 2, keytab entry valid``



Test Plan
---------

TBD
