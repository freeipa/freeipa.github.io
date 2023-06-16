Overview
--------

Users can authenticate to Kerberos in FreeIPA using many different
mechanism (i.e. Password, Password+OTP, RADIUS, etc.). Each mechanism
represents a different authentication strength. Similarly, Kerberos
enabled services had varying degrees of security sensitive content. It
would be valuable if services could specify that they require certain
authentication methods. This would permit, for example, the strongest
authentication methods to be used across all the services while not
allowing weaker authentication methods to have access to security
critical services.

This project enables authentication methods to insert tags, called
Authentication Indicators, into the TGT based upon the types of
authentication that are used. Service principals can then define a set
of authentication indicators which are required in order to obtain a
ticket for the service.

.. _use_cases3:

Use Cases
---------

.. _strong_authentication_on_selected_system:

Strong Authentication on Selected System
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~



User story
^^^^^^^^^^

As an Administrator, I want to setup authentication to a critical system
in my infrastructure (gateway VPN, accounting system) to only allow IdM
users authenticated via strong authentication methods (2FA). I do not
want to require strong authentication on other systems.

Description
^^^^^^^^^^^

A realm has two servers configured for ssh which use the following
principals configured with authentication indicators:

-  host/lowsecurity.example.com []
-  host/highesecurity.example.com [otp radius]

When the administrator logs in using both his password and an OTP token,
he can access both systems via ssh. However, when the administrator logs
in using just a password, he can only access lowsecurity.example.com.

Design
------

Background
~~~~~~~~~~

The IETF draft for Authentication Indicators is available here:
https://tools.ietf.org/html/draft-jain-kitten-krb-auth-indicator-01

The MIT Kerberos project for Authentication Indicators is available
here: http://k5wiki.kerberos.org/wiki/Projects/Authentication_indicator

MIT Kerberos now contains all the plumbing for Authentication Indicators
as of the 1.14 release. Our changes are mostly limited to the kdb plugin
and the UI/CLI.

.. _overview_1:

Overview
~~~~~~~~

.. figure:: Indicators.png
   :alt: Indicators.png

   Indicators.png

.. _enable_optional_otp_preauth:

Enable Optional OTP Preauth
~~~~~~~~~~~~~~~~~~~~~~~~~~~



Server Side
^^^^^^^^^^^

Currently, if OTP is enabled for a user (even optionally), only the otp
preauth mechanism is returned. Once we have authentication indicators,
it is the otp preauth mechanism that will set the indicator. Thus, we
have to disallow 1FA through the otp preauth module so that only 2FA
authentications get a 2FA indicator.

To accomplish this, we have ipa-otpd set a critical control during its
bind operation which indicates to the server that an OTP MUST be
validated for authentication to succeed. If the server does not support
this control, the bind will fail. If the server does support the control
but is unable to validate an OTP, the bind will fail. The bind control
has the following OID: TBD.

Once authentications through ipa-otpd enforce the presence of a second
factor, we can enable other preauthentication mechanisms (encrypted
challenge/timestamp) for 1FA.

.. _client_side:

Client Side
^^^^^^^^^^^

The kinit utility should work out of the box. Versions that did not
support OTP previously failed authentication. However, with the above
server change these clients now work for 1FA only. Further, clients
without FAST enabled will now be able to obtain a 1FA ticket; enabling
them to step up to 2FA by enabling FAST with their 1FA ticket. Finally,
clients that both understand OTP and have FAST enabled will chose the
first mechanism returned from the server; which is OTP. This is probably
the best behavior we can expect from kinit.

SSSD will have to learn how to handle receiving both encrypted challenge
and OTP when FAST is present. Currently, it should force the use of OTP.
We need to make this optional.

.. _insert_authentication_indicators_during_authentication_asreq:

Insert Authentication Indicators During Authentication (ASReq)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once the above task is done, we can start inserting authentication
indicators. This is a simple configuration change to the KDB plugin to
return a specially formatted user string for the otp preauthentication
mechanism.

.. _verify_authentication_indicators_during_ticket_issuance_tgsreq:

Verify Authentication Indicators During Ticket Issuance (TGSReq)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

During a TGS request, the KDC will load the service principal from KDB
like normal. In this case, we simply need to return a tl_data string
with the key as "require_auth" and the value a space separated list of
authentication indicators. This will be stored in LDAP as the optional
multi-valued attribute krbPrincipalAuthInd on krbPrincipal objects. This
will require a patch to the upstream LDAP KDB plugin's schema to support
this new attribute. It will also require a patch to FreeIPA's KDB to
support this new schema.

Once the KDC has a list of acceptable indicators, it can make the proper
policy evaluation.

Implementation
--------------

==================================== ==================
New Dependencies                     Backup and Restore
==================================== ==================
krb5 >= 1.14.0 (freeipa-server only) N/A
==================================== ==================

.. _otprequired_bind_control:

OTPRequired Bind Control
~~~~~~~~~~~~~~~~~~~~~~~~

=== ============
OID Contents
=== ============
TBD None (empty)
=== ============

.. _kdb_string_attributes:

KDB String Attributes
~~~~~~~~~~~~~~~~~~~~~

+-------------------+---------------+--------------+------------------------------+
| Principal Type    | Configuration | Key          | Value                        |
+===================+===============+==============+==============================+
| User (AS Req)     | RADIUS        | otp          | [{"indicators": ["radius"]}] |
+-------------------+---------------+--------------+------------------------------+
| User (AS Req)     | OTP           | otp          | [{"indicators": ["otp"]}]    |
+-------------------+---------------+--------------+------------------------------+
| Service (TGS Req) | N/A           | require_auth |                              |
+-------------------+---------------+--------------+------------------------------+

.. _feature_management3:

Feature Management
------------------

UI
~~

Services should gain checkboxes for the required authentication
indicators.

CLI
~~~

Overview of the CLI commands. Example:

=========== ============================
Command     Options
=========== ============================
service-add --auth-ind=['otp', 'radius']
service-mod --auth_ind=['otp', 'radius']
=========== ============================

Upgrades
--------

Upgrades will need to learn the new schema.

Troubleshooting
---------------

TBD

.. _how_to_test3:

How to Test
-----------

Easy to follow instructions how to test the new feature. FreeIPA user
needs to be able to follow the steps and demonstrate the new features.

The chapter may be divided in sub-sections per `Use
Case <#Use_Cases>`__.

.. _test_plan3:

Test Plan
---------

Test scenarios that will be transformed to test cases for FreeIPA
`Continuous Integration <V3/Integration_testing>`__ during
implementation or review phase. This can be also link to `source in
cgit <https://git.fedorahosted.org/cgit/freeipa.git/>`__ with the test,
if appropriate.

`Authentication Indicators V4.4 test
plan <V4/Authentication_Indicators/Test_Plan>`__
