RHEL6_Package_List
==================

\__NOTOC_\_



SRPM Packages
=============

Current page provides the list of the IPA related SRPM packages. The
package names listed here can be used to determine which component
should be used to report issues if found in the Red Hat Enterprise Linux
distribution. Please, do not report bugs against internal packages
unless you are absolutely sure that they are the cause of the problem.

-  Directory server components

   -  **389-ds-base** - code of the directory server
   -  **svrcore** - internal
   -  **ds-replication** - replication bits
   -  **perl-Mozilla-LDAP** - internal

-  Certificate System and dependencies

   -  **ipa-pki-theme** - internal
   -  **pki-core** - all PKI functionality of IPA
   -  **tomcatjss** - internal
   -  **osutil** - internal
   -  **nss** - internal
   -  **jss** - internal

-  IPA packages:

   -  **ipa** - main IPA component
   -  **slapi-nis** - NIS and other compatibility capabilities of IPA
   -  **python-krbV** - internal
   -  **python-kerberos** - internal
   -  **python-pyasn1** - internal
   -  **python-netaddr** - internal
   -  **python-nss** - internal
   -  **bind-dyndb-ldap** - ldap support for the DNS server

-  Keberos packages

   -  **krb5** - Kerberos KDC and library

-  Client packages

   -  **ipa-client** - client enrollment package
   -  **certmonger** - certificate tracking
   -  **sssd** - independent project that provides client side framework
      for authentication and identity providers.
   -  **ding-libs** - internal
   -  **openldap** - LDAP client library