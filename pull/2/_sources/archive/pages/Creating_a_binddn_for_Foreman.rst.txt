Creating a binddn for Foreman

For setting up freeIPA authentication for Foreman I liked to have a
seperate system account binddn.

In order to do this you first need to create a foreman-binddn.update
file like this:

| `` dn: uid=foreman,cn=sysaccounts,cn=etc,$SUFFIX``
| `` default:objectclass:account``
| `` default:objectclass:simplesecurityobject``
| `` default:uid:foreman``
| `` only:userPassword:averysecurepassword``
| `` only:passwordExpirationTime:20380119031407Z``
| `` only:nsIdleTimeout:0``

and then you import it into the FreeIPA (as root) like this:

`` ipa-ldap-updater foreman-binddn.update``

You can check if the new user is present running:

`` ldapsearch -D "cn=Directory Manager" -x uid=foreman -W``

Optional you can also add a group in freeIPA where you put all Foreman
admins inside:

`` ipa group-add --desc="Foreman Admins" foreman_admins``

This one is used below as the optional LDAP filter.

On the Foreman you supply the following information: LDAP Server

| `` Server: ``
| `` port: 389``
| `` Server type: FreeIPA``

Account

| `` Account Username: uid=foreman,cn=sysaccounts,cn=etc,dc=ipa,dc=example``
| `` Account Password: averysecurepassword``
| `` Base DN: cn=users,cn=accounts,dc=ipa,dc=example``
| `` Groups Base DN: cn=groups,cn=accounts,dc=ipa,dc=example``
| `` LDAP filter: (memberOf=cn=foreman_admins,cn=groups,cn=accounts,dc=ipa,dc=example)``
| `` Check both boxes of: ``
| `` Automatically create accounts in Foreman``
| `` Usergroup sync``

Atrribute mappings

| `` Login name attribue: uid``
| `` First name attribute: givenName``
| `` Surname attribute: sn``
| `` Email address attribute: mail``
