Serving_legacy_clients_for_trusts
=================================

\__NOTOC_\_

Overview
========

Since version 3.0 FreeIPA supports cross-realm trusts with Active
Directory. In order to allow AD users to utilize services on IPA
clients, up to date version of SSSD should be configured at the IPA
client. In case it is not possible to install and configure SSSD > 1.9,
Active Directory users cannot access services on IPA clients.

This feature is designed to bridge the gap and provide minimal
compatibility level that allows to log-in to IPA clients for AD users.
IPA clients will be able to use any reasonable nss_ldap/pam_ldap/sssd
version.



Use Cases
=========

Access to IPA client machine resources for AD users in case IPA client
cannot utilize up to date version of SSSD with native support for IPA
cross-realm trusts.

Design
======

Since IPA client is configured with the use of older SSSD or
nss_ldap/pam_ldap, all work should be performed at the IPA master.
Primary design decision is to provide a separate LDAP tree, similar to
compat tree, that has following features:

-  information about both IPA and AD users can be queried;
-  it is possible to enumerate members of IPA and AD groups;
-  authentication bind to IPA LDAP as AD users should automatically
   trigger obtaining ticket from AD DC; in case TGT is obtained,
   authentication bind should be treated as successful.

Since old clients are often unable to use multiple search bases, LDAP
tree will be served by a modified slapi-nis plugin which will be able to
augment queries to main IPA tree with requests to SSSD cache.

Authentication
----------------------------------------------------------------------------------------------

There are two types of Kerberos based authentication involved. First and
preferred one, if the client can handle Kerberos authentication it
should do it directly against AD. In most cases this would give the user
a valid ticket in his credential cache which he can use for SSO to other
services. This case splits to two sub-cases:

-  AD application understands Kerberos and uses it to connect to the IPA
   client service. In such case AD client obtains a ticket to IPA client
   service through the AD DC which first gives cross-realm TGT and then
   issues referral to IPA KDC for the ticket to a service. AD client
   then connects to IPA client service with already obtained service
   ticket for the service.
-  AD user uses application that doesn't obtain Kerberos ticket prior to
   use of IPA client service. This sub-case include log-in into IPA
   client machine console where AD user credentials are entered
   directly. In this case PAM stack configuration is used to
   authenticate. The client (pam_ldap module) does an authenticated LDAP
   bind to validate the credentials the user gave at the login prompt.
   Since the IPA server cannot do the validation itself, it will in the
   end do a kinit with the forwarded credentials of the user against AD
   DC. In this case the user on the client will not have any Kerberos
   tickets after login, but at least authentication of AD users would be
   possible even for an LDAP only client.

In order to allow Kerberos authentication afterwards, IPA KDC needs to
issue referrals to the trusted domains KDCs (AD DCs). On client side
what is needed is **dns_lookup_kdc = true** in **/etc/krb5.conf**.

On the server side a plugin is needed for 389-ds to allow authentication
bind of AD user. This plugin will define bind pre-op hook to intercept
bind and also create virtual DN for the user. Authentication will happen
through an attempt to obtain a TGT for the user in question against AD
DC in AD realm. Actual information about the virtualized DN will be
served via slap-nis plugin.

Implementation
==============

#. IPA server sets SSSD configuration to 'ipa_server_mode = true' on
   install or upgrade
#. ipa-adtrust-install configures additional directory server plugin to
   serve trusted domains tree
#. Directory server plugin uses getpwnam\_r(), getgrnam\_r() and related
   calls to obtain information about AD user. For IPA users the
   information is fetched directly from the LDAP. For this purpose
   slapi-nis plugin would be extended to allow queries against SSSD
   (getpwnam\_/getgrnam\_r) instead of LDAP directory.
#. Directory server plugin will provide bind pre-op hook to perform
   authentication for AD user. The DN for AD user will be virtual one,
   handled by the previous plugin.
#. IPA KDC database driver adds MS-PAC information into ticket granting
   ticket for host/fqdn@REALM, cifs/fqdn@REALM, and HTTP/fqdn@REALM
   principal of IPA masters. This is required to allow SSSD on IPA
   master to authenticate against AD using host/fqdn@REALM principal and
   Python code in IPA server to do trust range discoveries. Additionally
   cifs/fqdn@REALM service needs to communicate to AD DC.

For SSSD design see
https://fedorahosted.org/sssd/wiki/DesignDocs/IPAServerMode



Feature Management
==================

UI

The feature is transparent and not exposed in UI

CLI

The feature is not directly exposed in CLI.

IPA idrange management is expanded to specify idrange type (IPA local,
AD trust, AD with winsync, IPA trust, ..) to affect the way how AD users
SIDs are mapped to POSIX IDs.



Major configuration options and enablement
==========================================

Given complexity of the configuration, it is advisable to create a
script on IPA server that will read out existing configuration and
provide generated scriplets that configure different systems.

The generated scriptlets should support following configurations. Each
scriplet when run should output recommended configuration as a text on
standard output or instruction to run appropriate system commands in
case where direct modification of the configuration files in not
possible (Solaris LDAP configuration).

The utility will be called **ipa-advise** and provide pluggable way to
add new "advises". Each "advise" will be named **--** to allow them
being structured. Below you can find an imaginary example:

| ``  ipa-advise config-solaris11-padl``
| ``      ....    config-freebsd7-padl``
| ``      ....    config-aix63-native``
| ``      ....    setup-ipa-trust2ad``
| ``      ....    setup-ipa-dnsdelegation``

and so on.

``   ipa-advise``

would show all plugins (filtering itself, i.e. list and help plugin)
with their short descriptions.



SSSD 1.9 and after
----------------------------------------------------------------------------------------------

SSSD 1.9 and onwards natively supports IPA cross-realm trusts with AD.
No need to explicitly use AD compatibility tree



SSSD 1.11
----------------------------------------------------------------------------------------------

Additionally, on IPA master **sssd.conf** will have **ipa_server_mode =
true** set. This is the mode that will allow IPA master to ask SSSD for
resolution of AD users using Global Catalog.



SSSD prior to 1.9
-----------------

Compat tree can be configured to search both main IPA LDAP tree and AD
compatibility data.



PADL pam_ldap/nss_ldap
----------------------

PADL **pam_ldap** is in use by all GNU/Linux distributions and many
other UNIX-like operating systems.



Vendor-specific pam_ldap
------------------------

While PADL pam_ldap supports AIX 5L, FreeBSD 3.x and above, HP-UX 11i,
IRIX 6.x, Linux, Mac OS X 10.2 and above, and Solaris 2.6 and above,
many vendors provide their own version also called **pam_ldap**.

Solaris pam_ldap implementation does not use directly editable files.
Instead, special utility is used to configure LDAP options.

Replication
===========

No effect on replication. Since directory server plugin is only
configured when ipa-adtrust-install is run, IPA masters may opt out from
serving AD clients.



Updates and Upgrades
====================

During upgrade of IPA master, sssd.conf should be updated to set
'ipa_server_mode = true'.

Dependencies
============

Depends on SSSD implementing IPA server mode (sssd 1.11)



External Impact
===============

https://fedorahosted.org/sssd/wiki/DesignDocs/IPAServerMode



Backup and Restore
==================

No external configuration files are affected



Legacy clients and HBAC rules
=============================

One of limitations of legacy support is the fact that authentication and
authorization is first performed at IPA server side using system-auth
PAM service. At this point what is checked by HBAC rules is access by
the user to the service called 'system-auth' on IPA master, not on the
legacy client.



Test Plan
=========

-  FreeIPA server: ipa.example.org
-  Active Directory: ad.example.org



RFE Author
==========

-  `ab <User:Ab>`__ (`talk <User_talk:Ab>`__)
-  `tbabej <User:Tbabej>`__ (`talk <User_talk:Tbabej>`__)