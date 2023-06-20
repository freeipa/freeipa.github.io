Owncloud_Authentication_against_FreeIPA
=======================================

`` ``

**HOWTO: Owncloud Authentication against FreeIPA**

This document describes how to setup owncloud (7.0.4) against FreeIPA
(4.1.2) demo1 server.

This howto was tested by deploying owncloud to openshift.com as capsule
and demo1 ipa server.

For the purpose of this document, the following information is given

| `` Server: ipa.demo1.freeipa.org``
| `` base dn: dc=demo1,dc=freeipa,dc=org``

Prerequisite:
-------------

We will need to create a bind account for owncloud to authenticate to
IPA as a service account. For this purpose I use admin account. Owncloud
needs to be able to connect to IPA server on port 389 or 636 to LDAP
sync works.



Owncloud Authentication
-----------------------

-  Login to owncloud
-  Go to Apps (left corner)
-  Enable LDAP user and group backend
-  Go to Admin page (right corner)
-  **Server**

| `` Server: ``\ ```ldap://ipa.demo1.freeipa.org`` <ldap://ipa.demo1.freeipa.org>`__
| `` Port: 389``
| `` User DN: uid=admin,cn=users,cn=accounts,dc=demo1,dc=freeipa,dc=org``
| `` Password: Secret123``
| `` Base DN: dc=demo1,dc=freeipa,dc=org``

.. figure:: Own_ldap_server.png
   :alt: Own_ldap_server.png

   Own_ldap_server.png

-  **User Filter**

`` Edit raw filter instead: (objectclass=*)``

.. figure:: Own_ldap_user_filter.png
   :alt: Own_ldap_user_filter.png

   Own_ldap_user_filter.png

-  **Login Filter**

| `` LDAP Username: checked``
| `` Edit raw filter instead: (&(objectclass=*)(uid=%uid))``

.. figure:: Own_ldap_login_filter.png
   :alt: Own_ldap_login_filter.png

   Own_ldap_login_filter.png

-  **Group filter** (it depends on which user group you want allow to
   access owncloud)

`` Edit raw filter instead: (\|(cn=ipausers))``

.. figure:: Own_ldap_group_filter.png
   :alt: Own_ldap_group_filter.png

   Own_ldap_group_filter.png

-  **Advanced**

   -  Connection Settings

`` Configuration Active: checked``

.. figure:: Own_ldap_adv_conn_set.png
   :alt: Own_ldap_adv_conn_set.png

   Own_ldap_adv_conn_set.png

-  

   -  Directory Settings

| `` User Display Name Field: displayname``
| `` Base User Tree: cn=users,cn=accounts,dc=demo1,dc=freeipa,dc=org``
| `` Group Display Name Field: cn``
| `` Base Group Tree: cn=groups,cn=accounts,dc=demo1,dc=freeipa,dc=org``
| `` Group-Member association: uniqueMember``
| `` Paging chunksize: 500``

.. figure:: Own_ldap_adv_dir_set.png
   :alt: Own_ldap_adv_dir_set.png

   Own_ldap_adv_dir_set.png

-  

   -  Special Attributes

| `` Email Field: mail``
| `` User Home Folder Naming Rule: cn``

.. figure:: Own_ldap_adv_spec_att.png
   :alt: Own_ldap_adv_spec_att.png

   Own_ldap_adv_spec_att.png

-  **Expert**

`` nothing``

.. figure:: Own_ldap_expert.png
   :alt: Own_ldap_expert.png

   Own_ldap_expert.png

`Category:How to <Category:How_to>`__ `Category:Draft
documentation <Category:Draft_documentation>`__