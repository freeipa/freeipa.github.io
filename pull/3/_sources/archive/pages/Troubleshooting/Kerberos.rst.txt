This page contains **Kerberos** troubleshooting advice, including
**trusts**. For other issues, refer to the index at
`Troubleshooting <Troubleshooting>`__.

.. _kinit_does_not_work:

kinit does not work
===================

-  On client, see the debug messages from the ``kinit`` process itself:

      ::

         KRB5_TRACE=/dev/stdout kinit admin

-  Make sure that there are no `DNS Issues <#DNS_Issues>`__ and that
   forward (A and/or AAAA) records of the client are OK.
-  Make sure that ``krb5kdc`` and ``dirsrv`` services on the FreeIPA
   server are running
-  Check for errors in ``/var/log/krb5kdc.log``

.. _service_does_not_start:

Service does not start
======================

-  See service log of the respective service for the exact error text.
   For example, the `Directory Server <Directory_Server>`__ stores the
   log in ``/var/log/dirsrv/slapd-REALM-NAME/errors``
-  Make sure that the server the service is running on has a fully
   qualified domain name
-  Make sure that if /etc/hosts contains an entry for this server, the
   fully qualified domain name comes first, e.g.:

      ::

         192.168.1.1 ipa.example.com ipa

-  See what keys are in the keytab used for authentication of the
   service, e.g.:

      ::

         # klist -kt /etc/dirsrv/ds.keytab

-  Make sure that the stored principals match the system FQDN system
   name
-  Make sure that the version of the keys (KVNO) stored in the keytab
   and in the FreeIPA server match:

      ::

         $ kvno ldap/ipa.example.com@EXAMPLE.COM

-  Make sure that there are no `DNS Issues <#DNS_Issues>`__ and both
   forward and reverse DNS records of the are OK and match the system
   name and the stored principal keys
-  Make sure that the **system time difference** on the host and FreeIPA
   server is not greater than 5 minutes

.. _cannot_authenticate_on_client:

Cannot authenticate on client
=============================

-  If FreeIPA was re-enrolled against different FreeIPA server, try
   removing SSSD caches (``/var/lib/sss/db/*``) and restarting the SSSD
   service (`freeipa-users
   thread <https://www.redhat.com/archives/freeipa-users/2015-June/msg00116.html>`__)

For further advise, see `SSSD
guide <https://fedorahosted.org/sssd/wiki/Troubleshooting>`__ for
troubleshooting problems on clients, including tips for gathering SSSD
log files.

.. _failed_auth_increments_failed_login_count_by_2:

Failed auth increments failed login count by 2
==============================================

-  This happens when migration mode is enabled. After normal auth
   attempt SSSD performs LDAP bind to generate Kerberos keys. This
   failure raises the counter for second time.
-  Resolution: disable migration mode when all users are migrated by

      ::

         ipa config-mod --enable-migration=False

.. _cannot_authenticate_user_with_otp_with_google_authenticator:

Cannot authenticate user with OTP with Google Authenticator
===========================================================

-  This happens when hash function **other that SHA-1** is used and OTP
   code is generated using Google Authenticator (encountered with 4.74).
   Google Authenticator ignores the hash function and uses SHA-1 anyway
   making the generated codes unusable. Use FreeOTP application or OTP
   tokens with SHA-1 hash function. `related freeipa-users
   thread <https://www.redhat.com/archives/freeipa-users/2016-November/msg00356.html>`__.

.. _smart_card_authentication:

Smart Card authentication
=========================

See `Troubleshooting SmartCard
authentication <https://floblanc.wordpress.com/2017/06/02/freeipa-troubleshooting-smartcard-authentication/>`__
for SmartCard authentication issues.

For Kerberos PKINIT authentication both client and server (KDC) side
must have support for PKINIT enabled. On Fedora/RHEL/CentOS systems this
means an RPM package krb5-pkinit or similar should be installed. If a
client system lacks krb5-pkinit package, a client will not be able to
use a smartcard to obtain an initial Kerberos ticket (TGT). This is hard
to notice as Kerberos client will simply have no way to respond to the
pre-authentication scheme for PKINIT. Thus, a first step in resolving
issues with PKINIT would be to check that krb5-pkinit package is
installed.

Trusts
======

Ubuntu distributions at this time don't support Trust feature of
FreeIPA. See
https://bugs.launchpad.net/ubuntu/+source/samba/+bug/1552249 for more
details.

.. _cannot_create_trust_with_trust_add:

Cannot create trust with trust-add
----------------------------------

See `separate page <Active_Directory_trust_setup#Debugging_trust>`__
with instructions how to debug `trust <Trusts>`__ creating issues.
