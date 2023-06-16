Description
===========

This page explains how to setup and configure a KRA to an IPA server
installed with a `Dogtag <http://pki.fedoraproject.org>`__ CA. A KRA is
used for key escrow and recovery. This is also referred to as the Data
Recovery Manager (DRM).

**NOTE**: This is being developed as a Proof-of-Concept.

Prerequisites
=============

-  Fedora 18
-  ipa-server 3.1.4
-  pki-core 10.0.3

This procedure is only tested to work with the above versions

.. _install_and_configure_ipa_server:

Install and configure IPA server
================================

.. _make_sure_all_packages_are_up_to_date:

Make sure all packages are up to date
-------------------------------------

``# yum update -y``

.. _install_required_packages:

Install required packages
-------------------------

``# yum install -y freeipa-server bind bind-dyndb-ldap pki-kra``

.. _install_ipa_server:

Install IPA server
------------------

``# ipa-server-install -a mypassword1 -p mypassword2 --domain=``\ *``ipa_domain``*\ `` --realm=``\ *``IPA_DOMAIN``*\ `` --setup-dns --no-forwarders -U``

**NOTE**: This configures IPA with its own DNS server. This is not an
absolute requirement.

.. _install_the_kra:

Install the KRA
===============

.. _create_kra_installation_configuration_file:

Create KRA installation configuration file
------------------------------------------

Create a file with these contents somewhere in your filesystem. It is
only needed during installation of the KRA.

Replace the password, host name and realm names as appropriate to your
installation.

::

   [KRA]
   pki_security_domain_https_port=443
   pki_security_domain_password= mypassword2
   pki_security_domain_user=admin
   pki_enable_proxy = True
   pki_restart_configured_instance = False
   pki_backup_keys = True
   pki_backup_password = mypassword2
   pki_client_database_dir = /tmp/tmp-ce2oQN
   pki_client_database_password = mypassword2
   pki_client_database_purge = False
   pki_client_pkcs12_password = mypassword2
   pki_admin_name = admin
   pki_admin_uid = admin
   pki_admin_email = root@localhost
   pki_admin_password = mypassword2
   pki_admin_nickname = ipa-ca-agent
   pki_admin_subject_dn = cn=ipa-ca-agent,O=EXAMPLE.COM
   pki_import_admin_cert=True
   pki_admin_cert_file=/root/.dogtag/pki-tomcat/ca_admin.cert
   pki_client_admin_cert_p12 = /root/ca-agent.p12
   pki_ds_ldap_port = 389
   pki_ds_password = mypassword2
   pki_ds_base_dn = o=ipakra
   pki_ds_database = ipakra
   pki_storage_subject_dn=cn=DRM Storage Certificate,o=EXAMPLE.COM
   pki_transport_subject_dn=cn=DRM Transport Certificate,o=EXAMPLE.COM
   pki_subsystem_subject_dn = cn=DRM Subsystem,O=EXAMPLE.COM
   pki_ssl_server_subject_dn = cn=ipa.example.com,O=EXAMPLE.COM
   pki_audit_signing_subject_dn = cn=DRM Audit,O=EXAMPLE.COM
   pki_subsystem_nickname = subsystemCert cert-pki-kra
   pki_ssl_server_nickname = Server-Cert cert-pki-ca
   pki_audit_signing_nickname = auditSigningCert cert-pki-kra
   pki_storage_nickname=storageCert cert-pki-kra
   pki_transport_nickname=transportCert cert-pki-kra

.. _update_ipa_proxy_configuration:

Update IPA proxy configuration
------------------------------

Replace /etc/httpd/conf.d/ipa-pki-proxy.conf with this:

::

   # VERSION 3 - DO NOT REMOVE THIS LINE

   ProxyRequests Off

   # matches for ee port
   <LocationMatch "^/ca/ee/ca/checkRequest|^/ca/ee/ca/getCertChain|^/ca/ee/ca/getTokenInfo|^/ca/ee/ca/profileSubmit|^/ca/ee/ca/tokenAuthenticate|^/ca/ocsp|^/ca/ee/ca/updateNumberRange|^/ca/ee/ca/getCRL">
       NSSOptions +StdEnvVars +ExportCertData +StrictRequire +OptRenegotiate
       NSSVerifyClient none
       ProxyPassMatch ajp://localhost:8009
       ProxyPassReverse ajp://localhost:8009
   </LocationMatch>

   # matches for admin port and installer
   <LocationMatch "^/ca/admin/ca/getCertChain|^/ca/admin/ca/getConfigEntries|^/ca/admin/ca/getCookie|^/ca/admin/ca/getStatus|^/ca/admin/ca/getSubsystemCert|^/ca/admin/ca/securityDomainLogin|^/ca/admin/ca/getDomainXML|^/ca/rest/installer/installToken|^/ca/admin/ca/updateNumberRange|^/ca/rest/securityDomain/domainInfo|^/ca/rest/account/login|^/ca/admin/ca/tokenAuthenticate|^/ca/admin/ca/updateConnector|^/ca/admin/ca/updateNumberRange|^/ca/admin/ca/updateDomainXML|^/ca/rest/account/logout|^/ca/rest/securityDomain/installToken">
       NSSOptions +StdEnvVars +ExportCertData +StrictRequire +OptRenegotiate
       NSSVerifyClient none
       ProxyPassMatch ajp://localhost:8009
       ProxyPassReverse ajp://localhost:8009
   </LocationMatch>

   # matches for agent port and eeca port
   <LocationMatch "^/ca/agent/ca/displayBySerial|^/ca/agent/ca/doRevoke|^/ca/agent/ca/doUnrevoke|^/ca/agent/ca/updateDomainXML|^/ca/eeca/ca/profileSubmitSSLClient">
       NSSOptions +StdEnvVars +ExportCertData +StrictRequire +OptRenegotiate
       NSSVerifyClient require
       ProxyPassMatch ajp://localhost:8009
       ProxyPassReverse ajp://localhost:8009
   </LocationMatch>

   # Only enable this on servers that are not generating a CRL
   #RewriteRule ^/ipa/crl/MasterCRL.bin https://dart.greyoak.com/ca/ee/ca/getCRL?op=getCRL&crlIssuingPoint=MasterCRL [L,R=301,NC]

   <LocationMatch "^/kra/agent/kra/connector">
       NSSOptions +StdEnvVars +ExportCertData +StrictRequire +OptRenegotiate
       NSSVerifyClient require
       ProxyPassMatch ajp://localhost:8009
       ProxyPassReverse ajp://localhost:8009
   </LocationMatch>

.. _restart_httpd:

Restart httpd
-------------

``# systemctl restart httpd.service``

.. _add_the_kra:

Add the KRA
-----------

``# pkispawn -s KRA -f /path/to/kra.cfg``

.. _restart_tomcat:

Restart Tomcat
==============

``# systemctl restart pki-tomcatd@pki-tomcat.service``

.. _configure_a_browser_for_kra_adminisrtrative_work:

Configure a browser for KRA adminisrtrative work
================================================

.. _copy_the_agent_pkcs12_file:

Copy the Agent PKCS#12 file
---------------------------

The PKCS#12 file that contains the IPA RA agent that we'll use to do the
KRA work is in /root/ca-agent.p12. Copy this to your client machine, or
to a location on the server that is readable by the user you want to run
Firefox. Fix permissions as needed.

| ``# cp /root/ca-agent.p12 /home/someuser``
| ``# chown someuser /home/someuser/ca-agent.p12``

.. _import_the_cert:

Import the cert
---------------

Start Firefox and select Edit -> Preferences -> Advanced -> Encryption
-> View Certificates

select Import

Enter the path to ca-agent.p12

Enter the PKCS#12 password (the Directory manager password, mypassword2
in the example)

.. _test_the_cert:

Test the cert
-------------

We will be using the CA directly as opposed to going through the IPA
GUI.

Browse to https://ipa.example.com:8443/

You may be prompted to trust the CA. You can import it directly by
instead by going to http://ipa.example.com/ipa/config/ca.crt

Select Agent Services and you should be prompted to select a client
certificate to use. If you imported the certificate correctly then
selecting it and clicking Ok should display the CA agent page.

.. _issue_and_recover_a_certificate:

Issue and Recover a Certificate
-------------------------------

There are further instructions for testing the KRA at
https://access.redhat.com/site/documentation/en-US/Red_Hat_Certificate_System/8.1/html/Admin_Guide/Testing_the_Key_Archival_and_Recovery_Setup.html
