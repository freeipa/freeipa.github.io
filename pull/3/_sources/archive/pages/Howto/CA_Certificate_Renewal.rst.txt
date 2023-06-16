This page provides manual instructions to renew the IPA CA certificate.

.. _before_you_start:

Before you start
----------------

**Important:** This article is about renewing Certificate Authority (CA)
certificate which by default expires in 20 years. In \``getcert list`\`
its nickname is 'caSigningCert'. If you want to renew other certificate,
e.g., a host or service certificate which typically has expiration
period 2 years and is managed by Certmonger please check `manually
renewal section of Certmonger
page <Certmonger#Manually_renew_a_certificate>`__

This renewal must take place in the period in which your other
certificates are still valid. Your CA needs to be running in order to
renew its own subsystem certificates. If you try to renew the CA
certificate after it has expired such that its validity dates are past
the expiration date of the CA subsystem certificates then your IPA
server will not work.

So for example, your CA is set to expire on 12/23, along with all the CA
subsystem certificates and likely the server certificates used by Apache
and 389-ds.

You come in after Christmas to find that nothing works because the
certificates have expired. When you provide your CSR to the external CA,
they will need to issue a certificate that is valid from at least Dec
22. It **must overlap** the old validity time. You will need to reset
the system clock on the CA back to the 22nd in order to renew the other
certificates, then you can bring time forward.

.. _procedure_in_current_ipa:

Procedure in current IPA
------------------------

FreeIPA 4.1 introduced `integrated CA certificate renewal
tools <V4/CA_certificate_renewal>`__. To manually renew the CA
certificate, run:

``# ipa-cacert-manage renew``

.. _procedure_in_ipa_4.1:

Procedure in IPA < 4.1
----------------------

.. _request_renewed_certificate:

Request renewed certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~

On a random FreeIPA server with CA installed run:

::

   # getcert start-tracking -d /etc/pki/pki-tomcat/alias -n 'caSigningCert cert-pki-ca' -P <pin> -c dogtag-ipa-ca-renew-agent
   # getcert resubmit -i <id> -T ipaCSRExport
   # getcert stop-tracking -i <id>

This will get a certificate request file (CSR) in ``/root/ipa.csr``.
Sign the CSR file with the *external CA* and the save the resulting
certificate to ``ipa.crt`` in PEM format.

.. _renew_ca_certificate_on_ca_servers:

Renew CA Certificate on CA Servers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On all FreeIPA servers **with CA** installed, run:

::

   # /usr/lib64/ipa/certmonger/stop_pkicad
   # certutil -A -d /etc/pki/pki-tomcat/alias -n 'caSigningCert cert-pki-ca' -t CT,C,C -a -i ipa.crt
   # /usr/lib64/ipa/certmonger/renew_ca_cert 'caSigningCert cert-pki-ca'

On all FreeIPA servers run:

::

   # certutil -A -d /etc/dirsrv/slapd-REALM -n 'REALM IPA CA' -t CT,,C -f /etc/dirsrv/slapd-REALM/pwdfile.txt -a -i ipa.crt
   # certutil -A -d /etc/httpd/alias -n 'REALM IPA CA' -t CT,C,C -f /etc/httpd/alias/pwdfile.txt -a -i ipa.crt
   # systemctl restart dirsrv@REALM.service
   # systemctl restart httpd

.. _renew_ca_certificate_on_freeipa_clients:

Renew CA Certificate on FreeIPA clients
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On all FreeIPA servers and FreeIPA clients run

::

   # certutil -A -d /etc/pki/nssdb -n 'IPA CA' -t CT,C,C -a -i ipa.crt
   # cp ipa.crt /etc/ipa/ca.crt 

.. _procedure_in_ipa_4.0:

Procedure in IPA < 4.0
----------------------

You need the original CSR in order to obtain a new certificate. You may
be able to find this in one of three places:

#. The external CA may still have a copy of it
#. In ``/root/ipa.csr`` on the initial IPA master
#. In ``/etc/pki-ca/CS.cfg`` in ``ca.signing.certreq`` on the initial
   IPA master. This will need to be converted to PEM format.

You'll need to know the nickname of your CA in the NSS databases. This
is usually '" IPA CA". I will use "EXAMPLE.COM IPA CA".

You can query the Apache database to find out what this currently is:

``# certutil -L -d /etc/httpd/alias``

.. _renew_the_certificate:

Renew the Certificate
~~~~~~~~~~~~~~~~~~~~~

Give the CSR to your external CA and have them issue you a new
certificate. This document assumes that the resulting certificate is
saved into /root/ipa.crt and that the external CA certificate chain is
saved into /root/external-ca.pem.

You will need to do this renewal on the IPA CA designated for managing
renewals.

One way to identify the initial IPA master is to see if the value for
``subsystem.select`` is New.

::

   # grep subsystem.select /etc/pki-ca/CS.cfg
   subsystem.select=New

An alternative method is to look at the post-save command in this
output. It needs to be renew_ca_cert:

::

   # getcert list
   Number of certificates and requests being tracked: 8.
   Request ID '20131125153455':
           status: MONITORING
           stuck: no
           key pair storage: type=NSSDB,location='/var/lib/pki-ca/alias',nickname='auditSigningCert cert-pki-ca',token='NSS Certificate DB',pin='455536908955'
           certificate: type=NSSDB,location='/var/lib/pki-ca/alias',nickname='auditSigningCert cert-pki-ca',token='NSS Certificate DB'
           CA: dogtag-ipa-renew-agent
           issuer: CN=Certificate Authority,O=EXAMPLE.COM
           subject: CN=CA Audit,O=EXAMPLE.COM
           expires: 2015-11-15 15:34:12 UTC
           pre-save command: /usr/lib64/ipa/certmonger/stop_pkicad
           post-save command: /usr/lib64/ipa/certmonger/renew_ca_cert "auditSigningCert cert-pki-ca"
           track: yes
           auto-renew: yes
   ...

.. _install_the_new_ca_certificate_on_your_ipa_master_ca:

Install the new CA certificate on your IPA master CA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The CA needs to be shut down in order to update its certificate:

``# service ipa stop``

Update the CA certificate NSS database:

``# certutil -A -d /var/lib/pki-ca/alias -n 'caSigningCert cert-pki-ca' -t CT,C,C -a -i /root/ipa.crt``

Replace the value of ``ca.signing.cert`` in ``/etc/pki-ca/CS.cfg``. This
is the base64 value of the certificate. You can obtain this by removing
the BEGIN/END blocks from ipa.crt and compressing it into a single line.

Update the Apache NSS database:

``# certutil -A -d /etc/httpd/alias -n 'EXAMPLE.COM IPA CA'  -t CT,C,C -a -i /root/ipa.crt``

Update the LDAP server instances:

| ``# certutil -A -d /etc/dirsrv/slapd-REALM -n 'EXAMPLE.COM IPA CA' -t CT,C,C -a -i /root/ipa.crt``
| ``# certutil -A -d /etc/dirsrv/slapd-PKI-IPA -n 'EXAMPLE.COM IPA CA' -t CT,C,C -a -i /root/ipa.crt``

Update the shared system database:

``# certutil -A -d /etc/pki/nssdb -n 'EXAMPLE.COM IPA CA'  -t CT,C,C -a -i /root/ipa.crt``

Update the CA in the filesystem:

| ``# cat /root/ipa.crt /root/external-ca.pem >/usr/share/ipa/html/ca.crt``
| ``# cp /root/ipa.crt /etc/ipa/ca.crt``

Restart the world

``# service ipa start``

Update the CA in LDAP

First convert the certificate to DER form:

``# openssl x509 -outform DER -in /root/ipa.crt  -out /tmp/ipa.der``

Add to LDAP:

::

   # kinit admin
   # ldapmodify -Y GSSAPI
   SASL/GSSAPI authentication started
   SASL username: admin@EXAMPLE.COM
   SASL SSF: 56
   SASL data security layer installed.
   dn: cn=CAcert,cn=ipa,cn=etc,dc=example,dc=com
   changetype: modify
   replace: cacertificate;binary
   cacertificate;binary:<file:///tmp/ipa.der

.. _install_new_ca_on_other_ipa_masters_with_a_ca:

Install new CA on other IPA masters with a CA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The CA needs to be shut down in order to update its certificate:

``# service ipa stop``

Update the CA certificate NSS database:

``# certutil -A -d /var/lib/pki-ca/alias -n 'caSigningCert cert-pki-ca' -t CT,C,C -a -i /root/ipa.crt``

Replace the value of ``ca.signing.cert`` in ``/etc/pki-ca/CS.cfg``. This
is the base64 value of the certificate. You can obtain this by removing
the BEGIN/END blocks from ipa.crt and compressing it into a single line.

Update the Apache NSS database:

``# certutil -A -d /etc/httpd/alias -n 'EXAMPLE.COM IPA CA'  -t CT,C,C -a -i /root/ipa.crt``

Update the LDAP server instances:

| ``# certutil -A -d /etc/dirsrv/slapd-REALM -n 'EXAMPLE.COM IPA CA' -t CT,C,C -a -i /root/ipa.crt``
| ``# certutil -A -d /etc/dirsrv/slapd-PKI-IPA -n 'EXAMPLE.COM IPA CA' -t CT,C,C -a -i /root/ipa.crt``

Update the shared system database:

``# certutil -A -d /etc/pki/nssdb -n 'EXAMPLE.COM IPA CA'  -t CT,C,C -a -i /root/ipa.crt``

Update the CA in the filesystem:

| ``# cat /root/ipa.crt /root/external-ca.pem >/usr/share/ipa/html/ca.crt``
| ``# cp /root/ipa.crt /etc/ipa/ca.crt``

Restart the world

``# service ipa start``

.. _install_new_ca_on_other_ipa_masters_without_a_ca:

Install new CA on other IPA masters without a CA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Copy the updated CA to the machine. I'm assuming it is in /root/ipa.crt.

Stop the world

``# service ipa stop``

Update the Apache NSS database:

``# certutil -A -d /etc/httpd/alias -n 'EXAMPLE.COM IPA CA'  -t CT,C,C -a -i /root/ipa.crt``

Update the LDAP server instances:

| ``# certutil -A -d /etc/dirsrv/slapd-REALM -n 'EXAMPLE.COM IPA CA' -t CT,C,C -a -i /root/ipa.crt``
| ``# certutil -A -d /etc/dirsrv/slapd-PKI-IPA -n 'EXAMPLE.COM IPA CA' -t CT,C,C -a -i /root/ipa.crt``

Update the shared system database:

``# certutil -A -d /etc/pki/nssdb -n 'EXAMPLE.COM IPA CA'  -t CT,C,C -a -i /root/ipa.crt``

Update the CA in the filesystem:

| ``# cat /root/ipa.crt /root/external-ca.pem >/usr/share/ipa/html/ca.crt``
| ``# cp /root/ipa.crt /etc/ipa/ca.crt``

Restart the world

``# service ipa start``

.. _install_the_new_ca_on_all_ipa_client_machines:

Install the new CA on all IPA client machines
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Retrieve the updated IPA CA, I've put it into /root/ipa.crt.

Update the shared system database:

``# certutil -A -d /etc/pki/nssdb -n 'EXAMPLE.COM IPA CA'  -t CT,C,C -a -i /root/ipa.crt``

Update the CA in the filesystem:

``# cp /root/ipa.crt /etc/ipa/ca.crt``

Troubleshooting
---------------

See the `Troubleshooting Guide <Troubleshooting#PKI_Issues>`__.
