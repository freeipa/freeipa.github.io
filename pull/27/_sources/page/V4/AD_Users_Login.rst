AD_Users_Login
==============

Overview
--------

AD users are able to use CLI after they obtain Kerberos ticket. It would
be useful to allow them also using WebUI.



Use Cases
---------

As AD user I want to see my certificates or SSH keys in FreeIPA using
WebUI.

As AD user I want to add SSH key to my FreeIPA profile.

Design
------

Only AD user self-service is supported. The self-service contains the
same fields as idoverrideuser facet. New menu specification is needed
(we've already had one for FreeIPA admin, one for FreeIPA user
self-service). The new one serves for all AD users (admins and users).
For determining which menu specification should be shown is used new ipa
whoami command (`V4/Who Am I Command <V4/Who_Am_I_Command>`__).

Each AD user who wants to log in FreeIPA WebUI has to have created
idoverrideuser. In case that one does not have created the
iduseroverride then the log in is not possible.

Implementation
--------------

After loging in, the WebUI calls ipa whoami command which returns the
name of object which is trying to log in. With this information the
WebUI also receive the command name and necessary arguments which needs
to be passed to that command to get the information about logged
identity. According to these, the WebUI choose which menu specification
should be displayed.

In case of AD user, the ipa idoverrideuser-show command is called.



Feature Management
------------------

UI

New menu specification for AD users which shows idoverrideuser facet
with changed titles to 'Profile'.

CLI

No CLI.

Configuration
----------------------------------------------------------------------------------------------

Each AD user which wants to log in into FreeIPA WebUI has to have
created idoverrideuser.

Upgrade
-------



How to Use
----------

-  Establish trust between FreeIPA server and AD.
-  Create AD user on AD machine
-  Create ID User Override on FreeIPA server, where the id override user
   name is Ad_user@domain.ad.example.com

| ``$ ipa idoverrideuser-add "Default Trust View" Ad_user@domain.ad.example.com``
| ``-----------------------------------------------------------------------------``
| ``Added User ID override "Ad_user@domain.ad.example.com"``
| ``-----------------------------------------------------------------------------``
| ``Anchor to override: ad_user@domain.ad.example.com``

-  Log in to FreeIPA WebUI using ad_username@ad.domain and password for
   AD user.



Test Plan
---------