Kerberos_PKINIT
===============

Overview
--------

Kerberos authentication framework allows to obtain Kerberos tickets
based on a number of different mechanisms. Standard approach implies use
of a secret shared by both a Kerberos KDC and a Kerberos client.

RFC 4556 describes a way to allow initial authentication in Kerberos
protocol to happen with the help of Public Key Cryptography (PKINIT).

This feature page describes PKINIT implementation in FreeIPA.



Use Cases
---------

Following use cases are defined for the scope of this FreeIPA feature:

-  As an administrator, I want to enable use of PKINIT for Kerberos
   authentication

   -  By default, FreeIPA CA should issue certificates for the KDCs
   -  Externally provided certificates should be accepted as well when
      deploying PKINIT configuration

-  As an administrator, I want to allow users to issue certificates that
   can be associated with their Kerberos identities
-  As an administrator, I want to define exact criteria for PKI
   certificates to be accepted for PKINIT purpose by the Kerberos KDC.
-  As an administrator, I want to define exact criteria for PKI
   certificates to be chosen for PKINIT purpose by Kerberos clients.
-  As a user, I want to be able to manage public key certificates
   associated with my Kerberos identity, regardless which CA did issue
   them.
-  As a user, I want to use Anonymous PKINIT feature to obtain an
   initial ticket granting ticket (TGT) to allow FAST exchange with two
   factor authentication without access to a different Kerberos
   principal.

Design
------

MIT Kerberos KDC and a Kerberos client library already support PKINIT
for initial authentication. The scope of supported functionality is
defined in the manual page for \`krb5.conf\` in the section **PKINIT
Options**.

From Kerberos perspective, there are two separate actors: a Kerberos
client has to choose which PKI certificate to use for initial Kerberos
authentication, and a Kerberos KDC has to decide whether a PKINIT
authentication request with the selected PKI certificate can be accepted
to represent operations on behalf of the claimed Kerberos identity.

PKINIT initial authentication is implemented as a pre-authentication
scheme in Kerberos protocol. Briefly, RFC 4556 document defines the
following extensions to RFC 4120:

#. The client indicates the use of public-key authentication by
   including a special preauthenticator in the initial request. This
   preauthenticator contains the client's public-key data and a
   signature.
#. The KDC tests the client's request against its authentication policy
   and trusted Certification Authorities (CAs).
#. If the request passes the verification tests, the KDC replies as
   usual, but the reply is encrypted using either:

   #. a key generated through a Diffie-Hellman (DH) key exchange with
      the client, signed using the KDC's signature key; or
   #. a symmetric encryption key, signed using the KDC's signature key
      and encrypted using the client's public key.
      Any keying material required by the client to obtain the
      encryption key for decrypting the KDC reply is returned in a
      pre-authentication field accompanying the usual reply.

#. The client validates the KDC's signature, obtains the encryption key,
   decrypts the reply, and then proceeds as usual.

Given that actual work to handle PKINIT is already implemented in MIT
Kerberos, high level scope for FreeIPA integration includes the
following changes:

-  Changes to the FreeIPA server and replica installers to request
   PKINIT-compatible certificates for use in KDC configuration
-  Addition of a special profile with Kerberos KDC specific extensions
-  Changes to certificate management plugin (`cert.py`) to allow
   generating PKINIT-compatible certificate for KDC using the profile
   defined above
-  Creation of a special principal to assist in Anonymous PKINIT initial
   authentication

Since the `External
Authentication <https://www.freeipa.org/page/V4/External_Authentication>`__
workflow now isolates FreeIPA framework from all Kerberos keys/ccaches
care must be taken to ensure that the password logins to the master
(e.g. via WebUI) are able to leverage some FAST armoring mechanisms to
enable login using OTP. To allow this, local anonymous PKINIT may be
performed using a self-signed KDC keypair. Note that such PKINIT setup
is not usable on enrolled clients and should be employed only in cases
when proper external PKINIT certificates signed by IPA CA are not
available, e.g. in CA-less case or when PKINIT certificate issuance was
suppressed by installer option. In order to differentiate between
client-consumable PKINIT and the local variant, the former capability
will be advertised in LDAP and an API will be provided to display its'
status on individual masters.

Implementation
--------------



Addition of a special profile with Kerberos KDC specific extensions
----------------------------------------------------------------------------------------------

A profile named **KDCs_PKINIT_Certs** is added to Dogtag and is loaded
into the LDAP store by means of an upgrade script.



Changes to certificate management plugin
----------------------------------------------------------------------------------------------

Certificate management plugin (`cert.py`) is changed to allow generating
PKINIT-compatible certificate for KDC using the **KDCs_PKINIT_Certs**
profile. The difference with normal certificate issuing workflow is that
the public certificate is not stored in the LDAP entry for the
coresponding principal, \`krbtgt/REALM@REALM`. Since KDC
pre-authentication pkinit module is configured through \`kdc.conf`,
there is no need to store the public certificate in the LDAP at all.

As a result, each KDC will have own certificate issued for
\`krbtgt/REALM@REALM\` using **KDCs_PKINIT_Certs** profile.



Creation of a special principal for Anonymous PKINIT
----------------------------------------------------------------------------------------------

A special principal, \`WELLKNOWN/ANONYMOUS@REALM`, is created during KDC
configuration stage. A pre-authentication is set to be required for the
principal because PKINIT use requires pre-authentication mechanism.



FreeIPA server and replica installer changes
----------------------------------------------------------------------------------------------

MIT Kerberos KDC can be built with either NSS and OpenSSL libraries for
PKINIT support. Depending on the compile time default, when KDC is
configured to enable PKINIT, KDC and CA certificates are configured in a
different way. Both Red Hat Enterprise Linux and Fedora distributions
build MIT KDC with OpenSSL. Thus, KDC configuration must specify
separately a private key and a public certificate of the KDC
certificates.

Certmonger is capable to generate PKI certificates pair in separate
files but FreeIPA installer framework does not support this option. This
means certmonger shim in FreeIPA installer framework is extended to
support issuing certificates in ceparate private and public key files.



Backup and Restore
----------------------------------------------------------------------------------------------

When backing the server up, KDC certificate pair needs to be backed up.
When restoring, KDC certificate pair needs to be properly restored,
including certmonger tracking of it.



Feature Management
------------------

Since anonymous PKINIT will now always be required given the changes
during Privilege separation, **ipa pkinit-anonymous** command will be
deprecated and do nothing but issue a warning. This is a safeguard to
prevent the locking of anonymous principal and hence breaking
password-based authentication to IPA framework (e.g. WebUI logins).

``ipa config-show`` will be enhanced to display active KDCs supporting
client PKINIT. To improve the usability of PKINIT in mixed topologies, a
separate command (``ipa pkinit-status``) will be provided to query
PKINIT status on selected master or on the whole topology.

To allow PKINIT authentication for users, their PKI certificates should
include SAN extensions that map them directly to their primary Kerberos
principal name. More details can be found in the PKINIT section of the
**krb5.conf** manual page.

Configuration
----------------------------------------------------------------------------------------------

[STRIKEOUT:For deployment with integrated CA PKINIT is always enabled.
For deployments with external CA it is possible to supply externally
signed KDC certificate pair.]

The exact PKINIT configuration when deploying new replica depends on the
domain level, FreeIPA version and CA status of the remote master as is
shown in the following table:

+----------------------------------+----------------------------------+
| Remote master configuration      | PKINIT feature status\*          |
+==================================+==================================+
| <4.5, no CA                      | local PKINIT                     |
+----------------------------------+----------------------------------+
| <4.5 with CA                     | try out full PKINIT, fall back   |
|                                  | to local configuration if it     |
|                                  | fails                            |
+----------------------------------+----------------------------------+
| 4.5 no CA, no external PKINIT    | local PKINIT                     |
| cert                             |                                  |
+----------------------------------+----------------------------------+
| 4.5 no CA, external PKINIT cert  | external PKINIT                  |
+----------------------------------+----------------------------------+
| 4.5 with CA                      | full PKINIT                      |
+----------------------------------+----------------------------------+
|                                  |                                  |
+----------------------------------+----------------------------------+

-  status: local PKINIT: locally configured KDC with self-signed
   certificate, external PKINIT: proper PKINIT configured externally
   provided KDC certificate and CA cert, full PKINIT: proper PKINIT
   configured using certificate signed by IPA CA

In addition, ``ipa-server-certinstall`` command should be extended to
allow for installing a PKINIT certificate from PKCS#12 file or request a
new PKINIT keypair signed by integrated CA. This should allow users to
configure proper PKINIT support for clients post-hoc after
installing/upgrading masters with no client PKINIT support.

NOTE: The design does not consider completely disabled anonymous PKINIT
as a configuration option. In order to emulate disabled PKINIT the
administrator can provide a PKCS#12 file via ``ipa-server-certinstall``
and just not distribute the trusted CA certificate as a pkinit anchor to
the clients. In this way the local PKINIT will work but clients would
not be able to issue anonymous tickets to clients. Since this state will
be recorded as a 'external PKINIT', the upgrade code will not attempt to
configure full PKINIT.

Upgrade
-------

[STRIKEOUT:On upgrade PKINIT is not automatically enabled. To enable
PKINIT, run ipa-pkinit-manage command.]

The exact upgrade path depends on the state of the feature on the
master. We shall distinguish the following configurations:

-  PKINIT is not configured at all (pre 4.5 masters)
-  only local PKINIT w/ self-signed KDC keypair is configured (DL0
   masters, CA-less masters without supplied PKINT certificates)
-  PKINIT is configured with supplied 3rd party certificate
-  full PKINIT is configured with certificates signed by IPA CA

+-------------+-------------+-------------+-------------+-------------+
| Server      | absent      | local       | external    | full PKINIT |
| co          | PKINIT      | PKINIT      | PKINIT      |             |
| nfiguration |             |             |             |             |
+=============+=============+=============+=============+=============+
| no CA       | configure   | no action   | no action   | N/A         |
|             | local       |             |             |             |
|             | PKINIT      |             |             |             |
+-------------+-------------+-------------+-------------+-------------+
| with CA on  | configure   | configure   | no action   | no action   |
| the master  | full PKINIT | full PKINIT |             |             |
+-------------+-------------+-------------+-------------+-------------+
| with CA not | try to      | try to      | no action   | no action   |
| on the      | configure   | configure   |             |             |
| master      | full        | full        |             |             |
|             | PKINIT,     | PKINIT,     |             |             |
|             | fallback to | fallback to |             |             |
|             | local       | local       |             |             |
|             | PKINIT      | PKINIT      |             |             |
+-------------+-------------+-------------+-------------+-------------+
|             |             |             |             |             |
+-------------+-------------+-------------+-------------+-------------+

As can be inferred from the table, there may be failures arising from
PKINIT cert request being routed to a CA master which does not hold the
**KDCs_PKINIT_Certs** profile or does not yet possess upgraded cert.py
plugin. In this case the upgrader will issue a warning and fall back to
issuing self-signed PKINIT certificates.



Update Sep 1 2017
----------------------------------------------------------------------------------------------

Following information should be incorporated in the design page in order
to be up to date with what was implemented in FreeIPA 4.5.x

When you install 4.5 with --no-pkinit, the installer will generate
self-signed certificate for PKINIT. This certificate is only used and
trusted by IPA Web UI running on the same server to obtain an anonymous
ticket.

How to upgrade that certificate to a full-featured PKINIT KDC
certificate depends on what is your CA.

A new tool, ipa-pkinit-manage, exists in FreeIPA 4.5.

``ipa-pkinit-manage status``

will tell you current status. "Enabled" means you have properly working
PKINIT infrastructure on this system. "Disabled" means you only have
self-signed PKINIT.

``ipa-pkinit-manage enable``

will try to request PKINIT KDC certificate from IdM CA. You should make
sure that all IdM CAs are at the same level (FreeIPA 4.5+) before
running this command.

If you are not using IdM CA (external CA is in use), the tool will tell
you to use ipa-server-certinstall to install an externally procured KDC
certificate. That certificate should have following properties:

-  it be issued with CN=fqdn,$SUBJECT_BASE
-  it should have Kerberos principal krbtgt/REALM@REALM
-  it should have id-pkinit-KPkdc OID (1.3.6.1.5.2.3.5)

You can see /usr/share/ipa/profiles/KDCs_PKINIT_Certs.cfg which is the
KDC profile we use to issue this certificate with integrated Dogtag CA.
Compare it to /usr/share/ipa/profiles/caIPAserviceCert.cfg to see the
difference to a 'normal' IPA service certificate.



How to Use
----------

FreeIPA clients are always configured to trust FreeIPA CA in their
**/etc/krb5.conf**. Thus, if PKINIT KDC certificate is issued by FreeIPA
CA, no additional configuration on the client side is needed.

[STRIKEOUT:To use Anonymous PKINIT, make sure ipa pkinit-anonymous
enable is run (default if installed with PKINIT enabled).]

Then any client can use

``   kinit -n``

to request an anonymous Kerberos ticket.

The ticket granting ticket (TGT) obtained as result of the **kinit -n**
request can only be used to create a FAST channel for second factor
authentication:

::

   | ``   kinit -n``
   | ``   klist``
   | ``   ARMOR_CCACHE=$(klist|grep cache:|cut -d' ' -f3-)``
   | ``   kinit -T $ARMOR_CCACHE principal@REALM ``

**-T** option of kinit allows to specify existing credentials cache with
a valid TGT to create a FAST channel between the Kerberos client and the
KDC when requesting the initial ticket for the specified
**principal@REALM**. This allows to safely pass details of the
multi-factor pre-authentication exchange to the KDC.

To query whether the master supports client PKINIT via ``config-show``:

| ``  $ ipa config-show``
| ``  Maximum username length: 32``
| ``  Home directory base: /home``
| ``  Default shell: /bin/sh``
| ``  Default users group: ipausers``
| ``  ...``
| ``  IPA masters supporting PKINIT: master1.ipa.test``
| ``  ...``

To obtain the same information via ``ipa pkinit-status``:

| ``  $ ipa pkinit-status``
| ``  Server name: master1.ipa.test``
| ``  PKINIT status: enabled``
| ``  ...``
| ``  Server name: replica1.ipa.test``
| ``  PKINIT status: disabled``
| ``  ...``