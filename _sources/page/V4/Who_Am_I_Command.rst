Who_Am_I_Command
================

Overview
--------

The command \`whoami\` is designed to provide details on how to obtain
information about currently authenticated identity.



Use Cases
---------

Primary use case for \`whoami\` command is to allow Web UI to choose
correct initial page for the authenticated identity. For self-service of
Active Directory users from trusted forests, their starting page should
be corresponding ID user override. For FreeIPA users their starting page
is their own user page.

Design
------

The \`whoami\` command uses following features of FreeIPA design:

-  Any identity authenticated to FreeIPA framework should be able to
   authenticate to an LDAP server. If authentication to LDAP server is
   successful, there is an LDAP object that represents this identity.
   FreeIPA framework can query DN of this LDAP object using \`whoami\`
   extension to LDAPv3 as documented in
   `https://tools.ietf.org/html/rfc4532 RFC
   4532 <https://tools.ietf.org/html/rfc4532_RFC_4532>`__.
-  FreeIPA uses flat DIT to store objects in LDAP database. Therefore, a
   corresponding FreeIPA framework object can be found by matching DN of
   the authenticated LDAP object to a FreeIPA framework's object with
   closest container DN, e.g. leftmost RDN is the only difference. There
   is only one exception to this rule, ID user overrides, which have
   container DN two levels below because they are grouped by the ID
   views.
-  FreeIPA framework objects are introspectable and have associated
   commands, including a command to show the object in question.
-  Finally, all objects that could be used to bind to FreeIPA framework
   have \`bindable = True\` property.

Implementation
--------------

When LDAP bind is successfully performed, an LDAP client can request DN
value of the bound LDAP object. FreeIPA framework always uses SASL
GSSAPI to perform its LDAP bind, thus SASL mapping rules in 389-ds are
used to find proper LDAP object which corresponds to SASL identity.
There are two primary SASL mappings in FreeIPA: Kerberos principal
mapping for FreeIPA users, hosts, and services, and ID user override
mapping for users from trusted Active Directory forests.

For FreeIPA objects which are known to be exceptions (ID users
override), a correction of the container DN is applied before testing.
SASL mapping for trusted Active Directory users always searches for ID
user override in the 'Default Trust View' ID View. This means container
DN for ID user override can be extended to always use 'Default Trust
View' ID view.

\`whoami\` command retrieves DN of the authenticated LDAP object and the
object itself from LDAP. It then finds FreeIPA framework object that is
bindable and has container DN immediately preceding the DN of the
authenticated object. In terms of \`ipapython.dn.DN\` class, this means
DN.find(foo) returns 1.

The command returns following information:

-  object class name
-  function to call to get actual details about the object
-  arguments to pass to the function



Feature Management
------------------

The \`whoami\` command does not require any arguments. It uses implicit
authenticated identity as its input.

Configuration
----------------------------------------------------------------------------------------------

No configuration is needed

Upgrade
-------

No upgrade impact



How to Use
----------

Below is an example of how communication looks like for an Active
Directory user which has ID override in 'Default Trust View'.

| ``       $ ipa -vv console``
| ``       >>> api.Command.whoami()``
| ``       ipa: INFO: trying ``\ ```https://ipa.example.com/ipa/session/json`` <https://ipa.example.com/ipa/session/json>`__
| ``       ipa: INFO: Forwarding 'whoami/1' to json server '``\ ```https://ipa.example.com/ipa/session/json`` <https://ipa.example.com/ipa/session/json>`__\ ``'``
| ``       ipa: INFO: Request: {``
| ``           "id": 0,``
| ``           "method": "whoami/1",``
| ``           "params": [``
| ``               [],``
| ``               {``
| ``                   "version": "2.220"``
| ``               }``
| ``           ]``
| ``       }``
| ``       ipa: INFO: Response: {``
| ``           "error": null,``
| ``           "id": 0,``
| ``           "principal": "Administrator@AD.DOMAIN",``
| ``           "result": {``
| ``               "arguments": [``
| ``                   "default trust view",``
| ``                   "administrator@ad.domain"``
| ``               ],``
| ``               "command": "idoverrideuser_show/1",``
| ``               "object": "idoverrideuser",``
| ``               "options": []``
| ``           },``
| ``           "version": "``\ ``"``
| ``       }``
| ``      {'command': 'idoverrideuser_show/1', 'object': 'idoverrideuser', 'arguments': ('default trust view', 'administrator@ad.domain')}``



Test Plan
---------

There are five types of objects that could bind to IPA using their
credentials. \`ipa whoami\` call expects one of the following:

-  users
-  staged users
-  hosts
-  Kerberos services
-  ID user override from the default trust view

The latter category of objects is automatically mapped by SASL GSSAPI
mapping rule in 389-ds for users from trusted Active Directory forests.

Below is a short summary demonstrating possible test cases for the
\`whoami\` command.



Using host principal
----------------------------------------------------------------------------------------------

::

   [root@ipa ~]# klist
   Ticket cache: KEYRING:persistent:0:krb_ccache_uA6VDOR
   Default principal: host/ipa.example.com@EXAMPLE.COM

   Valid starting       Expires              Service principal
   03/08/2017 15:37:47  03/09/2017 15:37:42  HTTP/ipa.example.com@EXAMPLE.COM
   03/08/2017 15:37:42  03/09/2017 15:37:42  krbtgt/EXAMPLE.COM@EXAMPLE.COM
   [root@ipa ~]# ipa -vv console
   ipa: INFO: trying https://ipa.example.com/ipa/session/json
   ipa: INFO: Forwarding 'schema' to json server 'https://ipa.example.com/ipa/session/json'
   ipa: INFO: trying https://ipa.example.com/ipa/session/json
   (Custom IPA interactive Python console)
   >>> api.Command.whoami()
   ipa: INFO: Forwarding 'whoami/1' to json server 'https://ipa.example.com/ipa/session/json'
   ipa: INFO: Request: {
       "id": 0, 
       "method": "whoami/1", 
       "params": [
           [], 
           {
               "version": "2.220"
           }
       ]
   }
   ipa: INFO: Response: {
       "error": null, 
       "id": 0, 
       "principal": "host/ipa.example.com@EXAMPLE.COM", 
       "result": {
           "arguments": [
               "ipa.example.com"
           ], 
           "command": "host_show/1", 
           "object": "host"
       }, 
       "version": "4.4.90.dev201703081319+git708d826"
   }
   {u'command': u'host_show/1', u'object': u'host', u'arguments': (u'ipa.example.com',)}