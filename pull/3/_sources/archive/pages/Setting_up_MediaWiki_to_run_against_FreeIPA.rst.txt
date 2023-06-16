**Introduction**

--------------

The following documentation is practical guide to set up MediaWiki to
run against FreeIPA. It will show how to secure access to MediaWiki with
Kerberos and SSL in FreeIPA environment.

We suppose that there are one server and several clients configured in
following way:

-  Domain: example.com
-  Realm: EXAMPLE.COM
-  *Server* hostname: master.example.com
-  *Client 0* hostname: client-0.example.com
-  *Client 1* hostname: client-1.example.com etc.
-  All passwords used by FreeIPA: secret123

*Server* runs **freeipa-server** with **Dogtag** and **DNS**. *Client-0*
has **freeipa-client**, **freeipa-admintools**, **MediaWiki**,
**mod_nss** and **mod_auth_kerb**. Other clients run **freeipa-client**
only.

MediaWiki was installed and configured with default settings(it's
located in /var/www/wiki) and its
configuration(/etc/httpd/conf.d/mediawiki.conf) looks like this:

| ``Alias /wiki/skins /usr/share/mediawiki/skins``
| ``Alias /wiki       /var/www/wiki``

**Secure access to MediaWiki by Kerberos**

--------------

We are going to permit access to MediaWiki only to those users who has
valid kerberos ticket. In order to do it we must have **mod_auth_kerb**
installed on *client-0*. Because we have freeipa-admintools installed on
*client-0*, we can get kerberos ticket for Admin and then run all
commands without any need to logging into server.

-  Get kerberos ticket for admin user:

``kinit admin``

-  Create service for MediaWiki:

``ipa service-add HTTP/client-0.example.com``

-  Get keytab for service and make the keytab readable by Apache(httpd):

``ipa-getkeytab -s master.example.com -p HTTP/client-0.example.com -k /etc/httpd/conf/http.keytab``

``chown apache /etc/httpd/conf/http.keytab``

-  Allow access to MediaWiki only to those who has valid kerberos
   ticket. Add these lines into mediawiki.conf:

| ``<Location "/wiki">``
| ``  AuthType Kerberos``
| ``  AuthName "Kerberos Login"``
| ``  KrbMethodNegotiate on``
| ``  KrbMethodK5Passwd off``
| ``  KrbServiceName HTTP``
| ``  KrbAuthRealms EXAMPLE.COM``
| ``  Krb5KeyTab /etc/httpd/conf/http.keytab``
| ``  KrbSaveCredentials off``
| ``  Require valid-user``

.. _automatic_login_of_users_into_mediawiki:

Automatic login of users into MediaWiki
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Additionally we can use one of MediaWiki extensions to automatically log
in every user with valid kerberos ticket. The extension is called
**Auth_remoteuser** and could be downloaded from:

```http://upload.wikimedia.org/ext-dist/Auth_remoteuser-trunk-r85015.tar.gz`` <http://upload.wikimedia.org/ext-dist/Auth_remoteuser-trunk-r85015.tar.gz>`__

-  Extract the archive into directory /var/www/wiki/extensions
-  Edit /var/www/wiki/LocalSettings.php by adding these lines:

| ``require_once('extensions/Auth_remoteuser/Auth_remoteuser.php');``
| ``$wgAuth = new Auth_remoteuser();``

-  Additionally you can add also these lines:

| ``$wgAuthRemoteuserAuthz = true; /* Your own authorization test */``
| ``$wgAuthRemoteuserName = $_SERVER["AUTHENTICATE_CN"]; /* User's name */``
| ``$wgAuthRemoteuserMail = $_SERVER["AUTHENTICATE_MAIL"]; /* User's Mail */``
| ``$wgAuthRemoteuserNotify = false; /* Do not send mail notifications */``
| ``$wgAuthRemoteuserDomain = "EXAMPLE.COM"; /* Remove EXAMPLE.COM\ from the beginning or @EXAMPLE.COM at the end of a IWA username */``
| ``/* User's mail domain to append to the user name to make their email address */``
| ``$wgAuthRemoteuserMailDomain = "example.com";``
| ``// Don't let anonymous people do things...``
| ``$wgGroupPermissions['*']['createaccount']   = false;``
| ``$wgGroupPermissions['*']['read']            = false;``
| ``$wgGroupPermissions['*']['edit']            = false;``

Instructions for installing Auth_remoteuser extension were taken from
`www.mediawiki.org <http://www.mediawiki.org/wiki/Extension:AutomaticREMOTE_USER>`__.

**Configuring SSL**

--------------

Last step is to secure connection to our Wiki by SSL. In order to do
that we must have **mod_nss** installed on *client-0* and server must be
running CA(Dogtag).

-  First of all we must generate new NSS database(by default stored in
   /etc/httpd/alias):

``certutil -N -d /etc/httpd/alias``

-  Save the database password into a file and make it accessible by
   Apache(httpd):

| ``echo "internal:mypassword" > /etc/httpd/alias/passwd.txt``
| ``chown apache /etc/httpd/alias/passwd.txt``
| ``chmod 600 /etc/httpd/alias/passwd.txt``

-  Create a certificate request

``certutil -R -s "CN=client-0.example.com, O=EXAMPLE.COM" -d /etc/httpd/alias -a > mediawiki.csr``

-  Request the certificate from CA and remember its number

``ipa cert-request --principal=HTTP/client-0.example.com mediawiki.csr``

-  Save the certificate into a file

``ipa cert-show #number --out=wikicert.crt``

-  Add the certificate to *client-0* NSS database under the name
   "Https-cert"

``certutil -A -d /etc/httpd/alias/ -n "Https-cert" -a -i wikicert.crt -t ",,"``

-  Edit NSS configuration file, usually stored in
   /etc/httpd/conf.d/nss.conf. Some/all of these settings could already
   be there, so check for duplicity.

| ``Listen 443``
| ``<VirtualHost _default_:443>``
| ``NSSRenegotiation on``
| ``NSSRequireSafeNegotiation on``
| ``NSSEnforceValidCerts off``
| ``NSSNickName "Https-cert"``
| ``NSSPassPhraseDialog "``\ ```file:/etc/httpd/alias/passwd.txt`` <file:/etc/httpd/alias/passwd.txt>`__\ ``"``

-  Add rewrite rules to activate SSL. Following lines must be added into
   MediaWiki configuration file (/etc/httpd/conf.d/mediawiki.conf):

| ``RewriteEngine on``
| ``RewriteCond %{SERVER_PORT}  !^443$``
| ``RewriteCond %{REQUEST_URI}  ^/wiki/``
| ``RewriteRule ^/(.*) https://client-0.example.com/$1 [L,R]``

-  Restart httpd service
