Overview
--------

FreeIPA code is using certificates in many different ways, through
various API and tools (NSS, openssl, ...) The goal of this refactoring
is to provide consistent methods to handle certificates, and will
consist in 3 different efforts:

-  Use python-cryptography instead of ipalib.x509 and ipalib.pkcs10
   (ticket #6398)
-  Use certmonger to request certificates during installation (ticket
   #6433)
-  Provide a consistent certificate DB API

Design
------

.. _certificate_issuance_during_installation:

Certificate issuance during installation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

During FreeIPA installation, 3 different certificates are requested:

-  a certificate for the renew agent (CN=IPA RA,O=$DOMAIN) stored in
   /etc/httpd/alias with the nickname **ipaCert**
-  a certificate for the LDAP server (CN=$FQDN,$DOMAIN) stored in
   /etc/dirsrv/slapd-$DOMAIN) with the nickname **Server-Cert**
-  a certificate for the HTTP server (CN=$FQDN,$DOMAIN) stored in
   /etc/httpd/alias) with the nickname **Server-Cert**

The methods to obtain those certificates are not consistent but could be
unified by using certmonger to process the issuance. Furthermore, the
certificates are later tracked by certmonger. The code needs to take
into consideration the profile that will be used for each certificate,
as well as certmonger's CA helper responsible for the request and the
tracking.

ipaCert
^^^^^^^

This certificate is requested early in the installation process, after
Dogtag is configured. At this point, Dogtag has created a certificate
for ipa-ca-agent in a temporary NSS DB (specified in the pkispawn config
file as the parameter pki_client_database_dir) that can be used to
request the RA agent cert.

ipaCert must be requested and tracked by dogtag-ipa-ca-renew-agent CA
helper, meaning that the helper needs to be modified in order to use the
temp cert ipa-ca-agent to communicate with Dogtag. Pseudo-code for the
ra request would be the following:

-  configure dogtag-ipa-ca-renew-agent CA helper with helper-location:
   /usr/libexec/certmonger/dogtag-ipa-ca-renew-agent-submit
-  RA request:

   -  temporarily modify dogtag-ipa-ca-renew-agent CA helper to use
      ipa-ca-agent stored in the temp NSS DB
   -  request ipaCert through certmonger, with the profile caServerCert
   -  restore original values for dogtag-ipa-ca-renew-agent CA helper

It is important to use the profile caServerCert as it does not define
auth.instance_id, meaning that it allows submission without
authentication.

.. _ldaphttp_server_certificate:

LDAP/HTTP server certificate
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

These certificate must be requested and tracked using IPA CA helper,
with the profile caIPAserviceCert. This profile is defined in Dogtag
with auth.instance_id=raCertAuth, meaning that the submission must be
performed with authentication, using a SSL certificate for a user member
of cn=Registration Manager Agents,ou=groups,o=ipaca. The ipaCert
certificate can be used in this case. The issue is that IPA is not
completely configured at this point and IPA CA helper will not be able
to handle the certificate request. To workaround this, we need to
temporarily reconfigure IPA CA helper to use
/usr/libexec/certmonger/dogtag-submit instead of
/usr/libexec/certmonger/ipa-submit. Pseudo code:

-  temporarily modify IPA CA helper to use dogtag-submit and agent
   authentication with ipaCert
-  request Server-Cert through certmonger, with the profile
   caIPAserviceCert
-  restore original values for IPA CA helper

.. _refactor_certificate_processing_code_to_use_python_cryptography:

Refactor certificate processing code to use python-cryptography
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using python-nss for certificate processing has several problems,
including:

-  problems with opening multiple NSSDBs
-  problems with opening a single NSSDB in multiple processes
-  python-nss bugs and poor error reporting
-  lots of NSS functionality not exposed by python-nss

It was desired that we should refactor certificate and CSR processing
code to use python-cryptography instead of python-nss. This is largely a
mechanical procedure. Aspects of this refactoring where substantive
changes were required are detailed in the following subsections.

.. _dealing_with_distinguished_names:

Dealing with Distinguished Names
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Whereas python-nss exposed DNs as strings (in LDAP order),
python-cryptography exposes ``cryptography.x509.name.Name`` objects.
FreeIPA often needs to work with ``ipapython.dn.DN`` objects, as well as
string representations. ``DN`` objects can be stringified, so it was
necessary and sufficient to extend the ``DN`` constructor to accept
``Name`` objects. When converting a ``Name``, the constructor consults a
mapping of python-cryptography ``ObjectIdentifier`` values to LDAP
attribute names.

The current version of python-cryptography does not properly handle
multi-value RDNs. This will be fixed in python-cryptography 1.6. Once
v1.6 is available, the ``DN`` constructor should be updated accordingly.
In practice, due to the rarity of multi-value RDNs in practice, and the
fact that "flattened" DNs should still compare equal, the lack of proper
handling is unlikely to cause problems.

.. _dealing_with_known_othername_types:

Dealing with known OtherName types
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

python-cryptography exposes a single type
``cryptography.x509.general_name.OtherName`` for OtherName GeneralNames.
There are no hooks for informing python-cryptography of more specific
types that you would like to be recognised.

FreeIPA does need to parse and inspect *KRBPrincipalName* and *UPN*
OtherName values. We define corresponding subclasses of ``OtherName``,
which are constructed by application to an ``OtherName`` value, which
perform additional decoding and expose the more specific data through
additional attributes. When processing general names, we provide a
function that is applied a list of
``cryptography.x509.general_name.GeneralName`` values, and maps
``OtherName`` values of known types into the corresponding subtype,
returning the resulting list.

.. _avoiding_pkcs_10_friendlyname_attribute:

Avoiding PKCS #10 *friendlyName* attribute
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``dogtag-ipa-ca-renew-agent`` Certmonger renewal helper needs to
determine the NSSDB nickname of the certificate being renewed.
Certmonger includes this datum in the *friendlyName* attribute in the
PKCS #10 CSR. python-nss does not provide a way to access this datum, so
we used hand-rolled *pyasn1* definitions to decode this information
ourselves. We wished to remove this.

python-cryptography, like python-nss, does not include a way of reading
the *friendlyName* attribute. Therefore a different approach was taken.
In ``dogtag-ipa-ca-renew-agent``, using the FreeIPA deployment's
certificate *subject base*, we construct a mapping of subject DNs to
NSSDB nicknames. We then use the value of the ``CERTMONGER_REQ_SUBJECT``
environment variable as the key to look up the NSSDB nickname.

All CSR-related *pyasn1* definitions and decoding were removed.

.. _decoding_certificate_subject_alternative_name_extension:

Decoding certificate Subject Alternative Name extension
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

python-cryptography does successfully decode certificates with
unrecognised *critical* extensions, because the decoding of extensions
is deferred until the user attempts to access the ``extensions``
attribute (``UnrecognisedExtension`` will be thrown at this point).
Until such time as python-cryptography provides a way to handle this
scenario, to process the certificate Subject Alternative Name (SAN)
extension, we must decode the *TBSCertificate* data ourselves.

Using definitions provided by the *pyasn1-modules* library, we decode
the value of the ``cryptography.x509.Certificate.tbs_certificate_bytes``
attribute and look for the SAN extension. Then for each general name, we
instantiate the relevant ``cryptography.x509.general_name.GeneralName``
subclass (this is a forward-compatiblity measure and ensures commonality
among higher-level functions that process general names). We *do not*
convert known OtherName types into subclasses of ``OtherName`` at this
point (the function discuss above can be used to do this).
