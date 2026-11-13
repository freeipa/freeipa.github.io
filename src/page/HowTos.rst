How-tos
=======



Working with FreeIPA
--------------------

-  :doc:`Change Directory Manager password </page/Howto/Change_Directory_Manager_Password>`
-  `Creating
   permissions <https://vda.li/en/posts/2016/08/30/Creating-permissions-in-FreeIPA/>`__
-  `Giving permissions to service 
   accounts <https://fy.blackhats.net.au/blog/html/2015/07/06/FreeIPA:_Giving_permissions_to_service_accounts.html/>`__
-  :doc:`DNS classless IN-ADDR.ARPA delegation </page/Howto/DNS_classless_IN-ADDR.ARPA_delegation>` - How to
   delegate reverse zone for e.g. 192.0.2.0/26 and not only /24
-  :doc:`DNS updates and zone transfers with TSIG </page/Howto/DNS_updates_and_zone_transfers_with_TSIG>`
-  :doc:`DNS in isolated networks </page/Howto/DNS_in_isolated_networks>` -
   Adding root zone to IPA
-  :doc:`DNSSEC </page/Howto/DNSSEC>`
-  :doc:`IPA (DNS) Locations </page/Howto/IPA_locations>`
-  :doc:`Updating FreeIPA system DNS records on a remote DNS server </page/Howto/Updating_FreeIPA_system_DNS_records_on_a_remote_DNS_server>`
-  `Firewall (iptables) rules for common FreeIPA
   server <http://adam.younglogic.com/2013/03/iptables-rules-for-freeipa/>`__
-  :doc:`FreeIPA with integrated BIND inside chroot </page/Howto/FreeIPA_with_integrated_BIND_inside_chroot>`
-  `Delegate DNS zone management to
   users <http://adam.younglogic.com/2012/02/dns-managers-in-freeipa/>`__
-  :doc:`Migrating FreeIPA </page/Howto/Migration>` to new machines

   -  :doc:`Migrating FreeIPA servers with CA installed prior to 3.1 </page/Howto/Dogtag9ToDogtag10Migration>`

-  :doc:`Setting up S4U2Proxy with FreeIPA </page/Howto/Setting_up_S4U2Proxy_with_FreeIPA>`
-  :doc:`Host based access control and allow_all </page/Howto/HBAC_and_allow_all>`
-  :doc:`Inspecting the PAC </page/Howto/Inspecting_the_PAC>`
-  `Hardening FreeIPA for servers exposed on the
   Internet <https://www.redhat.com/archives/freeipa-users/2014-April/msg00243.html>`__
-  `Let's Encrypt certificate installation for FreeIPA web
   UI <https://github.com/freeipa/freeipa-letsencrypt>`__
-  `Using FreeIPA JSON
   API <https://www.redhat.com/archives/freeipa-users/2015-November/msg00132.html>`__:
   `example with
   Perl <https://www.redhat.com/archives/freeipa-users/2015-November/msg00132.html>`__
   ; `supported API
   commands <https://git.fedorahosted.org/cgit/freeipa.git/tree/API.txt>`__
-  `Lightweight Sub-CAs in FreeIPA
   4.4 <http://blog-ftweedal.rhcloud.com/2016/07/lightweight-sub-cas-in-freeipa-4-4/>`__
-  `Configuring IPA's OCSP to use with httpd and
   mod_nss <http://akasurde.github.io/ocsp-mod-nss-httpd-centos.html#ocsp-mod-nss-httpd-centos>`__
-  :doc:`Wildcard certificates </page/Howto/Wildcard_certificates>`
-  :doc:`Recover from a deleted admin user </page/Howto/Recover_from_a_deleted_admin_user>`



Interoperability with other systems
-----------------------------------

-  :doc:`Create a Trust between FreeIPA and Active Directory </page/Active_Directory_trust_setup>`
-  :doc:`Using third party Certificates </page/Using_3rd_part_certificates_for_HTTP/LDAP>`
-  :doc:`NIS accounts migration preserving Passwords </page/NIS_accounts_migration_preserving_Passwords>`
-  `Samba 3
   Integration <http://techslaves.org/2011/08/24/freeipa-and-samba-3-integration/>`__
   -- guide involves patching the code!
-  :doc:`Adding a KRA to an IPA Installation (proof of concept) </page/Howto/IPAv3_Add_a_KRA>` (Partially integrated into
   **FreeIPA 4.2+**, see :doc:`Vault </page/V4/Password_Vault>`)
-  `How to use FreeIPA in AWS
   EC2 <http://cloud-mechanic.blogspot.com/2013/10/diversion-kerberos-freeipa-in-aws-ec2.html>`__
-  `DokuWiki authLDAP and freeIPA / Enterprise
   IPA <https://www.dokuwiki.org/plugin:authldap:ipa>`__
-  :doc:`ISC DHCPd and Dynamic DNS updates </page/Howto/ISC_DHCPd_and_Dynamic_DNS_update>`
-  `Integration with Apache Synscope Identity Management
   system <http://blog.tirasa.net/unlock-full-freeipa-features.html>`__
-  `google2ipa <https://github.com/neverlless/google2ipa>`__ - Sync
   Google Workspace users and groups into FreeIPA: provisioning with
   one-time passwords, Google group to IPA group mapping, and offboarding
   (lock, then delayed preserve-delete). Uses the FreeIPA JSON API with a
   least-privilege role, no schema changes.
-  `Tunelling Kerberos over HTTP on a firewalled
   network <https://www.dragonsreach.it/2014/10/24/kerberos-over-http-on-a-firewalled-network/>`__
-  :doc:`PolicyKit integration </page/Howto/FreeIPA_PolicyKit>`
-  :doc:`FreeIPA on Banana Pi </page/HowTo/FreeIPA_on_banana_pi>`

Windows
----------------------------------------------------------------------------------------------

-  :doc:`Implementing FreeIPA in a mixed Environment (Windows/Linux) - Step by step </page/Implementing_FreeIPA_in_a_mixed_Environment_Windows-Linux_-_Step_by_step>`
-  :doc:`Windows authentication against FreeIPA </page/Windows_authentication_against_FreeIPA>`

UNIX
----------------------------------------------------------------------------------------------

-  :doc:`Generic LDAP configuration </page/HowTo/LDAP>`
-  `FreeBSD client
   configuration <https://forums.freebsd.org/threads/freebsd-freeipa-via-sssd.46526/>`__
-  `FreeBSD custom binary package
   repository <https://blog-ftweedal.rhcloud.com/2014/10/configuring-freebsd-as-a-freeipa-client/>`__
-  :doc:`Configuring IBM AIX Clients </page/ConfiguringAixClients>`
-  :doc:`Configuring Unix Clients (Solaris / HP-UX / AIX) </page/ConfiguringUnixClients>`
-  `Using FreeIPA for User Authentication on Mac OS X
   10.7/10.8 <http://linsec.ca/Using_FreeIPA_for_User_Authentication#Mac_OS_X_10.7.2F10.8>`__
-  :doc:`Setup FreeIPA Services for MacOS X 10.12/10.13 </page/HowTo/Setup_FreeIPA_Services_for_MacOS_X_10.12/10.13>`



3rd party Applications Integration
----------------------------------

General
----------------------------------------------------------------------------------------------

-  :doc:`FreeIPA demonstration tools </page/FreeIPA_demonstration_tools>`
-  `SUDO client
   how-to <https://www.redhat.com/archives/freeipa-users/2013-June/msg00064.html>`__
   (using `SSSD <https://fedorahosted.org/sssd/>`__)
-  SUDO Integration for RHEL 5.8 (historical article unavailable)
-  :doc:`SUDO Integration for AIX </page/SUDO_Integration_for_AIX>`



Mail Services
----------------------------------------------------------------------------------------------

-  :doc:`Dovecot Integration </page/Dovecot_Integration>`
-  :doc:`Dovecot IMAPS Integration with FreeIPA using Single Sign On </page/Dovecot_IMAPS_Integration_with_FreeIPA_using_Single_Sign_On>`
-  `Postfix relaying using IPA for Kerberos
   Authentication <https://stomp.colorado.edu/blog/blog/2013/07/09/on-freeipa-postfix-and-a-relaying-smtp-client/>`__
-  :doc:`Zimbra Collaboration Server 7.2 Authentication and GAL lookups against FreeIPA </page/Zimbra_Collaboration_Server_7.2_Authentication_and_GAL_lookups_against_FreeIPA>`
-  `Kerberizing PostgreSQL with FreeIPA for
   Keystone <http://adam.younglogic.com/2013/05/kerberizing-postgresql-with-freeipa-for-keystone/>`__
   (see related
   `discussion <http://www.redhat.com/archives/freeipa-devel/2013-September/msg00408.html>`__)



Web Services
----------------------------------------------------------------------------------------------

-  :doc:`Setting up MediaWiki to run against FreeIPA </page/Setting_up_MediaWiki_to_run_against_FreeIPA>`
-  `Apache and Kerberos for Django Authentication +
   Authorization <http://www.roguelynn.com/words/apache-kerberos-for-django/>`__
-  :doc:`LDAP authentication for Atlassian JIRA using FreeIPA </page/HowTos/LDAP_authentication_for_Atlassian_JIRA_using_FreeIPA>`
-  `LDAP authentication for JIRA using
   FreeIPA <https://www.redhat.com/archives/freeipa-users/2015-June/msg00199.html>`__
-  :doc:`Using IPA server and sssd for web application's authentication and identity needs </page/Web_App_Authentication>`
-  :doc:`Setting up Redmine to authenticate users against FreeIPA </page/HowTo/Authenticating_Redmine_with_IPA>`
-  :doc:`Setting up Rhodecode to authenticate users against FreeIPA </page/HowTos/Setting_up_Rhodecode_to_authenticate_users_against_FreeIPA>`
-  :doc:`Setting up The Bug Genie to authenticate users against FreeIPA </page/HowTos/Setting_up_The_Bug_Genie_to_authenticate_users_against_FreeIPA>`



Web Infrastructure
----------------------------------------------------------------------------------------------

-  :doc:`Squid Integration with FreeIPA using Single Sign On </page/Squid_Integration_with_FreeIPA_using_Single_Sign_On>`
-  `FreeIPA behind SSL
   proxy <https://www.adelton.com/freeipa/freeipa-behind-ssl-proxy>`__
-  `FreeIPA behind HTTP proxy with different
   hostname <https://www.adelton.com/freeipa/freeipa-behind-proxy-with-different-name>`__
-  `FreeIPA behind load
   balancer <https://www.adelton.com/freeipa/freeipa-behind-load-balancer>`__



Instant Messaging
----------------------------------------------------------------------------------------------

-  :doc:`eJabberd Integration with FreeIPA using LDAP Group memberships </page/EJabberd_Integration_with_FreeIPA_using_LDAP_Group_memberships>`

Virtualization
----------------------------------------------------------------------------------------------

-  :doc:`Authenticating libvirt (with VNC) against IPA </page/Libvirt_with_VNC_Consoles>`
-  `Setup IPA Server + Client with Vagrant VMs - sample
   Vagrantfile <https://gist.github.com/econchick/99699a6fee2eb44d13b0>`__
-  :doc:`vSphere 5 integration </page/HowTo/vsphere5_integration>`

OpenShift
^^^^^^^^^

-  :doc:`OpenShift Broker and IPA DNS Server with Dynamic Updates with GSS-TSIG </page/OpenShift_Broker_and_IPA_DNS_Server_with_Dynamic_Updates_with_GSS-TSIG>`
-  :doc:`OpenShift Broker Apache + mod_auth_kerb for IdM </page/OpenShift_Broker_Apache_+_mod_auth_kerb_for_IdM>`
-  :doc:`OpenShift Enterprise on top of a trust between IPA/IdM and Windows Active Directory </page/OpenShift_Enterprise_on_top_of_a_trust_between_IPA/IdM_and_Windows_Active_Directory>`

OpenStack
^^^^^^^^^

-  `Keystone integration with IdM
   (FreeIPA) <https://www.rdoproject.org/documentation/keystone-integration-with-idm/>`__

Certificates
----------------------------------------------------------------------------------------------

-  `Lets Encrypt With
   FreeIPA <https://github.com/antevens/letsencrypt-freeipa>`__: Scripts
   to use Let's Encrypt certs with FreeIPA
-  :doc:`Implementing SNI on Apache with IPA for certificate management and Kerberos Authentication </page/Apache_SNI_With_Kerberos>`
-  :doc:`Using FreeIPA CA for Puppet </page/Howto/Using_FreeIPA_CA_for_Puppet>`
-  `Puppet: Using the FreeIPA PKI
   (outdated) <http://jcape.name/2012/01/16/using-the-freeipa-pki-with-puppet/>`__
-  :doc:`Recovering from expired CA subsystem certificates in IPA 2.x </page/IPA_2x_Certificate_Renewal>`
-  :doc:`Promoting a self-signed IPA CA </page/Howto/Promoting_a_self-signed_FreeIPA_CA>`
-  :doc:`CA Certificate Renewal </page/Howto/CA_Certificate_Renewal>`
-  :doc:`Promoting a CA to Renewal and CRL Master </page/Howto/Promote_CA_to_Renewal_and_CRL_Master>`

-  :doc:`Client certificate authentication with LDAP </page/Howto/Client_Certificate_Authentication_with_LDAP>`
-  `Using FreeIPA CA as an ACME provider for
   cert-manager <https://www.hamzahkhan.com/using-freeipa-ca-as-an-acme-provider-for-cert-manager>`__:
   Issue Kubernetes certificates from the IPA ACME service, with DNS-01
   challenges answered through IPA DNS dynamic updates (RFC 2136)

Authentication
----------------------------------------------------------------------------------------------

-  :doc:`Creating a binddn for Foreman </page/Creating_a_binddn_for_Foreman>`
-  :doc:`YubiRadius integration with group-validated FreeIPA Users using LDAPS </page/YubiRadius_integration_with_group-validated_FreeIPA_Users_using_LDAPS>`
-  `NFS and FreeIPA
   integration <http://wiki.linux-nfs.org/wiki/index.php/NFS_and_FreeIPA>`__
   (at `linux-nfs.org <http://www.linux-nfs.org/>`__)
-  `NFS and FreeIPA
   integration <http://linsec.ca/Using_FreeIPA_for_User_Authentication#Setting_up_Kerberized_NFSv4_Server>`__
   (at `linsec.ca <http://linsec.ca/>`__)
-  :doc:`Integration with Okta SSO </page/HowTo/Integrate_With_Okta>`
-  :doc:`Using FreeIPA and FreeRadius as a RADIUS based software token OTP system with CentOS/RedHat 7 </page/Using_FreeIPA_and_FreeRadius_as_a_RADIUS_based_software_token_OTP_system_with_CentOS/RedHat_7>`
-  `FreeRadius and
   FreeIPA <https://www.redhat.com/archives/freeipa-users/2015-December/msg00170.html>`__:
   deployment considerations

   -  `Using mschapv2 with
      FreeIPA <https://fy.blackhats.net.au/blog/html/2016/01/13/FreeRADIUS:_Using_mschapv2_with_freeipa.html>`__

-  `Pulse Secure device
   authentication <https://www.redhat.com/archives/freeipa-users/2016-January/msg00152.html>`__
-  :doc:`Using Yubikey 4 Nano to authenticate to FreeIPA enrolled host </page/Using_Yubikey_4_Nano_to_authenticate_to_FreeIPA_enrolled_host>`

Storage
----------------------------------------------------------------------------------------------

-  `Setup Kerberised NFS server on ONTAP with
   FreeIPA <https://whyistheinternetbroken.wordpress.com/2020/03/24/nfs-kerberos-ontap-freeipa/>`__
-  :doc:`NetApp integration in a mixed environment </page/NetApp_integration_in_a_mixed_environment>`
-  :doc:`NexentaStor integration in a mixed environment </page/NexentaStor_integration_in_a_mixed_environment>`
-  :doc:`Integrating a Samba File Server With IPA </page/Howto/Integrating_a_Samba_File_Server_With_IPA>`
-  `Synology NAS DSM and FreeIPA Setup for Samba, NFS and
   Kerberos <https://blog.cubieserver.de/2018/synology-nas-samba-nfs-and-kerberos-with-freeipa-ldap/>`__
-  :doc:`Integrating Dell EMC Unity with IPA </page/Howto/Integrating_Dell_EMC_Unity>`
-  :doc:`Integrating Dell EMC Isilon OneFS with IPA </page/Howto/Integrating_Dell_EMC_Isilon_OneFS>`



Content Distribution Systems
----------------------------------------------------------------------------------------------

-  :doc:`Plan: FreeIPA and OpenShift Enterprise integration with Puppet </page/Plan:_FreeIPA_and_OpenShift_Enterprise_integration_with_Puppet>`
-  :doc:`Using IPA's CA for Puppet </page/Howto/Using_FreeIPA_CA_for_Puppet>`

Logging
----------------------------------------------------------------------------------------------

-  :doc:`Centralised Logging with Logstash/ElasticSearch/Kibana </page/Howto/Centralised_Logging_with_Logstash/ElasticSearch/Kibana>`



Fancy things (user Avatars etc.)
----------------------------------------------------------------------------------------------

-  `Adding Display Pictures/Avatars to Red Hat
   IDM/FreeIPA <https://www.dalemacartney.com/2013/12/05/adding-display-picturesavatars-red-hat-idmfreeipa/>`__
-  `Loading Display Pictures/Avatars from Red Hat IDM/FreeIPA into
   GNOME3 <https://www.dalemacartney.com/2013/12/05/loading-display-picturesavatars-red-hat-idmfreeipa-gnome3/>`__

See :doc:`how to contribute a how-to </page/HowTo/Writing_how_to_documentation_on_the_wiki>`.
