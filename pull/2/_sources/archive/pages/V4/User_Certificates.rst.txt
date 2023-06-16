Overview
--------

FreeIPA `4.1.0 <Releases/4.1.0>`__ or older and its `PKI <PKI>`__
component can release certificates for *hosts* and *services*, both are
using the same `PKI <PKI>`__ profile. This functionality covers basic
needs of servers and their services, mostly TLS encryption, rarely also
TLS authentication (current profile can serve both as client and server
certificate for authentication). However, it does not allow **users** to
also leverage the benefits of the integrated `PKI <PKI>`__, making it a
long standing `PKI <PKI>`__ debt. With `multiple certificate
profiles <V4/Certificate_Profiles>`__ and `ability to create lightweight
sub-CAs <V4/Sub-CAs>`__, the user certificate extension is finally
realizable.



Use Cases
---------

.. _smart_card_authentication:

Smart Card Authentication
~~~~~~~~~~~~~~~~~~~~~~~~~

This is the primary use case for this feature. FreeIPA administrators
should be able to issue Smart Cards (or X509 certificates in general) to
their users and configure FreeIPA to enable matching of the certificate
to the user entry itself. The feature will primarily focus on the new
SSSD PKCS#11 interface (`SSSD RFE ticket
#546 <https://fedorahosted.org/sssd/ticket/546>`__). The alternative for
SSSD PKCS#11 interface is the standard ``pam_pkcs11`` that is available
also on non-Linux platforms, but does not provide tight integration with
FreeIPA - certificate matching has to be configured on each client
(`example
config <https://github.com/OpenSC/pam_pkcs11/blob/master/doc/README.ldap_mapper>`__).

This use case is generic and can result in different implementations,
given there are several options how the Smart Card certificate is loaded
to FreeIPA and if it is available before hand at all.

.. _client_certificate_authentication:

Client Certificate Authentication
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`PKI <PKI>`__ needs to be able to issue user certificates that can be
used for user certificate authentication against server, for example to
`mod_nss <https://fedorahosted.org/mod_nss/>`__ or
`mod_ssl <http://www.modssl.org/>`__
`Apache <http://httpd.apache.org/>`__ modules.

.. _smime_and_user_signing_certificates:

S/MIME and User Signing Certificates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Certificates can be used not only for user authentication, but also for
signing (`S/MIME <http://en.wikipedia.org/wiki/S/MIME>`__). Mail user
agents are even able to catch them automatically from LDAP, as long as
the certificate is in user entry in ``userCertificate;binary`` attribute
(`related MozillaZine
discussion <http://forums.mozillazine.org/viewtopic.php?t=465963>`__).
FreeIPA should be able to to both issue the certificate for signing in
its default profiles, but be able to serve as a server for MUAs to
automatically download the user S/MIME certificates.

Design
------

This feature consists of 4 main parts that need to be designed to met
the proposed use cases:

-  Ability to issue user certificate with different profiles and key
   usage (client/server authentication, signing or subject). This is
   being solved by `V4/Certificate Profiles <V4/Certificate_Profiles>`__
   and `V4/Sub-CAs <V4/Sub-CAs>`__ designs.
-  Requesting and storing user certificates from `PKI <PKI>`__ in the
   tree so that they can be searched, displayed by Web UI and revoked,
   when needed
-  Searching certificates by clients - Smart Card, S/MIME use cases
-  Enrolling external certificates to the system - Smart Card use case

.. _storing_user_certificates:

Storing User Certificates
~~~~~~~~~~~~~~~~~~~~~~~~~

Service and Host certificates are currently stored in the entries
themselves, when the certificate is request by the ``cert-request``
command. When service/host is deleted, the stored certificate is
decoded, serial number identified and certificate then revoked in the
`PKI <PKI>`__. The certificate is also revoked when the service/host
requests another certificate. There is thus always just one active
certificate for a service/host. Web UI is able to display the
service/host certificate and allow also manual revocation.

User certificates, multiple certificate profiles and sub-CAs add a twist
to this design as not all certificates need to be stored directly in the
user/service/host entry. For example, while S/MIME signing certificates
should be in the user entry so that mail clients can search for them,
TLS encryption or especially short-lived certificates (`Web
SSO <http://www.ietf.org/staging/draft-mccallum-websso-00.txt>`__ or
`kx509 <https://tools.ietf.org/html/rfc6717>`__) should not be in the
user entry at all to avoid unnecessary pollution of the `Directory
Server <Directory_Server>`__ replication changelog or making user entry
unnecessary big (and thus adding a risk of LDAP performance problems).

This design therefore removes the assumption that all certificates are
stored in user entry.

.. _searching_certificates_by_clients:

Searching Certificates by Clients
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In both cases, when (user) certificate is stored in an entry or not,
clients searching user authenticating with such certificate (read -
Smart Card use case) have to identify the user entry. The certificate
entry itself does not always identify the user entry in LDAP, the
clients (SSSD or pam_pkcs11) have to search for the user by other means.

There are several options how the clients can run the search:

-  **Binary match on whole certificate**: this is the simplest approach
   to do as the user certificate can be retrieved from the Smart Card
   and used in an LDAP filter. This approach requires the certificate to
   be loaded in FreeIPA before Smart Card authentication. The
   performance cost for searching by the whole blob (around 1K of binary
   certificate) was not found to be significant enough to not allow this
   as an option.
-  **CertificateMatch or CertificateExactMatch**: support of the `RFC
   4523, section 3.2 <https://tools.ietf.org/html/rfc4523>`__ in the
   `Directory Server <Directory_Server>`__ would allow search by a
   certificate field directly in the LDAP search. However, this standard
   is not supported in the 389 Directory Server yet, it is a complex
   feature to add. See for example research done based on other LDAP
   server variants (`OpenLDAP
   paper <http://www.openldap.org/pub/slim/CMPaper.pdf>`__, `related
   presentation <http://www.openldap.org/conf/odd-sandiego-2004/Sangseok.pdf>`__).
   This approach also assumes that the certificate is in the user entry.
-  **Match on custom certificate attributes**: in some environments it
   is not possible for FreeIPA to dictate what should be in the
   certificate as the certificate is provided by a 3rd party CA. However
   this CA includes information in the certificate that is known in
   advance and is unique per user. In this case is important to be able
   to pre-provision user objects with the matching attribute+value that
   will be found on the user's Smart Card. This also means SSSD needs to
   find, from a global configuration entry, what attribute it needs to
   extract in order to use it to search the correct user entry. This
   allows use of certificates without an explicit per-user provisioning
   step when they are given a certificate/Smart Card.
-  **Match on a Kerberos principal**: this is a special case of the
   *Match on custom certificate attributes* option, but it is being
   called out specifically as it would be the best default for FreeIPA
   issued certificates that already contain the Kerberos principal SAN
   extension and thus no modification to FreeIPA is needed.

Given the high development costs of a *CertificateMatch*, this design
aims to cover support for *Binary match on whole certificate* option and
also discusses the proposal for the *Match on custom certificate
attributes* option.

.. _certificate_identity_mapping:

Certificate Identity Mapping
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As noted in the *Match on custom certificate attributes* option, admin
should be able to specify custom rules for extracting the selected
certificate fields and use them in a user search filter. This leads to a
new **Certificate Identity Mapping** concept - a server side
configuration that would specify how the user should be search when
specific certificates are used.

**Certificate Identity Mapping parts**

-  **Selector**: what certificates does it apply to - especially issuer.
   It can also select key usage or other certificate fields that the
   client (SSSD) will use to match the authentication Smart
   Card/certificate
-  **LDAP filter**: the actual LDAP filter with placeholders from the
   certificate that will be used for searching the user. The placeholder
   should be an ASN.1 compatible way of telling SSSD the path to the
   attribute itself. It should also support custom certificate
   extensions, i.e. custom OID

When Smart Card is authenticating to the system, SSSD would check all
the Certificate Identity Mapping, see which matches the public
certificate, extract the selected fields and use the resulting LDAP
filter for user search.

The search itself is only one part of the problem. FreeIPA also needs to
be able to **store** the unique per user configuration and an attribute
and let admin set it.

.. _storing_custom_user_identifier:

Storing Custom User Identifier
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When *Certificate Identity Mapping* are used, FreeIPA should be able to
provide a convenient attribute that can be used for storing the custom
matching field value (unless standard user attribute like ``employeID``
is used). There should be a new attribute just for this purpose, with
Syntax and MatchingRule flexible enough to cover most use cases:

=== ================ ====== ====================================
OID Attribute Name   Syntax Description
=== ================ ====== ====================================
TBD ipaUserCertMatch TBD    Custom matching attribute selection.
=== ================ ====== ====================================

.. _granularity_of_the_certificate_identity_mapping:

Granularity of the Certificate Identity Mapping
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Admins should be able to set the profile both for internal CA, but
especially for the **external CA**, from which each may use different
combination of the fields for the matching.

.. _changes_to_certificate_bookkeeping:

Changes to Certificate Bookkeeping
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This design changes the default expectation of current cert management
API for hosts and services as it always expected ``userCertificate`` to
be filled and used for certificate manipulation and bookkeeping
(listing, revoking). Instead, some certificates may not be stored in the
user entry at all and only stored in Dogtag (especially the short lived
certificates).

.. _revocation_of_the_certificates:

Revocation of the Certificates
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All older version of FreeIPA always revoked the certificate when an
object (host, service) was deleted or disabled. With every revocation,
the certificate has to be added to the revocation list and properly
distributed in the CRL object. Big CRL revocation lists may cause issues
with replication
(`#4048 <https://fedorahosted.org/freeipa/ticket/4048>`__) or CRL
processing.

With this feature, objects are likely to have multiple certificates and
the revocation list growth would increase even more. This design
therefore plans to **stop revoking certificates automatically** as in
most cases (deleting objects/certificates when not useful) the
certificate does not have to be added to the revocation list and can be
simply deleted and left for natural expiration. However, there are still
cases when revocation is due (the key was compromised, user leaves
organization and retained a copy of the private key) and FreeIPA needs
to have ability to revoke these certificates.



Feature Management
------------------

UI
~~

When viewing an active user's entry in FreeIPA WebUI the number of
certificates issued to the user will be displayed along with a button to
view a list of these certificates as Base64 encoded blobs. The
functionality to add and remove arbitrary certificates to the user (UI
counterpart of commands discussed in the section below) will also be
developed.

WebUI with also be extended to allow to request certificates for users
when a suitable `Certificate Profile <V4/Certificate_Profiles>`__ to
handle this task is configured.

CLI
~~~

Both ``userCertificate`` values for externally issued certificates and
the special matching attribute (``ipaUserCertMatch``) can be added with
the standard ``object-mod`` command. These entries can live in similar
tree as the `Sub-CAs <V4/Sub-CAs>`__.

Using `Certificate Profiles <V4/Certificate_Profiles>`__ feature it is
possible to issue certificates to users using ``ipa cert-request``
command. If the profile has ``ipaCertProfileStoreIssued`` attribute set
to ``TRUE``, then the whole DER encoded certificate blob will be stored
in the ``userCertificate;binary`` attribute of the user entry.

Moreover, ``ipa user-add-cert`` and ``ipa user-remove-cert`` commands
were developed to add or remove arbitrary certificates to/from user's
``userCertificate;binary`` attribute. Both command share the following
syntax:

::

    ipa user-{add|remove}-cert [UID] --certificate=[BASE64 BLOB] 

where ``UID`` corresponds to the user login and ``BASE64 BLOB`` is the
Base64 encoded blob between ``-----BEGIN CERTIFICATE-----`` and
``-----END CERTIFICATE-----`` lines in standard PEM certificate. The
``--certificate`` can be specified more that once to add or remove
multiple certificates in one call.

Configuration
~~~~~~~~~~~~~

A **per certificate profile configuration** (`Certificate Profile
design <V4/Certificate_Profiles>`__) should be added, allowing admin
select the proper ``userCertificate`` field treatment for the respective
profile, mostly based whether the certificate is short-lived or
long-lived:

-  Store certificate
-  Do not track issued certificate

If the *Store certificate* option is selected, there should be another
option available:

-  Automatically revoke certificate when identity is disabled or deleted

Upgrade
-------

Upgraded FreeIPA servers should default to *Store issued and enrolled
certificates* to avoid change of behavior with service and host
certificates. New installations should default to only *Record issued
and enrolled certificate* to avoid storing unnecessary data.

.. _how_to_test40:

How to Test
-----------

.. _using_freeipadogtag_pki_to_issue_user_certificates:

Using FreeIPA/Dogtag PKI to issue user certificates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  create/import a new certificate profile for handling requests for
   user certificates. For quick testing of the feature you can just
   export the default FreeIPA certificate profile to a file, change the
   ``profileId`` and ``desc`` fields to values you like and import the
   modified profile back to FreeIPA:

::

   $ ipa certprofile-show caIPAserviceCert --out=caIPAuserCert.txt
   ...edit the resulting file...
   $ ipa certprofile-import caIPAuserCert --file=caIPAuserCert.txt --store=True

For a comprehensive guide to preparation of certificate profile tailored
to a specifc use case see `this blog post from Fraser
Tweedale <https://blog-ftweedal.rhcloud.com/2015/08/user-certificates-and-custom-profiles-with-freeipa-4-2/>`__

-  add a new CA ACL which permits requesting certificates for user
   entries and add the custom profile to this CA ACL

::

   $ ipa caacl-add users_caIPAuserCert --usercat=all
   $ ipa caacl-add-profile users_caIPAuserCert --certprofiles=caIPAuserCert

-  generate a certificate request for the user e.g. using OpenSSL

::

   $ openssl req -new -newkey rsa:2048 -days 365 -nodes -keyout private.key -out cert.csr -subj '/CN=tuser'

-  use ``ipa cert-request`` command to request a new certificate for the
   user.

::

   $ ipa cert-request cert.csr --principal=tuser --profile-id=caIPAuserCert

-  If using WebUI go to *Authentication* → *Certificates* and click the
   ``Issue`` button. Select the custom profile in the *Profile ID*
   drop-down menu, fill in the user login in *Principal* field and paste
   the Base64 encoded CSR into the text field
-  ``ipa user-show`` and the user entry in WebUI should now display the
   Base64 encoded certificate in addition to other attributes

::

   $ ipa user-show tuser
     User login: tuser
     First name: Test
     Last name: User
     Home directory: /home/tuser
     Login shell: /bin/sh
     Email address: tuser@ipadom.org
     UID: 553000003
     GID: 553000003
     Certificate: MIID/zCCAuegAwIBAgIBCz...
     Account disabled: False
     Password: False
     Member of groups: ipausers
     Kerberos keys available: False

-  the newly issued certificate should be visible when viewing list of
   certificates in WebUI

.. _using_cli_commands_to_manager_user_certificates:

Using CLI commands to manager user certificates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  generate one or more self-signed certificates using e.g. OpenSSL

::

   $ openssl req -x509 -newkey rsa:2048 -days 365 -nodes -keyout private.key -out cert.pem -subj '/CN=tuser'

-  convert the certificate do DER for easier handling through CLI

::

   $ openssl x509 -outform der -in cert.pem -out cert.der

-  use ``ipa user-add-cert`` to add the certificate(s) to the user:

::

   $ ipa user-add-cert tuser --certificate="$(base64 cert.der)"

-  use ``ipa user-show tuser`` or view the user in the WebUI to verify
   that the newly added certificate is displayed

::

   $ ipa user-show tuser
     User login: tuser
     First name: Test
     Last name: User
     Home directory: /home/tuser
     Login shell: /bin/sh
     Email address: tuser@ipadom.org
     UID: 553000003
     GID: 553000003
     Certificate: MIIC8zCCAdugAwIBAgI...
     Account disabled: False
     Password: False
     Member of groups: ipausers
     Kerberos keys available: False

-  check that the following error is raised when you try to add the same
   certificate again

::

   ipa: ERROR: 'usercertificate;binary' already contains one or more values

-  remove the certificate from the user entry using
   ``ipa user-remove-cert``

::

   $ ipa user-remove-cert tuser --certificate="$(base64 cert.der)"

-  run ``ipa user-show tuser`` or view the user in the WebUI; the
   certificate will be no longer present

::

   $ ipa user-show tuser
     User login: tuser
     First name: Test
     Last name: User
     Home directory: /home/tuser
     Login shell: /bin/sh
     Email address: tuser@ipadom.org
     UID: 553000003
     GID: 553000003
     Account disabled: False
     Password: False
     Member of groups: ipausers
     Kerberos keys available: False

-  check that an attempt to remove already removed certificate will
   raise the error:

::

   ipa: ERROR: usercertificate;binary does not contain 'one or more values to remove'

.. _using_sssd_to_lookup_users_by_certificate:

Using SSSD to lookup users by certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Starting with `version
1.13.0 <https://fedorahosted.org/sssd/wiki/Releases/Notes-1.13.0>`__
SSSD is now able to lookup user entries by the certificates issued to
them. To test this feature *sssd-dbus* package must be installed.

-  enable SSSD Infopipe D-Bus interface by adding ``ifp`` to the
   ``services`` entry in the ``[sssd]`` section of SSSD configuration
   file (``/etc/sssd/sssd.conf`` on Fedora).
-  restart sssd
-  add a certificate to the user entry with one of the methods discussed
   in previous sections. Make sure you have the PEM certificate file at
   hand
-  `Query the SSSD D-Bus
   interface <https://fedorahosted.org/sssd/wiki/DesignDocs/LookupUsersByCertificate>`__
   for the user entry associated with the certificate by running the
   following command as root:

::

   # dbus-send --system --print-reply  --dest=org.freedesktop.sssd.infopipe /org/freedesktop/sssd/infopipe/Users \
   org.freedesktop.sssd.infopipe.Users.FindByCertificate string:"$(cat cert.pem)"

-  check that the last element of the object path returned by the D-Bus
   interface is the same as the UID of the user possessing the
   certificate:

::

   method return sender=:1.792 -> dest=:1.793 reply_serial=2 object path 
   "/org/freedesktop/sssd/infopipe/Users/ipadom_2eorg/883600001"

.. _use_case_smart_card_authentication_using_sssd_and_freeipa:

Use case: smart card authentication using SSSD and FreeIPA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Nathan Kinder put together blog series focused on practical provisioning
of Smart Cards using OpenSC and testing it with FreeIPA:

-  `Part 1: Using smart cards with
   FreeIPA <https://blog-nkinder.rhcloud.com/?p=179>`__
-  `Part 2: Using smart cards with
   FreeIPA <https://blog-nkinder.rhcloud.com/?p=184>`__

References
----------

-  `Nathan Kinder: Part 1: Using smart cards with
   FreeIPA <https://blog-nkinder.rhcloud.com/?p=179>`__
-  `Nathan Kinder: Part 2: Using smart cards with
   FreeIPA <https://blog-nkinder.rhcloud.com/?p=184>`__
-  `Fraser Tweedale: User certificates and custom profiles with FreeIPA
   4.2 <https://blog-ftweedal.rhcloud.com/2015/08/user-certificates-and-custom-profiles-with-freeipa-4-2/>`__



Test Plan
---------

TBD
