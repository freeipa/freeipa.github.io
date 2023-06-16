.. _ldap_schema_for_pkcs11_data:

LDAP schema for PKCS#11 data
============================

Introduction
------------

Components of FreeIPA need to use PKCS#11 interface (`v2.20 and draft
v2.30 <http://www.emc.com/emc-plus/rsa-labs/standards-initiatives/pkcs-11-cryptographic-token-interface-standard.htm>`__
; `draft
v2.40 <https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=pkcs11>`__)
to manipulate key material and certificates.

As usual, FreeIPA wants to have all the data distributed so LDAP
database seems as natural choice for backend. The problem is that the
required schema does not yet exist. This is an initial proposal for a
schema definition, which can be extended if more PKCS#11 data will be
stored in LDAP.

Discussion about this schema is happening on freeipa-devel list:

-  `2014
   February <https://www.redhat.com/archives/freeipa-devel/2014-February/msg00223.html>`__
-  `2014
   March <https://www.redhat.com/archives/freeipa-devel/2014-March/msg00007.html>`__

Assumption: There is a base OID defined for used by FreeIPA.

General OIDs used in the schema definition:

| ``.``\ ``     # all ipk11 schema elements``
| ``.``\ ``.1       # objectclasses``
| ``.``\ ``.2       # attributeTypes``
| ``# ``\ ``.``\ ``.3..n  # syntaxes, matching rules ... if needed``
| `` := ``\ ``.``

All objectclass and attribute type names will be prefixed by ipk11.

.. _attribute_types:

Attribute types
---------------

.. _uniqued_identifier:

Uniqued identifier
~~~~~~~~~~~~~~~~~~

-  ipk11UniqueId

   -  LDAP-only, no corresponding PKCS#11 attribute

| ``(``\ ``.1.1 NAME 'ipk11UniqueId'``
| `` DESC 'Unique identifier'``
| `` EQUALITY caseIgnoreMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

.. _common_metadata:

Common metadata
~~~~~~~~~~~~~~~

Metadata attributes common to multiple object classes.

-  ipk11Private

   -  corresponds to CKA_PRIVATE

| ``(``\ ``.1.11 NAME 'ipk11Private'``
| `` DESC 'Is private to application'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Modifiable

   -  corresponds to CKA_MODIFIABLE

| ``(``\ ``.1.12 NAME 'ipk11Modifiable'``
| `` DESC 'Can be modified by application'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Label

   -  corresponds to CKA_LABEL

| ``(``\ ``.1.13 NAME 'ipk11Label'``
| `` DESC 'Description'``
| `` EQUALITY caseExactMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Copyable

   -  corresponds to CKA_COPYABLE (PKCS#11 2.30)

| ``(``\ ``.1.14 NAME 'ipk11Copyable'``
| `` DESC 'Can be copied by application'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Destroyable

   -  corresponds to CKA_DESTROYABLE (PKCS#11 2.40)

| ``(``\ ``.1.15 NAME 'ipk11Destroyable'``
| `` DESC 'Can be destroyed by application'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Trusted

   -  corresponds to CKA_TRUSTED

| ``(``\ ``.1.16 NAME 'ipk11Trusted'``
| `` DESC 'Can be trusted by application'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11CheckValue

   -  corresponds to CKA_CHECK_VALUE

| ``(``\ ``.1.17 NAME 'ipk11CheckValue'``
| `` DESC 'Checksum'``
| `` EQUALITY octetStringMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.40``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11StartDate

   -  corresponds to CKA_START_DATE

| ``(``\ ``.1.18 NAME 'ipk11StartDate'``
| `` DESC 'Validity start date'``
| `` EQUALITY generalizedTimeMatch``
| `` ORDERING generalizedTimeOrderingMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.24``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11EndDate

   -  corresponds to CKA_END_DATE

| ``(``\ ``.1.19 NAME 'ipk11EndDate'``
| `` DESC 'Validity end date'``
| `` EQUALITY generalizedTimeMatch``
| `` ORDERING generalizedTimeOrderingMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.24``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11PublicKeyInfo

   -  corresponds to CKA_PUBLIC_KEY_INFO (PKCS#11 2.40) /
      CKA_X_PUBLIC_KEY_INFO (p11-kit)
   -  SubjectPublicKeyInfo is defined in `RFC
      5280 <http://tools.ietf.org/html/rfc5280#section-4.1>`__
   -  SubjectPublicKey for RSA public keys is defined in `RFC
      4055 <http://tools.ietf.org/html/rfc4055#section-1.2>`__

| ``(``\ ``.1.20 NAME 'ipk11PublicKeyInfo'``
| `` DESC 'DER-encoding of SubjectPublicKeyInfo of associated public key'``
| `` EQUALITY octetStringMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.40``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Distrusted

   -  corresponds to CKA_X_DISTRUSTED (p11-kit)

| ``(``\ ``.1.21 NAME 'ipk11Distrusted'``
| `` DESC 'Must not be trusted by application'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Subject

   -  corresponds to CKA_SUBJECT

| ``(``\ ``.1.22 NAME 'ipk11Subject'``
| `` DESC 'DER-encoding of subject name'``
| `` EQUALITY octetStringMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.40``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Id

   -  corresponds to CKA_ID

| ``(``\ ``.1.23 NAME 'ipk11Id'``
| `` DESC 'Key association identifier'``
| `` EQUALITY octetStringMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.40``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Local

   -  corresponds to CKA_LOCAL

| ``(``\ ``.1.24 NAME 'ipk11Local'``
| `` DESC 'Was created locally on token'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

.. _certificate_metadata:

Certificate metadata
~~~~~~~~~~~~~~~~~~~~

Metadata attributes specific to certificates.

-  ipk11Issuer

   -  corresponds to CKA_ISSUER

| ``(``\ ``.1.33 NAME 'ipk11Issuer'``
| `` DESC 'DER-encoding of issuer name'``
| `` EQUALITY octetStringMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.40``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11SerialNumber

   -  corresponds to CKA_SERIAL_NUMBER

| ``(``\ ``.1.34 NAME 'ipk11SerialNumber'``
| `` DESC 'DER-encoding of serial number'``
| `` EQUALITY octetStringMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.40``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11SubjectKeyHash

   -  corresponds to CKA_HASH_OF_SUBJECT_PUBLIC_KEY and
      CKA_NAME_HASH_ALGORITHM
   -  valid values: "*mechanism* *hexdigest*"

| ``(``\ ``.1.37 NAME 'ipk11SubjectKeyHash'``
| `` DESC 'Hash of subject public key'``
| `` EQUALITY caseIgnoreMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11IssuerKeyHash

   -  corresponds to CKA_HASH_OF_ISSUER_PUBLIC_KEY and
      CKA_NAME_HASH_ALGORITHM
   -  valid values: "*mechanism* *hexdigest*"

| ``(``\ ``.1.38 NAME 'ipk11IssuerKeyHash'``
| `` DESC 'Hash of issuer public key'``
| `` EQUALITY caseIgnoreMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11SecurityDomain

   -  corresponds to CKA_JAVA_MIDP_SECURITY_DOMAIN
   -  valid values: "manufacturer", "operator", "thirdParty"

| ``(``\ ``.1.39 NAME 'ipk11SecurityDomain'``
| `` DESC 'Java MIDP security domain'``
| `` EQUALITY caseIgnoreMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

.. _common_key_metadata:

Common key metadata
~~~~~~~~~~~~~~~~~~~

Metadata attributes common to all key object classes.

-  ipk11KeyType

   -  corresponds to CKA_KEY_TYPE

| ``(``\ ``.1.41 NAME 'ipk11KeyType'``
| `` DESC 'Key type'``
| `` EQUALITY caseIgnoreMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Derive

   -  corresponds to CKA_DERIVE

| ``(``\ ``.1.42 NAME 'ipk11Derive'``
| `` DESC 'Key supports key derivation'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11KeyGenMechanism

   -  corresponds to CKA_KEY_GEN_MECHANISM
   -  valid values: any mechanism name

| ``(``\ ``.1.43 NAME 'ipk11KeyGenMechanism'``
| `` DESC 'Mechanism used to generate this key'``
| `` EQUALITY caseIgnoreMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11AllowedMechanisms

   -  corresponds to CKA_ALLOWED_MECHANISMS
   -  valid values: one or more mechanism names separated by space

| ``(``\ ``.1.44 NAME 'ipk11AllowedMechanisms'``
| `` DESC 'Space-separated list of mechanisms allowed to be used with this key'``
| `` EQUALITY caseIgnoreMatch``
| `` SUBSTR caseIgnoreSubstringsMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

.. _public_key_metadata:

Public key metadata
~~~~~~~~~~~~~~~~~~~

Metadata attributes specific to public and secret keys.

-  ipk11Encrypt

   -  corresponds to CKA_ENCRYPT

| ``(``\ ``.1.51 NAME 'ipk11Encrypt'``
| `` DESC 'Key supports encryption'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Verify

   -  corresponds to CKA_VERIFY

| ``(``\ ``.1.52 NAME 'ipk11Verify'``
| `` DESC 'Key supports verification where the signature is an appendix to the data'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11VerifyRecover

   -  corresponds to CKA_VERIFY_RECOVER

| ``(``\ ``.1.53 NAME 'ipk11VerifyRecover'``
| `` DESC 'Key supports verification where data is recovered from the signature'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Wrap

   -  corresponds to CKA_WRAP

| ``(``\ ``.1.54 NAME 'ipk11Wrap'``
| `` DESC 'Key supports wrapping'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11WrapTemplate

   -  corresponds to CKA_WRAP_TEMPLATE

| ``(``\ ``.1.55 NAME 'ipk11WrapTemplate'``
| `` DESC 'DN of template of keys which can be wrapped using this key'``
| `` EQUALITY distinguishedNameMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.12``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

.. _private_key_metadata:

Private key metadata
~~~~~~~~~~~~~~~~~~~~

Metadata attributes specific to private and secret keys.

-  ipk11Sensitive

   -  corresponds to CKA_SENSITIVE

| ``(``\ ``.1.61 NAME 'ipk11Sensitive'``
| `` DESC 'Key is sensitive'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Decrypt

   -  corresponds to CKA_DECRYPT

| ``(``\ ``.1.62 NAME 'ipk11Decrypt'``
| `` DESC 'Key supports decryption'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Sign

   -  corresponds to CKA_SIGN

| ``(``\ ``.1.63 NAME 'ipk11Sign'``
| `` DESC 'Key supports signatures where the signature is an appendix to the data'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11SignRecover

   -  corresponds to CKA_SIGN_RECOVER

| ``(``\ ``.1.64 NAME 'ipk11SignRecover'``
| `` DESC 'Key supports signatures where data can be recovered from the signature'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Unwrap

   -  corresponds to CKA_UNWRAP

| ``(``\ ``.1.65 NAME 'ipk11Unwrap'``
| `` DESC 'Key supports unwrapping'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Extractable

   -  corresponds to CKA_EXTRACTABLE

| ``(``\ ``.1.66 NAME 'ipk11Extractable'``
| `` DESC 'Key is extractable and can be wrapped'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11AlwaysSensitive

   -  corresponds to CKA_ALWAYS_SENSITIVE

| ``(``\ ``.1.67 NAME 'ipk11AlwaysSensitive'``
| `` DESC 'Key has always been sensitive'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11NeverExtractable

   -  corresponds to CKA_NEVER_EXTRACTABLE

| ``(``\ ``.1.68 NAME 'ipk11NeverExtractable'``
| `` DESC 'Key has never been extractable'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11WrapWithTrusted

   -  corresponds to CKA_WRAP_WITH_TRUSTED

| ``(``\ ``.1.69 NAME 'ipk11WrapWithTrusted'``
| `` DESC 'Key can only be wrapped with a trusted wrapping key'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11UnwrapTemplate

   -  corresponds to CKA_UNWRAP_TEMPLATE

| ``(``\ ``.1.70 NAME 'ipk11UnwrapTemplate'``
| `` DESC 'DN of template to apply to keys unwrapped using this key'``
| `` EQUALITY distinguishedNameMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.12``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11AlwaysAuthenticate

   -  corresponds to CKA_ALWAYS_AUTHENTICATE

| ``(``\ ``.1.71 NAME 'ipk11AlwaysAuthenticate'``
| `` DESC 'User has to authenticate for each use with this key'``
| `` EQUALITY booleanMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.7``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

.. _encoded_key_data:

Encoded key data
~~~~~~~~~~~~~~~~

In PKCS#11 objects are defined as sets of attributes, but for keys and
certificates there should be the possibility to store the complete
entity in one attribute in a specific format.

-  ipaPublicKey

   -  was previously called `ipaPublicKeyInfo <#ipaPublicKey>`__

| ``(2.16.840.1.113730.3.8.11.53 NAME 'ipaPublicKey'``
| `` DESC 'Public key as DER-encoded SubjectPublicKeyInfo (RFC 5280)'``
| `` EQUALITY octetStringMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.40``
| `` X-ORIGIN 'IPA v4' )``

-  ipaPrivateKey

   -  was previously called ipaEPrivateKeyInfo

| ``(2.16.840.1.113730.3.8.11.54 NAME 'ipaPrivateKey'``
| `` DESC 'Private key as encrypted DER-encoded PrivateKeyInfo (RFC 5958)'``
| `` EQUALITY octetStringMatch``
| `` SINGLE-VALUE``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.40``
| `` X-ORIGIN 'IPA v4' )``

-  ipaSecretKey

   -  The attribute is single-valued on purpose. You should combine
      `ipk11SecretKey <#ipk11SecretKey>`__ and
      `ipaSecretKeyRefObject <#ipaSecretKeyRefObject>`__ object classes
      to store multiple variants of the secret key in separate objects.
      This groups wrapped blobs with metadata like `wrapping
      mechanism <#ipaWrappingMech>`__ and `wrapping key
      URI <#ipaWrappingKey>`__.

| ``(2.16.840.1.113730.3.8.11.55 NAME 'ipaSecretKey'``
| `` DESC 'Encrypted secret key data'``
| `` EQUALITY octetStringMatch``
| `` SINGLE-VALUE``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.40``
| `` X-ORIGIN 'IPA v4' )``

.. _wrapping_key_reference:

Wrapping key reference
~~~~~~~~~~~~~~~~~~~~~~

-  ipaWrappingKey

   -  Pointer to wrapping key
   -  PKCS#11 URI according to
      `draft-pechanec-pkcs11uri <http://tools.ietf.org/html/draft-pechanec-pkcs11uri>`__,
      including "pkcs11:" prefix

| `` (2.16.840.1.113730.3.8.11.61 NAME 'ipaWrappingKey'``
| `` DESC 'PKCS#11 URI of the wrapping key'``
| `` EQUALITY caseExactMatch``
| `` SINGLE-VALUE``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 )``

-  ipaWrappingMech

   -  corresponds to wrapping mechanism used for key wrapping

| ``(2.16.840.1.113730.3.8.11.65 'ipaWrappingMech'``
| `` DESC 'PKCS#11 wrapping mechanism equivalent to CK_MECHANISM_TYPE'``
| `` EQUALITY caseIgnoreMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| `` SINGLE-VALUE``
| `` X-ORIGIN 'IPA v4' )``

-  ipaSecretKeyRef

   -  Pointer to `ipaSecretKeyObject <#ipaSecretKeyObject>`__ or
      `ipaPrivateKeyObject <#ipaPrivateKeyObject>`__
   -  This multi-valued attribute allows you to share one metadata
      object (e.g. `ipk11SecretKey <#ipk11SecretKey>`__) among multiple
      encrypted key blobs, i.e. one key wrapped with more than one key

| `` (2.16.840.1.113730.3.8.11.64 NAME 'ipaSecretKeyRef'``
| `` DESC 'DN of the ipaSecretKeyObject'``
| `` EQUALITY distinguishedNameMatch``
| `` SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 )``

.. _object_classes:

Object classes
--------------

.. _structural_object_class:

Structural object class
~~~~~~~~~~~~~~~~~~~~~~~

For use in a PKCS#11 only database a structural objectclass is defined.

-  ipk11Object

   -  LDAP-only, no corresponding PKCS#11 object class

| ``(``\ ``.2.1 NAME 'ipk11Object'``
| `` DESC 'Object'``
| `` SUP top STRUCTURAL``
| `` MUST   ``\ ```ipk11UniqueId`` <#ipk11UniqueId>`__
| `` X-ORIGIN 'IPA v4' )``

.. _storage_objects:

Storage objects
~~~~~~~~~~~~~~~

This schema defines a mapping of PKCS#11 storage object classes
CKO_CERTIFICATE, CKO_PUBLIC_KEY and CKO_PRIVATE_KEY. These objectclasses
are auxiliary and can be used to extend other objects.

-  ipk11StorageObject

   -  abstract base class of all PKCS#11 storage objects

| ``(``\ ``.2.2 NAME 'ipk11StorageObject'``
| `` DESC 'Storage object'``
| `` SUP top ABSTRACT``
| `` MAY  ( ``\ ```ipk11Private`` <#ipk11Private>`__\ `` $ ``\ ```ipk11Modifiable`` <#ipk11Modifiable>`__\ `` $ ``\ ```ipk11Label`` <#ipk11Label>`__\ `` $ ``\ ```ipk11Copyable`` <#ipk11Copyable>`__\ `` $``
| ``        ``\ ```ipk11Destroyable`` <#ipk11Destroyable>`__\ `` )``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Certificate

   -  abstract base class of CKO_CERTIFICATE objects

| ``(``\ ``.2.3 NAME 'ipk11Certificate'``
| `` DESC 'Certificate'``
| `` SUP ``\ ```ipk11StorageObject`` <#ipk11StorageObject>`__\ `` ABSTRACT``
| `` MAY  ( ``\ ```ipk11Trusted`` <#ipk11Trusted>`__\ `` $ ``\ ```ipk11CheckValue`` <#ipk11CheckValue>`__\ `` $ ``\ ```ipk11StartDate`` <#ipk11StartDate>`__\ `` $ ``\ ```ipk11EndDate`` <#ipk11EndDate>`__\ `` $``
| ``        ``\ ```ipk11PublicKeyInfo`` <#ipk11PublicKeyInfo>`__\ `` $ ``\ ```ipk11Distrusted`` <#ipk11Distrusted>`__\ `` )``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11X509Certificate

   -  corresponds to CKO_CERTIFICATE of type CKC_X_509

| ``(``\ ``.2.4 NAME 'ipk11X509Certificate'``
| `` DESC 'X.509 certificate'``
| `` SUP ``\ ```ipk11Certificate`` <#ipk11Certificate>`__\ `` AUXILIARY``
| `` MAY  ( ``\ ```ipk11Subject`` <#ipk11Subject>`__\ `` $ ``\ ```ipk11Id`` <#ipk11Id>`__\ `` $ ``\ ```ipk11Issuer`` <#ipk11Issuer>`__\ `` $ ``\ ```ipk11SerialNumber`` <#ipk11SerialNumber>`__\ `` $``
| ``        ``\ ```ipk11SubjectKeyHash`` <#ipk11SubjectKeyHash>`__\ `` $ ``\ ```ipk11IssuerKeyHash`` <#ipk11IssuerKeyHash>`__\ `` $ ``\ ```ipk11SecurityDomain`` <#ipk11SecurityDomain>`__\ `` )``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11Key

   -  abstract base class of all PKCS#11 key objects

| ``(``\ ``.2.5 NAME 'ipk11Key'``
| `` DESC 'Key'``
| `` SUP ``\ ```ipk11StorageObject`` <#ipk11StorageObject>`__\ `` ABSTRACT``
| `` MAY  ( ``\ ```ipk11KeyType`` <#ipk11KeyType>`__\ `` $ ``\ ```ipk11Id`` <#ipk11Id>`__\ `` $ ``\ ```ipk11StartDate`` <#ipk11StartDate>`__\ `` $ ``\ ```ipk11EndDate`` <#ipk11EndDate>`__\ `` $ ``\ ```ipk11Derive`` <#ipk11Derive>`__\ `` $``
| ``        ``\ ```ipk11Local`` <#ipk11Local>`__\ `` $ ``\ ```ipk11KeyGenMechanism`` <#ipk11KeyGenMechanism>`__\ `` $ ``\ ```ipk11AllowedMechanisms`` <#ipk11AllowedMechanisms>`__\ `` )``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11PublicKey

   -  corresponds to CKO_PUBLIC_KEY

| ``(``\ ``.2.6 NAME 'ipk11PublicKey'``
| `` DESC 'Public key'``
| `` SUP ``\ ```ipk11Key`` <#ipk11Key>`__\ `` AUXILIARY``
| `` MAY  ( ``\ ```ipk11Subject`` <#ipk11Subject>`__\ `` $ ``\ ```ipk11Encrypt`` <#ipk11Encrypt>`__\ `` $ ``\ ```ipk11Verify`` <#ipk11Verify>`__\ `` $ ``\ ```ipk11VerifyRecover`` <#ipk11VerifyRecover>`__\ `` $ ``\ ```ipk11Wrap`` <#ipk11Wrap>`__\ `` $``
| ``        ``\ ```ipk11Trusted`` <#ipk11Trusted>`__\ `` $ ``\ ```ipk11WrapTemplate`` <#ipk11WrapTemplate>`__\ `` $ ``\ ```ipk11Distrusted`` <#ipk11Distrusted>`__\ `` $ ``\ ```ipk11PublicKeyInfo`` <#ipk11PublicKeyInfo>`__\ `` )``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11PrivateKey

   -  corresponds to CKO_PRIVATE_KEY

| ``(``\ ``.2.7 NAME 'ipk11PrivateKey'``
| `` DESC 'Private key'``
| `` SUP ``\ ```ipk11Key`` <#ipk11Key>`__\ `` AUXILIARY``
| `` MAY  ( ``\ ```ipk11Subject`` <#ipk11Subject>`__\ `` $ ``\ ```ipk11Sensitive`` <#ipk11Sensitive>`__\ `` $ ``\ ```ipk11Decrypt`` <#ipk11Decrypt>`__\ `` $ ``\ ```ipk11Sign`` <#ipk11Sign>`__\ `` $``
| ``        ``\ ```ipk11SignRecover`` <#ipk11SignRecover>`__\ `` $ ``\ ```ipk11Unwrap`` <#ipk11Unwrap>`__\ `` $ ``\ ```ipk11Extractable`` <#ipk11Extractable>`__\ `` $ ``\ ```ipk11AlwaysSensitive`` <#ipk11AlwaysSensitive>`__\ `` $``
| ``        ``\ ```ipk11NeverExtractable`` <#ipk11NeverExtractable>`__\ `` $ ``\ ```ipk11WrapWithTrusted`` <#ipk11WrapWithTrusted>`__\ `` $ ``\ ```ipk11UnwrapTemplate`` <#ipk11UnwrapTemplate>`__\ `` $``
| ``        ``\ ```ipk11AlwaysAuthenticate`` <#ipk11AlwaysAuthenticate>`__\ `` $ ``\ ```ipk11PublicKeyInfo`` <#ipk11PublicKeyInfo>`__\ `` )``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11SecretKey

   -  corresponds to CKO_SECRET_KEY

| ``(``\ ``.2.8 NAME 'ipk11SecretKey'``
| `` DESC 'Secret key'``
| `` SUP ``\ ```ipk11Key`` <#ipk11Key>`__\ `` AUXILIARY``
| `` MAY  ( ``\ ```ipk11Sensitive`` <#ipk11Sensitive>`__\ `` $ ``\ ```ipk11Encrypt`` <#ipk11Encrypt>`__\ `` $ ``\ ```ipk11Decrypt`` <#ipk11Decrypt>`__\ `` $ ``\ ```ipk11Sign`` <#ipk11Sign>`__\ `` $ ``\ ```ipk11Verify`` <#ipk11Verify>`__\ `` $``
| ``        ``\ ```ipk11Wrap`` <#ipk11Wrap>`__\ `` $ ``\ ```ipk11Unwrap`` <#ipk11Unwrap>`__\ `` $ ``\ ```ipk11Extractable`` <#ipk11Extractable>`__\ `` $ ``\ ```ipk11AlwaysSensitive`` <#ipk11AlwaysSensitive>`__\ `` $``
| ``        ``\ ```ipk11NeverExtractable`` <#ipk11NeverExtractable>`__\ `` $ ``\ ```ipk11CheckValue`` <#ipk11CheckValue>`__\ `` $ ``\ ```ipk11WrapWithTrusted`` <#ipk11WrapWithTrusted>`__\ `` $``
| ``        ``\ ```ipk11Trusted`` <#ipk11Trusted>`__\ `` $ ``\ ```ipk11WrapTemplate`` <#ipk11WrapTemplate>`__\ `` $ ``\ ```ipk11UnwrapTemplate`` <#ipk11UnwrapTemplate>`__\ `` )``
| `` X-ORIGIN 'IPA v4' )``

-  ipk11DomainParameters

   -  corresponds to CKO_DOMAIN_PARAMETERS

| ``(``\ ``.2.9 NAME 'ipk11DomainParameters'``
| `` DESC 'Domain parameters'``
| `` SUP ``\ ```ipk11StorageObject`` <#ipk11StorageObject>`__\ `` AUXILIARY``
| `` MAY  ( ``\ ```ipk11KeyType`` <#ipk11KeyType>`__\ `` $ ``\ ```ipk11Local`` <#ipk11Local>`__\ `` )``
| `` X-ORIGIN 'IPA v4' )``

.. _encoded_key_data_1:

Encoded key data
~~~~~~~~~~~~~~~~

-  ipaPublicKeyObject

   -  was previously called `ipaPublicKey <#ipaPublicKey>`__

| ``(2.16.840.1.113730.3.8.12.24 NAME 'ipaPublicKeyObject'``
| `` DESC 'Wrapped public key'``
| `` SUP top AUXILIARY``
| `` MUST  ( ``\ ```ipaPublicKey`` <#ipaPublicKey>`__\ `` )``
| `` X-ORIGIN 'IPA v4' )``

-  ipaPrivateKeyObject

   -  was previously called ipaEPrivateKey

| ``(2.16.840.1.113730.3.8.12.25 NAME 'ipaPrivateKeyObject'``
| `` DESC 'Wrapped private key'``
| `` SUP top AUXILIARY``
| `` MUST ( ``\ ```ipaWrappingKey`` <#ipaWrappingKey>`__\ `` $ ``\ ```ipaWrappingMech`` <#ipaWrappingMech>`__\ `` $ ``\ ```ipaPrivateKey`` <#ipaPrivateKey>`__\ `` )``
| `` X-ORIGIN 'IPA v4' )``

-  ipaSecretKeyObject

| ``(2.16.840.1.113730.3.8.12.26 NAME 'ipaSecretKeyObject'``
| `` DESC 'Wrapped secret key'``
| `` SUP top AUXILIARY``
| `` MUST ( ``\ ```ipaWrappingKey`` <#ipaWrappingKey>`__\ `` $ ``\ ```ipaWrappingMech`` <#ipaWrappingMech>`__\ `` $ ``\ ```ipaSecretKey`` <#ipaSecretKey>`__\ `` )``
| `` X-ORIGIN 'IPA v4' )``

-  ipaSecretKeyRefObject

   -  Allows to extend `ipk11SecretKey <#ipk11SecretKey>`__ with
      reference to key material stored in another object(s)
   -  Use case is with DNSSEC master key: One master key shares PKCS#11
      metadata object but its key data are wrapped with multiple replica
      keys -> are stored as multiple distinct blobs.
   -  To be clear, `ipaSecretKeyRef <#ipaSecretKeyRef>`__ attribute is
      multi-valued and application has to walk through set of referenced
      LDAP entries and find suitable unwrapping key

| `` (2.16.840.1.113730.3.8.12.34 NAME 'ipaSecretKeyRefObject'``
| `` DESC 'Indirect storage for encoded key material'``
| `` SUP top AUXILIARY``
| `` MUST ``\ ```ipaSecretKeyRef`` <#ipaSecretKeyRef>`__
| `` X-ORIGIN 'IPA v4' )``

.. _pkcs11_mapping:

PKCS#11 mapping
---------------

.. _attribute_types_1:

Attribute types
~~~~~~~~~~~~~~~

-  Boolean attributes

======== =====
CK_BBOOL LDAP
======== =====
CK_TRUE  TRUE
CK_FALSE FALSE
======== =====

-  `ipk11StartDate <#Common_metadata>`__,
   `ipk11EndDate <#Common_metadata>`__

==================================================== =================
CK_DATE                                              LDAP
==================================================== =================
{ .year = "*yyyy*", .month = "*mm*", .day = "*dd*" } *yyyymmdd*\ 0000Z
==================================================== =================

-  `ipk11SecurityDomain <#Certificate_metadata>`__

=============================== ============
CK_SECURITY_DOMAIN              LDAP
=============================== ============
CK_SECURITY_DOMAIN_UNSPECIFIED  *empty*
CK_SECURITY_DOMAIN_MANUFACTURER manufacturer
CK_SECURITY_DOMAIN_OPERATOR     operator
CK_SECURITY_DOMAIN_THIRD_PARTY  thirdParty
=============================== ============

-  `ipk11KeyType <#ipk11KeyType>`__

================== =============
CK_MECHANISM_TYPE  LDAP
================== =============
CKK_RSA            rsa
CKK_DSA            dsa
CKK_DH             dh
CKK_ECDSA          ec
CKK_EC             ec
CKK_X9_42_DH       x942Dh
CKK_KEA            kea
CKK_GENERIC_SECRET genericSecret
CKK_RC2            rc2
CKK_RC4            rc4
CKK_DES            des
CKK_DES2           des2
CKK_DES3           des3
CKK_CAST           cast
CKK_CAST3          cast3
CKK_CAST5          cast128
CKK_CAST128        cast128
CKK_RC5            rc5
CKK_IDEA           idea
CKK_SKIPJACK       skipjack
CKK_BATON          baton
CKK_JUNIPER        juniper
CKK_CDMF           cdmf
CKK_AES            aes
CKK_BLOWFISH       blowfish
CKK_TWOFISH        twofish
CKK_SECURID        securid
CKK_HOTP           hotp
CKK_ACTI           acti
CKK_CAMELLIA       camellia
CKK_ARIA           aria
CKK_MD5_HMAC       md5Hmac
CKK_SHA_1_HMAC     sha1Hmac
CKK_RIPEMD128_HMAC ripemd128Hmac
CKK_RIPEMD160_HMAC ripemd160Hmac
CKK_SHA256_HMAC    sha256Hmac
CKK_SHA384_HMAC    sha384Hmac
CKK_SHA512_HMAC    sha512Hmac
CKK_SHA224_HMAC    sha224Hmac
CKK_SEED           seed
CKK_GOSTR3410      gostr3410
CKK_GOSTR3411      gostr3411
CKK_GOST28147      gost28147
================== =============

-  `ipk11KeyGenMechanism <#ipk11KeyGenMechanism>`__,
   `ipk11AllowedMechanisms <#ipk11AllowedMechanisms>`__,
   `ipaWrappingMech <#ipaWrappingMech>`__

================================== =========================
CK_MECHANISM_TYPE                  LDAP
================================== =========================
CKM_RSA_PKCS_KEY_PAIR_GEN          rsaPkcsKeyPairGen
CKM_RSA_PKCS                       rsaPkcs
CKM_RSA_9796                       rsa9796
CKM_RSA_X_509                      rsaX509
CKM_MD2_RSA_PKCS                   md2RsaPkcs
CKM_MD5_RSA_PKCS                   md5RsaPkcs
CKM_SHA1_RSA_PKCS                  sha1RsaPkcs
CKM_RIPEMD128_RSA_PKCS             ripemd128RsaPkcs
CKM_RIPEMD160_RSA_PKCS             ripemd160RsaPkcs
CKM_RSA_PKCS_OAEP                  rsaPkcsOaep
CKM_RSA_X9_31_KEY_PAIR_GEN         rsaX931KeyPairGen
CKM_RSA_X9_31                      rsaX931
CKM_SHA1_RSA_X9_31                 sha1RsaX931
CKM_RSA_PKCS_PSS                   rsaPkcsPss
CKM_SHA1_RSA_PKCS_PSS              sha1RsaPkcsPss
CKM_DSA_KEY_PAIR_GEN               dsaKeyPairGen
CKM_DSA                            dsa
CKM_DSA_SHA1                       dsaSha1
CKM_DSA_SHA224                     dsaSha224
CKM_DSA_SHA256                     dsaSha256
CKM_DSA_SHA384                     dsaSha384
CKM_DSA_SHA512                     dsaSha512
CKM_DH_PKCS_KEY_PAIR_GEN           dhPkcsKeyPairGen
CKM_DH_PKCS_DERIVE                 dhPkcsDerive
CKM_X9_42_DH_KEY_PAIR_GEN          x942DhKeyPairGen
CKM_X9_42_DH_DERIVE                x942DhDerive
CKM_X9_42_DH_HYBRID_DERIVE         x942DhHybridDerive
CKM_X9_42_MQV_DERIVE               x942MqvDerive
CKM_SHA256_RSA_PKCS                sha256RsaPkcs
CKM_SHA384_RSA_PKCS                sha384RsaPkcs
CKM_SHA512_RSA_PKCS                sha512RsaPkcs
CKM_SHA256_RSA_PKCS_PSS            sha256RsaPkcsPss
CKM_SHA384_RSA_PKCS_PSS            sha384RsaPkcsPss
CKM_SHA512_RSA_PKCS_PSS            sha512RsaPkcsPss
CKM_SHA224_RSA_PKCS                sha224RsaPkcs
CKM_SHA224_RSA_PKCS_PSS            sha224RsaPkcsPss
CKM_RC2_KEY_GEN                    rc2KeyGen
CKM_RC2_ECB                        rc2Ecb
CKM_RC2_CBC                        rc2Cbc
CKM_RC2_MAC                        rc2Mac
CKM_RC2_MAC_GENERAL                rc2MacGeneral
CKM_RC2_CBC_PAD                    rc2CbcPad
CKM_RC4_KEY_GEN                    rc4KeyGen
CKM_RC4                            rc4
CKM_DES_KEY_GEN                    desKeyGen
CKM_DES_ECB                        desEcb
CKM_DES_CBC                        desCbc
CKM_DES_MAC                        desMac
CKM_DES_MAC_GENERAL                desMacGeneral
CKM_DES_CBC_PAD                    desCbcPad
CKM_DES2_KEY_GEN                   des2KeyGen
CKM_DES3_KEY_GEN                   des3KeyGen
CKM_DES3_ECB                       des3Ecb
CKM_DES3_CBC                       des3Cbc
CKM_DES3_MAC                       des3Mac
CKM_DES3_MAC_GENERAL               des3MacGeneral
CKM_DES3_CBC_PAD                   des3CbcPad
CKM_DES3_CMAC_GENERAL              des3CmacGeneral
CKM_DES3_CMAC                      des3Cmac
CKM_CDMF_KEY_GEN                   cdmfKeyGen
CKM_CDMF_ECB                       cdmfEcb
CKM_CDMF_CBC                       cdmfCbc
CKM_CDMF_MAC                       cdmfMac
CKM_CDMF_MAC_GENERAL               cdmfMacGeneral
CKM_CDMF_CBC_PAD                   cdmfCbcPad
CKM_DES_OFB64                      desOfb64
CKM_DES_OFB8                       desOfb8
CKM_DES_CFB64                      desCfb64
CKM_DES_CFB8                       desCfb8
CKM_MD2                            md2
CKM_MD2_HMAC                       md2Hmac
CKM_MD2_HMAC_GENERAL               md2HmacGeneral
CKM_MD5                            md5
CKM_MD5_HMAC                       md5Hmac
CKM_MD5_HMAC_GENERAL               md5HmacGeneral
CKM_SHA_1                          sha1
CKM_SHA_1_HMAC                     sha1Hmac
CKM_SHA_1_HMAC_GENERAL             sha1HmacGeneral
CKM_RIPEMD128                      ripemd128
CKM_RIPEMD128_HMAC                 ripemd128Hmac
CKM_RIPEMD128_HMAC_GENERAL         ripemd128HmacGeneral
CKM_RIPEMD160                      ripemd160
CKM_RIPEMD160_HMAC                 ripemd160Hmac
CKM_RIPEMD160_HMAC_GENERAL         ripemd160HmacGeneral
CKM_SHA256                         sha256
CKM_SHA256_HMAC                    sha256Hmac
CKM_SHA256_HMAC_GENERAL            sha256HmacGeneral
CKM_SHA224                         sha224
CKM_SHA224_HMAC                    sha224Hmac
CKM_SHA224_HMAC_GENERAL            sha224HmacGeneral
CKM_SHA384                         sha384
CKM_SHA384_HMAC                    sha384Hmac
CKM_SHA384_HMAC_GENERAL            sha384HmacGeneral
CKM_SHA512                         sha512
CKM_SHA512_HMAC                    sha512Hmac
CKM_SHA512_HMAC_GENERAL            sha512HmacGeneral
CKM_SECURID_KEY_GEN                securidKeyGen
CKM_SECURID                        securid
CKM_HOTP_KEY_GEN                   hotpKeyGen
CKM_HOTP                           hotp
CKM_ACTI                           acti
CKM_ACTI_KEY_GEN                   actiKeyGen
CKM_CAST_KEY_GEN                   castKeyGen
CKM_CAST_ECB                       castEcb
CKM_CAST_CBC                       castCbc
CKM_CAST_MAC                       castMac
CKM_CAST_MAC_GENERAL               castMacGeneral
CKM_CAST_CBC_PAD                   castCbcPad
CKM_CAST3_KEY_GEN                  cast3KeyGen
CKM_CAST3_ECB                      cast3Ecb
CKM_CAST3_CBC                      cast3Cbc
CKM_CAST3_MAC                      cast3Mac
CKM_CAST3_MAC_GENERAL              cast3MacGeneral
CKM_CAST3_CBC_PAD                  cast3CbcPad
CKM_CAST5_KEY_GEN                  cast128KeyGen
CKM_CAST128_KEY_GEN                cast128KeyGen
CKM_CAST5_ECB                      cast128Ecb
CKM_CAST128_ECB                    cast128Ecb
CKM_CAST5_CBC                      cast128Cbc
CKM_CAST128_CBC                    cast128Cbc
CKM_CAST5_MAC                      cast128Mac
CKM_CAST128_MAC                    cast128Mac
CKM_CAST5_MAC_GENERAL              cast128MacGeneral
CKM_CAST128_MAC_GENERAL            cast128MacGeneral
CKM_CAST5_CBC_PAD                  cast128CbcPad
CKM_CAST128_CBC_PAD                cast128CbcPad
CKM_RC5_KEY_GEN                    rc5KeyGen
CKM_RC5_ECB                        rc5Ecb
CKM_RC5_CBC                        rc5Cbc
CKM_RC5_MAC                        rc5Mac
CKM_RC5_MAC_GENERAL                rc5MacGeneral
CKM_RC5_CBC_PAD                    rc5CbcPad
CKM_IDEA_KEY_GEN                   ideaKeyGen
CKM_IDEA_ECB                       ideaEcb
CKM_IDEA_CBC                       ideaCbc
CKM_IDEA_MAC                       ideaMac
CKM_IDEA_MAC_GENERAL               ideaMacGeneral
CKM_IDEA_CBC_PAD                   ideaCbcPad
CKM_GENERIC_SECRET_KEY_GEN         genericSecretKeyGen
CKM_CONCATENATE_BASE_AND_KEY       concatenateBaseAndKey
CKM_CONCATENATE_BASE_AND_DATA      concatenateBaseAndData
CKM_CONCATENATE_DATA_AND_BASE      concatenateDataAndBase
CKM_XOR_BASE_AND_DATA              xorBaseAndData
CKM_EXTRACT_KEY_FROM_KEY           extractKeyFromKey
CKM_SSL3_PRE_MASTER_KEY_GEN        ssl3PreMasterKeyGen
CKM_SSL3_MASTER_KEY_DERIVE         ssl3MasterKeyDerive
CKM_SSL3_KEY_AND_MAC_DERIVE        ssl3KeyAndMacDerive
CKM_SSL3_MASTER_KEY_DERIVE_DH      ssl3MasterKeyDeriveDh
CKM_TLS_PRE_MASTER_KEY_GEN         tlsPreMasterKeyGen
CKM_TLS_MASTER_KEY_DERIVE          tlsMasterKeyDerive
CKM_TLS_KEY_AND_MAC_DERIVE         tlsKeyAndMacDerive
CKM_TLS_MASTER_KEY_DERIVE_DH       tlsMasterKeyDeriveDh
CKM_TLS_PRF                        tlsPrf
CKM_SSL3_MD5_MAC                   ssl3Md5Mac
CKM_SSL3_SHA1_MAC                  ssl3Sha1Mac
CKM_MD5_KEY_DERIVATION             md5KeyDerivation
CKM_MD2_KEY_DERIVATION             md2KeyDerivation
CKM_SHA1_KEY_DERIVATION            sha1KeyDerivation
CKM_SHA256_KEY_DERIVATION          sha256KeyDerivation
CKM_SHA384_KEY_DERIVATION          sha384KeyDerivation
CKM_SHA512_KEY_DERIVATION          sha512KeyDerivation
CKM_SHA224_KEY_DERIVATION          sha224KeyDerivation
CKM_PBE_MD2_DES_CBC                pbeMd2DesCbc
CKM_PBE_MD5_DES_CBC                pbeMd5DesCbc
CKM_PBE_MD5_CAST_CBC               pbeMd5CastCbc
CKM_PBE_MD5_CAST3_CBC              pbeMd5Cast3Cbc
CKM_PBE_MD5_CAST5_CBC              pbeMd5Cast5Cbc
CKM_PBE_MD5_CAST128_CBC            pbeMd5Cast128Cbc
CKM_PBE_SHA1_CAST5_CBC             pbeSha1Cast5Cbc
CKM_PBE_SHA1_CAST128_CBC           pbeSha1Cast128Cbc
CKM_PBE_SHA1_RC4_128               pbeSha1Rc4128
CKM_PBE_SHA1_RC4_40                pbeSha1Rc440
CKM_PBE_SHA1_DES3_EDE_CBC          pbeSha1Des3EdeCbc
CKM_PBE_SHA1_DES2_EDE_CBC          pbeSha1Des2EdeCbc
CKM_PBE_SHA1_RC2_128_CBC           pbeSha1Rc2128Cbc
CKM_PBE_SHA1_RC2_40_CBC            pbeSha1Rc240Cbc
CKM_PKCS5_PBKD2                    pkcs5Pbkd2
CKM_PBA_SHA1_WITH_SHA1_HMAC        pbaSha1WithSha1Hmac
CKM_WTLS_PRE_MASTER_KEY_GEN        wtlsPreMasterKeyGen
CKM_WTLS_MASTER_KEY_DERIVE         wtlsMasterKeyDerive
CKM_WTLS_MASTER_KEY_DERIVE_DH_ECC  wtlsMasterKeyDeriveDhEcc
CKM_WTLS_PRF                       wtlsPrf
CKM_WTLS_SERVER_KEY_AND_MAC_DERIVE wtlsServerKeyAndMacDerive
CKM_WTLS_CLIENT_KEY_AND_MAC_DERIVE wtlsClientKeyAndMacDerive
CKM_KEY_WRAP_LYNKS                 keyWrapLynks
CKM_KEY_WRAP_SET_OAEP              keyWrapSetOaep
CKM_CMS_SIG                        cmsSig
CKM_KIP_DERIVE                     kipDerive
CKM_KIP_WRAP                       kipWrap
CKM_KIP_MAC                        kipMac
CKM_CAMELLIA_KEY_GEN               camelliaKeyGen
CKM_CAMELLIA_ECB                   camelliaEcb
CKM_CAMELLIA_CBC                   camelliaCbc
CKM_CAMELLIA_MAC                   camelliaMac
CKM_CAMELLIA_MAC_GENERAL           camelliaMacGeneral
CKM_CAMELLIA_CBC_PAD               camelliaCbcPad
CKM_CAMELLIA_ECB_ENCRYPT_DATA      camelliaEcbEncryptData
CKM_CAMELLIA_CBC_ENCRYPT_DATA      camelliaCbcEncryptData
CKM_CAMELLIA_CTR                   camelliaCtr
CKM_ARIA_KEY_GEN                   ariaKeyGen
CKM_ARIA_ECB                       ariaEcb
CKM_ARIA_CBC                       ariaCbc
CKM_ARIA_MAC                       ariaMac
CKM_ARIA_MAC_GENERAL               ariaMacGeneral
CKM_ARIA_CBC_PAD                   ariaCbcPad
CKM_ARIA_ECB_ENCRYPT_DATA          ariaEcbEncryptData
CKM_ARIA_CBC_ENCRYPT_DATA          ariaCbcEncryptData
CKM_SEED_KEY_GEN                   seedKeyGen
CKM_SEED_ECB                       seedEcb
CKM_SEED_CBC                       seedCbc
CKM_SEED_MAC                       seedMac
CKM_SEED_MAC_GENERAL               seedMacGeneral
CKM_SEED_CBC_PAD                   seedCbcPad
CKM_SEED_ECB_ENCRYPT_DATA          seedEcbEncryptData
CKM_SEED_CBC_ENCRYPT_DATA          seedCbcEncryptData
CKM_SKIPJACK_KEY_GEN               skipjackKeyGen
CKM_SKIPJACK_ECB64                 skipjackEcb64
CKM_SKIPJACK_CBC64                 skipjackCbc64
CKM_SKIPJACK_OFB64                 skipjackOfb64
CKM_SKIPJACK_CFB64                 skipjackCfb64
CKM_SKIPJACK_CFB32                 skipjackCfb32
CKM_SKIPJACK_CFB16                 skipjackCfb16
CKM_SKIPJACK_CFB8                  skipjackCfb8
CKM_SKIPJACK_WRAP                  skipjackWrap
CKM_SKIPJACK_PRIVATE_WRAP          skipjackPrivateWrap
CKM_SKIPJACK_RELAYX                skipjackRelayx
CKM_KEA_KEY_PAIR_GEN               keaKeyPairGen
CKM_KEA_KEY_DERIVE                 keaKeyDerive
CKM_FORTEZZA_TIMESTAMP             fortezzaTimestamp
CKM_BATON_KEY_GEN                  batonKeyGen
CKM_BATON_ECB128                   batonEcb128
CKM_BATON_ECB96                    batonEcb96
CKM_BATON_CBC128                   batonCbc128
CKM_BATON_COUNTER                  batonCounter
CKM_BATON_SHUFFLE                  batonShuffle
CKM_BATON_WRAP                     batonWrap
CKM_ECDSA_KEY_PAIR_GEN             ecKeyPairGen
CKM_EC_KEY_PAIR_GEN                ecKeyPairGen
CKM_ECDSA                          ecdsa
CKM_ECDSA_SHA1                     ecdsaSha1
CKM_ECDSA_SHA224                   ecdsaSha224
CKM_ECDSA_SHA256                   ecdsaSha256
CKM_ECDSA_SHA384                   ecdsaSha384
CKM_ECDSA_SHA512                   ecdsaSha512
CKM_ECDH1_DERIVE                   ecdh1Derive
CKM_ECDH1_COFACTOR_DERIVE          ecdh1CofactorDerive
CKM_ECMQV_DERIVE                   ecmqvDerive
CKM_JUNIPER_KEY_GEN                juniperKeyGen
CKM_JUNIPER_ECB128                 juniperEcb128
CKM_JUNIPER_CBC128                 juniperCbc128
CKM_JUNIPER_COUNTER                juniperCounter
CKM_JUNIPER_SHUFFLE                juniperShuffle
CKM_JUNIPER_WRAP                   juniperWrap
CKM_FASTHASH                       fasthash
CKM_AES_KEY_GEN                    aesKeyGen
CKM_AES_ECB                        aesEcb
CKM_AES_CBC                        aesCbc
CKM_AES_MAC                        aesMac
CKM_AES_MAC_GENERAL                aesMacGeneral
CKM_AES_CBC_PAD                    aesCbcPad
CKM_AES_CTR                        aesCtr
CKM_AES_CTS                        aesCts
CKM_AES_CMAC                       aesCmac
CKM_AES_CMAC_GENERAL               aesCmacGeneral
CKM_BLOWFISH_KEY_GEN               blowfishKeyGen
CKM_BLOWFISH_CBC                   blowfishCbc
CKM_TWOFISH_KEY_GEN                twofishKeyGen
CKM_TWOFISH_CBC                    twofishCbc
CKM_AES_GCM                        aesGcm
CKM_AES_CCM                        aesCcm
CKM_AES_KEY_WRAP                   aesKeyWrap
CKM_AES_KEY_WRAP_PAD               aesKeyWrapPad
CKM_BLOWFISH_CBC_PAD               blowfishCbcPad
CKM_TWOFISH_CBC_PAD                twofishCbcPad
CKM_DES_ECB_ENCRYPT_DATA           desEcbEncryptData
CKM_DES_CBC_ENCRYPT_DATA           desCbcEncryptData
CKM_DES3_ECB_ENCRYPT_DATA          des3EcbEncryptData
CKM_DES3_CBC_ENCRYPT_DATA          des3CbcEncryptData
CKM_AES_ECB_ENCRYPT_DATA           aesEcbEncryptData
CKM_AES_CBC_ENCRYPT_DATA           aesCbcEncryptData
CKM_GOSTR3410_KEY_PAIR_GEN         gostr3410KeyPairGen
CKM_GOSTR3410                      gostr3410
CKM_GOSTR3410_WITH_GOSTR3411       gostr3410WithGostr3411
CKM_GOSTR3410_KEY_WRAP             gostr3410KeyWrap
CKM_GOSTR3410_DERIVE               gostr3410Derive
CKM_GOSTR3411                      gostr3411
CKM_GOSTR3411_HMAC                 gostr3411Hmac
CKM_GOST28147_KEY_GEN              gost28147KeyGen
CKM_GOST28147_ECB                  gost28147Ecb
CKM_GOST28147                      gost28147
CKM_GOST28147_MAC                  gost28147Mac
CKM_GOST28147_KEY_WRAP             gost28147KeyWrap
CKM_DSA_PARAMETER_GEN              dsaParameterGen
CKM_DH_PKCS_PARAMETER_GEN          dhPkcsParameterGen
CKM_X9_42_DH_PARAMETER_GEN         x942DhParameterGen
CKM_AES_OFB                        aesOfb
CKM_AES_CFB64                      aesCfb64
CKM_AES_CFB8                       aesCfb8
CKM_AES_CFB128                     aesCfb128
CKM_RSA_PKCS_TPM_1_1               rsaPkcsTpm11
CKM_RSA_PKCS_OAEP_TPM_1_1          rsaPkcsOaepTpm11
================================== =========================

.. _object_classes_1:

Object classes
~~~~~~~~~~~~~~

-  `ipk11X509Certificate <#Storage_objects>`__

+--------------------------------+------------------------------------+
| Attribute                      | Value                              |
+================================+====================================+
| CKA_CLASS                      | CKO_CERTIFICATE                    |
+--------------------------------+------------------------------------+
| CKA_TOKEN                      | CK_TRUE                            |
+--------------------------------+------------------------------------+
| CKA_PRIVATE                    | `ipk11Private <#ipk11Private>`__   |
+--------------------------------+------------------------------------+
| CKA_MODIFIABLE                 | `ipk                               |
|                                | 11Modifiable <#ipk11Modifiable>`__ |
+--------------------------------+------------------------------------+
| CKA_LABEL                      | `ipk11Label <#ipk11Label>`__       |
+--------------------------------+------------------------------------+
| CKA_COPYABLE                   | `ipk11Copyable <#ipk11Copyable>`__ |
+--------------------------------+------------------------------------+
| CKA_DESTROYABLE                | `ipk11                             |
|                                | Destroyable <#ipk11Destroyable>`__ |
+--------------------------------+------------------------------------+
| CKA_CERTIFICATE_TYPE           | CKC_X_509                          |
+--------------------------------+------------------------------------+
| CKA_TRUSTED                    | `ipk11Trusted <#ipk11Trusted>`__   |
+--------------------------------+------------------------------------+
| CKA_CHECK_VALUE                | `ipk                               |
|                                | 11CheckValue <#ipk11CheckValue>`__ |
+--------------------------------+------------------------------------+
| CKA_START_DATE                 | `i                                 |
|                                | pk11StartDate <#ipk11StartDate>`__ |
+--------------------------------+------------------------------------+
| CKA_END_DATE                   | `ipk11EndDate <#ipk11EndDate>`__   |
+--------------------------------+------------------------------------+
| CKA_PUBLIC_KEY_INFO            | `ipk11Publ                         |
|                                | icKeyInfo <#ipk11PublicKeyInfo>`__ |
+--------------------------------+------------------------------------+
| CKA_X_DISTRUSTED               | `ipk                               |
|                                | 11Distrusted <#ipk11Distrusted>`__ |
+--------------------------------+------------------------------------+
| CKA_SUBJECT                    | `ipk11Subject <#ipk11Subject>`__   |
+--------------------------------+------------------------------------+
| CKA_ISSUER                     | `ipk11Issuer <#ipk11Issuer>`__     |
+--------------------------------+------------------------------------+
| CKA_SERIAL_NUMBER              | `ipk11Se                           |
|                                | rialNumber <#ipk11SerialNumber>`__ |
+--------------------------------+------------------------------------+
| CKA_HASH_OF_SUBJECT_PUBLIC_KEY | `ipk11Subjec                       |
|                                | tKeyHash <#ipk11SubjectKeyHash>`__ |
+--------------------------------+------------------------------------+
| CKA_HASH_OF_ISSUER_PUBLIC_KEY  | `ipk11Issu                         |
|                                | erKeyHash <#ipk11IssuerKeyHash>`__ |
+--------------------------------+------------------------------------+
| CKA_JAVA_MIDP_SECURITY_DOMAIN  | `ipk11Securi                       |
|                                | tyDomain <#ipk11SecurityDomain>`__ |
+--------------------------------+------------------------------------+
| CKA_NAME_HASH_ALGORITHM        | `ipk11Subjec                       |
|                                | tKeyHash <#ipk11SubjectKeyHash>`__ |
|                                | and                                |
|                                | `ipk11Issu                         |
|                                | erKeyHash <#ipk11IssuerKeyHash>`__ |
+--------------------------------+------------------------------------+

-  `ipk11PublicKey <#Storage_objects>`__

+------------------------+--------------------------------------------+
| Attribute              | Value                                      |
+========================+============================================+
| CKA_CLASS              | CKO_PUBLIC_KEY                             |
+------------------------+--------------------------------------------+
| CKA_TOKEN              | CK_TRUE                                    |
+------------------------+--------------------------------------------+
| CKA_PRIVATE            | `ipk11Private <#ipk11Private>`__           |
+------------------------+--------------------------------------------+
| CKA_MODIFIABLE         | `ipk11Modifiable <#ipk11Modifiable>`__     |
+------------------------+--------------------------------------------+
| CKA_LABEL              | `ipk11Label <#ipk11Label>`__               |
+------------------------+--------------------------------------------+
| CKA_COPYABLE           | `ipk11Copyable <#ipk11Copyable>`__         |
+------------------------+--------------------------------------------+
| CKA_DESTROYABLE        | `ipk11Destroyable <#ipk11Destroyable>`__   |
+------------------------+--------------------------------------------+
| CKA_KEY_TYPE           | `ipk11KeyType <#ipk11KeyType>`__           |
+------------------------+--------------------------------------------+
| CKA_ID                 | `ipk11Id <#ipk11Id>`__                     |
+------------------------+--------------------------------------------+
| CKA_START_DATE         | `ipk11StartDate <#ipk11StartDate>`__       |
+------------------------+--------------------------------------------+
| CKA_END_DATE           | `ipk11EndDate <#ipk11EndDate>`__           |
+------------------------+--------------------------------------------+
| CKA_DERIVE             | `ipk11Derive <#ipk11Derive>`__             |
+------------------------+--------------------------------------------+
| CKA_LOCAL              | `ipk11Local <#ipk11Local>`__               |
+------------------------+--------------------------------------------+
| CKA_KEY_GEN_MECHANISM  | `ipk11                                     |
|                        | KeyGenMechanism <#ipk11KeyGenMechanism>`__ |
+------------------------+--------------------------------------------+
| CKA_ALLOWED_MECHANISMS | `ipk11Allo                                 |
|                        | wedMechanisms <#ipk11AllowedMechanisms>`__ |
+------------------------+--------------------------------------------+
| CKA_SUBJECT            | `ipk11Subject <#ipk11Subject>`__           |
+------------------------+--------------------------------------------+
| CKA_ENCRYPT            | `ipk11Encrypt <#ipk11Encrypt>`__           |
+------------------------+--------------------------------------------+
| CKA_VERIFY             | `ipk11Verify <#ipk11Verify>`__             |
+------------------------+--------------------------------------------+
| CKA_VERIFY_RECOVER     | `i                                         |
|                        | pk11VerifyRecover <#ipk11VerifyRecover>`__ |
+------------------------+--------------------------------------------+
| CKA_WRAP               | `ipk11Wrap <#ipk11Wrap>`__                 |
+------------------------+--------------------------------------------+
| CKA_TRUSTED            | `ipk11Trusted <#ipk11Trusted>`__           |
+------------------------+--------------------------------------------+
| CKA_WRAP_TEMPLATE      | `ipk11WrapTemplate <#ipk11WrapTemplate>`__ |
+------------------------+--------------------------------------------+
| CKA_PUBLIC_KEY_INFO    | `i                                         |
|                        | pk11PublicKeyInfo <#ipk11PublicKeyInfo>`__ |
+------------------------+--------------------------------------------+
| CKA_X_DISTRUSTED       | `ipk11Distrusted <#ipk11Distrusted>`__     |
+------------------------+--------------------------------------------+

-  `ipk11PrivateKey <#Storage_objects>`__

+-------------------------+-------------------------------------------+
| Attribute               | Value                                     |
+=========================+===========================================+
| CKA_CLASS               | CKO_PRIVATE_KEY                           |
+-------------------------+-------------------------------------------+
| CKA_TOKEN               | CK_TRUE                                   |
+-------------------------+-------------------------------------------+
| CKA_PRIVATE             | `ipk11Private <#ipk11Private>`__          |
+-------------------------+-------------------------------------------+
| CKA_MODIFIABLE          | `ipk11Modifiable <#ipk11Modifiable>`__    |
+-------------------------+-------------------------------------------+
| CKA_LABEL               | `ipk11Label <#ipk11Label>`__              |
+-------------------------+-------------------------------------------+
| CKA_COPYABLE            | `ipk11Copyable <#ipk11Copyable>`__        |
+-------------------------+-------------------------------------------+
| CKA_DESTROYABLE         | `ipk11Destroyable <#ipk11Destroyable>`__  |
+-------------------------+-------------------------------------------+
| CKA_KEY_TYPE            | `ipk11KeyType <#ipk11KeyType>`__          |
+-------------------------+-------------------------------------------+
| CKA_ID                  | `ipk11Id <#ipk11Id>`__                    |
+-------------------------+-------------------------------------------+
| CKA_START_DATE          | `ipk11StartDate <#ipk11StartDate>`__      |
+-------------------------+-------------------------------------------+
| CKA_END_DATE            | `ipk11EndDate <#ipk11EndDate>`__          |
+-------------------------+-------------------------------------------+
| CKA_DERIVE              | `ipk11Derive <#ipk11Derive>`__            |
+-------------------------+-------------------------------------------+
| CKA_LOCAL               | `ipk11Local <#ipk11Local>`__              |
+-------------------------+-------------------------------------------+
| CKA_KEY_GEN_MECHANISM   | `ipk11K                                   |
|                         | eyGenMechanism <#ipk11KeyGenMechanism>`__ |
+-------------------------+-------------------------------------------+
| CKA_ALLOWED_MECHANISMS  | `ipk11Allow                               |
|                         | edMechanisms <#ipk11AllowedMechanisms>`__ |
+-------------------------+-------------------------------------------+
| CKA_SUBJECT             | `ipk11Subject <#ipk11Subject>`__          |
+-------------------------+-------------------------------------------+
| CKA_SENSITIVE           | `ipk11Sensitive <#ipk11Sensitive>`__      |
+-------------------------+-------------------------------------------+
| CKA_DECRYPT             | `ipk11Decrypt <#ipk11Decrypt>`__          |
+-------------------------+-------------------------------------------+
| CKA_SIGN                | `ipk11Sign <#ipk11Sign>`__                |
+-------------------------+-------------------------------------------+
| CKA_SIGN_RECOVER        | `ipk11SignRecover <#ipk11SignRecover>`__  |
+-------------------------+-------------------------------------------+
| CKA_UNWRAP              | `ipk11Unwrap <#ipk11Unwrap>`__            |
+-------------------------+-------------------------------------------+
| CKA_EXTRACTABLE         | `ipk11Extractable <#ipk11Extractable>`__  |
+-------------------------+-------------------------------------------+
| CKA_ALWAYS_SENSITIVE    | `ipk11A                                   |
|                         | lwaysSensitive <#ipk11AlwaysSensitive>`__ |
+-------------------------+-------------------------------------------+
| CKA_NEVER_EXTRACTABLE   | `ipk11Nev                                 |
|                         | erExtractable <#ipk11NeverExtractable>`__ |
+-------------------------+-------------------------------------------+
| CKA_WRAP_WITH_TRUSTED   | `ipk11W                                   |
|                         | rapWithTrusted <#ipk11WrapWithTrusted>`__ |
+-------------------------+-------------------------------------------+
| CKA_UNWRAP_TEMPLATE     | `ipk1                                     |
|                         | 1UnwrapTemplate <#ipk11UnwrapTemplate>`__ |
+-------------------------+-------------------------------------------+
| CKA_ALWAYS_AUTHENTICATE | `ipk11AlwaysA                             |
|                         | uthenticate <#ipk11AlwaysAuthenticate>`__ |
+-------------------------+-------------------------------------------+
| CKA_PUBLIC_KEY_INFO     | `ip                                       |
|                         | k11PublicKeyInfo <#ipk11PublicKeyInfo>`__ |
+-------------------------+-------------------------------------------+

-  `ipk11SecretKey <#Storage_objects>`__

+------------------------+--------------------------------------------+
| Attribute              | Value                                      |
+========================+============================================+
| CKA_CLASS              | CKO_SECRET_KEY                             |
+------------------------+--------------------------------------------+
| CKA_TOKEN              | CK_TRUE                                    |
+------------------------+--------------------------------------------+
| CKA_PRIVATE            | `ipk11Private <#ipk11Private>`__           |
+------------------------+--------------------------------------------+
| CKA_MODIFIABLE         | `ipk11Modifiable <#ipk11Modifiable>`__     |
+------------------------+--------------------------------------------+
| CKA_LABEL              | `ipk11Label <#ipk11Label>`__               |
+------------------------+--------------------------------------------+
| CKA_COPYABLE           | `ipk11Copyable <#ipk11Copyable>`__         |
+------------------------+--------------------------------------------+
| CKA_DESTROYABLE        | `ipk11Destroyable <#ipk11Destroyable>`__   |
+------------------------+--------------------------------------------+
| CKA_KEY_TYPE           | `ipk11KeyType <#ipk11KeyType>`__           |
+------------------------+--------------------------------------------+
| CKA_ID                 | `ipk11Id <#ipk11Id>`__                     |
+------------------------+--------------------------------------------+
| CKA_START_DATE         | `ipk11StartDate <#ipk11StartDate>`__       |
+------------------------+--------------------------------------------+
| CKA_END_DATE           | `ipk11EndDate <#ipk11EndDate>`__           |
+------------------------+--------------------------------------------+
| CKA_DERIVE             | `ipk11Derive <#ipk11Derive>`__             |
+------------------------+--------------------------------------------+
| CKA_LOCAL              | `ipk11Local <#ipk11Local>`__               |
+------------------------+--------------------------------------------+
| CKA_KEY_GEN_MECHANISM  | `ipk11                                     |
|                        | KeyGenMechanism <#ipk11KeyGenMechanism>`__ |
+------------------------+--------------------------------------------+
| CKA_ALLOWED_MECHANISMS | `ipk11Allo                                 |
|                        | wedMechanisms <#ipk11AllowedMechanisms>`__ |
+------------------------+--------------------------------------------+
| CKA_SENSITIVE          | `ipk11Sensitive <#ipk11Sensitive>`__       |
+------------------------+--------------------------------------------+
| CKA_ENCRYPT            | `ipk11Encrypt <#ipk11Encrypt>`__           |
+------------------------+--------------------------------------------+
| CKA_DECRYPT            | `ipk11Decrypt <#ipk11Decrypt>`__           |
+------------------------+--------------------------------------------+
| CKA_SIGN               | `ipk11Sign <#ipk11Sign>`__                 |
+------------------------+--------------------------------------------+
| CKA_VERIFY             | `ipk11Verify <#ipk11Verify>`__             |
+------------------------+--------------------------------------------+
| CKA_WRAP               | `ipk11Wrap <#ipk11Wrap>`__                 |
+------------------------+--------------------------------------------+
| CKA_UNWRAP             | `ipk11Unwrap <#ipk11Unwrap>`__             |
+------------------------+--------------------------------------------+
| CKA_EXTRACTABLE        | `ipk11Extractable <#ipk11Extractable>`__   |
+------------------------+--------------------------------------------+
| CKA_ALWAYS_SENSITIVE   | `ipk11                                     |
|                        | AlwaysSensitive <#ipk11AlwaysSensitive>`__ |
+------------------------+--------------------------------------------+
| CKA_NEVER_EXTRACTABLE  | `ipk11Ne                                   |
|                        | verExtractable <#ipk11NeverExtractable>`__ |
+------------------------+--------------------------------------------+
| CKA_CHECK_VALUE        | `ipk11CheckValue <#ipk11CheckValue>`__     |
+------------------------+--------------------------------------------+
| CKA_WRAP_WITH_TRUSTED  | `ipk11                                     |
|                        | WrapWithTrusted <#ipk11WrapWithTrusted>`__ |
+------------------------+--------------------------------------------+
| CKA_TRUSTED            | `ipk11Trusted <#ipk11Trusted>`__           |
+------------------------+--------------------------------------------+
| CKA_WRAP_TEMPLATE      | `ipk11WrapTemplate <#ipk11WrapTemplate>`__ |
+------------------------+--------------------------------------------+
| CKA_UNWRAP_TEMPLATE    | `ipk                                       |
|                        | 11UnwrapTemplate <#ipk11UnwrapTemplate>`__ |
+------------------------+--------------------------------------------+

-  `ipk11DomainParameters <#Storage_objects>`__

=============== ========================================
Attribute       Value
=============== ========================================
CKA_CLASS       CKO_DOMAIN_PARAMETERS
CKA_TOKEN       CK_TRUE
CKA_PRIVATE     `ipk11Private <#ipk11Private>`__
CKA_MODIFIABLE  `ipk11Modifiable <#ipk11Modifiable>`__
CKA_LABEL       `ipk11Label <#ipk11Label>`__
CKA_COPYABLE    `ipk11Copyable <#ipk11Copyable>`__
CKA_DESTROYABLE `ipk11Destroyable <#ipk11Destroyable>`__
CKA_KEY_TYPE    `ipk11KeyType <#ipk11KeyType>`__
CKA_LOCAL       `ipk11Local <#ipk11Local>`__
=============== ========================================

-  `ipaPublicKeyObject <#Encoded_key_data_2>`__

==================== ===============================================
Attribute            Value
==================== ===============================================
CKA_CLASS            CKO_PUBLIC_KEY
CKA_TOKEN            CK_TRUE
CKA_PUBLIC_KEY_INFO  `ipaPublicKey <#ipaPublicKey>`__
CKA_MODULUS          extracted from `ipaPublicKey <#ipaPublicKey>`__
CKA_MODULUS_BITS     extracted from `ipaPublicKey <#ipaPublicKey>`__
CKA_PUBLIC_EXPONENT  extracted from `ipaPublicKey <#ipaPublicKey>`__
CKA_PRIME            extracted from `ipaPublicKey <#ipaPublicKey>`__
CKA_SUBPRIME         extracted from `ipaPublicKey <#ipaPublicKey>`__
CKA_BASE             extracted from `ipaPublicKey <#ipaPublicKey>`__
CKA_VALUE            extracted from `ipaPublicKey <#ipaPublicKey>`__
CKA_VALUE_BITS       extracted from `ipaPublicKey <#ipaPublicKey>`__
CKA_EC_PARAMS        extracted from `ipaPublicKey <#ipaPublicKey>`__
CKA_EC_POINT         extracted from `ipaPublicKey <#ipaPublicKey>`__
CKA_GOSTR3410_PARAMS extracted from `ipaPublicKey <#ipaPublicKey>`__
CKA_GOSTR3411_PARAMS extracted from `ipaPublicKey <#ipaPublicKey>`__
CKA_GOST28147_PARAMS extracted from `ipaPublicKey <#ipaPublicKey>`__
==================== ===============================================

-  `ipaPrivateKeyObject <#Encoded_key_data_2>`__

==================== =================================================
Attribute            Value
==================== =================================================
CKA_CLASS            CKO_PRIVATE_KEY
CKA_TOKEN            CK_TRUE
CKA_PUBLIC_KEY_INFO  derived from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_MODULUS          extracted from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_PUBLIC_EXPONENT  extracted from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_PRIVATE_EXPONENT extracted from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_PRIME_1          extracted from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_PRIME_2          extracted from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_EXPONENT_1       extracted from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_EXPONENT_2       extracted from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_COEFFICIENT      extracted from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_PRIME            extracted from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_SUBPRIME         extracted from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_BASE             extracted from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_VALUE            extracted from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_VALUE_BITS       extracted from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_EC_PARAMS        extracted from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_GOSTR3410_PARAMS extracted from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_GOSTR3411_PARAMS extracted from `ipaPrivateKey <#ipaPrivateKey>`__
CKA_GOST28147_PARAMS extracted from `ipaPrivateKey <#ipaPrivateKey>`__
==================== =================================================

-  `ipaSecretKeyObject <#Encoded_key_data_2>`__

============= =============================================
Attribute     Value
============= =============================================
CKA_CLASS     CKO_SECRET_KEY
CKA_TOKEN     CK_TRUE
CKA_VALUE     derived from `ipaSecretKey <#ipaSecretKey>`__
CKA_VALUE_LEN derived from `ipaSecretKey <#ipaSecretKey>`__
============= =============================================

-  ipaCertificate (`V4/CA certificate
   renewal <V4/CA_certificate_renewal>`__)

========= ===============
Attribute Value
========= ===============
CKA_CLASS CKO_CERTIFICATE
CKA_TOKEN CK_TRUE
CKA_LABEL cn
========= ===============

-  ipaKeyPolicy (`V4/CA certificate
   renewal <V4/CA_certificate_renewal>`__)

================ ========================
Attribute        Value
================ ========================
CKA_TOKEN        CK_TRUE
CKA_TRUSTED      derived from ipaKeyTrust
CKA_X_DISTRUSTED derived from ipaKeyTrust
================ ========================

============= ===========================
Attribute     Value
============= ===========================
CKA_CLASS     CKO_X_CERTIFICATE_EXTENSION
CKA_TOKEN     CK_TRUE
CKA_OBJECT_ID DER-encoding of 2.5.29.15
CKA_VALUE     derived from ipaKeyUsage
============= ===========================

============= ===========================
Attribute     Value
============= ===========================
CKA_CLASS     CKO_X_CERTIFICATE_EXTENSION
CKA_TOKEN     CK_TRUE
CKA_OBJECT_ID DER-encoding of 2.5.29.37
CKA_VALUE     derived from ipaExtKeyUsage
============= ===========================

-  pkiUser (`RFC 4523 <http://tools.ietf.org/html/rfc4523>`__)

======================== ==================================
Attribute                Value
======================== ==================================
CKA_CLASS                CKO_CERTIFICATE
CKA_TOKEN                CK_TRUE
CKA_CERTIFICATE_TYPE     CKC_X_509
CKA_CERTIFICATE_CATEGORY CK_CERTIFICATE_CATEGORY_TOKEN_USER
CKA_VALUE                userCertificate
======================== ==================================

-  pkiCA (`RFC 4523 <http://tools.ietf.org/html/rfc4523>`__)

======================== =================================
Attribute                Value
======================== =================================
CKA_CLASS                CKO_CERTIFICATE
CKA_TOKEN                CK_TRUE
CKA_CERTIFICATE_TYPE     CKC_X_509
CKA_CERTIFICATE_CATEGORY CK_CERTIFICATE_CATEGORY_AUTHORITY
CKA_VALUE                cACertificate
======================== =================================

.. _default_values_used_by_freeipa:

Default values used by FreeIPA
------------------------------

Some attributes have default values which do not need to be stored in
LDAP. Default values depend on LDAP object classes present in the
object.

ipk11publickey
~~~~~~~~~~~~~~

================== =====
Attribute          Value
================== =====
ipk11copyable      True
ipk11derive        False
ipk11encrypt       False
ipk11local         True
ipk11modifiable    True
ipk11private       True
ipk11trusted       False
ipk11verify        True
ipk11verifyrecover True
ipk11wrap          False
================== =====

ipk11privatekey
~~~~~~~~~~~~~~~

======================= =====
Attribute               Value
======================= =====
ipk11alwaysauthenticate False
ipk11alwayssensitive    True
ipk11copyable           True
ipk11decrypt            False
ipk11derive             False
ipk11extractable        True
ipk11local              True
ipk11modifiable         True
ipk11neverextractable   False
ipk11private            True
ipk11sensitive          True
ipk11sign               True
ipk11signrecover        True
ipk11unwrap             False
ipk11wrapwithtrusted    False
======================= =====

ipk11secretkey
~~~~~~~~~~~~~~

======================= =====
Attribute               Value
======================= =====
ipk11alwaysauthenticate False
ipk11alwayssensitive    True
ipk11copyable           True
ipk11decrypt            False
ipk11derive             False
ipk11encrypt            False
ipk11extractable        True
ipk11local              True
ipk11modifiable         True
ipk11neverextractable   False
ipk11private            True
ipk11sensitive          True
ipk11sign               False
ipk11trusted            False
ipk11unwrap             True
ipk11verify             False
ipk11wrap               True
ipk11wrapwithtrusted    False
======================= =====
