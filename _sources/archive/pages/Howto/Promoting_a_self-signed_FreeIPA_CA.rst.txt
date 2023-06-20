Promoting_a_self-signed_FreeIPA_CA
==================================



Promote a self-signed FreeIPA CA
--------------------------------

FreeIPA officially never supported installations with --selfsign option,
i.e. FreeIPA servers which do not use Certificate Authority but only use
a self-signed certificate stored in a local NSS certificate database to
sign certificates. Thus, official user guides do not have a procedure
how to promote a replica in such environment to serve as a new master
CA.

This *HOWTO* should help Administrator promote such replica to a master
CA which is able to sign new certificates. This may be useful for
example when the old FreeIPA master server is to be decommissioned and
is being replaced with a new replica. Please note, that if you have two
promoted FreeIPA replicas, certificate serial numbers may clash unless
you manually configure proper offset in */var/lib/ipa/ca_serialno*.



Procedure to install and promote a new self-signed replica
----------------------------------------------------------------------------------------------

1. Install a replica

| ``# ipa-replica-install``

2. Copy CA serial number setting from master to replica:

| ``# scp /var/lib/ipa/ca_serialno root@REPLICA:/var/lib/ipa/``

3. On replica, set correct owner and permissions:

| ``# chown root:apache /var/lib/ipa/ca_serialno``
| ``# chmod 550 /var/lib/ipa/ca_serialno``

4. Restore SELinux context on serial file:

| ``# restorecon  var/lib/ipa/ca_serialno``

5. Copy master CA certificate and pwdfile.txt to replica:

| ``# scp /etc/httpd/alias/cacert.p12 /etc/httpd/alias/pwdfile.txt root@REPLICA:~/``

6. On replica, import the CA certificate:

| ``# pk12util -i ~/cacert.p12 -w ~/pwdfile.txt -d /etc/httpd/alias/ -k /etc/httpd/alias/pwdfile.txt``

7. The list of certificates in NSS database (including the one imported)
can be listed with:

| ``# certutil -L -d /etc/httpd/alias/``

However, since *pk12util* import util is not capable of setting a
correct certificate nickname, the imported certificate will have a
nickname like *CN=$REALM Certificate Authority*, which is not recognized
by IPA certificate system.

The following procedure can be used **set a correct nickname** of the
certificate (replace *$REALM* with your realm):

a) Export the certificate

``# certutil -d /etc/httpd/alias/ -L -n 'CN=$REALM Certificate Authority' -a > ~/cacert.crt``

b) Delete the old certificate (NSS database /etc/httpd/alias/ should be
backed up before this step):

``# certutil -d /etc/httpd/alias/ -D -n 'CN=$REALM Certificate Authority'``

c) Import the certificate with correct nickname:

| ``# certutil -A -n "$REALM IPA CA" -d /etc/httpd/alias/ -f /etc/httpd/alias/pwdfile.txt \``
| ``  -i /root/cacert.crt -a -t CTu,u,Cu``

8. Enable certificate operations on IPA replica:

``# echo "enable_ra=True" >> /etc/ipa/default.conf``

9. Reload replica web server to pick up new configuration:

``# service httpd reload``

`Category:How to <Category:How_to>`__