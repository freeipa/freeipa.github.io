Authenticating_Redmine_with_IPA
===============================

In order to authenticate against Redmine, read `Redmine LDAP
guide <http://www.redmine.org/projects/redmine/wiki/RedmineLDAP>`__
first. Information specific to FreeIPA follows.

Configuration
=============

Redmine can use LDAP in 2 modes. Let's call them "Basic" and "Full".
Basic is easier to configure, while Full gives better user experience.

It's recommended to start with Basic configuration, and after it's
verified, add the missing information for Full configuration.



Basic Configuration
-------------------

::

   Name: whatever you want
   Host: your ldap server name (e.g., "ldap_server", or "ldap_server.example.com")
   Port: 636
   LDAPS: check
   Account: leave empty
   Password: leave empty
   Base DN: cn=users,cn=accounts,dc=example,dc=com
   LDAP filter: leave empty
   On-the-fly user creation: check
   Login attribute: uid
   Firstname/Lastname/Email attributes: leave empty

That's it! Click "Save"; and then "Test", you should get "Successful
connection".

Now you have working LDAP authentication. Logout and test it.

You *don't have* LDAP synchronization. I.e., whenever a new user login,
he will have to manually fill in his details - First/Last names and
mail.



Full Configuration
------------------

Full Configuration adds in synchronization, so whenever a new user
logins, Redmine will "know" everything it wants about it.

Did you read `HowTo/LDAP <HowTo/LDAP>`__ "System Accounts" section?
Redmine needs a system account.

Add the following fields: (after creating the System Account, named
*redmine* in this example):

::

   Account: uid=redmine,cn=sysaccounts,cn=etc,dc=example,dc=com
   Password:
   Firstname attribute: givenName
   Lastname attribute: sn
   Email attribute: mail

Again - save, logout and login.

If you made mistake - you'll lose even the Basic configuration
capabilities. But it's easy to go back - just delete the extra fields.

That's all!

`Category:How to <Category:How_to>`__