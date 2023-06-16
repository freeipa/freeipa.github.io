Tested on CentOS 6.5, The Bug Genie 3.2.7.1, FreeIPA 3.0.0-37.el6

Log in as admin. Bottom of the page, click Configure The Bug Genie.

Authentification tab, select LDAP authentification as Authentication
backend, save.

Now go to LDAP authentification tab, and fill in the following values
(don’t forget to follow the « Important information part of the page !)
:

::

   Hostname: ldaps://youripaserver1,ldaps://youripaserver2
   Base DN: DC=yourlocaldaim,DC=tld
   Object DN attribute: entrydn
   User class: person
   Username attribute: uid
   Full name attribute: cn
   Email address attribute: mail
   Group class: group

Next entries are empty. May be useful in a more complex setup. Feel free
to complete this (mini) howto.

Test connection, import your users. You should be OK.
