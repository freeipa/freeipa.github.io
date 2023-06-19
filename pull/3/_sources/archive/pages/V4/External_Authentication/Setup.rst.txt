.. _ipa_server_authentication_with_user_certificate_or_smart_card_setup:

IPA Server Authentication with User Certificate or Smart Card Setup
===================================================================

The ``ipa-experimental-x509-auth-plugin`` enables external
authentication for the FreeIPA server web UI to log in using a
certificate or smart card.

**Warning: This plug-in is experimental. Do not use it in production
environments.**

*Note: This plug-in does not verify if a certificate has been revoked.
Configure the Apache web server to enable revocation checks.*

The procedure has been tested with Red Hat Enterprise Linux 7.3 and
FreeIPA 4.4.

**Warning: Certificate-based authentication for Web UI was integrated
into FreeIPA 4.5.0. The setup described below is not required anymore**

--------------

.. _freeipa_server_configuration:

FreeIPA Server Configuration
----------------------------

-  Red Hat Enterprise Linux 7.3
-  FreeIPA 4.4.0 or newer

--------------

.. _installing_the_web_ui_plug_in:

Installing the Web UI Plug-in
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To to download, install and configure the web UI plug-in, run on your
FreeIPA server:

-  Download the repository file from
   https://copr.fedorainfracloud.org/coprs/g/freeipa/ipa-experimental-x509-auth-plugin/
   and store it in the ``/etc/yum.repos.d/`` directory. For example, for
   Red Hat Enterprise Linux 7.3:

| ``# cd /etc/yum.repos.d/``
| ``# curl -O ``\ ```https://copr.fedorainfracloud.org/coprs/g/freeipa/ipa-experimental-x509-auth-plugin/repo/epel-7/group_freeipa-ipa-experimental-x509-auth-plugin-epel-7.repo`` <https://copr.fedorainfracloud.org/coprs/g/freeipa/ipa-experimental-x509-auth-plugin/repo/epel-7/group_freeipa-ipa-experimental-x509-auth-plugin-epel-7.repo>`__

-  To install the plug-in:

``# yum install -y ipa-experimental-x509-auth-plugin``

   After the package is installed in the
   ``/usr/share/ipa/ui/js/plugins/`` FreeIPA plug-in directory, ``yum``
   automatically enables the ``mod_lookup_identity`` module and
   configures the System Security Services Daemon (SSSD).

-  Set the ``OK-AS-DELEGATE`` flag to the web server's Kerberos service
   principal:

``# ipa service-mod --ok-to-auth-as-delegate=True HTTP/$(hostname)``

   This Kerberos flag enables the service to forward Kerberos tickets.
   Use this flag only with security-reviewed and trusted services.

--------------

.. _example_authentication_using_a_certificate:

Example Authentication Using a Certificate
------------------------------------------

Smart cards use a digitally signed certificated, issued from a public
key infrastructure (PKI) provider to authenticate a user. Authenticating
using a soft token works like smart cards with user certificates.
However, smart cards additionally require a hardware reader and a driver
for the smart card. Follow your smart card provider's documentation, how
to generate the keys and how to add them to the smart card.

.. _authenticating_to_the_web_ui_using_a_freeipa_ca_signed_certificate:

Authenticating to the Web UI Using a FreeIPA CA-signed Certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To create a new account and authenticate to the web UI using a
certificate issued by the FreeIPA certificate authority (CA):

-  Create a ``demo`` user account in FreeIPA with a certificate and
   store the private key in the ``~/demo.key`` file.

   For details, see `Using FreeIPA/Dogtag PKI to Issue User
   Certificates <http://www.freeipa.org/page/V4/User_Certificates#Using_FreeIPA.2FDogtag_PKI_to_issue_user_certificates>`__.

-  Verify that the certificate is displayed in the output of the
   ``ipa user-show`` command:

| ``# ipa user-show demo``
| ``...``
| ``Certificate: MIIDjjCCAnagAwIBAgIBGTANBgkqhkiG.....``

   Alternatively, verify that the certificate is shown in the user's
   account details in the web UI.

-  To download the certificate for the ``demo`` user to the
   ``~/demo_cert.pem`` file, run:

| ``# echo '-----BEGIN CERTIFICATE-----' > ~/demo_cert.pem``
| ``# ipa user-show demo | grep Certificate:\  | cut -d ' ' -f 4 | fold -64 >> ~/demo_cert.pem``
| ``# echo '-----END CERTIFICATE-----' >> ~/demo_cert.pem``

-  Convert the certificate and private key to a PKCS #12-formated
   ``~/demo_cert.pfx`` file:

``# openssl pkcs12 -export -out ~/demo_cert.pfx -inkey ~/demo.key -in ~/demo_cert.pem``

   PKCS #12-formatted files are password protected and store private and
   public keys. Additionally, it can optionally include intermediate and
   root certificates. You can import these files to browsers and web
   servers, such as Apache Tomcat.

-  Import the ``~/demo_cert.pfx`` file to your web browsers certificate
   store. For details, see your web browser's documentation.

You are now able to authenticate to the web UI using the certificate.

--------------

.. _authenticating_to_the_web_ui_using_a_certificate_signed_by_an_external_ca:

Authenticating to the Web UI using a Certificate Signed by an External CA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are running FreeIPA with certificates signed by an external
certificate authority (CA), the certificates are usually stored in a
local NSS database, such as ``~/nssdb/``.

To create a new account and authenticate to the web UI using a
certificate issued by an external CA:

-  Create a new user account:

``# ipa user-add demo --first=Demo --last=Demo``

-  Assign the certificate from the ``~/nssdb/`` NSS database to the
   FreeIPA ``demo`` user account:

``# ipa user-add-cert --certificate="$( certutil -L -d ~/nssdb/ -a -n alpha | grep -v '.---' )" demo``

-  Export the certificate in PKCS #12 format from the ``~/nssdb/`` NSS
   database:

``# pk12util -o demo_cert.pfx -n alpha -d ~/nssdb/``

-  To verify external signed certificates, the Apache web server must
   use the CA certificate:

:\* To export the CA certificate from the ``~/nssdb/`` NSS database:

``# certutil -L -d ~/nssdb/ -a -n cacert > ca_cert.pem``

:\* Import the CA certificate to the Apache web server's certificate
store:

``# certutil -A -n ext_authCA -t CT,C,C  -d /etc/httpd/alias/ -a -i ca_cert.pem``

-  Restart the web server service:

``# systemctl restart httpd``

-  Import the ``~/demo_cert.pfx`` file to your web browsers certificate
   store. For details, see your web browser's documentation.

You are now able to authenticate to the web UI using the certificate.

--------------

.. _verifying_the_web_ui_log_in_using_the_command_line:

Verifying the Web UI Log-in Using the Command Line
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To verify the authentication to the web UI with certificates using the
command line, run:

``# curl --cert demo_cert.pem --key demo.key ``\ ```https://ipaserver/ipa/session/login_x509`` <https://ipaserver/ipa/session/login_x509>`__\ `` -siv``

--------------

.. _developer_notes:

Developer NOTES
---------------

-  Sources: https://github.com/Tiboris/ipa-experimental-x509-auth-plugin

-  Repositories:
   https://copr.fedorainfracloud.org/coprs/tdudlak/ipa-experimental-x509-auth-plugin/

-  For further information about the plug-in infrastructure of the
   FreeIPA web UI, see
   https://pvoborni.fedorapeople.org/doc/#!/guide/Plugins
