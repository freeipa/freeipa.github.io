Overview
--------

FreeIPA already supports Smart Card authentication: the user provides a
Smart Card containing a certificate and the user lookup is performed
with a binary match of the whole certificate (see `User
Certificates <V4/User_Certificates>`__).

The goal is to extend this feature to support the following cases:

-  the Smart Card contains multiple certificates. The administrator must
   be able to define Matching rules that will check which certificates
   are valid for authentication.
-  the Smart Card contains multiple certificates that are valid for
   authentication. The user must be able to select the certificate he
   wants to use for login.
-  the Certificate presented by the user is mapped to multiple accounts.
   The user must be able to disambiguate by providing a username.
-  the mapping between a Certificate and a user account can be done
   either through binary match of the whole certificate or a match on
   custom certificate attributes (such as Subject + Issuer).

The feature is tightly linked to SSSD features:

-  `Smartcards and Multiple
   Identities <https://fedorahosted.org/sssd/wiki/DesignDocs/SmartcardsAndMultipleIdentities>`__
-  `Matching and Mapping
   Certificates <https://docs.pagure.org/SSSD.sssd/design_pages/matching_and_mapping_certificates.html>`__



Use Cases
---------

-  As a user with multiple accounts linked to my Smart Card certificate
   in FreeIPA server, I want to authenticate with my SmartCard to
   FreeIPA server WebUI as a selected role
-  As a user with multiple accounts linked to my Smart Card certificate
   in FreeIPA server, I want to authenticate with my SmartCard to a
   desktop system joined to FreeIPA as a selected role
-  As a user with multiple accounts linked to my Smart Card certificate
   in FreeIPA server, I want to authenticate with my SmartCard to a
   remote system joined to FreeIPA as a selected role
-  As an administrator, I want to configure SmartCard authentication
   policy (allow the system to display a prompt for the username when
   multiple accounts match a single certificate) at the FreeIPA level
-  As an administrator, I want to manage links between a certificate and
   user accounts (create a link, remove a link, list links for an
   account, find accounts linked to a certificate), either with the full
   certificate content or with Subject and Issuer information
-  As an application developer, I want my application using FreeIPA
   server as authentication backend to enable authentication of users
   with multiple accounts linked to their SmartCard



Feature Management
------------------

The feature will provide 3 different configuration items:

-  the global feature configuration: feature behavior options
-  the rules configuration: each rule will define a scope for the rule
   (apply to certificates issued by a given issuer), the matching rule
   (only certificates following some criteria are considered valid for
   authentication), the mapping rule (the mechanism used to map a
   certificate to a user), the domains where the user account can be
   searched, and a boolean to enable/disable the rule
-  the mapping information for a given user

CLI
~~~

Configuration
^^^^^^^^^^^^^

+--------------------+-----------------------+-----------------------+
| Command            | Options               | Description           |
+====================+=======================+=======================+
| certmapconfig-mod  | --promptusername=BOOL | Prompt for the        |
|                    |                       | username when         |
|                    |                       | multiple identities   |
|                    |                       | are mapped to a       |
|                    |                       | certificate           |
+--------------------+-----------------------+-----------------------+
| certmapconfig-show |                       | Show the current      |
|                    |                       | Certificate Identity  |
|                    |                       | Mapping               |
|                    |                       | configuration.        |
+--------------------+-----------------------+-----------------------+

Rules
^^^^^

The CLI will allow to define Matching and Mapping rules.

+-----------------------+---------------------------------------------+
| Command               | Options                                     |
+=======================+=============================================+
| certmaprule-{add/mod} | RULENAME --desc DESCRIPTION --maprule       |
|                       | MAPRULE --matchrule MATCHRULE --domain      |
|                       | DOMAIN --priority PRIORITY                  |
+-----------------------+---------------------------------------------+
| certmaprule-enable    | RULENAME                                    |
+-----------------------+---------------------------------------------+
| certmaprule-disable   | RULENAME                                    |
+-----------------------+---------------------------------------------+
| certmaprule-del       | RULENAME                                    |
+-----------------------+---------------------------------------------+
| certmaprule-find      | RULENAME --rulename=STR --desc STR          |
|                       | --maprule STR --matchrule MATCHRULE         |
|                       | --domain DOMAIN --priority INT              |
+-----------------------+---------------------------------------------+
| certmaprule-show      | RULENAME                                    |
+-----------------------+---------------------------------------------+

All the options are optional.

-  DESCRIPTION: free form text
-  DOMAIN must contain a domain name, either IPA domain name or a
   trusted AD domain (for instance: ipadomain.com)
-  MAPRULE: the format is described in `SSSD /
   Mapping <https://docs.pagure.org/SSSD.sssd/design_pages/matching_and_mapping_certificates.html#id4>`__
   and sss-certmap(5) man page
-  MATCHRULE: the format is aligned with `pkinit_cert_match
   syntax <http://web.mit.edu/Kerberos/krb5-1.14/doc/admin/conf_files/krb5_conf.html#pkinit-krb5-conf-options>`__
   and described in `SSSD /
   Matching <https://docs.pagure.org/SSSD.sssd/design_pages/matching_and_mapping_certificates.html#id3>`__
   and sss-certmap(5) man page
-  PRIORITY: integer >= 0. The higher the value, the lower the priority.
   Rules with the same priority will all be considered.

.. _user_mapping:

User mapping
^^^^^^^^^^^^

The CLI already provides commands related to certificates (ipa user-add,
ipa user-mod --certificate and ipa user-{add/remove}-cert). They will be
kept when the full certificate needs to be stored in the user entry.

New CLI will allow to link a certificate to a user, with different
options:

-  directly with the mapping data
-  through a certificate, from which subject and issuer will be
   extracted
-  through the certificate subject and issuer

============================= =======================================
Command                       Options
============================= =======================================
user-{add/remove}-certmapdata LOGIN CERTMAPDATA
user-{add/remove}-certmapdata LOGIN --subject SUBJECT --issuer ISSUER
user-{add/remove}-certmapdata LOGIN --certificate BLOB
============================= =======================================

-  LOGIN
-  CERTMAPDATA will be a free-form text.
-  SUBJECT and ISSUER are DNs (using LDAP ordering) of the certificate
   subject/issuer
-  BLOB is the base-64 encoded user certificate, from which subject and
   issuer will be extracted

A new CLI will allow to look for all the users corresponding to the
provided certificate:

============= ==================
Command       Options
============= ==================
certmap-match FILE
certmap-match --certificate BLOB
============= ==================

-  FILE file containing the certificate
-  BLOB is the base-64 encoded user certificate

The output will contain the matching user names, grouped by domain:

| ``---------------``
| ``2 users matched``
| ``---------------``
| ``  Domain: DOMAIN.EXAMPLE.COM``
| ``  Usernames: user1, user2``
| ``----------------------------``
| ``Number of entries returned 2``
| ``----------------------------``

UI
~~

-  Add a new tab below "Authentication", with the title "Certificate
   Identity Mapping"
-  The window will contain 2 sub sections (drop-down menu):

   -  Certificate Identity Mapping configuration
   -  Certificate Identity Mapping Rules

-  Modify the "User" page to also display "Mapped Certificates" with
   "Add"/"Delete" buttons

Design
------

This document concentrates on the management part of the feature
(configuration, provisioning of user certificates and mappings). The
design for SSSD modifications is out of scope.

.. _high_level_schema:

High Level schema
~~~~~~~~~~~~~~~~~

The feature will be delivered as a new plugin in FreeIPA and
modifications in existing plugins (user, host).

The CLI and GUI tools will write the configuration and mappings in the
LDAP backend, thus requiring new schema and permissions/ACIs in order to
protect the data.

LDAP
~~~~

.. _objectclasses_and_attributes:

Objectclasses and attributes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following schema will be used:

``attributeTypes: (2.16.840.1.113730.3.8.22.1.1 NAME 'ipaCertMapPromptUsername' DESC 'Prompt for the username when multiple identities are mapped to a certificate' EQUALITY booleanMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.7 SINGLE-VALUE X-ORIGIN 'IPA v4.5' )``

``attributeTypes: (2.16.840.1.113730.3.8.22.1.2 NAME 'ipaCertMapMapRule' DESC 'Certificate Mapping Rule' SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 SINGLE-VALUE X-ORIGIN 'IPA v4.5' )``

``attributeTypes: (2.16.840.1.113730.3.8.22.1.3 NAME 'ipaCertMapMatchRule' DESC 'Certificate Matching Rule' SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 SINGLE-VALUE X-ORIGIN 'IPA v4.5' )``

``attributeTypes: (2.16.840.1.113730.3.8.22.1.4 NAME 'ipaCertMapData' DESC 'Certificate Mapping Data' EQUALITY caseIgnoreMatch SUBSTR caseIgnoreSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 X-ORIGIN 'IPA v4.5' )``

``attributeTypes: (2.16.840.1.113730.3.8.22.1.5 NAME 'ipaCertMapPriority' DESC 'Rule priority' SYNTAX 1.3.6.1.4.1.1466.115.121.1.27 SINGLE-VALUE X-ORIGIN 'IPA v4.5' )``

``objectClasses: (2.16.840.1.113730.3.8.22.2.1 NAME 'ipaCertMapConfigObject' DESC 'IPA Certificate Mapping global config options' AUXILIARY MAY ipaCertMapPromptUsername X-ORIGIN 'IPA v4.5' )``

``objectClasses: (2.16.840.1.113730.3.8.22.2.2 NAME 'ipaCertMapRule' DESC 'IPA Certificate Mapping rule' SUP top STRUCTURAL MUST cn MAY ( description $ ipaCertMapMapRule $ ipaCertMapMatchRule $ associatedDomain $ ipaCertMapPriority $ ipaEnabledFlag ) X-ORIGIN 'IPA v4.5' )``

``objectClasses: (2.16.840.1.113730.3.8.22.2.3 NAME 'ipaCertMapObject' DESC 'IPA Object for Certificate Mapping' AUXILIARY MAY ipaCertMapData X-ORIGIN 'IPA v4.5' )``

Example
^^^^^^^

| ``dn: cn=certmap,$BASEDN``
| ``objectClass: top``
| ``objectClass: nsContainer``
| ``objectClass: ipaCertMapConfigObject``
| ``cn: certmap``
| ``ipaCertMapPromptUsername: FALSE``

| ``dn: cn=certmaprules,cn=certmap,$BASEDN``
| ``objectClass: top``
| ``objectClass: nsContainer``
| ``cn: certmaprules``

| ``dn: cn=rule1,cn=certmaprules,cn=certmap,$BASEDN``
| ``cn: rule1``
| ``objectClass: ipacertmaprule``
| ``associatedDomain: domain.com``
| ``ipaCertMapMapRule: (ipacertmapdata=X509:<I>{issuer_dn}<S>{subject_dn})``
| ``ipaCertMapPriority: 1``
| ``ipaCertMapMatchRule: <ISSUER>CN=Certificate Authority,O=IPA.DEVEL``
| ``ipaEnabledFlag: TRUE``
| ``description: rule1 description``

| ``dn: uid=user1,cn=users,cn=accounts,$BASEDN``
| ``objectclass: top``
| ``objectclass: (all IPA user objectclasses)``
| ``objectclass: ipacertmapobject``
| ``ipacertmapdata: X509:<I>CN=Certificate Authority,O=IPA.DEVEL<S>CN=certmaptest.ipa.devel,O=IPA.DEVEL``

.. _access_control:

Access control
~~~~~~~~~~~~~~

New privilege: **Certificate Identity Mapping Administrators**

New Self-service permission: **Users can manage their own X.509
certificate identity mappings**

New permissions:

-  **System: Read Certmap Configuration**: allows to read the
   configuration in the certmap configuration container
-  **System: Modify Certmap Configuration**: allows to modify the
   configuration in the certmap configuration container
-  **System: Read Certmap Rules**: allows to read the rules in the rules
   container
-  **System: Add Certmap Rules**: allows to add new rules in the rules
   container
-  **System: Modify Certmap Rules**: allows to modify rules in the rules
   container
-  **System: Delete Certmap Rules**: allows to delete rules in the rules
   container
-  **System: Manage User Certificate Mappings**: allow to add/remove a
   certificate identity mapping to a user

The **System: Read Certmap Configuration** and **System: Read Certmap
Rules** permissions will be granted to ldap:///all, and all the other
permissions will be added to the **Certificate Identity Mapping
Administrators** privilege.

Implementation
--------------

Upgrade
-------

The upgrade needs to install the new schema and create the entry
cn=certmap,cn=ipa,cn=etc,$BASEDN and the container entry
cn=certmaprules,cn=certmap,cn=ipa,cn=etc,$BASEDN.

In prevision of future modifications, the configuration
cn=certmap,cn=ipa,cn=etc,$BASEDN will contain a ipacertmapversion
attribute.



How to Use
----------

-  Allow to display the prompt for username disambiguation

``ipa certmapconfig-mod --promptusername=TRUE``

-  Define a mapping rule based on subject and issuer

``ipa certmaprule-add defaultrule --desc "Default mapping rule" --maprule "(ipacertmapdata=X509:<I>{issuer_dn}<S>{subject_dn})"``

-  Configure the mapping between the user testuser and a certificate
   issued by cn=extca,dc=example,dc=com with subject
   cn=myname,dc=example,dc=com

``ipa user-add-certmapdata testuser --subject cn=myname,dc=example,dc=com --issuer cn=extca,dc=example,dc=com``

or

``ipa user-add-certmapdata testuser "X509:<I>cn=extca,dc=example,dc=com<S>cn=myname,dc=example,dc=com"``

-  On an enrolled client, login to GDM using a smart card containing the
   user cert. The authenticate user will be "testuser"



Test Plan
---------

Test scenarios that will be transformed to test cases for FreeIPA
`Continuous Integration <V3/Integration_testing>`__ during
implementation or review phase. This can be also link to `source in
cgit <https://git.fedorahosted.org/cgit/freeipa.git/>`__ with the test,
if appropriate.

Troubleshooting
---------------

Please check `FreeIPA: Troubleshooting SmartCard
authentication <https://floblanc.wordpress.com/2017/06/02/freeipa-troubleshooting-smartcard-authentication/>`__
blog post for tips.
