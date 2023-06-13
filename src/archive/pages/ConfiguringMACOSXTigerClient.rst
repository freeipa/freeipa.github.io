.. _ipa_client_configuration:

IPA Client Configuration
========================

setup
-----

| `` Follow this guide. I found the step-by-step description to be very useful.``
| `` ``\ ```http://clc.its.psu.edu/Labs/Mac/Resources/authdoc/default.aspx`` <http://clc.its.psu.edu/Labs/Mac/Resources/authdoc/default.aspx>`__

| `` Specifically for Kerberos Configuration , follow this. No changes.``
| ``  ``\ ```http://clc.its.psu.edu/Labs/Mac/Resources/authdoc/kerberosauthentication.aspx`` <http://clc.its.psu.edu/Labs/Mac/Resources/authdoc/kerberosauthentication.aspx>`__
| `` ``
| `` Specifically for LDAP Client Configuration, follow this.``
| `` There are some changes, which are described below.``
| ``  ``\ ```http://clc.its.psu.edu/Labs/Mac/Resources/authdoc/ldapauthorization.aspx`` <http://clc.its.psu.edu/Labs/Mac/Resources/authdoc/ldapauthorization.aspx>`__

| ``  PrimaryGroupID - use gidNumber attribute from LDAP``
| ``  UniqueID - use uidNumber attribute from LDAP``

NTP
---

-  Open the "Date&Time" utility and point it to the
   ipaserver.example.com to set the date and time automatically.

kinit
-----

-  Attempt to get a kerberos ticket.

| ``   kinit admin``
| ``   klist ( to verify )``

ssh
---

| `` if you have a valid kerberos ticket, ssh would proceed with GSSAPI``
| `` auth without asking for a password.``

-  ssh admin@ipaserver.example.com

.. _system_login:

system login
------------

-  On the MAC system console, login as an ipa user with its password.
   Once logged in , open a terminal and try these commands:

``id ( look for userid and group id correctness )``

-  After login, if you have kerberos configured, make sure you have a
   valid kerberos ticket. klist will help here.

nfsv4/kerberos
--------------

``TBD. not sure what status this code is in. I'm not able to find any docs for this from apple.``

.. _browser___firefox:

Browser - firefox
-----------------

| ``Do the normal kerberos configuration for firefox. ``
| ``Open firefox. goto ``\ ```about:config`` <about:config>`__
| `` set network.negotiate-auth.delegation-uris to .example.com``
| `` set network.negotiate-auth.trusted-uris to .example.com``

| ``Goto ``\ ```https://ipaserver.example.com`` <https://ipaserver.example.com>`__
| ``If you have a valid kerberos ticket, you should be authenticated at this point.``
