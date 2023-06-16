Overview
========

This page describes the steps to configure IPA server.

Prerequisites
=============

-  Install IPA server.

Configuration
=============

::

   % ipa-server-install
   Server host name [ipa.example.com]:
   Please confirm the domain name [example.com]:
   Please provide a realm name [EXAMPLE.COM]:
   Directory Manager password: Secret123
   Password (confirm): Secret123
   IPA admin password: Secret123
   Password (confirm): Secret123

To restart IPA:

::

   % ipactl restart

If you need to uninstall:

::

   % ipa-server-install --uninstall

Verification
============

::

   % kinit admin
   Password for admin@EXAMPLE.COM: Secret123

::

   % klist

::

   % ipa-finduser admin

.. _web_ui:

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

.. _ldap_client:

LDAP Client
===========

| Hostname: ipa.example.com
| Port: 389
| Bind DN: cn=Directory Manager
| Password: Secret123

::

   % ldapsearch -x -b dc=example,dc=com

::

   % ldapsearch -Y GSSAPI -b dc=example,dc=com

.. _configure_listen_host:

Configure Listen Host
=====================

::

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

::

   % service dirsrv restart

.. _configure_kerberos:

Configure Kerberos
==================

Edit /etc/krb5.conf:

::

   [dbmodules]
     EXAMPLE.COM = {
       ...
       ldap_servers = ldap://ipa.example.com/
       ...
     }

Restart Kerberos:

::

   % service krb5kdc restart

.. _enable_change_log:

Enable Change Log
=================

Enable Retro Changelog plugin:

::

   % ldapmodify -h ipa.example.com -p 389 -x -D "cn=Directory Manager" -w Secret123
   dn: cn=Retro Changelog Plugin,cn=plugins,cn=config
   changetype: modify
   replace: nsslapd-pluginEnabled
   nsslapd-pluginEnabled: on
   -

Restart DS:

::

   % service dirsrv restart

::

   % ldapsearch -h ipa.example.com -p 389 -x -D "cn=Directory Manager" -w Secret123 -b "cn=changelog"

.. _create_sync_user_account:

Create Sync User Account
========================

::

   % ldapadd -h ipa.example.com -p 389 -x -D "cn=Directory Manager" -w Secret123
   dn: uid=sync,cn=sysaccounts,cn=etc,dc=example,dc=com
   objectClass: account
   objectClass: simpleSecurityObject
   uid: sync
   userPassword: Secret123

::

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
