External_Authentication
=======================

Overview
--------

In modern systems sometimes users need to be allowed to authenticate
using alternative protocols, like Federation protocols (SAML) or
Hardware Security Modules like Smart Cards (X509). This Feature captures
the changes needed to allow these alternative authentication methods to
interact with the FreeIPA UI and HTTP RPC pipes.



Use Cases
---------

An Active Directory Domain User uses ADFS (SAML) to authenticate to a
trusted IPA server, the user is known to the system but doesn't have
Krb5 creds to interact with the framework.

A Smartcard owner wants to log in to the Web UI using the smartcard
without having to rely on PKINIT for Single Sign On. The user can be
authenticated via X509 certs to Apache but lacks krb5 creds to interact
with the framework.

Design
------

The solution is to allow the apache authentication modules to map the
user to an identity known by IPA and then impersonate the user by
requesting a ticket from the KDC (Protocol Transitioning).

.. figure:: Ext-Auth.svg
   :alt: Ext-Auth.svg

   Ext-Auth.svg

The image above represent the final architecture implementing external
authentication.

Workflows
----------------------------------------------------------------------------------------------

The classic workflow where mod_auth_gssapi obtains a ticket and stores
it in a credential cache to be used by the ipa sever framework changes
to handle two different workflows: - External Authentication workflow. -
Negotiate authentication with kerberos credentials workflow. The form
based authentication internal workflow also changes slightly but it is
substantially the same as the Negotiate workflow so it won't be
explicitly covered.



External Authentication workflow
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. A user authenticates to an Apache module (A1)
#. After positive authentication the mod_lookup_identity module matches
   the authenticated user to the correct IPA user via SSSD (A2).
#. With the correct principal name, mod_auth_gssapi performs a s4u2self
   operation to obtain a ticket for the HTTP service on behalf of the
   authenticating users (A3). The s4u2self step is actually done via
   privilege separation interposition by gssproxy.
#. An (encrypted) credential is returned to mod_auth_gssapi, which
   proceeds to store it in a ccache file (A4) that can be read by both
   the Apache user and the 'ipaapi' user used to run the framework.
#. The workflow proceeds with step X described later.



Negotiate authentication with kerberos credentials workflow
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. A user with a kerberos ticket authenticates to mod_auth_gssapi (B1),
   this happens via gssproxy interposition (B2) as the Apache server
   does not have direct access to the HTTP keytab.
#. An (encrypted) credential is returned to mod_auth_gssapi, which
   proceeds to store it (B3) in a ccache file that can be read by both
   the Apache user and the 'ipaapi' user used to run the framework.
#. The wrokflow proceeds with step X described later.



Applications operations
^^^^^^^^^^^^^^^^^^^^^^^

   X. When the applications (ipa framewrok/dogtag/etc..) needs to talk
   to the LDAP server (C1) it performs the usual s4u2proxy step using
   these encrypted credentials (C2). the encrypted credentials are read
   from the file ccache and are passed back to gssproxy which performs
   the actual s4u2proxy operation as well as the context establishment
   operation between HTTP and LDAP.



Main Changes
----------------------------------------------------------------------------------------------

A few key changes to the current FreeIPA framework setup are needed:

-  GSS-Proxy will need to be started on the box and given exclusive
   access to the HTTP keytab, the configuration of GSS-Proxy must allow
   impersonation only by the apache process (either via SELinux labels
   or process uid) and allow proxying only by the Framework process
   (again SELinux labels or process uid).
   `4189 <https://www.fedorahosted.org/freeipa/ticket/4189>`__
-  All the processes comprising the FreeIPA framework (Apache, framework
   user, helper scrips) will need to be configured to be intercepted by
   GSS-Proxy `4189 <https://www.fedorahosted.org/freeipa/ticket/4189>`__
-  The Framework will need to be moved to a separate process interfaced
   via mod_wsgi, so that the framework is not operating as the apache
   user. `5959 <https://www.fedorahosted.org/freeipa/ticket/5959>`__
-  credentials caches will need to be passed between the apache process
   and the framework process, either by setting up a shared file area,
   or by message passing of some kind (initially a shared file area
   where both the apache user and the framework user can write is
   probably sufficient).
-  The dogtag integration will finally need to be changed to use GSSAPI
   authentication instead of using a certificate for the RA agent.
   `5011 <https://www.fedorahosted.org/freeipa/ticket/5011>`__ (this
   step can be done separately after the rest)

Implementation
--------------

The implementation requires changed to mod_auth_gssapi
(`PR92 <https://github.com/modauthgssapi/mod_auth_gssapi/pull/92>`__)
and other parts of the apache authentication configuration. It also
requires the introduction of GSS-Proxy to properly separate privileges
between the components.

-  **Dependencies**:

   -  New mod_auth_gssapi module including upstream PR92
   -  New GSS-Proxy including access control for s4u2self VS s4u2proxy
      functionality



Feature Management
------------------

UI

How the feature will be managed via the Web UI.

CLI

Authntication plugins for the CLI are TBD, the first goal is to get the
browser workflow working.

Configuration
----------------------------------------------------------------------------------------------

Any configuration options? Any commands to enable/disable the feature or
turn on/off its parts?

Upgrade
-------

Configureation changes will be needed in various parts of FreeIPA. The
Framework will need to operate as a different user from the local apache
user:

-  Create a new system user
-  Change/check file permissions for this new user
-  Change/check access permissions for any socket based interaction

GSS-Proxy needs to be started:

-  New service to be started by FreeIPA
-  New configuration snippets specific to the FreeIPA use case
-  SELinux Policy to make sure all the parts can properly communicate
   with GSS-Proxy
-  IPA Keytab permissions need to be changed so that only GSS-Proxy can
   access it



How to Use
----------

Easy to follow instructions how to use the new feature according to the
`use cases <#Use_Cases>`__ described above. FreeIPA user needs to be
able to follow the steps and demonstrate the new features.

The chapter may be divided in sub-sections per `Use
Case <#Use_Cases>`__.



Test Plan
---------

Test scenarios that will be transformed to test cases for FreeIPA
`Continuous Integration <V3/Integration_testing>`__ during
implementation or review phase. This can be also link to `source in
cgit <https://git.fedorahosted.org/cgit/freeipa.git/>`__ with the test,
if appropriate.



External links
--------------

-  `The current smart card/x509 certificate
   setup <V4/External_Authentication/Setup>`__
-  `https://fedorahosted.org/gss-proxy/ticket/133 GSS-Proxy
   ticket <https://fedorahosted.org/gss-proxy/ticket/133_GSS-Proxy_ticket>`__
-  `Using mod_nss's NSSVerifyClient require + LookupUserByCertificate +
   GssapiImpersonate <V4/External_Authentication/NSS_Impersonation>`__
   -- developer investigation