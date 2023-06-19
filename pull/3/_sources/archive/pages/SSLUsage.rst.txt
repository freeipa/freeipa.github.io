\__NOTOC_\_

Introduction
------------

SSL is used in a lot of different places within IPA. This is an attempt
to explain where it is used, where the certificates come from, and how
everything inter-operates.

We use two different SSL libraries within IPA: NSS and OpenSSL.

The following servers are configured with SSL server certificates:

-  Apache, using the NSS database in /etc/httpd/alias
-  389-ds, using the NSS database in /etc/dirsrv/slapd-INSTANCE
-  dogtag uses an NSS database for its LDAP server and the CA itself but
   we don't need visibility into those.

.. _certificate_authorities:

Certificate Authorities
-----------------------

There are two options for Certificate authorities in IPA: dogtag and
selfsign.

dogtag
~~~~~~

dogtag gives us a full CA that can do all the things you'd expect a CA
to do:

-  Issue certificates
-  Revoke certificates
-  Be an OCSP responder
-  Generate CRLs
-  and lots more that we don't necessarily use

selfsign
~~~~~~~~

selfsign self-signed CA using certutil to generate certificates. This CA
resides in the Apache NSS database (``/etc/httpd/alias``). This also
uses the file ``/var/lib/ipa/ca_serialno`` in an attempt to guarantee
unique serial numbers.

.. _certificate_subjects:

Certificate Subjects
--------------------

dogtag allows us to have great control over the subject of the
certificates we issue. The profile
``/var/lib/pki-ca/profiles/ca/caIPAserviceCert.cfg`` controls how
certificates are issued. This is controlled using the
``policyset.serverCertSet.1.default.params.name`` directive. It is set
with the --subject argument to the IPA server installer and defaults to
``CN=fqdn, O=IPA``. Regardless of the certificate subject requested we
pull the CN out of the request and use only that.

selfsign does not allow for a configurable subject. The request must
match the expected subject configured.

Servers
-------

Apache
~~~~~~

Apache uses the NSS database in ``/etc/httpd/alias``. mod_nss is the SSL
engine.

At a minimum you'll find 3 certificates here:

-  CA certificate: the IPA CA certificate (for selfsign the private key
   also resides here)
-  Server-Cert: the SSL server cert for Apache
-  Signing-Cert: the SSL object signing certificate for signing the jar
   file that allows the browser to be automatically configured

If using dogtag then the RA agent certificate will also reside here.

389-ds
~~~~~~

389-ds uses the NSS database in ``/etc/dirsrv/slapd-INSTANCE``.

There should be only two certificates here:

-  CA certificate: the IPA CA certificate
-  Server-Cert: the SSL server cert for Apache

.. _signing_certificate:

Signing Certificate
-------------------

In order to modify a browser configuration using javascript the code
must be signed by a signing certificate of a trusted CA. We issue an
object signing certificate to sign the code in
/usr/share/ipa/html/configure.jar during installation. This is signed
with the nickname 'Signing-Cert' from the Apache NSS database.

.. _client_authentication:

Client authentication
---------------------

The only client authentication done is done by the IPA XML-RPC server
connecting to dogtag to do agent commands (request, revoke, etc). This
uses python-nss and the RA agent certificate created during dogtag
installation.

.. _ldap_replication:

LDAP replication
----------------

LDAP replication must use SSL. Because all replicas are provided with an
SSL certificate (by ``ipa-replica-prepare``) this is straightforward.

.. _what_happens_during_installation:

What happens during installation?
---------------------------------

Regardless of the CA type we install a copy of the CA into two places on
disk:

-  /usr/share/ipa/html/ca.crt: so clients can download the CA
-  /etc/ipa/ca.crt: for the command-line tools and other clients

This duplication only exists on servers, on client machines the CA is
only put into /etc/ipa/ca.crt.

.. _dogtag_1:

dogtag
~~~~~~

.. _selfsign_1:

selfsign
~~~~~~~~

.. _what_happens_during_an_ipa_command_request:

What happens during an ipa command request?
-------------------------------------------

We use Kerberos to authenticate to the web server but this
authentication is done in the clear over HTTP. For LDAP we can use SASL
which provides encryption as well as authentication.

The basic process is:

-  Get Kerberos credentials
-  Stick then into the Authorization header of the HTTP request we will
   make
-  Make an SSL connection
-  Send the request over this SSL connection

Most HTTP authentication is a two-step process. You make an HTTP request
and the server responds with a 401 if it needs authentication. We know
in advance that authentication is required so we always include the
Authorization header.

The connection occurs in ipalib/rpc.py. The xmlrpclib ServerProxy()
class takes as an argument the transport class to use. Transport is a
class in xmlrpclib that makes requests using httplib. There is also a
SafeTransport that makes requests over SSL but it doesn't fit our needs
(it doesn't verify CAs, for one thing). Instead we override the methods
in Transport() that we need to make an SSL connection.

The simple SSLSocket() class does this for us. It returns an
SSLConnection object. This class does all the SSL heavy lifting. We pass
it the location of the IPA CA (``/etc/ipa/ca.crt``) so we can validate
the request.

Newer versions of httplib have dropped some classes we need, notably
SSLFile and FakeSocket. We have slurped in a copy of what we need in
ipapython/ipasslfile.py for those systems that don't have it.
