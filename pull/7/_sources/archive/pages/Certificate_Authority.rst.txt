\__TOC_\_

Introduction
------------

Status
------

Feature complete, needs testing.

Fedora
------

The dogtag packages are now available in Fedora. The required packages
should be pulled in as dependencies when ipa-server is installed.

This just makes the binaries available for the IPA installer script. The
installer creates and configures the necessary dogtag components to
stand up a CA.

Installing
----------

A dogtag CA is installed by default by IPA. To install using a
self-signed CA instead of dogtag pass in the ``--selfsign`` argument to
``ipa-server-install``.

The CA uses a separate instance of DS that is used only for the CA. This
instance is named PKI-IPA.

It will install a CA instance into /var/lib/pki-ca.

A copy of the root CA certificate and private key will be put into
/root/cacert.p12.

A copy of the CA agent certificate will be put into /root/ca-agent.p12.
This agent certificate can be imported into a browser and used to
administer CS using the web interface (not recommended).

.. _use_a_different_ca_to_sign_the_ipa_ca_certificate:

Use a Different CA to sign the IPA CA certificate
-------------------------------------------------

If you have an existing CA you can use it make the IPA CA a subordinate.

This is a three-step process:

-  Have ipa-server-install generate a Certificate Signing Request (CSR)
-  Take the CSR to your CA and have it signed
-  Provide the resulting certificate to ipa-server-install to complete
   the installation

.. _detailed_instructions:

Detailed instructions
~~~~~~~~~~~~~~~~~~~~~

Run ``ipa-server-install`` with whatever arguments are appropriate for
your environment and include the ``--external_ca`` flag:

``# ipa-server-install --external-ca``

This will generate a CSR in ``/root/ipa.csr``. This is the file you need
to provide to your CA for signing. You will also need to obtain a PEM
copy of your CA trust chain.

Once you have both of these you can continue the installer:

``# ipa-server-install --external_cert_file=/root/ipa.crt --external_ca_file=/root/existing_ca.crt``

The server caches the answers the first time you run the installer so
you don't need to answer the questions again the second time. This cache
is removed when the installer is run again.

The paths to the certificate and CA must be absolute paths. The dogtag
silent installer will fail if they are not.

Once the installation is complete you will have the same files as a
standalone IPA CA: /root/cacert.p12 and /root/ca-agent.p12.

The only difference is that the CA certificate is signed by your
external CA in this mode and self-signed in the default mode.

.. _using_certificates_from_a_different_ca:

Using Certificates From a Different CA
--------------------------------------

If don't you want to use the new IPA CA features at all that is ok but
you'll need to take a few extra steps.

There are two ways to achieve this:

-  Install IPA using the selfsign CA and replace the server certs
   post-installation
-  Provide PKCS#12 files to the installer (and still use the selfsign
   CA, it just won't generate any certs)

The step setting ``enable_ra`` to ``False`` disables the cert plugin in
the XML-RPC interface. Your IPA server will be unable to issue
certificates.

.. _install_and_replace:

Install and Replace
~~~~~~~~~~~~~~~~~~~

To use the Install and Replace method do the following:

-  Install IPA server with the ``--selfsign`` option
-  Once IPA is up and working run ``ipa-server-certinstall`` once for
   the DS and once for Apache to replace the server certificates
-  If you want the Firefox autoconfiguration to work use an object
   signing certificate to sign the jar file in
   ``/usr/share/ipa/html/configure.jar``
-  Replace the CA certificate in ``/etc/ipa/ca.crt`` and
   ``/usr/share/ipa/html/ca.crt``
-  Edit ``/etc/ipa/default.conf`` and set ``enable_ra`` to ``False``
-  Restart Apache

.. _install_with_your_own_certificates:

Install with your own certificates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To use the Install your own method do the following:

-  Install IPA server with the ``--http_pkcs12`` and ``--dirsrv_pkcs12``
   and their respective pin arguments. Your PKCS#12 files should contain
   the server cert, key and the CA cert chain.
-  If you want the Firefox autoconfiguration to work use an object
   signing certificate to sign the jar file in
   ``/usr/share/ipa/html/configure.jar``
-  Verify that the CA certificate in ``/etc/ipa/ca.crt`` and
   ``/usr/share/ipa/html/ca.crt`` are correct
-  Edit ``/etc/ipa/default.conf`` and set ``enable_ra`` to ``False``
-  Restart Apache

.. _working_with_certificates:

Working with Certificates
-------------------------

Once your CA is configured authorized users can perform a number of
operations.

.. _request_a_certificate:

Request a certificate
~~~~~~~~~~~~~~~~~~~~~

You start with a Certificate Signing Request (CSR). In our case this is
a base-64 encoded PKCS#10 request which looks something like:

::

   -----BEGIN CERTIFICATE REQUEST-----
   MIIBnTCCAQYCAQAwXTELMAkGA1UEBhMCU0cxETAPBgNVBAoTCE0yQ3J5cHRvMRIw
   EAYDVQQDEwlsb2NhbGhvc3QxJzAlBgkqhkiG9w0BCQEWGGFkbWluQHNlcnZlci5l
   ...
   ...
   9rsQkRc9Urv9mRBIsredGnYECNeRaK5R1yzpOowninXC
   -----END CERTIFICATE REQUEST-----

To generate a CSR using OpenSSL run:

::

   % openssl req -new -nodes -out host.csr

You will be prompted for the contents of the certificate subject
(country, state, organization, etc). The only critical piece is the
common name, this needs to be set to the FQDN of your host.

The CSR is in the file ``host.csr`` and the key for this request is in
``privkey.pem``

In NSS you need to start with a cert database so you have an extra step.
An NSS CSR generation looks like:

::

   % certutil -N -d /tmp/test
   % certutil -R -s "CN=ipa.example.com" -d /tmp/test -o test.csr -g 1024 -a

The CSR needs to be passed as one single string on the command-line:

$ ipa cert-request --principal=ldap2/zeus.example.com --add
'MIIBejCB5AIBADA7MQwwCgYDVQQKEwNJUEExEDAOBgNVBAsTB3BraS1pcGExGTAXBgNVBAMTEHpldXMuZ3JleW9hay5jb20wgZ8wDQYJKoZIhvcNAQEBBQADgY0AMIGJAoGBANSiseMKVF/ft44rJDM5XKaMo6jo03TBAC2i61D1GZL6nZ1trl6Oc4YfXccrbQLQ4RGLB6vwDE8vHyYh36OICb1EiWJ+bRaPsn9FaO2mk4qyZp2U/om52BCTSrOq+O+EhTdqLs+hUmUFRDpzmGX3x3UU0JR7cPcvNbcnNQvqfb2NAgMBAAGgADANBgkq
>
hkiG9w0BAQUFAAOBgQAjWFSgv3KZbcjn8V3rhAnuXG9xFzsqD5XsDRBsIMIrG/KNtw4VZBzuXlU2zOdoYm1vlSlzwep9xWXJi5L8HejyqPiCf2mLB60ZxBJLbe1UQ07+oCBMrxck4VXmnySWekRzfYy9lqV0lP/3A5UC6jbtrqJ6t5mp3yiwkjEzEJGp3A=='

certificate:
MIIC7DCCAdSgAwIBAgIBCzANBgkqhkiG9w0BAQUFADAuMQwwCgYDVQQKEwNJUEExHjAcBgNVBAMTFUNlcnRpZmljYXRlIEF1dGhvcml0eTAeFw0wOTEwMDkxNTU5MzJaFw0xMDA0MDcxNTU5MzJaMDsxDDAKBgNVBAoTA0lQQTEQMA4GA1UECxMHcGtpLWlwYTEZMBcGA1UEAxMQemV1cy5ncmV5b2FrLmNvbTCBnzANBgkqhkiG9w0BAQEFAAOBjQAwgYkCgYEA1KKx4wpUX9+3jiskMzlcpoyjqOjTdMEALaLrUPUZkvqdnW2uXo5zhh9dxyttAtDhEYsHq/AMTy8fJiHfo4gJvUSJYn5tFo+yf0Vo7aaTirJmnZT+ibnYEJNKs6r474SFN2ouz6FSZQVEOnOYZffHdRTQlHtw9y81tyc1C+p9vY0CAwEAAaOBizCBiDAfBgNVHSMEGDAWgBSaCxEepKwX6jI6LVrOiHgmiM4EwjBABggrBgEFBQcBAQQ0MDIwMAYIKwYBBQUHMAGGJGh0dHA6Ly96ZXVzLmdyZXlvYWsuY29tOjkxODAvY2Evb2NzcDAOBgNVHQ8BAf8EBAMCBPAwEwYDVR0lBAwwCgYIKwYBBQUHAwEwDQYJKoZIhvcNAQEFBQADggEBAGljf00jgdkyivJFOONlu/lNb25vpWnM/Wu3N6F6hcYC722YjzAFFAFA6wJf4QybrUGQiihh4veqRlknZeoR8Wlc9491E+lGwyk4SZxmSWMHc/TEoTDKPqrngTaf3YsjzBqPhQQF6TkM1TL68uMsU4uHl7mWtD9ueDVQkWlWmTJEJXcT8OdvIFOC/Wb+Vq9FV1lgJAosXmt8poZyHPGX+GiVxdtdZpoZSb7vyMg0FvjhSHNoX8tVo5QKlzxsci+xRkT4tcAe+ReFyQ7+aX851QMluttEUT1Gz1OxnfGojvHbiqYkH92fk+eUJtTZHGpK5H1U5gJVFauCKf/8SzU4awE=
request_id: 11 serial_number: 0xb status: 0 subject:
CN=zeus.example.com,OU=pki-ipa,O=IPA

There is also a short-cut method though it isn't working today (Oct 9).

::

   ipa cert-request file://test.csr --principal=ldap3/zeus.example.com --add

.. _put_a_cert_on_hold:

Put a cert on hold
~~~~~~~~~~~~~~~~~~

::

   $ ipa cert-revoke --revocation-reason=6 0xb

.. _remove_a_cert_from_hold:

Remove a cert from hold
~~~~~~~~~~~~~~~~~~~~~~~

::

   $ ipa cert-revoke --revocation-reason=6 0xb

.. _revoke_a_certificate:

Revoke a certificate
~~~~~~~~~~~~~~~~~~~~

::

   $ ipa cert-revoke --revocation-reason=1 0xb
   revoked: True
   status: 0

.. _retrieve_a_revoked_certificate:

Retrieve a revoked certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   $ ipa cert-get 0xb
   certificate: MIIC7D...
   revocation_reason: 1
   status: 0

Revocation
----------

When you want a certificate to go away you revoke it. `RFC
5280 <http://www.ietf.org/rfc/rfc5280.txt>`__ defines the following
reasons for revocation:

-  0 - unspecified
-  1 - keyCompromise
-  2 - cACompromise
-  3 - affiliationChanged
-  4 - superseded
-  5 - cessationOfOperation
-  6 - certificateHold
-  8 - removeFromCRL
-  9 - privilegeWithdrawn
-  10 - aACompromise

Note that reason code 7 is not used.

A CRL is generated by the CA that contains a list of revoked
certificates. This may be retrieved from an IPA server at:
http://ipa.example.com/ipa/crl/MasterCRL.bin

.. _working_with_services:

Working with Services
---------------------

**Note** this services capability is not in alpha 1

When requesting a certificate a number of things happen:

-  The server checks the virtual ACI "request certificate" to see if the
   requestor has permission to request certificates.
-  The subject of the certificate is compared to the hostname in the
   requested principal. They must match.
-  The requested service record is retrieved. If the service already has
   a userCertificate attribute then the request stops
-  If the service does not exist and the --add argument wasn't provided
   then the request stops
-  If the --add option was requested and the requestor doesn't have
   permission to add services the request stops
-  The requestor must be listed in the managedBy attribute of the
   service record.

.. _managedby_attribute:

managedBy attribute
~~~~~~~~~~~~~~~~~~~

This is a multi-value attribute that contains the distinguished name of
all hosts that are allowed to write the userCertificate attribute of
this service. Use the service-add-host and service-remove-host to
control access.

Example
~~~~~~~

This example demonstrates the steps need to be taken by an administrator
and a client machine that is attempting to request a certificate for
itself. This assumes that the CSR has already been generated and is in
the file web.csr in the current directory.

**admin**

| ``ipa host-add client.example.com --password=secret123``
| ``ipa service-add HTTP/client.example.com``
| ``ipa service-add-host --hosts=client.example.com HTTP/client.example.com``
| ``ipa rolegroup-add-member --hosts=client.example.com certadmin``

**client**

| ``ipa-client-install``
| ``ipa-join -w secret123``
| ``kinit -kt /etc/krb5.keytab host/client.example.com``
| ``ipa -d cert-request ``\ ```file://web.csr`` <file://web.csr>`__\ `` --principal=HTTP/client.example.com``

.. _what_this_does:

What this does
^^^^^^^^^^^^^^

**admin**

-  Add a new host with a one-time password
-  Create an HTTP service principal for the host
-  Allow the host to manage its own userCertificate attribute
-  Allow the host to manage certificates (request, revoke, etc)

**client**

-  Configure the client to use the IPA realm
-  join the host to the realm and retrieve a keytab
-  get a kerberos ticket for the machine
-  request a certificate

.. _vpn_example:

VPN Example
~~~~~~~~~~~

Lets consider the case of a VPN server. On an IPA client we want to set
up a VPN tunnel to a remote host, perhaps at another company. To do this
we need to create an entry for that remote host, a service principal to
store the certificate, then we can issue the cert.

Some of this work needs to be done as an admin:

| ``% kinit admin``
| ``% ipa host-add vpn.remote.com``
| ``% ipa service-add vpn/vpn.remote.com``
| ``% ipa service-add-host --hosts=ipa.example.com vpn/vpn.remote.com``

We've created the remote host and a service principal for it, then gave
permission for the host ipa.example.com to request a certificate on
behalf of vpn.remote.com. This assumes that the certificate request for
vpn.remote.com is in the file **vpn.csr**.

On ipa.example com:

| ``% kinit -kt /etc/krb5.keytab host/ipa.example.com@EXAMPLE.COM``
| ``% ipa cert-request --principal=vpn/vpn.remote.com vpn.csr ``
| ``% ipa service-show vpn/vpn.remote.com``
