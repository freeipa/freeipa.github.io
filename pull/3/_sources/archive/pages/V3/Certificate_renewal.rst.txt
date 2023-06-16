.. _ipa_certificate_renewal:

IPA Certificate Renewal
=======================

{{ admon/important \| \| This is a design page for a specific feature.
As project evolves it will become outdated and will not be maintained.
It is already known that FreeIPA 3.1 does not exactly work this way. }}

Certificates are valid for a varying period of time, capped by the
validity time of the root CA itself. The server certificates that IPA
issues are automatically renewed by certmonger before they expire. There
is currently no mechanism to renew the CA itself or the certificates it
requires.

The following certificates are issued for our two CA types.

.. _dogtag_ca:

Dogtag CA
---------

#. the root certificate itself (8 years in initial implementation, 15+
   years when `#3315 <https://fedorahosted.org/freeipa/ticket/3315>`__
   was implemented )

   -  subject name CN=Certificate Authority,O=$REALM
   -  enrolled using caCACert profile
   -  private key in /var/lib/$CAINSTANCE/alias as "caSigningCert
      cert-$CAINSTANCE"
   -  exported to /root/cacert.p12 for use on replicas
   -  stored in CS.cfg as ca.signing.cert
   -  nickname in CS.cfg as ca.signing.nickname
   -  copies stored in

      -  /var/lib/$CAINSTANCE/alias as "caSigningCert cert-$CAINSTANCE"
      -  /etc/httpd/alias as "$REALM IPA CA"
      -  /etc/dirsrv/slapd-PKI-CA as "$REALM IPA CA"
      -  /etc/dirsrv/slapd-$DSINSTANCE as "$REALM IPA CA"
      -  /etc/ipa/ca.crt

#. the OCSP subsystem certificate (2 years)

   -  subject name CN=OCSP Subsystem,O=$REALM
   -  enrolled using caOCSPCert profile
   -  private key in /var/lib/$CAINSTANCE/alias as "ocspSigningCert
      cert-$CAINSTANCE"
   -  exported to /root/cacert.p12 for use on replicas
   -  stored in CS.cfg as ca.ocsp_signing.cert
   -  nickname in CS.cfg as ca.ocsp_signing.nickname

#. the CA SSL services certificate (2 years)

   -  subject name CN=$fqdn,O=$REALM
   -  enrolled using caServerCert profile
   -  private key in /var/lib/$CAINSTANCE/alias as "Server-Cert
      cert-$CAINSTANCE"
   -  stored in CS.cfg as ca.sslserver.cert
   -  nickname in CS.cfg as ca.sslserver.nickname

#. CA subsystem certificate (2 years)

   -  subject name CN=CA Subsystem,O=$REALM
   -  enrolled using caServerCert profile
   -  private key in /var/lib/$CAINSTANCE/alias as "subsystemCert
      cert-$CAINSTANCE"
   -  exported to /root/cacert.p12 for use on replicas
   -  stored in CS.cfg as ca.subsystem.cert
   -  nickname in CS.cfg as ca.subsystem.nickname
   -  published to uid=CA-rapier.bos.redhat.com-9443,ou=people,o=ipaca

#. CA audit log certificate (2 years)

   -  subject name CN=CA Audit,O=$REALM
   -  enrolled using caSignedLogCert profile
   -  private key in /var/lib/$CAINSTANCE/alias as "auditSigningCert
      cert-$CAINSTANCE"
   -  exported to /root/cacert.p12 for use on replicas
   -  stored in CS.cfg as ca.audit_signing.cert
   -  nickname in CS.cfg as ca.audit_signing.nickname

#. IPA CA agent (2 years)

   -  subject name CN=ipa-ca-agent,O=$REALM
   -  enrolled using caAdminCert profile
   -  exported to /root/ca-agent.p12
   -  published to uid=admin,ou=people,o=ipaca

#. IPA RA agent (2 years)

   -  subject name CN=IPA RA,O=$REALM
   -  enrolled using caServerCert profile
   -  private key in /etc/httpd/alias as "ipaCert"
   -  published to uid=ipara,ou=people,o=ipaca

#. IPA service (2 years)

   -  subject name CN=$fqdn,O=$REALM
   -  enrolled using caIPAserviceCert profile
   -  issued by IPA RA agent
   -  revoked by IPA RA agent, superseded

#. IPA service (2 years)

   -  subject name CN=$fqdn,O=$REALM
   -  enrolled using caIPAserviceCert profile
   -  issued by IPA RA agent
   -  revoked by IPA RA agent, superseded

#. IPA service (2 years)

   -  subject name CN=$fqdn,O=$REALM
   -  enrolled using caIPAserviceCert profile
   -  issued by IPA RA agent
   -  revoked by IPA RA agent, superseded

#. CA jar signing (4 years)

   -  subject name CN=Object Signing Cert,O=$REALM
   -  enrolled using caJarSigningCert profile
   -  private key in /etc/httpd/alias as "Signing-Cert"
   -  issued by IPA RA agent
   -  used to sign browser auto-configuration jar

#. IPA service (2 years)

   -  subject name CN=$fqdn,O=$REALM
   -  enrolled using caIPAserviceCert profile
   -  private key in /etc/dirsrv/slapd-$DSINSTANCE as "Server-Cert"
   -  issued by IPA RA agent
   -  monitored by certmonger

#. IPA service (2 years)

   -  subject name CN=$fqdn,O=$REALM
   -  enrolled using caIPAserviceCert profile
   -  private key in /etc/dirsrv/slapd-PKI-IPA as "Server-Cert"
   -  issued by IPA RA agent
   -  monitored by certmonger

#. IPA service (2 years)

   -  subject name CN=$fqdn,O=$REALM
   -  enrolled using caIPAserviceCert profile
   -  private key in /etc/httpd/alias as "Server-Cert"
   -  issued by IPA RA agent
   -  monitored by certmonger

.. _selfsign_ca:

Selfsign CA
-----------

#. the root certificate itself (10 years)

   -  subject name CN=$REALM Certificate Authority
   -  private key in /etc/httpd/alias as "$REALM IPA CA"
   -  copies stored in

      -  /etc/dirsrv/slapd-$DSINSTANCE as "$REALM IPA CA"
      -  /etc/ipa/ca.crt

#. CA jar signing (10 years)

   -  subject name CN=Object Signing Cert,O=$REALM
   -  enrolled using caJarSigningCert profile
   -  private key in /etc/httpd/alias as "Signing-Cert"
   -  used to sign browser auto-configuration jar

This document is going to focus on renewing Dogtag CA certificates. The
solutions are broken into short, medium and long-term, moving from more
manual to more automated.

.. _short_term_plan:

Short term plan
---------------

Warn
~~~~

certmonger can track certificates and make a note into a file what
certificate(s) need to be renewed. IPA can read this file and display
the contents on any ipa CLI or UI command when the user is in the admins
group. This will put the renewal front and center to any administrator.

We will warn at a configurable interval, storing the warning value in
cn=ipaconfig.

Manual instructions will be provided to users to handle renewing the
certificates and installing the result in the appropriate locations(s).

Information on renewals can be found at:

-  http://docs.redhat.com/docs/en-US/Red_Hat_Certificate_System/8.1/html/Admin_Guide/Renewing_Certificates.html
-  http://docs.redhat.com/docs/en-US/Red_Hat_Certificate_System/8.1/html/Admin_Guide/renewing-certificates.html

.. _medium_term_plan:

Medium term plan
----------------

Renew
~~~~~

certmonger will be trained to talk directly to the CA and can renew all
but the CA root certificate. This renewal will only happen on the master
doing the CRL generation.

All CA system certificates have specific but different requirements.
Renewal enrollments allow one to submit serial number of a certificate
to be renewed. The CA recovers original certificate request and profile
used to generate original system certificate based on certificate serial
number provided in a renewal request. Knowing the original profile
allows the CA to regenerate an identical system certificate.

There is currently no way to determine the profile or request ID
required for renewing these CA certificates. A mechanism will need to be
created (where is TBD, though likely in the CA) to reconcile this.
Whatever process we come up with will need to work with existing
installations that did not store this information.

The certmonger on other IPA servers can use the EE public interface post
renewal time to see if updated certificates are available. If they are
it will retrieve them and install them as appropriate.

The time for doing the renewal should be configurable in IPA. certmonger
already has this as a configuration. I'm not sure if we want to tweak
that file on the fly or have certmonger query IPA in cn=ipaconfig. The
renewal time should be > than the warning time to give renewal a chance
to occur before we start warning users of expiration.

There is also the issue of what machine renews these shared
certificates. The convention will be the CRL master will do this.
certmonger will periodically check to see if new certificates exist. We
will not revoke old CA subsystem certificates when a new one is issued.
This differs from what we do with IPA host and service certificates that
we issue.

.. _renewing_the_agent_cert:

Renewing the agent cert
^^^^^^^^^^^^^^^^^^^^^^^

Renewing the RA agent cert requires an additional step. The public key
is stored in the PKI-IPA LDAP database and will need to be updated when
the certificate is renewed. If it isn't then the agent will fail to
authenticate.

.. _long_term_plan:

Long term plan
--------------

.. _renewing_the_ca_certificate:

Renewing the CA certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Renewing the CA is a special case because it will require refreshing all
clients and servers in IPA. The renewal process is expected always to be
a manual process.

Updating the CA in the shared NSS database in /etc/pki/nssdb may be
handled by SSSD.

.. _subordinate_ca_renewal:

Subordinate CA renewal
~~~~~~~~~~~~~~~~~~~~~~

If the CA is installed as a subordinate of another then by definition
this is going to be a manual renewal process. The same problems of
sharing the updated CA certificate apply.

.. _special_cases:

Special cases
-------------

.. _rekeying_the_ca:

Rekeying the CA
~~~~~~~~~~~~~~~

Rekeying of the CA requires a new CA install. This may also be
necessary/desired as the CA ages and certificates are revoked, unused,
etc, as maintenance. We will need a way to install a new CA alongside
the old one and slowly migrate all users to the new one.

.. _rekeying_a_server_cert:

Rekeying a server cert
~~~~~~~~~~~~~~~~~~~~~~

If an IPA service wants new keys it can do this now using either
certmonger or can generate a CSR and use the ipa cert-request command.

Tickets
=======

IPA
---

-  `1985 Email notify Admin prior to CA certificate
   expiration <https://fedorahosted.org/freeipa/ticket/1985>`__
-  `2803 RFE: Auto-renew IPA subsystem
   certificates <https://fedorahosted.org/freeipa/ticket/2803>`__

`Category:NoLink <Category:NoLink>`__
`Category:CheckUpdate <Category:CheckUpdate>`__
