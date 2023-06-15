Overview
--------

IPA should support installing without an embedded Certificate Authority,
with user-provided SSL certificates for the HTTP and `Directory
Server <Directory_Server>`__.

.. _use_cases111:

Use Cases
---------

#. User installs IPA using the --http_pkcs12, --dirsrv_pkcs12,
   --http_pin, --dirsrv_pin, --root-ca-file options, giving appropriate
   certificates.
#. IPA installs and works normally, excluding the certificate management
   operations
#. User creates a replica using --http_pkcs12, --dirsrv_pkcs12,
   --http_pin, --dirsrv_pin options to ipa-replica-prepare
#. Replica works normally, excluding certificate management operaions

Design
------

To install without a CA, the following options must be provided to
ipa-server-install:

-  --http_pkcs12: a PKCS#12 file containing the HTTP server certificate
-  --dirsrv_pkcs12: a PKCS#12 file containing the Directory server
   certificate
-  --http_pin: password for the file given in --http_pkcs12
-  --dirsrv_pin: password for the file given in --dirsrv_pkcs12
-  --root-ca-file: a PEM file containing the root CA certificate

Each of the PKCS#12 files must contain exactly one certificate with a
private key. This will be used as the server certificate. It must also
contain certificates of any intermediate CAs. It may contain any number
of other certificates without matching private keys. The two PKCS#12
files may (and often will) be the same. The certificates must be valid
as server certs for the given host.

The PEM file given by --root-ca-file must contain exactly one
certificate. This cert will be trusted as a root CA on the IPA server
and all clients. Both server certs must be signed by this CA (possibly
via a trust chain). It will be available in ``/etc/ipa/ca.crt`` and in
the cACertificate attribute of cn=CACert,cn=ipa,cn=etc,$SUFFIX in LDAP,
just like the IPA CA cert in Dogtag-based installations.

Note that --root-ca-file gives the root of trust, which may not
necessarily be the root of a certificate chain. For example, if a user
wishes to trust a company-wide certificate, but not an external the CA
that signed it, --root-ca-file should give the company certificate.

These options are incompatible with --external-ca.

To install a replica, following options must be provided to
ipa-replica-prepare:

-  --http_pkcs12
-  --dirsrv_pkcs12
-  --http_pin
-  --dirsrv_pin

The root CA is taken from the existing master. All requirements
mentioned in the ipa-server-install case apply.

.. _certificate_rotation:

Certificate rotation
~~~~~~~~~~~~~~~~~~~~

Neither the Dogtag CA nor Certmonger are installed on IPA servers in
this scenario. Before the server certificates expire, new ones must be
issued and installed manually.

In this setup, no warnings are given about certs that are about to
expire.

.. _kdc_pkinit_considerations:

KDC pkinit considerations
~~~~~~~~~~~~~~~~~~~~~~~~~

In the future, another certificate will be needed for the KDC. It will
be specified using the --pkinit_pkcs and --pkinit_pin options, similarly
to the other two certs.

Currently KDC pkinit is disabled.

.. _affected_commands:

Affected commands
~~~~~~~~~~~~~~~~~

IPA's cert plugin and cert-\* commands will not be available at all.
Calling them will result in CommandError (code 905) No online help will
be available on them, or on the "cert" topic.

Certificates removed from LDAP will not be automatically revoked. This
affects the following commands:

-  host-del
-  host-mod
-  host-disable
-  service-del
-  service-mod
-  service-disable

Clients
~~~~~~~

Clients in a CA-less IPA installation will work normally, except host
certificates will not be assigned automatically.

Older clients configure certmonger to obtain the host certificate, which
will fail, with the following line appearing periodically in the system
log:

``   Server failed request, will retry: 905 (RPC failed at server.  unknown command 'cert_request').``

The errors can be stopped by issuing:

| ``   # getcert list  # to find out the certmonger request ID``
| ``   # getcert stop-tracking ``

If needed, machine certificates may be obtained from the external CA and
added to the server with:

``   ipa host-mod ``\ `` --certificate ``

.. _why___root_ca_file:

Why --root-ca-file?
~~~~~~~~~~~~~~~~~~~

IPA requires the user to specify the root CA to be trusted. This done to
ensure that all replicas and clients trust the IPA certificates, no
matter what their initial configuration is.

Theoretically the CA certificate could be extracted from PKCS#12 files,
but to ensure transparency and the administrator's control when it comes
to trusting a root CA, it must always be specified explicitly.

In the future it might be possible to trust more than one CA in this
way.

.. _certificate_management_faq:

Certificate management FAQ
--------------------------

.. _how_to_check_what_format_files_are:

How to check what format files are?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use the handy ``file`` command.

PEM files show up as such:

| ``   $ file /etc/ipa/ca.crt``
| ``   /etc/ipa/ca.crt: PEM certificate``

PKCS#12 files show up as just "data":

| ``   $ file dirsrv.p12``
| ``   dirsrv.p12: data``

To check a PKCS#12 file, you need to know the password:

| ``   $ pk12util -l dirsrv.p12``
| ``   Enter password for PKCS12 file:``
| ``   Certificate(has private key):``
| ``       <...>``
| ``   Certificate:``
| ``       <...>``
| ``   Key(shrouded):``
| ``       <...>``

.. _how_many_certs_are_there_in_a_file:

How many certs are there in a file?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For PKCS#12 files, use ``pk12util -l`` (see previous section).

For PEM files, simply open the file in a pager or text editor and count
the number of blocks. A certificate will look like this:

| ``   $ cat /etc/ipa/ca.crt``
| ``   -----BEGIN CERTIFICATE-----``
| ``   MIIDuzCCAqOgAwIBAgIBATANBgkqhkiG9w0BAQsFADBFMSMwIQYDVQQKExpJRE0u``
| ``   TEFCLkVORy5CUlEuUkVESEFULkNPTTEeMBwGA1UEAxMVQ2VydGlmaWNhdGUgQXV0``
| ``   aG9yaXR5MB4XDTEzMDMyMDE3MDQxNFoXDTMzMDMyMDE3MDQxNFowRTEjMCEGA1UE``
| ``   ChMaSURNLkxBQi5FTkcuQlJRLlJFREhBVC5DT00xHjAcBgNVBAMTFUNlcnRpZmlj``
| ``   YXRlIEF1dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMZi``
| ``   pF9Dz5O1rVTRnwIdttHl0sKpHeRqzi/S7bnAFh3Jb2UxzFmHTpgQFKqq72mYatpL``
| ``   O0BPc47IGh9gwGZNLcEaNCf7zYCbqBJso8RV6SxbHSEdo+JuSYhMxVasKQcojqeY``
| ``   /wx11A4NSQAco6mBZz255llZqMQcJVMW4T8aioUd19Yh35CM9vr6l6dgUnvA9fAF``
| ``   TOl144yfF8AjvF1hIAePjLyl+Y/xxh1U2j5hF4z7ZeUGHKVZR9pQ62kbM7TgAR6Y``
| ``   YLGpis44JPfgRVkDGEkc7Vzpct1D4Iz7/oGMV+0kbJbz+9DSIHWY10QTtf9mNQNn``
| ``   xKGa3wCf5u8ctfmms8cCAwEAAaOBtTCBsjAfBgNVHSMEGDAWgBQCHF1DVeHg3kUG``
| ``   VRm/j0f9eji6nzAPBgNVHRMBAf8EBTADAQH/MA4GA1UdDwEB/wQEAwIBxjAdBgNV``
| ``   HQ4EFgQUAhxdQ1Xh4N5FBlUZv49H/Xo4up8wTwYIKwYBBQUHAQEEQzBBMD8GCCsG``
| ``   AQUFBzABhjNodHRwOi8vdm0tMDg0LmlkbS5sYWIuZW5nLmJycS5yZWRoYXQuY29t``
| ``   OjgwL2NhL29jc3AwDQYJKoZIhvcNAQELBQADggEBAB3+or2Q/aPO4ZMBE4Q6xCMV``
| ``   09ESAXXT/0DLakAt28ljy1wWKVR3d54TxZJ4DEcYgbxDa1A87DZW8sn+LM4Uwap9``
| ``   DUyHA0mhBjROe6NXgJQl9aZ7IeE1ht+pw/n+JR2sg3ccYHvQjRcEZj2OPQuavyPn``
| ``   hwokDc3FVarlsQcrtfePG3e8TQXAnpSxV+KAMBEp4yib5nrkNZZoU+nqMI0ftXrk``
| ``   rP5q0SaEBEjC4+AoYje4Bv3+8RKT1kwBMkTL8eRRuWZmKvOy9sCnnFfU4HMMkPTK``
| ``   NJg9Gt8a/xU6GK239M1keCKct87VqWN1unXaD51bgotK1UJWj1q8H262mSYzfRg=``
| ``   -----END CERTIFICATE-----``

.. _how_to_extact_certs_or_or_combine_certs_into_files:

How to extact certs or or combine certs into files?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _pem_files:

PEM Files
^^^^^^^^^

PEM files are plain text; manipulate them using a text editor

.. _base64_encoded_der_certificates:

Base64-encoded DER certificates
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The letters and symbols between a PEM file's BEGIN CERTIFICATE and END
CERTIFICATE markers are a base64-encoded DER-encoded X.509 certificate.
To convert between PEM and base64-encoded DER, just add or remove the
markers in a text editor.

.. _using_a_nss_database:

Using a NSS database
^^^^^^^^^^^^^^^^^^^^

NSS databases can be manipulated using ``certutil`` and ``pk12util``.

In a NSS database, each certificate is identified using a "nickname".
The nickname can be set with -n option, or taken from the "Friendly
name" entry in a PKCS#12 file, or from the Subject of the certificate.
Note that nicknames and Friendly Names are \*not\* part of the cert
itself.

Create a temporary NSS database using:

``   certutil -N -d /path/to/nssdb``

Remember to set appropriate permissions if you're working with sensitive
data.

To list nicknames and trust flags in of the certs in the database,
enter:

``   certutil -L -d /path/to/nssdb/``

To import a PKCS#12 file to a database:

``   pk12util -i /path/to/pkcs12file.p12 -d /path/to/nssdb``

To export a PKCS#12 file from a database (this will export the
certificate chain and private key(s), if available):

``   pk12util -o /path/to/pkcs12file.p12 -d /path/to/nssdb -n ``

To import a PEM file:

``   certutil -A -d /path/to/nssdb -n ``\ `` -a -t ``\ `` -i ``

For an explicitly trusted (root) CA, use "CT,C,C" for flags. Otherwise
use ",,"

To export a PEM file (to stdout):

``   certutil -L -d /path/to/nssdb -n ``\ `` -a``

Note that PEM is referred to as "ASCII" in certutil documentation.

To create a self-signed root CA certificate and private key:

``   certutil -S -d /path/to/nssdb -s "CN=$(hostname)" -m $RANDOM -n RootCA -t CT,C,C -x``

You should substitute a unique serial number for $RANDOM.

To generate a Certificate Signing Request for a server:

``   certutil -R -d /path/to/nssdb -s "CN=$(hostname)" -1 -a -o request.csr``

Select Digital Signature, Non-Repudiation and Key Encipherment for the
extension.

To sign the CSR, and get a PEM file with the cert:

``   certutil -C -d /path/to/nssdb -m $RANDOM -a -i request.csr -c RootCA``

Again, substitute a unique serial number for $RANDOM.

.. _how_to_check_that_my_certificates_will_be_usable:

How to check that my certificates will be usable?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To inspect PKCS#12 files, use ``pk12util -l``. For other files, import
them in a NSS database and use ``certutil -L``. See above for details.

For the servers, you will need certs with a private key. These show up
as "Certificate(has private key):" in ``pk12util`` output, and with "u"
flags in ``certutil -L`` without ``-n`` The certs will need Digital
Signature, Non-Repudiation and Key Encipherment in the "Certificate Key
Usage" extension (visible in ``pk12util -l`` and ``certutil -L -n``
output). Also, server certs must have "CN=" in the Subject.

The server certs will need a valid trust chain leading up to the CA
certificate. You can check the trust chain following the "Subject" and
"Issuer" lines in the ``pk12util -l`` output. CAs should have
Certificate Signing and CRL Signing in their "Certificate Key Usage"
extension.

.. _feature_managment:

Feature Managment
-----------------

UI
~~

N/A

CLI
~~~

The --http_pkcs12, --dirsrv_pkcs12, --http_pin, --dirsrv_pin options to
ipa-server-install and ipa-replica-prepare work again. The
--root-ca-file option was added to ipa-server-install.

Configuration
~~~~~~~~~~~~~

The feature can be installed as detailed above. There is no supported
way to enable a CA once a CA-less IPA is installed, or to revert to
CA-less from a Dogtag installation.

Replication
-----------

When creating a replica file, certificates for that replica must be
specified. These must be signed by the CA given as --root-ca-file to the
original master (a copy of this CA cert is in /etc/ipa/ca.crt).



Updates and Upgrades
--------------------

Existing installs are not affected.

Upgrading CA-less instances should work normally.



Test Plan
---------

See `dedicated test page <V3/CA-less_install/Test>`__.



RFE Author
----------

`pviktori <User:pviktorin>`__
