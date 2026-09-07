NewCA
=====

Introduction
------------

By default during installation of IPA we generate a self-signed CA that
is used to issue the SSL server certs used by 389 and Apache. A mistake
was made in the generation of this CA, `bug
514027 <https://bugzilla.redhat.com/show_bug.cgi?id=514027>`__. The CA
Basic Constraint extension wasn't added to the certificate.

This wasn't a problem until NSS 3.12.3 which requires a certificate have
this extension to be considered a CA. The most obvious impact is that
Firefox 3.5 cannot import an IPA CA certificate.

This affects IPA 1.0 through 1.2.1.



Instructions for issuing a new CA certificate
---------------------------------------------

Some changes were made in IPA so that any new CA certificates generated
will have the appropriate extensions. So any new installations of IPA
1.2.2 or higher will not have this problem.

For existing installations we have written a tool,
`ipa-newca <http://freeipa.org/downloads/ipa-newca>`__, that will issue
a new CA certificate using the existing key pair. This means that any
existing SSL server certs (including on replicas) will not need to be
re-issued.

It does mean that the new CA certificate will need to be propagated
everywhere that the old CA certificate exists. This includes the 389 and
Apache server on every IPA server as well as any browser that has
trusted the old CA certificate.

To generate a new CA certificate, log into the initial IPA master. This
is the instance that contains the CA.

The tool requires that the 389 instance be shut down in order to run. It
is probably best to shut down all IPA services before running the tool.

**Step 1** Upgrade to IPA 1.2.2

**Step 2** Generate a new CA certificate

::

   # /usr/sbin/ipactl stop
   # chmod +x ipa-newca
   # ./ipa-newca
   # /usr/bin/ipactl restart

You will be prompted before a new CA is generated.

**Step 3** Distribute this new CA

This new CA certificate needs to be manually installed into all other
IPA replicas. The new certificate be install on other machines by
logging into it and running:

::

   # cd /usr/share/ipa/html
   # wget -O ipa.crt http://replica4.greyoak.com/ipa/config/ca.crt
   # chmod 444 ipa.crt
   # certutil -A -d /etc/httpd/alias -n "CA certificate" -t CT,,C -a < /usr/share/ipa/html/ca.crt
   # certutil -A -d /etc/dirsrv/slapd-GREYOAK-COM -n "CA certificate" -t CT,,C -a < /usr/share/ipa/html/ca.crt
   # /usr/sbin/ipactl restart

Browsers will need to trust this new CA certificate as well. The easiest
way to do this is to direct users to
```http://ipa.example.com/ipa/config/ca.crt`` <http://ipa.example.com/ipa/config/ca.crt>`__

This will download and attempt to install the new CA certificate. Have
users click all 3 trust boxes and import the cert.