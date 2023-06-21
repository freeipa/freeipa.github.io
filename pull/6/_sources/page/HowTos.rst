HowTos
======



Working with FreeIPA
--------------------

-  `Change Directory Manager
   password <Howto/Change_Directory_Manager_Password>`__
-  `Creating
   permissions <https://vda.li/en/posts/2016/08/30/Creating-permissions-in-FreeIPA/>`__
-  [https://fy.blackhats.net.au/blog/html/2015/07/06/FreeIPA:_Giving_permissions_to_service_accounts..html
   Giving permissions to service accounts]
-  `DNS classless IN-ADDR.ARPA
   delegation <Howto/DNS_classless_IN-ADDR.ARPA_delegation>`__ - How to
   delegate reverse zone for e.g. 192.0.2.0/26 and not only /24
-  `DNS updates and zone transfers with
   TSIG <Howto/DNS_updates_and_zone_transfers_with_TSIG>`__
-  `DNS in isolated networks <Howto/DNS_in_isolated_networks>`__ -
   Adding root zone to IPA
-  `DNSSEC <Howto/DNSSEC>`__
-  `IPA (DNS) Locations <Howto/IPA_locations>`__
-  `Updating FreeIPA system DNS records on a remote DNS
   server <Howto/Updating_FreeIPA_system_DNS_records_on_a_remote_DNS_server>`__
-  `Firewall (iptables) rules for common FreeIPA
   server <http://adam.younglogic.com/2013/03/iptables-rules-for-freeipa/>`__
-  `FreeIPA with integrated BIND inside
   chroot <Howto/FreeIPA_with_integrated_BIND_inside_chroot>`__
-  `Delegate DNS zone management to
   users <http://adam.younglogic.com/2012/02/dns-managers-in-freeipa/>`__
-  `Migrating FreeIPA <Howto/Migration>`__ to new machines

   -  `Migrating FreeIPA servers with CA installed prior to
      3.1 <Howto/Dogtag9ToDogtag10Migration>`__

-  `Setting up S4U2Proxy with
   FreeIPA <Howto/Setting_up_S4U2Proxy_with_FreeIPA>`__
-  `Host based access control and
   allow_all <Howto/HBAC_and_allow_all>`__
-  `Inspecting the PAC <Howto/Inspecting_the_PAC>`__
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
-  `Wildcard certificates <Howto/Wildcard_certificates>`__



Interoperability with other systems
-----------------------------------

-  `Create a Trust between FreeIPA and Active
   Directory <Active_Directory_trust_setup>`__
-  `Using third party
   Certificates <Using_3rd_part_certificates_for_HTTP/LDAP>`__
-  `NIS accounts migration preserving
   Passwords <NIS_accounts_migration_preserving_Passwords>`__
-  `Samba 3
   Integration <http://techslaves.org/2011/08/24/freeipa-and-samba-3-integration/>`__
   -- guide involves patching the code!
-  `Adding a KRA to an IPA Installation (proof of
   concept) <Howto/IPAv3_Add_a_KRA>`__ (Partially integrated into
   **FreeIPA 4.2+**, see `Vault <V4/Password_Vault>`__)
-  `How to use FreeIPA in AWS
   EC2 <http://cloud-mechanic.blogspot.com/2013/10/diversion-kerberos-freeipa-in-aws-ec2.html>`__
-  `DokuWiki authLDAP and freeIPA / Enterprise
   IPA <https://www.dokuwiki.org/plugin:authldap:ipa>`__
-  `ISC DHCPd and Dynamic DNS
   updates <Howto/ISC_DHCPd_and_Dynamic_DNS_update>`__
-  `Integration with Apache Synscope Identity Management
   system <http://blog.tirasa.net/unlock-full-freeipa-features.html>`__
-  `Tunelling Kerberos over HTTP on a firewalled
   network <https://www.dragonsreach.it/2014/10/24/kerberos-over-http-on-a-firewalled-network/>`__
-  `PolicyKit integration <Howto/FreeIPA_PolicyKit>`__
-  `HowTo/FreeIPA_on_banana_pi <HowTo/FreeIPA_on_banana_pi>`__

Windows
----------------------------------------------------------------------------------------------

-  `Implementing FreeIPA in a mixed Environment (Windows/Linux) - Step
   by
   step <Implementing_FreeIPA_in_a_mixed_Environment_(Windows/Linux)_-_Step_by_step>`__
-  `Windows authentication against
   FreeIPA <Windows_authentication_against_FreeIPA>`__

UNIX
----------------------------------------------------------------------------------------------

-  `Generic LDAP configuration <HowTo/LDAP>`__
-  `FreeBSD client
   configuration <https://forums.freebsd.org/threads/freebsd-freeipa-via-sssd.46526/>`__
-  `FreeBSD custom binary package
   repository <https://blog-ftweedal.rhcloud.com/2014/10/configuring-freebsd-as-a-freeipa-client/>`__
-  `Configuring Unix Clients (Solaris / HP-UX /
   AIX) <ConfiguringUnixClients>`__
-  `Using FreeIPA for User Authentication on Mac OS X
   10.7/10.8 <http://linsec.ca/Using_FreeIPA_for_User_Authentication#Mac_OS_X_10.7.2F10.8>`__
-  `HowTo/Setup FreeIPA Services for MacOS X
   10.12/10.13 <HowTo/Setup_FreeIPA_Services_for_MacOS_X_10.12/10.13>`__



3rd party Applications Integration
----------------------------------

General
----------------------------------------------------------------------------------------------

-  `FreeIPA demonstration tools <FreeIPA_demonstration_tools>`__
-  `SUDO client
   how-to <https://www.redhat.com/archives/freeipa-users/2013-June/msg00064.html>`__
   (using `SSSD <https://fedorahosted.org/sssd/>`__)
-  `SUDO Integration for RHEL 5.8 <SUDO_Integration_for_RHEL_5.8>`__
-  `SUDO Integration for AIX <SUDO_Integration_for_AIX>`__



Mail Services
----------------------------------------------------------------------------------------------

-  `Dovecot Integration <Dovecot_Integration>`__
-  `Dovecot IMAPS Integration with FreeIPA using Single Sign
   On <Dovecot_IMAPS_Integration_with_FreeIPA_using_Single_Sign_On>`__
-  `Postfix relaying using IPA for Kerberos
   Authentication <https://stomp.colorado.edu/blog/blog/2013/07/09/on-freeipa-postfix-and-a-relaying-smtp-client/>`__
-  `Zimbra Collaboration Server 7.2 Authentication and GAL lookups
   against
   FreeIPA <Zimbra_Collaboration_Server_7.2_Authentication_and_GAL_lookups_against_FreeIPA>`__
-  `Kerberizing PostgreSQL with FreeIPA for
   Keystone <http://adam.younglogic.com/2013/05/kerberizing-postgresql-with-freeipa-for-keystone/>`__
   (see related
   `discussion <http://www.redhat.com/archives/freeipa-devel/2013-September/msg00408.html>`__)



Web Services
----------------------------------------------------------------------------------------------

-  `Setting up MediaWiki to run against
   FreeIPA <Setting_up_MediaWiki_to_run_against_FreeIPA>`__
-  `Apache and Kerberos for Django Authentication +
   Authorization <http://www.roguelynn.com/words/apache-kerberos-for-django/>`__
-  `LDAP authentication for Atlassian JIRA using
   FreeIPA <HowTos/LDAP_authentication_for_Atlassian_JIRA_using_FreeIPA>`__
-  `LDAP authentication for JIRA using
   FreeIPA <https://www.redhat.com/archives/freeipa-users/2015-June/msg00199.html>`__
-  `Using IPA server and sssd for web application's authentication and
   identity
   needs <http://www.freeipa.org/page/Web_App_Authentication>`__
-  `Setting up Redmine to authenticate users against
   FreeIPA <HowTo/Authenticating_Redmine_with_IPA>`__
-  `Setting up Rhodecode to authenticate users against
   FreeIPA <HowTos/Setting_up_Rhodecode_to_authenticate_users_against_FreeIPA>`__
-  `Setting up The Bug Genie to authenticate users against
   FreeIPA <HowTos/Setting_up_The_Bug_Genie_to_authenticate_users_against_FreeIPA>`__



Web Infrastructure
----------------------------------------------------------------------------------------------

-  `Squid Integration with FreeIPA using Single Sign
   On <Squid_Integration_with_FreeIPA_using_Single_Sign_On>`__
-  `FreeIPA behind SSL
   proxy <https://www.adelton.com/freeipa/freeipa-behind-ssl-proxy>`__
-  `FreeIPA behind HTTP proxy with different
   hostname <https://www.adelton.com/freeipa/freeipa-behind-proxy-with-different-name>`__
-  `FreeIPA behind load
   balancer <https://www.adelton.com/freeipa/freeipa-behind-load-balancer>`__



Instant Messaging
----------------------------------------------------------------------------------------------

-  `eJabberd Integration with FreeIPA using LDAP Group
   memberships <eJabberd_Integration_with_FreeIPA_using_LDAP_Group_memberships>`__

Virtualization
----------------------------------------------------------------------------------------------

-  `Authenticating libvirt (with VNC) against
   IPA <Libvirt_with_VNC_Consoles>`__
-  `Setup IPA Server + Client with Vagrant VMs - sample
   Vagrantfile <https://gist.github.com/econchick/99699a6fee2eb44d13b0>`__
-  `vSphere 5 integration <HowTo/vsphere5_integration>`__

OpenShift
^^^^^^^^^

-  `OpenShift Broker and IPA DNS Server with Dynamic Updates with
   GSS-TSIG <OpenShift_Broker_and_IPA_DNS_Server_with_Dynamic_Updates_with_GSS-TSIG>`__
-  `OpenShift Broker Apache + mod_auth_kerb for
   IdM <OpenShift_Broker_Apache_+_mod_auth_kerb_for_IdM>`__
-  `OpenShift Enterprise on top of a trust between IPA/IdM and Windows
   Active
   Directory <OpenShift_Enterprise_on_top_of_a_trust_between_IPA/IdM_and_Windows_Active_Directory>`__

OpenStack
^^^^^^^^^

-  `Keystone integration with IdM
   (FreeIPA) <https://www.rdoproject.org/documentation/keystone-integration-with-idm/>`__

Certificates
----------------------------------------------------------------------------------------------

-  `Lets Encrypt With
   FreeIPA <https://github.com/antevens/letsencrypt-freeipa>`__: Scripts
   to use Let's Encrypt certs with FreeIPA
-  `Implementing SNI on Apache with IPA for certificate management and
   Kerberos Authentication <Apache_SNI_With_Kerberos>`__
-  `Using FreeIPA CA for Puppet <Howto/Using_FreeIPA_CA_for_Puppet>`__
-  `Puppet: Using the FreeIPA PKI
   (outdated) <http://jcape.name/2012/01/16/using-the-freeipa-pki-with-puppet/>`__
-  `Recovering from expired CA subsystem certificates in IPA
   2.x <IPA_2x_Certificate_Renewal>`__
-  `Promoting a self-signed IPA
   CA <Howto/Promoting_a_self-signed_FreeIPA_CA>`__
-  `CA Certificate Renewal <Howto/CA_Certificate_Renewal>`__
-  `Promoting a CA to Renewal and CRL
   Master <Howto/Promote_CA_to_Renewal_and_CRL_Master>`__

-  `Client certificate authentication with
   LDAP <Howto/Client_Certificate_Authentication_with_LDAP>`__

Authentication
----------------------------------------------------------------------------------------------

-  `Creating a binddn for Foreman <Creating_a_binddn_for_Foreman>`__
-  `YubiRadius integration with group-validated FreeIPA Users using
   LDAPS <YubiRadius_integration_with_group-validated_FreeIPA_Users_using_LDAPS>`__
-  `NFS and FreeIPA
   integration <http://wiki.linux-nfs.org/wiki/index.php/NFS_and_FreeIPA>`__
   (at `linux-nfs.org <http://www.linux-nfs.org/>`__)
-  `NFS and FreeIPA
   integration <http://linsec.ca/Using_FreeIPA_for_User_Authentication#Setting_up_Kerberized_NFSv4_Server>`__
   (at `linsec.ca <http://linsec.ca/>`__)
-  `Integration with Okta SSO <HowTo/Integrate_With_Okta>`__
-  `Using FreeIPA and FreeRadius as a RADIUS based software token OTP
   system with CentOS/RedHat
   7 <Using_FreeIPA_and_FreeRadius_as_a_RADIUS_based_software_token_OTP_system_with_CentOS/RedHat_7>`__
-  `FreeRadius and
   FreeIPA <https://www.redhat.com/archives/freeipa-users/2015-December/msg00170.html>`__:
   deployment considerations

   -  `Using mschapv2 with
      FreeIPA <https://fy.blackhats.net.au/blog/html/2016/01/13/FreeRADIUS:_Using_mschapv2_with_freeipa.html>`__

-  `Pulse Secure device
   authentication <https://www.redhat.com/archives/freeipa-users/2016-January/msg00152.html>`__
-  `Using Yubikey 4 Nano to authenticate to FreeIPA enrolled
   host <Using_Yubikey_4_Nano_to_authenticate_to_FreeIPA_enrolled_host>`__

Storage
----------------------------------------------------------------------------------------------

-  `Setup Kerberised NFS server on ONTAP with
   FreeIPA <https://whyistheinternetbroken.wordpress.com/2020/03/24/nfs-kerberos-ontap-freeipa/>`__
-  `NetApp integration in a mixed
   environment <NetApp_integration_in_a_mixed_environment>`__
-  `NexentaStor integration in a mixed
   environment <NexentaStor_integration_in_a_mixed_environment>`__
-  `Integrating a Samba File Server With
   IPA <Howto/Integrating_a_Samba_File_Server_With_IPA>`__
-  `Synology NAS DSM and FreeIPA Setup for Samba, NFS and
   Kerberos <https://blog.cubieserver.de/2018/synology-nas-samba-nfs-and-kerberos-with-freeipa-ldap/>`__
-  `Integrating Dell EMC Unity with
   IPA <Howto/Integrating_Dell_EMC_Unity>`__
-  `Integrating Dell EMC Isilon OneFS with
   IPA <Howto/Integrating_Dell_EMC_Isilon_OneFS>`__



Content Distribution Systems
----------------------------------------------------------------------------------------------

-  `Plan: FreeIPA and OpenShift Enterprise integration with
   Puppet <Plan:_FreeIPA_and_OpenShift_Enterprise_integration_with_Puppet>`__
-  `Using IPA's CA for Puppet <Using_IPA's_CA_for_Puppet>`__

Logging
----------------------------------------------------------------------------------------------

-  `Howto/Centralised Logging with
   Logstash/ElasticSearch/Kibana <Howto/Centralised_Logging_with_Logstash/ElasticSearch/Kibana>`__



Fancy things (user Avatars etc.)
----------------------------------------------------------------------------------------------

-  `Adding Display Pictures/Avatars to Red Hat
   IDM/FreeIPA <https://www.dalemacartney.com/2013/12/05/adding-display-picturesavatars-red-hat-idmfreeipa/>`__
-  `Loading Display Pictures/Avatars from Red Hat IDM/FreeIPA into
   GNOME3 <https://www.dalemacartney.com/2013/12/05/loading-display-picturesavatars-red-hat-idmfreeipa-gnome3/>`__

| 
| \__\_
| How to `add an HowTo on this
  wiki <HowTo/Writing_how_to_documentation_on_the_wiki>`__