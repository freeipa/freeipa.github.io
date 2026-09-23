Distribution_of_CA_certificates_to_clients
==========================================

\__NOTOC_\_

Overview
========

Allow automated and manual renewal of IPA CA certificate. Provide an
utility for manual renewal, including modification of chaining
(self-signed vs. signed by external CA). Store multiple CA certificates
in LDAP and distribute them to clients.

This page describes **phase 2** of the CA certificate management
feature, which consists of distribution of CA certificates to IPA
clients. Visit `V4/CA certificate renewal <V4/CA_certificate_renewal>`__
for description of **phase 1**, which consists of automated and manual
CA certificate renewal, CA certificate management utility and storage of
multiple CA certificate in LDAP.

Trac tickets:

-  `#4322 ([RFE] Automatically distribute CA certificates to
   clients) <https://fedorahosted.org/freeipa/ticket/4322>`__