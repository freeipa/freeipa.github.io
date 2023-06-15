\__NOTOC_\_

Overview
========

Relevant upstream tickets:
`#3547 <https://fedorahosted.org/freeipa/ticket/3547>`__,
`#3552 <https://fedorahosted.org/freeipa/ticket/3552>`__

Certificates issued by FreeIPA server 3.1 and later contains 2 CRL/OCSP
URIs server by Dogtag CA configured by FreeIPA:

::

   Certificate:
       Data:
           Version: 3 (0x2)
           Serial Number: 17 (0x11)
       Signature Algorithm: sha256WithRSAEncryption
           Issuer: O=EXAMPLE.COM, CN=Certificate Authority
           Validity
               Not Before: Apr  8 10:16:15 2013 GMT
               Not After : Apr  9 10:16:15 2015 GMT
           Subject: O=EXAMPLE.COM, CN=testcert.example.com
           ...
           X509v3 extensions:
               X509v3 Authority Key Identifier:
                   keyid:9F:25:93:2F:20:2A:79:9A:A8:88:CF:CC:EB:D0:F5:43:E7:3B:B1:EE

               Authority Information Access:
                   OCSP - URI:https://ipa-ca.example.com/ca/ocsp
                   OCSP - URI:https://server1.example.com/ca/ocsp

               X509v3 Key Usage: critical
                   Digital Signature, Non Repudiation, Key Encipherment, Data
   Encipherment
               X509v3 Extended Key Usage:
                   TLS Web Server Authentication, TLS Web Client Authentication
               X509v3 CRL Distribution Points:

                   Full Name:
                     URI:https://ipa-ca.example.com/ipa/crl/MasterCRL.bin
                   CRL Issuer:
                     DirName: O = ipaca, CN = Certificate Authority

                   Full Name:
                     URI:https://server1.example.com/ipa/crl/MasterCRL.bin
                   CRL Issuer:
                     DirName: O = ipaca, CN = Certificate Authority
           ...

One OCSP/CRL URI points to the original CA issuing the certificate and
one points to a general URL (managed by FreeIPA) pointing to any other
FreeIPA CA via CNAME/A DNS record that can serve the OCSP/CRL URI in
case if the original FreeIPA CA was decommissioned or unavailable at the
moment.

However, it `was
discovered <http://www.redhat.com/archives/freeipa-users/2013-April/msg00085.html>`__
there are 2 issues related to this change:

#. Having https in the URIs requires client (e.g. a web browser)
   validating the request to validate the machine serving the CRL/OCSP
   response itself even though he CRL/OCSP response is already signed by
   the CA and thus verifiable. Clients will fail to retrieve the
   CRL/OCSP in case of the general address as it is just a CNAME for a
   FreeIPA server whose certificate does not allow it to serve this
   address.
#. Even though we have 2 OCSP URIs in the certificate, the Firefox
   browser will not fail over to the general URI due to limitation in
   NSS which only tries the last OCSP and then fail when it is not
   available.

.. _use_cases10n:

Use Cases
=========

-  Client browser with strict OCSP check tries to validate certificate
   issued by FreeIPA for a web service. When OCSP URIs are not right
   (due to limitations above), client browser may refuse to open the
   page
-  Any other client software may try to validate a certificate issues by
   FreeIPA

Design
======

Issue certificates with just one OCSP/CRL URI pointing to the general
DNS name *ipa-ca.$DOMAIN*. This DNS name will contain A/AAAA DNS records
to all FreeIPA CAs and it will be automatically maintained by FreeIPA
(if it manages DNS). Otherwise, administrator would need to maintain
this DNS name on his own if he wants to keep the OCSP/CRL links
functional.

The general name *ipa-ca.$DOMAIN* was originally implemented to contain
a list of CNAMEs pointing to FreeIPA masters with CA installed. This DNS
record needs to be changed to use A/AAAA records instead as DNS
specification forbids multiple CNAMEs in one DNS name.

Implementation
==============

-  *ipa-replica-install* and ipa-ca-install needs to be updated to add
   A/AAAA records to *ipa-ca.$DOMAIN* when replica with CA is being
   installed
-  *ipa-csreplica-manage del* command should delete A/AAAA record from
   *ipa-ca.$DOMAIN* of the deleted CA replica
-  When IPA server is being upgraded, it should convert existing
   *ipa-ca.$DOMAIN* based on CNAMEs to version based on A/AAAA
   addresses. When there was a custom update to *ipa-ca.$DOMAIN*,
   upgrade should not touch it.



Feature Management
==================

UI
~~

No change to UI needed. The change is done under-the-hood and in the
installation tools.

CLI
~~~

No change to CLI needed.



Major configuration options and enablement
==========================================

The feature is enabled by default.

Replication
===========

Certificates are already replicated, so no change in this area.



Updates and Upgrades
====================

When FreeIPA is upgraded to the new version, it should:

#. Update *ipa-ca.$DOMAIN* DNS name to use A/AAAA instead of CNAMEs
#. Update FreeIPA Dogtag certificate profile to use single OCSP/CRL URI
   for new certificates. When already issued certificates needs to be
   fixed, a new certificate needs to be issued. certmonger can be used
   to simplify this task on a client if it tracks the certificate:

   #. *ipa-getcert list* - to identify tracking request ID of a
      certificate that needs to be re-issued
   #. *ipa-getcert resubmit -i $TRACKING_REQUEST_ID* - to ask for
      reissue of the certificate
   #. *ipa-getcert list -i $TRACKING_REQUEST_ID* - to check that
      certificate was successfully renewed

Dependencies
============

Certificate validation depends on capabilities of certificate libraries
(e.g. NSS, OpenSSL). This design was based on abilities of NSS library
used in Mozilla Firefox (see `NSS Bug
797815 <https://bugzilla.mozilla.org/show_bug.cgi?id=797815>`__ for
relevant discussion).



External Impact
===============

Change impacts projects processing certificate issued by FreeIPA server.



RFE Author
==========

Martin Kosek
