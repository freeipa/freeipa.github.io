LDAP
====

The following command will allow you to use a 3rd party certificate
after initially deploying the FreeIPA system. You will need the
following files:

-  ``mysite.key`` (your private SSL key)
-  ``mysite.crt`` (your SSL certificate)

Note: if FreeIPA is deployed on multiple servers (master and replicas),
the procedure must be applied on each server and requires a SSL
certificate/private SSL key for each server.

Note2: this procedure can be applied to change the HTTP/LDAP server
certificates even if FreeIPA was initially deployed with an embedded CA.



Procedure in current IPA
------------------------

Prerequisite
----------------------------------------------------------------------------------------------

The certificate in mysite.crt must be signed by a CA known by the
service you are loading the certificate into. If it is not the case, you
can use the commands *ipa-cacert-manage install* and *ipa-certupdate* to
load the CA's certificate prior to installing the new certificate.

| ``# ipa-cacert-manage -p DM_PASSWORD -n NICKNAME -t C,, install ca.crt``
| ``# ipa-certupdate``

Note: the command ipa-certupdate must be executed on all the IPA hosts
(master/replicas/clients) before moving to the next step.



Configuration of the 3rd part certificate
----------------------------------------------------------------------------------------------

You can install the new bundle using:

``# ipa-server-certinstall -w -d mysite.key mysite.crt``

The option -w|--http installs the certificate for the HTTP server, and
-d|--dirsrv installs the certificate for the LDAP server. Please see
ipa-server-certinstall(1) man page for more information regarding all
the available options.

Then restart your daemons:

| ``# systemctl restart httpd.service``
| ``# systemctl restart dirsrv@MY-REALM.service``



Procedure in IPA < 4.1
----------------------



Prerequisite
----------------------------------------------------------------------------------------------

The certificate in mysite.crt must be signed by a CA known by the
service you are loading the certificate into. If it is not the case, you
need to add the 3rd part CA (and its chain if it is a sub-CA) to the NSS
databases used by FreeIPA. For instance, if the chain contains a CA
(whose certificate is /root/ca1.crt) and a subCA (whose certificate is
/root/ca2.crt), run the following procedure on the master:

-  Add the CA and subCA certs to /etc/ipa/ca.crt and
   /usr/share/ipa/html/ca.crt

| ``# cat /root/ca1.crt /root/ca2.crt >> /etc/ipa/ca.crt``
| ``# cp /etc/ipa/ca.crt /usr/share/ipa/html/ca.crt``

-  Add the CA and subCA certs to the system DB

| ``# certutil -A -d /etc/pki/nssdb/ -n 'EXT-CA1' -t CT,C,C -a -i /root/ca1.crt``
| ``# certutil -A -d /etc/pki/nssdb/ -n 'EXT-CA2' -t CT,C,C -a -i /root/ca2.crt``

-  Add the CA and subCA certs to HTTP DB

| ``# certutil -A -d /etc/httpd/alias/ -n 'EXT-CA1' -t CT,C,C -a -i /root/ca1.crt``
| ``# certutil -A -d /etc/httpd/alias/ -n 'EXT-CA2' -t CT,C,C -a -i /root/ca2.crt``

-  Add the CA and subCA certs to DS main instance DB

| ``# certutil -A -d /etc/dirsrv/slapd-EXAMPLE-COM/ -n 'EXT-CA1' -t CT,C,C -a -i /root/ca1.crt``
| ``# certutil -A -d /etc/dirsrv/slapd-EXAMPLE-COM/ -n 'EXT-CA2' -t CT,C,C -a -i /root/ca2.crt``

-  Add the CA and subCA certs to DS PKI instance DB

| ``# certutil -A -d /etc/dirsrv/slapd-PKI-IPA/ -n 'EXT-CA1' -t CT,C,C -a -i /root/ca1.crt``
| ``# certutil -A -d /etc/dirsrv/slapd-PKI-IPA/ -n 'EXT-CA2' -t CT,C,C -a -i /root/ca2.crt``

-  Add the CA and subCA certs to PKI instance DB

| ``# certutil -A -d  /var/lib/pki-ca/alias/ -n 'EXT-CA1' -t CT,C,C -a -i /root/ca1.crt``
| ``# certutil -A -d  /var/lib/pki-ca/alias/ -n 'EXT-CA2' -t CT,C,C -a -i /root/ca2.crt``

-  Check that the trust flags are correct or fix them if needed in
   /etc/http/alias and /etc/dirsrv/slapd-EXAMPLE-COM:

| ``# certutil -M -d /etc/httpd/alias/ -t CT,C,C -n $ca1alias``
| ``# certutil -M -d /etc/httpd/alias/ -t CT,C,C -n $ca2alias``
| ``# certutil -M -d /etc/dirsrv/slapd-EXAMPLE-COM/ -n $ca1alias -t CT,C,C``
| ``# certutil -M -d /etc/dirsrv/slapd-EXAMPLE-COM/ -n $ca2alias -t CT,C,C``

-  restart the services

``# ipactl restart``

Note: the same procedure must be run on the replicas.

On the clients, you need to run only the following:

| ``# cat /root/ca1.crt /root/ca2.crt >> /etc/ipa/ca.crt``
| ``# certutil -A -d /etc/pki/nssdb/ -n 'EXT-CA1' -t CT,C,C -a -i /root/ca1.crt``
| ``# certutil -A -d /etc/pki/nssdb/ -n 'EXT-CA2' -t CT,C,C -a -i /root/ca2.crt``



Configuration of the 3rd part certificate
----------------------------------------------------------------------------------------------

First we want to create a new PKCS12 archive containing the
aforementioned certificates:

``# openssl pkcs12 -export -chain -CAfile /etc/ipa/ca.crt -in mysite.crt -inkey mysite.key -name MyIPA -out newcert.pk12 -passout pass:some_secret_password``

Once this command has completed, you can install the new bundle using:

::

   | ``# ipa-server-certinstall -w --http_pin=some_secret_password newcert.pk12 ``
   | ``# ipa-server-certinstall -d --dirsrv_pin=some_secret_password newcert.pk12``

Then restart your daemons:

| ``# systemctl restart httpd.service``
| ``# systemctl restart dirsrv@MY-REALM.service``