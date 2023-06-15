Overview
--------

Allow automated and manual renewal of IPA CA certificate. Provide an
utility for manual renewal, including modification of chaining
(self-signed vs. signed by external CA). Store multiple CA certificates
in LDAP and distribute them to clients.

This page describes **phase 1** of the CA certificate management
feature, which consists of automated and manual CA certificate renewal,
CA certificate management utility and storage of multiple CA certificate
in LDAP. Visit `V4/CA certificate renewal
(2) <V4/CA_certificate_renewal_(2)>`__ for description of **phase 2**,
which consists of distribution of CA certificates to IPA clients.



Use Cases
---------

.. _automated_ca_certificate_renewal:

Automated CA certificate renewal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When the CA certificate is nearing its expiration time, it should be
automatically renewed. The renewed certificate will use the same keypair
and subject name as the old certificate.

This only works for self-signed CA certificates in CA-ful installs.

.. _manual_ca_certificate_renewal:

Manual CA certificate renewal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Allow admin to manually renew the CA certificate and possibly change its
chaining (self-signed → signed by external CA, signed by external CA →
self-signed, signed by external CA → signed by other external CA)
(**phase 1**) and have the renewed certificate automatically distributed
to all systems in the domain (**phase 2**). The renewed certificate will
use the same keypair and subject name as the old certificate.

This works for any CA certificate in CA-ful installs.

.. _manual_install_of_ca_certificate:

Manual install of CA certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In CA-less installs, CA certificate renewal is completely in charge of a
3rd party CA. Provide means of installing CA certificates obtained from
such a CA (**phase 1**) and automatically distribute them to all systems
in the domain (**phase 2**).

Design
------

.. _shared_certificate_store:

Shared certificate store
~~~~~~~~~~~~~~~~~~~~~~~~

CA certificates will be stored in entries under
``cn=certificates,cn=ipa,cn=etc,``\ *``suffix``*:

| ``dn: cn=EXAMPLE.COM IPA CA,cn=certificates,cn=ipa,cn=etc,``\ *``suffix``*
| ``objectClass: ipaCertificate``
| ``objectClass: pkiCA``
| ``objectClass: ipaKeyPolicy``
| ``cn: cn=EXAMPLE.COM IPA CA``
| ``ipaCertSubject: CN=Certificate Authority,O=EXAMPLE.COM``
| ``ipaCertIssuerSerial: CN=Certificate Authority,O=EXAMPLE.COM;1``
| ``ipaCertIssuerSerial: CN=Certificate Authority,O=EXAMPLE.COM;22``
| ``ipaPublicKey: ``\ *``DER-encoded public key``*
| ``ipaConfigString: ipaCA``
| ``ipaConfigString: compatCA``
| ``cACertificate;binary:: ``\ *``DER-encoded certificate 1``*
| ``cACertificate;binary:: ``\ *``DER-encoded certificate 2``*
| ``ipaKeyTrust: trusted``
| ``ipaKeyExtUsage: 1.3.6.1.5.5.7.3.1``
| ``ipaKeyExtUsage: 1.3.6.1.5.5.7.3.2``
| ``ipaKeyExtUsage: 1.3.6.1.5.5.7.3.3``
| ``ipaKeyExtUsage: 1.3.6.1.5.5.7.3.4``

Each entry represents a set of certificates using a single subject name
and public key and contains trust policy for the set of certificates
(based on `p11-kit trust
policy <http://p11-glue.freedesktop.org/doc/storing-trust-policy/index.html>`__).
The attributes are:

-  ``cn``

   Nickname for the certificates. The value must be unique in the store.

-  ``ipaCertSubject``

   Subject name of the certificates. The value must be unique in the
   store. Semicolons in the subject name are escaped as ``\3b``.

-  ``ipaCertIssuerSerial``

   Issuer name and serial number of each certificate in the set,
   separated by semicolon. Each value must be unique in the store.
   Semicolons in the issuer name are escaped as ``\3b``.

-  ```ipaPublicKey`` <V4/PKCS11_in_LDAP/Schema#Encoded_key_data>`__

   Public key of the certificates as DER-encoded SubjectPublicKeyInfo.

-  ``ipaConfigString``

   Configuration flags:

   -  ``ipaCA``

      This entry represents the IPA CA certificate.

   -  ``compatCA``

      When a new certificate is added to the set, update
      ``cn=CAcert,cn=ipa,cn=etc,``\ *``suffix``* with it.

-  ``cACertificate``

   Each of the certificates in the set in DER encoding.

-  ``ipaKeyTrust``

   Trust setting for the certificates:

   -  ``unknown`` (default)

      Trust is not explicitly given. If there is other source of trust
      information, it should be used instead.

   -  ``trusted``

      The certificates in the set are explicitly trusted.

   -  ``distrusted``

      The certificates in the set are explicitly distrusted.

-  ``ipaKeyUsage``

   Allowed key usages for the certificates as purpose bit names from the
   key usage certificate extension (see `5280, section
   4.2.1.3 <http://tools.ietf.org/html/rfc5280#section-4.2.1.3%7CRFC>`__).
   A value of ``none`` means no key usages are allowed. Default value is
   the value of the key usage extension from each certificate.

-  ``ipaKeyExtUsage``

   Allowed extended key usages as key purpose OIDs (see `5280, section
   4.2.1.12 <http://tools.ietf.org/html/rfc5280#section-4.2.1.12%7CRFC>`__).
   A value of ``1.3.6.1.4.1.3319.6.10.16`` means no extended key usages
   are allowed. Default value is the value of the extended key usage
   extension from each certificate.
   For trusted CA certificates, the value of this attribute is mapped to
   NSS / certutil trust flags as follows:

   -  ``1.3.6.1.5.5.7.3.1`` ⇒ ``C,,``
   -  ``1.3.6.1.5.5.7.3.2`` ⇒ ``T,,``
   -  ``1.3.6.1.5.5.7.3.3`` ⇒ ``,,C``
   -  ``1.3.6.1.5.5.7.3.4`` ⇒ ``,C,``

The entries will be readable by everyone and writable only by the
directory manager (for installers and management tools) and the server
host (for automatic renewal).

The new schema used for the entries is:

| ``attributeTypes: (2.16.840.1.113730.3.8.11.???``
| ``                 NAME 'ipaCertSubject'``
| ``                 DESC 'Subject name'``
| ``                 EQUALITY caseIgnoreMatch``
| ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| ``                 SINGLE-VALUE``
| ``                 X-ORIGIN 'IPA v4' )``
| ``attributeTypes: (2.16.840.1.113730.3.8.11.???``
| ``                 NAME 'ipaCertIssuerSerial'``
| ``                 DESC 'Issuer name and serial number'``
| ``                 EQUALITY caseIgnoreMatch``
| ``                 SUBSTR caseIgnoreSubstringsMatch``
| ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| ``                 X-ORIGIN 'IPA v4' )``
| ``attributeTypes: (2.16.840.1.113730.3.8.11.???``
| ``                 NAME 'ipaKeyTrust'``
| ``                 DESC 'Key trust (unknown, trusted, distrusted)'``
| ``                 EQUALITY caseIgnoreMatch``
| ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| ``                 X-ORIGIN 'IPA v4') ``
| ``attributeTypes: (2.16.840.1.113730.3.8.11.???``
| ``                 NAME 'ipaKeyUsage'``
| ``                 DESC 'Allowed key usage'``
| ``                 EQUALITY caseIgnoreMatch``
| ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| ``                 X-ORIGIN 'IPA v4') ``
| ``attributeTypes: (2.16.840.1.113730.3.8.11.???``
| ``                 NAME 'ipaKeyExtUsage'``
| ``                 DESC 'Allowed extended key usage'``
| ``                 EQUALITY objectIdentifierMatch``
| ``                 SYNTAX 1.3.6.1.4.1.1466.115.121.1.38``
| ``                 X-ORIGIN 'IPA v4')``
| ``objectClasses: (2.16.840.1.113730.3.8.12.???``
| ``                NAME 'ipaCertificate'``
| ``                SUP top STRUCTURAL``
| ``                MUST ( cn $ ipaCertSubject $ ipaCertIssuerSerial $ ipaPublicKey )``
| ``                MAY  ( ipaConfigString )``
| ``                X-ORIGIN 'IPA v4' )``
| ``objectClasses: (2.16.840.1.113730.3.8.12.???``
| ``                NAME 'ipaKeyPolicy'``
| ``                SUP top AUXILIARY``
| ``                MAY  ( ipaKeyTrust $ ipaKeyUsage $ ipaExtKeyUsage )``
| ``                X-ORIGIN 'IPA v4')``

.. _automatic_renewal_of_ipa_ca_certificate:

Automatic renewal of IPA CA certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The CA certificate managed by Dogtag will be tracked by certmonger. If
the certificate is self-signed, it will be automatically renewed. If the
certificate is signed by an external CA, the renewal attempt will fail
with an error, advising the administrator to renew the certificate
manually. The error is syslogged with ALERT severity.

.. _ca_certificate_management_utility:

CA certificate management utility
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There will be new utility to manage CA certificates,
``ipa-cacert-manage``. It will have several subcommands:

-  ``renew``\ *``options``*

   This command will be available only for CA-ful installs and will be
   used to renew the IPA CA certificate. The certificate can be renewed
   either as self-signed or signed by an external CA. By default, the
   chaining used for the old certificate is used for the new certificate
   as well. Renewing a CA certificate signed by an external CA is a 2
   step operation: in the first step, a CSR is exported to
   ``/var/lib/ipa/ca.csr``; in the second step, the signed certificate
   is installed.
   The available options are:

   -  ``--self-signed``

      Renew the CA certificate as self-signed.

   -  ``--external-ca``

      Renew the CA certificate as signed by an external CA, step 1:
      Export CSR to ``/var/lib/ipa/ca.csr``.

   -  ``--external-cert-file``\ *``file``*

      Renew the CA certificate as signed by an external CA, step 2:
      Install the new CA certificate.

   -  ``--password``\ *``password``*

      Directory manager password. Required for external CA renewal step
      2.

-  ``install``\ *``options``*\ ````\ *``file``*

   Install CA certificate from a PEM file.
   The available options are:

   -  ``-n``\ *``nickname``*, ``--nickname``\ *``nickname``*

      Nickname for the certificate.

   -  ``-t``\ *``flags``*, ``--trust-flags``\ *``flags``*

      Trust flags for the certificate in NSS / certutil format.

.. _client_certificate_update_utility:

Client certificate update utility
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There will be new utility, ``ipa-certupdate``, for updating CA
certificates on clients with up-to-date data from LDAP. Until **phase
2** is complete, running it manually will be the only way to update the
CA certificates after installation.

Implementation
--------------

In CA-ful installs, CA certificate renewal is handled by certmonger.
Automatic renewal is handled by certmonger itself. In manual renewal,
``ipa-cacert-manage`` resubmits the certmonger request for the CA
certificate. If the CA certificate is self-signed, the request is
submitted directly to Dogtag. If the CA certificate is signed by an
external CA, ``ipa-cacert-manage`` exports the CSR created by certmonger
to ``/var/lib/ipa/ca.csr`` in the first step. In the seconds step, it
updates ``cn=ca_renewal,cn=ipa,cn=etc,``\ *``suffix``* so that the new
CA certificate can be picked up by certmonger and resubmits the
certmonger request. In the post-save command of the certmonger request,
the renewed CA certificate is added to
``cn=certificates,cn=ipa,cn=etc,``\ *``suffix``*.

When installing new CA certificate manually, ``ipa-cacert-manage`` adds
the certificate directly to
``cn=certificates,cn=ipa,cn=etc,``\ *``suffix``*.

When a CA certificate is renewed, its previous version is not removed to
allow rollover.



Feature Management
------------------

UI
~~

N/A

CLI
~~~

See `design <#CA_certificate_management_utility>`__.

Installers
~~~~~~~~~~

N/A

Upgrade
-------

Old clients will look for IPA CA certificate in
``cn=CAcert,cn=ipa,cn=etc,``\ *``suffix``*. A copy of the most recent
IPA CA certificate needs to be maintained in this entry for
compatibility with old clients.

Old servers do not have
``cn=certificates,cn=ipa,cn=etc,``\ *``suffix``*. Client installer has
to look for CA certificates both in this entry and in
``cn=CAcert,cn=ipa,cn=etc,``\ *``suffix``* for compatibility with old
servers.

.. _how_to_test13:

How to Test
-----------

.. _automated_ca_certificate_renewal_1:

Automated CA certificate renewal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Install IPA server with CA (either self-signed or signed by external
   CA)
#. Get the expiration date of the IPA CA certificate:

      ::

         # getcert list -d /etc/pki/pki-tomcat/alias -n 'caSigningCert cert-pki-ca'

#. Move system time 3 weeks before the expiration date
#. Check the status of the certmonger request:

      ::

         # getcert list -d /etc/pki/pki-tomcat/alias -n 'caSigningCert cert-pki-ca'

#. If the IPA CA was installed self-signed:

   #. Wait for the certmonger request to complete, it should end up with
      MONITORING status
   #. Check that the renewed CA certificate was added to the LDAP
      certificate store and to the ``/etc/pki/pki-tomcat/alias`` NSS
      database

#. If the IPA CA was installed signed by external CA:

   #. Wait for the certmonger request to complete, it should end up with
      CA_WORKING status
   #. Check that an error was syslogged with ALERT severity

.. _manual_ca_certificate_renewal_1:

Manual CA certificate renewal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Install IPA server with CA (either self-signed or signed by external
   CA)
#. To renew the IPA CA certificate as self-signed:

   #. Run ``ipa-cacert-manage renew``, if the IPA CA was not installed
      self-signed, add the ``--self-signed`` option
   #. Wait for the command to complete
   #. Check that the renewed CA certificate was added to the LDAP
      certificate store and to the ``/etc/pki/pki-tomcat/alias`` NSS
      database

#. To renew the IPA CA certificate as signed by external CA:

   #. Run ``ipa-cacert-manage renew``, if the IPA CA was not installed
      signed by external CA, add the ``--external-ca`` option
   #. The command will produce a CSR file at ``/var/lib/ipa/ca.csr``
   #. Sign the CSR file with the external CA to get the renewed CA
      certificate
   #. Run ``ipa-cacert-manage renew``, specify the renewed CA
      certificate and external CA certificate chain files in the
      ``--external-cert-file`` option
   #. Wait for the command to complete
   #. Check that the renewed CA certificate and the external CA
      certificate were added to the LDAP certificate store and to the
      ``/etc/pki/pki-tomcat/alias`` NSS database

.. _manual_install_of_ca_certificate_1:

Manual install of CA certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Install IPA server
#. Run ``ipa-cacert-manage install`` to install the CA certificate
#. Check that the certificate was added to the LDAP certificate store

.. _manual_update_of_local_ca_certificate_files:

Manual update of local CA certificate files
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Install IPA server(s) and possibly client(s)
#. Renew or install CA certificate(s)
#. Run ``ipa-certupdate`` on either a server or a client
#. Check that the ``/etc/ipa/nssdb`` and ``/etc/pki/nssdb`` NSS
   databases and the ``/etc/ipa/ca.crt`` file were updated with CA
   certificates from the LDAP certificate store
#. If on a server, additionaly check that the
   ``/etc/dirsrv/slapd-REALM`` and ``/etc/httpd/alias`` NSS databases
   and the ``/usr/share/ipa/html/ca.crt`` file were updated as well
#. If on a server with a CA, additionaly check that the
   ``/etc/pki/pki-tomcat/alias`` NSS database was updated as well



Test Plan
---------

TODO



RFE Author
----------

`Jan Cholasta <User:Jcholast>`__
