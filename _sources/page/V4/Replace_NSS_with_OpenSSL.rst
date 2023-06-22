Replace_NSS_with_OpenSSL
========================

Overview
========

This design describes the replacement of NSS with OpenSSL as the SSL
backend for client HTTPS connections across FreeIPA.

Design
======

NSS caused a lot of issues in the past when it comes to it being an SSL
backend to client HTTPS connections in FreeIPA. One of the biggest
issues was the NSS design flaw where the function to initialize the NSS
database can only be called once otherwise it breaks. NSS SSL
implementation also seems to have issues in FIPS mode where the library
requires password to access an NSS database even if the only required
information there is a CA certificate (see
https://bugzilla.redhat.com/show_bug.cgi?id=1410143 for details).

This design proposes getting rid of the FreeIPA *nsslib* module and
replacing the **NSSConnection** class with a class from standard Python
**httplib.HTTPSConnection** which uses OpenSSL as its backend for SSL
connections.

The proposed change is the main moving part for enabling FreeIPA to run
in FIPS-enabled systems. It will also simplify the life of the
developers as the machinery around NSS initialization will go away.

There are some concerns about standard Python OpenSSL bindings.
Generally, OpenSSL does not implement PKCS#11 and therefore it does not
allow client authentication with HSM/smart cards. In FreeIPA, though,
this is handled in PKINIT and is therefore not an issue. Another concern
is that the Python bindings don't allow caching of CRLs and they don't
implement OCSP. As FreeIPA does not cache nor check CRLs, this should
not be our concern. Checking of CRL is also already implemented in the
Python OpenSSL bindings so it can be turned on if there is a need for
it. As for the OCSP concerns, OpenSSL itself implements OCSP so an OCSP
check could be performed in a subprocess call if required.

Implementation
==============

FreeIPA will drop **NSSConnection** completely as a part of removal of
the *ipapython.nsslib* module. This means removing it from the
*ipalib.rpc* module as well as from connections with the Dogtag server.
**NSSConnection** is replaced by the **IPAHTTPSConnection** class. This
class inherits from the aforementioned **httplib.HTTPSConnection** for
which it configures SSL aspects according to the official documentation
(https://docs.python.org/2/library/ssl.html#context-creation).

In FreeIPA, we do not perform client authentication using a client
certificate to connect to a remote server. There is only one exception
and that is communication of IPA server to Dogtag server. This means
client certificate authentication is implemented using the standard
Python ssl.SSLContext method but solely for this purpose, on other
places PKINIT should be used instead.

This new HTTPS communication implementation requires CA certificates and
Dogtag client certificate (alongside with its private key, for the above
mentioned purpose) to be stored in files. We already do that - the CA
certificates we use are stored in */etc/ipa/ca.crt* and the Dogtag
certificate and key storage is found in
*/etc/httpd/alias/kra-agent.pem*. The changes in this design will
therefore put pressure on us handling these files right. The file
*/etc/httpd/alias/kra-agent.pem* also gets renamed to *ra-agent.pem* so
that the name better reflects its use and the file is moved to
*/var/lib/ipa* as that seems like a more appropriate directory.

Certmonger should no longer track **ipaCert** in */etc/httpd/alias* NSS
database. Instead, it should track the Dogtag client certificate and key
stored in the new location, */var/lib/ipa/ra-agent.pem*.

Upgrade
=======

Upgrade needs to perform the certmonger tracking transition of
**ipaCert** mentioned at the end of the *Implementation* chapter. This
means, that the previous *ra-agent.pem* needs to be exported to the
correct location during upgrade.



How to Use/Test Plan
====================

The changes are more-or-less code internal and should be seamless from
the outside. The only change is that the previously named
*kra-agent.pem* changes its location.