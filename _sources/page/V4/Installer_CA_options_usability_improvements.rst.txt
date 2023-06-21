Installer_CA_options_usability_improvements
===========================================

Overview
--------

External CA and CA-less installer command line options related to
certificate files are sometimes hard to use and often confusing to
users. Users are required to convert certificate files to the right
format before they can use them in installers. In CA-less install, users
also have to choose the right CA certificate to trust and make a file
containing only this certificate, both of which can be done
automatically.

Improve the situation by being more generous in accepted file formats
and automatically handling whatever can be handled automatically.



Use Cases
---------



External CA install
----------------------------------------------------------------------------------------------

-  Automatically determine which certificate is IPA CA certificate
-  Use 1, 2, or more certificate files in PEM, DER or PKCS#7 format



CA-less install
----------------------------------------------------------------------------------------------

-  Automatically determine which CA certificate to trust
-  Use 1 or more certificate files in PEM, DER, PKCS#7 or PKCS#12 format
   and 1 private key file in PKCS#1, PKCS#8 or PKCS#12 format

Design
------



External CA install
----------------------------------------------------------------------------------------------

Replace ``--external_cert_file`` and ``--external_ca_file`` options of
``ipa-server-install`` with a single ``--external-cert-file`` option.
The option must be specified one or more times and accepts PEM files
containing one or more certificates, DER certificate files and PKCS#7
files. The combined files from the option must contain the IPA CA
certificate and the whole CA certificate chain of the IPA CA
certificate's issuer. IPA CA certificate will be automatically picked
from the available certificates.



CA-less install
----------------------------------------------------------------------------------------------

Replace ``--dirsrv_pkcs12`` and ``--http_pkcs12`` options of
``ipa-server-install`` and ``ipa-replica-prepare`` with
``--dirsrv-cert-file`` and ``--http-cert-file`` options. Each of the
options must be specified one or more times and accept PEM file
containing one or more certificates and/or zero or one private keys, DER
certificate files and PKCS#7 and PKCS#12 files. The combined files from
each of the options must contain exactly one private key and one server
certificate and may contain the whole or part of CA certificate chain of
the server certificate's issuer.

Rename ``--dirsrv_pin`` and ``--http_pin`` of ``ipa-server-install`` and
``ipa-replica-prepare`` to ``--dirsrv-pin`` and ``--http-pin`` for
consistency. Note that in addition to PKCS#12 files, PKCS#1 and PKCS#8
files may also be PIN protected.

Add ``--ca-cert-file`` option to ``ipa-server-install``. The option may
be specified one or more times and accepts PEM files containing one or
more certificates, DER certificate files and PKCS#7 files. The combined
files may contain the whole or part of CA certificate chain of the DS
and HTTP server certificate's issuer.

Remove ``--root-ca-file`` option of ``ipa-server-install``. The option
is useless, because the trusted CA must always be the issuer of the DS
and HTTP server certificates. The CA certificate will be picked
automatically from the certificates specified by ``--dirsrv-cert-file``,
``--http-cert-file`` and ``--ca-cert-file``.

Update ``ipa-server-certinstall`` to follow the above convention as
well.

Implementation
--------------

A method for importing groups of files was added to the CertDB class.
PEM and DER certificate files and PKCS#12 files are imported directly.
PKCS#7 files are converted to certificate in PEM format using
``openssl pkcs7`` and imported. PKCS#1 and PKCS#8 private key files are
converted to PKCS#12 using ``openssl pkcs8`` and ``openssl pkcs12`` and
imported.

The old command line options of ``ipa-server-install`` and
``ipa-replica-prepare`` are hidden and kept for backward compatibility.



Feature Management
------------------

UI

N/A

CLI

N/A

Installers
----------------------------------------------------------------------------------------------

See `the design <#Design>`__.

Upgrade
-------

N/A



How to Test
-----------

Easy to follow instructions how to test the new feature. FreeIPA user
needs to be able to follow the steps and demonstrate the new features.

The chapter may be divided in sub-sections per `Use
Case <#Use_Cases>`__.



Test Plan
---------

Test scenarios that will be transformed to test cases for FreeIPA
`Continuous Integration <V3/Integration_testing>`__ during
implementation or review phase. This can be also link to `source in
cgit <https://git.fedorahosted.org/cgit/freeipa.git/>`__ with the test,
if appropriate.



RFE Author
----------

`Jan Cholasta <User:Jcholast>`__