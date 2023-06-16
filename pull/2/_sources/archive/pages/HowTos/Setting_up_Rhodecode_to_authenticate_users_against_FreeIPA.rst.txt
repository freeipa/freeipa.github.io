Tested on CentOS 6.5, Rhodecode 2.2.5, FreeIPA 3.0.0-37.el6

First of all, Rhodecode needs an account to connect to LDAP and then
perform searches and authentication. While any user account might work
for this purpose, there are few limitations:

-  user accounts are too powerful to what Rhodecode needs
-  user account's password is subject to expiration policies

Therefore, it is better to create a specialized system account for the
application access. By default system accounts give read-only access. To
do this create a file (eg rhodecodebind.ldif) containing the following:

| `` dn: uid=rhodecodebind,cn=sysaccounts,cn=etc,dc=yourlocaldomain,dc=tld``
| `` changetype: add``
| `` objectclass: account``
| `` objectclass: simplesecurityobject``
| `` uid: rhodecode``
| `` userPassword: ohaimakethissimethingtoughtobreak``
| `` passwordExpirationTime: 20380119031407Z``
| `` nsIdleTimeout: 0``

This can then be imported by:

| `` kinit admin``
| `` ldapmodify -Y GSSAPI -f rhodecodebind.ldif``

Next, log in to Rhodecode with your local admin account.

Admin menu -> authentification.

Enable rhodecode.lib.auth_modules.auth_ldap

Fill in the following values:

::

   LDAP Host youripaserver1,youripaserver2…
   Port 636
   Account: uid=rhodecodebind,cn=sysaccounts,cn=etc,dc=yourlocaldomain,dc=tld
   Password: ohaimakethissimethingtoughtobreak
   Connection Security LDAPS
   Certificate Checks NEVER (quite unsecure…other entries to be tested)
   Base DN cn=users,cn=accounts,dc=yourlocaldomain,dc=tld
   LDAP Search Filter (objectClass=person)
   LDAP Search Scope SUBTREE
   Login Attribute uid
   First Name Attribute givenname
   Last Name Attribute sn
   Email Attribute mail

Save. You’re done !

You can now auth with any user account (you may need to restart
rhodecode service)
