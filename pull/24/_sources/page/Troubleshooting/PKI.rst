PKI
===

This page contains **PKI** troubleshooting advice. For other issues,
refer to the index at `Troubleshooting <Troubleshooting>`__.



IPA won't start, expired certificates
=====================================

Where available (>= v4.8.0?), the ``ipa-cert-fix`` command can be used
to recover from expired system certificate scenarios. See `blog
post <https://frasertweedale.github.io/blog-redhat/posts/2019-05-24-ipa-cert-fix.html>`__.

Starting with IPA `3.0.0 <IPAv3_300_ga>`__ all FreeIPA certificates are
tracked by `Certmonger <Certmonger>`__ and should be renewed
automatically. In case of problems, see
`Certmonger#Manually_renew_a_certificate <Certmonger#Manually_renew_a_certificate>`__.

If your Certificate Authority certificate is expired, see `CA
Certificate Renewal page <Howto/CA_Certificate_Renewal>`__ .

For v2.0 see
`IPA_2x_Certificate_Renewal <IPA_2x_Certificate_Renewal>`__.

**DO NOT RUN** ``ipa-cacert-manage renew`` in an effort to renew
expirted certificates. This is for renewing the CA certificate only,
which by default is good for 20 years. It is very unlikely you need to
do this.



PKI-tomcatd fails to start
==========================

After an upgrade of IPA packages, pki-tomcatd fails to start. See
`Troubleshooting pki-tomcatd fails to
start <https://floblanc.wordpress.com/2017/09/11/troubleshooting-freeipa-pki-tomcatd-fails-to-start/>`__.



Authentication Errors
=====================

If you see something like
``4301 (RPC failed at server. Certificate operation cannot be completed: FAILURE``
``(Authentication Error))`` or ``Invalid Credential`` the likely culprit
is the RA agent certificate that IPA uses to authenticate against
`PKI <PKI>`__.

Use ``getcert list -d /etc/httpd/alias -n ipaCert`` to show the current
status of the certificate tracked by `Certmonger <Certmonger>`__. It
should be **MONITORING**.

If it isn't in MONITORING, or it is and things still aren't working,
compare the serial number of the certificate with that on other IPA
masters.

If you have the ``[free]ipa-healthcheck`` package available in your
distribution run that as it will check for this condition.

For IPA < 4.7.0:

``# certutil -L -d /etc/httpd/alias -n ipaCert | grep Serial``

For IPA >= 4.7.0:

``# openssl x509 -noout -serial -in /var/lib/ipa/ra-agent.pem``

The serial number should match the value of the 2nd integer at:

::

      ``# ldapsearch -x -h localhost -p 389 -b uid=ipara,ou=People,o=ipaca description ``

(use port 7389 for 2.x servers)

If they are different this suggests that one has been renewed. Only the
most recent is allowed by dogtag. To repair this, go to the master with
the most recent certificate:

For IPA < 4.7.0:

::

      ``# certutil -L -d /etc/httpd/alias -n ipaCert -a > /tmp/ra.crt ``

This will export all the certificates. Edit this file and remove all but
the first certificate, You can double-check the result with:

``# openssl x509 -text -in /tmp/ra.crt``

For IPA >= 4.7.0 the certificate is in ``/var/lib/ipa/ra-agent.pem``

It should have only the cert with the latest serial #.

After removing the unnecessary certificates from the file, for IPA <
4.7.0:

Now add it to your cert database:

| ``# certutil -A -n ipaCert -d /etc/httpd/alias -t u,u,u -a -i /tmp/ra.crt``
| ``# service httpd restart``

If the certificate is valid and the ou=People entry is ok then check the
`PKI <PKI>`__ logs ``/var/log/pki`` or ``/var/log/pki-ca``.

If you see an error like "Failed to connect LDAP server" then try
restarting the tomcat process, either pki-cad (for IPA 3.0) or
pki-tomcatd@pki-tomcat.service.



CRL gets very old
=================

If the main CRL file containing the list of invalidated certificates is
old and not updated, make sure you check that:

-  There is at least one `PKI <PKI>`__ master server generating the CRL
   - see `CVE-2012-4546 <CVE-2012-4546>`__ for instructions.
-  CRL generation on that server is not blocked by wrong ownership of
   ``/var/lib/ipa/pki-ca/publish/`` directory or there are no SELinux
   errors. Consult PKI ``system`` for details. (`related user
   case <https://www.redhat.com/archives/freeipa-users/2014-November/msg00012.html>`__)



External CA renewal with ipa-cacert-manage fails
================================================

The second step of external CA renewal may fail for a number of reasons:

-  Subject name mismatch

      The new CA certificate issued by the external CA uses a different
      subject name than the old CA certificate.

-  Subject name encoding mismatch

      The new CA certificate issued by the external CA uses the same
      subject name as the old CA certificate, but it is encoded
      differently. Some CAs like to re-encode the subject name from
      certificate signing requests in certificates they issue. This does
      not work well with NSS, which considers subject names to be equal
      only if they binary representation is exactly the same. To avoid
      the problem, configure the external CA to either respect the CSR's
      encoding, or use UTF8String encoding (which is the FreeIPA
      default). Microsoft Certificate Services / AD-CS uses
      PrintableString as the default encoding in subject names. See
      `this GitHub
      thread <https://github.com/freeipa/freeipa/pull/930#issuecomment-332748881>`__
      for how to observe and configure the AD-CS treatment of Subject DN
      encoding.

-  Subject public key info mismatch

      The new CA certificate issued by the external CA uses a different
      public / private key pair than the old CA certificate.

-  Not a valid CA certificate

      The new CA certificate issued by the external CA is either missing
      the Basic Constraints extension, or the Basic Constraints
      extension indicates that the certificate is not a CA certificate.