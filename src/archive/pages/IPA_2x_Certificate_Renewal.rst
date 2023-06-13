\__NOTOC_\_

Introduction
------------

Automated certificate renewal of the CA subsystem certificates was added
in `IPA 3.0 <IPAv3_300_ga>`__. If you can't upgrade, here are some
manual steps that should get you moving forward.

You probably have several certificates tracked by certmonger in a
CA_UNREACHABLE state, like:

::

   Request ID '20111111152846':
           status: CA_UNREACHABLE
           ca-error: Server failed request, will retry: -504 (libcurl failed to execute the HTTP POST transaction.  Peer certificate cannot be authenticated with known CA certificates).
           stuck: yes
           key pair storage: type=NSSDB,location='/etc/httpd/alias',nickname='Server-Cert',token='NSS Certificate DB',pinfile='/etc/httpd/alias/pwdfile.txt'
           certificate: type=NSSDB,location='/etc/httpd/alias',nickname='Server-Cert',token='NSS Certificate DB'
           CA: IPA
           issuer: CN=Certificate Authority,O=EXAMPLE.COM
           subject: CN=ipa.example.com,O=EXAMPLE.COM
           expires: 2013-11-11 15:28:46 UTC
           eku: id-kp-serverAuth,id-kp-clientAuth
           pre-save command: 
           post-save command: /usr/lib64/ipa/certmonger/restart_httpd
           track: yes
           auto-renew: yes

Process
-------

The first thing you need to do is update certmonger to at least 0.58-1:

``# yum update certmonger``

This provides the dogtag-ipa-renew-agent CA that can directly renew the
dogtag CA subsystem certificates. You only need to do this on those
masters with a CA (so it won't hurt if you upgrade it in other places
too but it won't help with this problem either).

In order for this to work you are going to need to go back in time to
when the certificates are all still valid. First we need to stop the NTP
service:

``# /sbin/service ntpd stop``

To find out when the certificates were still valid, run:

::

    # for nickname in "auditSigningCert cert-pki-ca" "ocspSigningCert cert-pki-ca" "subsystemCert cert-pki-ca" "Server-Cert cert-pki-ca"
      do
        echo $nickname
        certutil -L -d /var/lib/pki-ca/alias -n "${nickname}" | grep -i after
      done

This tells us when to set time back to. I recommend setting time back at
least 24 hours before expiration.

Next, you need to determine the PIN for the CA NSS database:

``# grep internal= /var/lib/pki-ca/conf/password.conf``

Now we need to tell certmonger about all the CA certificates it needs to
renew

::

   # for nickname in "auditSigningCert cert-pki-ca" "ocspSigningCert cert-pki-ca" "subsystemCert cert-pki-ca" "Server-Cert cert-pki-ca"
    do
        /usr/bin/getcert start-tracking -d /var/lib/pki-ca/alias -n "${nickname}" -c dogtag-ipa-renew-agent -P <internal pin>
    done

We also need to renew the agent certificate that IPA uses to
authenticate:

`` # /usr/bin/getcert start-tracking -d /etc/httpd/alias -n ipaCert -c dogtag-ipa-renew-agent -p /etc/httpd/alias/pwdfile.txt``

Use "getcert list" to confirm that these 5 certs are now being tracked
and note the Request IDs. You should be tracking a total of 8
certificates now.

Now it's time to go back in time to when the certificates are valid,
something like:

``# date 102910262013``

Let's force renewal on all of the certificates:

:literal:`# for line in `getcert list | grep Request | cut -d "'" -f2`; do getcert resubmit -i $line; done`

This will renew the CA subsystem certificates but the server
certificates for Apache and 389-ds will have failed. We need to do a
little more work to get those renewed, and to get the CA fully
operational again.

Running ``getcert list`` should confirm this. The two 389-ds and the
Apache certs are still in CA_UNREACHABLE, though the reason is now SSL
peer rejected your certificate as expired.

Let's continue getting the CA up and running. The audit subsystem
certificate is recreated with the wrong trust permissions. To fix this
run:

``# certutil -M -n "auditSigningCert cert-pki-ca" -d /var/lib/pki-ca/alias -t u,u,Pu ``

The dogtag configuration file has a base64-encoded copy of most of these
certificates in it. You'll need to update those by hand.

To get this run:

``# for nickname in "auditSigningCert cert-pki-ca" "ocspSigningCert cert-pki-ca" "subsystemCert cert-pki-ca" "Server-Cert cert-pki-ca"; do certutil -L -d /var/lib/pki-ca/alias -n "${nickname}" -a > /tmp/"${nickname}"; done``

Stop the CA service:

``# /sbin/service pki-cad stop``

Then edit /etc/pki-ca/CS.cfg and find the cert entry for each one and
replace the blobs. The option names are like ca.audit_signing.cert,
ca.ocsp_signing.cert, etc. It should be fairly straightforward. There
are 4 certs you need to replace.

Here are the nicknames and values:

::

   'auditSigningCert cert-pki-ca': 'ca.audit_signing.cert'
   'ocspSigningCert cert-pki-ca': 'ca.ocsp_signing.cert'
   'subsystemCert cert-pki-ca': 'ca.subsystem.cert'
   'Server-Cert cert-pki-ca': 'ca.sslserver.cert'

The PEM exported by certutil is going to be broken into several
64-character lines. You will need to combine them into a single line.

Backing up this file in advance would be a good idea.

Now you can try to restart the CA to see what happens. It should come up
fine:

``# /sbin/service pki-cad start``

For ipaCert, stored in /etc/httpd/alias you have another job to do. This
certificate is used to authenticate with the CA. You'll need to use
ldapmodify to fix things up.

Start by looking at the new value for ipaCert. You need to do two
things:

``# certutil -L -d /etc/httpd/alias -n ipaCert | grep -i serial``

Next you need the base64-encoded value of the cert like before:

``# certutil -L -d /etc/httpd/alias -n ipaCert -a``

Again you'll need to drop the header/footer and combine this into a
single line.

Next see what is already there with:

``# ldapsearch -x -h localhost -p 7389 -D 'cn=directory manager' -W -b uid=ipara,ou=People,o=ipaca``

You need to replace the serial number in the description attribute with
the new one. The serial number is the 2nd number. The format of the
description line is:

``2;``\ ``;``\ ``;``

The change is going to look something like:

::

   # ldapmodify -x -h localhost -p 7389 -D 'cn=directory manager' -w password
   dn: uid=ipara,ou=people,o=ipaca
   changetype: modify
   add: usercertificate
   usercertificate:: MII...PNQ=
   -
   replace: description
   description: 2;16;CN=Certificate Authority,O=EXAMPLE.COM;CN=IPA RA,O=EXAMPLE.COM
   <extra blank line to finish> 
   ^D to exit

Now restart the Apache service

``# /sbin/service httpd restart``

Next we need to renew the two 389-ds and the Apache server certificates.

``# ipa-getcert list``

For each of the three Request IDs run something like this:

``# ipa-getcert resubmit -i ``

Restart the world:

``# /sbin/service ipa restart``

Return to the present time.

| ``# /sbin/service ntpd start``
| ``# date (confirm it is now)``

To make sure that communication with the CA is working run:

``# ipa cert-show 1``

Notes
-----

I tested this on a RHEL 6.4 system that I installed ipa-server-2.2.0 and
krb5-server-1.9. I did this by:

| ``# date 111110262011``
| ``# ipa-server-install -N ...``

I confirmed that things were working, then I brought time to today:

``# rdate -s ``

So I basically simulated an installation 2 years in the past and see
today that my certificates are expired. Then I did the renewal
procedure. I did the install without an NTP server because otherwise it
would have reset the current time to today during the install, and I
wanted to be in the past.
