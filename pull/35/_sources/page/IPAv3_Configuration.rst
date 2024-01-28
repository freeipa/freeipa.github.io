IPAv3_Configuration
===================

Overview
========

This page describes the steps to configure IPA server.

Prerequisites
=============

-  Install IPA server.

Configuration
=============

.. code-block:: text

   % ipa-server-install
   Server host name [ipa.example.com]:
   Please confirm the domain name [example.com]:
   Please provide a realm name [EXAMPLE.COM]:
   Directory Manager password: Secret123
   Password (confirm): Secret123
   IPA admin password: Secret123
   Password (confirm): Secret123

To restart IPA:

.. code-block:: text

   % ipactl restart

If you need to uninstall:

.. code-block:: text

   % ipa-server-install --uninstall

Verification
============

.. code-block:: text

   % kinit admin
   Password for admin@EXAMPLE.COM: Secret123

.. code-block:: text

   % klist

.. code-block:: text

   % ipa-finduser admin



Web UI
======

Start Firefox, open http://ipa.example.com.

Open about:config, set the following parameters as follows:

-  network.auth.use-sspi: false
-  network.negotiate-auth.trusted-uris: .example.com.
-  network.negotiate-auth.delegation-uris: .example.com.

Go back to http://ipa.example.com, click Import the IPA Certificate
Authority.

Click Configure Firefox button.

Reload the page.



LDAP Client
===========

| Hostname: ipa.example.com
| Port: 389
| Bind DN: cn=Directory Manager
| Password: Secret123

.. code-block:: text

   % ldapsearch -x -b dc=example,dc=com

.. code-block:: text

   % ldapsearch -Y GSSAPI -b dc=example,dc=com



Configure Listen Host
=====================

.. code-block:: text

   % ldapmodify -h ipa.example.com -p 389 -x -D "cn=Directory Manager" -w Secret123
   dn: cn=config
   changetype: modify
   replace: nsslapd-listenhost
   nsslapd-listenhost: ipa.example.com
   -
   replace: nsslapd-securelistenhost
   nsslapd-securelistenhost: ipa.example.com
   -

Restart DS:

.. code-block:: text

   % service dirsrv restart



Configure Kerberos
==================

Edit /etc/krb5.conf:

.. code-block:: text

   [dbmodules]
     EXAMPLE.COM = {
       ...
       ldap_servers = ldap://ipa.example.com/
       ...
     }

Restart Kerberos:

.. code-block:: text

   % service krb5kdc restart



Enable Change Log
=================

Enable Retro Changelog plugin:

.. code-block:: text

   % ldapmodify -h ipa.example.com -p 389 -x -D "cn=Directory Manager" -w Secret123
   dn: cn=Retro Changelog Plugin,cn=plugins,cn=config
   changetype: modify
   replace: nsslapd-pluginEnabled
   nsslapd-pluginEnabled: on
   -

Restart DS:

.. code-block:: text

   % service dirsrv restart

.. code-block:: text

   % ldapsearch -h ipa.example.com -p 389 -x -D "cn=Directory Manager" -w Secret123 -b "cn=changelog"



Create Sync User Account
========================

.. code-block:: text

   % ldapadd -h ipa.example.com -p 389 -x -D "cn=Directory Manager" -w Secret123
   dn: uid=sync,cn=sysaccounts,cn=etc,dc=example,dc=com
   objectClass: account
   objectClass: simpleSecurityObject
   uid: sync
   userPassword: Secret123

.. code-block:: text

   % ldapmodify -h ipa.example.com -p 389 -x -D "cn=Directory Manager" -w Secret123
   dn: dc=example,dc=com
   changetype: modify
   add: aci
   aci: (targetattr="*")(version 3.0; acl "Sync user can access everything."; allow 
    (all) userdn = "ldap:///uid=sync,cn=sysaccounts,cn=etc,dc=example,dc=com";)
   -

   dn: cn=changelog
   changetype: modify
   add: aci
   aci: (targetattr="*")(version 3.0; acl "Sync user can access everything."; allow 
    (all) userdn = "ldap:///uid=sync,cn=sysaccounts,cn=etc,dc=example,dc=com";)
   -

References
==========

-  `IPA - Install and
   Deploy <http://www.freeipa.org/page/InstallAndDeploy>`__
-  `RHE IPA - Installation and Deployment
   Guide <http://www.redhat.com/docs/en-US/Red_Hat_Enterprise_IPA/1.0/html/Installation_Deployment_Guide/index.html>`__
-  `How to use access
   control <http://directory.fedoraproject.org/wiki/Howto:AccessControl>`__

`Category:Obsolete <Category:Obsolete>`__