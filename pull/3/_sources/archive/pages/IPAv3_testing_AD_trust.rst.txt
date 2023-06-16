See up-to-date article `Active Directory trust
setup <Active_Directory_trust_setup>`__.

.. _testing_trust_between_freeipa_and_ad:

Testing trust between FreeIPA and AD
====================================

Please note that this page is in a **draft** stage and will change
regularly as new features are added to FreeIPA.

**NOTE: An updated howtois now
available**\ `here <Active_Directory_trust_setup>`__

.. _install_freeipa:

Install FreeIPA
---------------

The easiest way is to
`configure <http://docs.fedoraproject.org/en-US/Fedora/12/html/Deployment_Guide/sec-Configuring_Yum_and_Yum_Repositories.html>`__
`the freeipa nightly devel
repository <http://jdennis.fedorapeople.org/ipa-devel/>`__ on a fully
updated Fedora 17 box. Please note that this repository will always
contain the very latest bits but may also be more unstable.

For Fedora 18 you need to enable updates and updates-testing
repositories and you \*don't\* need to use nightly devel repository
since everything is available in Fedora 18 already.

List of required package versions:

-  389-ds-base: at least 1.2.11.5. Earlier versions do not work with
   FreeIPA.
-  samba4: For FreeIPAv3 beta2 at least samba4 4.0.0beta2 is required,
   built with --without-ad-dc and --with-system-mitkrb5 options. Samba4
   4.0.0beta2 has changed an API that FreeIPA cross-realm trust support
   is relying on, thus the requirement. Any samba4 versions before
   samba4 4.0.0beta1 also do not have proper MIT krb5 support.
-  If SELinux is in use, policy equal or greater then version
   3.10.0-134.fc17 should be used.
-  MIT krb5 1.10 is required (krb5 1.10.2 or later is required)

Run ipa-server-install with --setup-dns and you favourite options to
setup an FreeIPA server. It is preferred to use the DNS server of
FreeIPA, otherwise a couple of settings must be added manually to the
external DNS server.

If there is a DNS server which can route DNS traffic between the FreeIPA
and AD domain this sould be used as forwarder with the option
'--forwarder=ip.address'. If there is no such server alternatives are
discussed in the following section.

Please do not forget to adjust the firewall settings as recommended by
ipa-server-install.

.. _setting_up_active_directory_domain_for_testing_purposes:

Setting up Active Directory domain for testing purposes
-------------------------------------------------------

Please follow article `Setting up Active Directory domain for testing
purposes <Setting_up_Active_Directory_domain_for_testing_purposes>`__

.. _check_dns:

Check DNS
---------

Is is important the both domains, FreeIPA and AD, can resolve names and
especially service records of the other domain. Please add the needed
configurations and test e.g. with

-  dig SRV \_ldap._tcp.ipa.domain
-  dig SRV \_ldap._tcp.ad.domain

If DNS fails against the AD domain you can try to add a special
configuration to your named.conf.

.. _adding_a_conditional_forwarder_on_the_freeipa_side:

Adding a conditional forwarder on the FreeIPA side
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If FreeIPA is configured with DNS, you can use following command:

   ipa dnszone-add ad.domain --name-server=$AD_DC
   --admin-email="hostmaster@ad.domain" --force --forwarder=$AD_DC_IP
   --forward-policy=only

alternatively you can define it manually in named.conf:

| `` zone "ad.domain" IN {``
| ``   type forward;``
| ``   forwarders { ip-address.of-one-or-more.ad.domain_controllers; };``
| ``   forward only;``
| `` };``

As result, the FreeIPA DNS server will send all DNS request for the
domain ad.domain to the specified forwarders.

.. _adding_a_conditional_forwarder_on_the_ad_side:

Adding a conditional forwarder on the AD side
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If DNS resolution does not work on the AS side, e.g. 'ping
server.ipa.domain' does not work on the command prompt a conditional
forwarder can be added to the AD DNS.

-  Open Start->Administrative Tools->DNS
-  make a right-click on 'Conditional Forwarders' in the left column of
   the window
-  select 'New Conditional Forwarder...'
-  add the DNS domain name of your FreeIPA domain name and the IP
   adresses of one or more DNS servers of your FreeIPA domain

You can also check out `this
video <http://people.redhat.com/ssorce/freeipa/ad-dns-forwarder.webm>`__
on how to set up the forwarders via the Windows GUI.

To test the new configuration you can try to ping your FreeIPA server
again. It might be necessary to call 'ipconfig /flushdns' to removed any
cached results.

Alternatively you can use command line utility dnscmd to configure the
forwarder:

-  Open Start -> Command Prompt
-  Enter: dnscmd 127.0.0.1 /ZoneAdd FreeIPA.Domain /Forwarder
   IP.AD.DR.ESS

The command should report that zone FreeIPA.Domain was successfully
added.

.. _adding_a_common_forwarder:

Adding a common forwarder
~~~~~~~~~~~~~~~~~~~~~~~~~

As an alternative to add conditional forwarders to the configuration of
the FreeIPA and AD servers a common forwarder can be used which routes
the DNS request between the different domains. If there are more than
two domains this might be the preferred solution because otherwise each
domain controller needs to know the DNS servers of all other domains.

One way to create a common forwarder is to run dnsmasq on a separate
server/VM like

`` dnsmasq --no-dhcp-interface=eth0,eth1 --server=/ipa.domain/10.16.10.10 - -server=/ad.domain/192.168.10.10``

DHCP is not needed so all interfaces on the server should be listed in
'--no-dhcp-interface'. If you want to monitor the DNS traffic you can
add '-d -q' so that dnsmasq runs in the foreground and prints debugging
messages.

This server/VM now can be used as the default forwarder on the FreeIPA
and AD servers.

.. _prepare_freeipa_server_for_trusts:

Prepare FreeIPA server for trusts
---------------------------------

Please note that currently SELinux denies winbind to access the Kerberos
credential file of the smbd
(https://bugzilla.redhat.com/show_bug.cgi?id=828619). Until there is an
updated SELinux policy available SELinux should be set into the
permissive more by calling 'setenforce 0'.

Call 'kinit admin' and call 'ipa-adtrust-install -p
directory_manager_password' to create all the needed objects in the
FreeIPA server.

Please set 'dns_lookup_kdc = true' in /etc/krb5.conf
(https://fedorahosted.org/freeipa/ticket/2515).

ipa-adtrust-install configures new services and extended operations on
the FreeIPA directory server. To make sure everything is started
properly you can call 'ipactl restart'.

Now you should call

-  'kinit admin' again, because the new TGT will now contain the PAC
   data
-  the following two steps are not needed if FreeIPA 3.0beta2 or a newer
   version is used:

   -  'ipa passwd admin', to create the needed data so that the admin
      account can be use in NTLM authentication
   -  if the password change fails with 'Constraint violation: Too soon
      to change password' you can wait one hour or change the password
      policy with 'ipa pwpolicy-mod --minlife=0'

.. _some_sanity_checks:

Some sanity checks
~~~~~~~~~~~~~~~~~~

The following commands can be used to check that smbd and winbindd are
basically working:

-  'smbclient -L server.ipa-domain -k'
-  'wbinfo --online-status'

.. _populating_ipantsecurityidentifier_sid_for_existing_users_and_groups:

Populating ipaNTSecurityIdentifier (SID) for existing users and groups
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After running ipa-adtrust-install new users and groups will
automatically get the needed attributes and objectclasses to be used
with trust. This mainly means the SID which is stored in the
ipaNTSecurityIdentifier LDAP attribute.

If an existing IPA installation is upgrade a SID must be assigned to
existing users and groups. A directory server task was added for this
purpose. Since this task can cause some replication traffic in setups
with multiple IPA servers and many users and groups, is is not run
automatically during the update or while running ipa-adtrust-install. To
start the task the following LDIF file

| `` dn: cn=sidgen,cn=ipa-sidgen-task,cn=tasks,cn=config``
| `` changetype: add``
| `` objectClass: top``
| `` objectClass: extensibleObject``
| `` cn: sidgen``
| `` nsslapd-basedn: dc=YOUR,dc=BASEDN ``
| `` delay: 0``

must be loaded with

   ldapmodify -H ldapi://%2fvar%2frun%2fslapd-YOUR-REALM.socket -f
   ipa-sidgen-task-start.ldif

as root or with directory manager credentials. There are two parameters,
nsslapd-basedn sould be set to your base DN. delay is the time between
two modifications in nano seconds. It can be used to spread the
replication traffic over a longer period of time.

In the logs a message like

   sidgen_task_thread - [file ipa_sidgen_task.c, line 191]: Sidgen task
   starts ...

is shown when the task starts and

   sidgen_task_thread - [file ipa_sidgen_task.c, line 196]: Sidgen task
   finished [0].

when the task finished successfully.

.. _create_a_trust_to_an_ad_domain:

Create a trust to an AD domain
------------------------------

Currently it is not possible to create a trust from the AD side, because
AD expect a directory server with an AD layout on the other side. We
have to investigate further what can be done to let AD create a trust
with FreeIPA. But since there are no plans to create a directory
structure similar to AD it might be possible that a trust can only be
created from the FreeIPA side.

The following command can be used to create a trust:

   ipa trust-add --type=ad ad.domain --admin Administrator --password

You will be asked for the password of the AD Administrator.

Check out `this
video <http://people.redhat.com/ssorce/freeipa/setup-ad-ipa-trust.webm>`__
to see the whole process of setting up a trust, incuding a quick test
that it is working (Note: the video was taken before we changed the
command format from "ipa trust-add-ad" to "ipa trust-add --type=ad", the
latter is the correct command now).

.. _testing_cross_realm_kerberos_configuration:

Testing cross-realm Kerberos configuration
------------------------------------------

If you request a TGT for a FreeIPA user with

   kinit ipauser@IPA.TEST

You should be able to request service tickets for services form the
FreeIPA domain:

   kvno host/other-host.ipa.test@IPA.TEST

and also for services from the AD domain

   kvno cifs/ad-dom-member.ad.test@AD.TEST

If you successful request a service ticket from the AD domain you should
also find a cross-realm TGT 'krbtgt/AD.TEST@IPA.TEST'.

.. _validating_the_trust_from_the_windows_side:

Validating the trust from the Windows side
------------------------------------------

With recent versions of the samba4 package (newer than 2011-12-21) it is
possible to validate the trust from the windows side. To do this open
the 'Active Directory Domains and Trusts' tool. Open the Properties of
your local domain, jump to the Trust tab and open the properties of the
FreeIPA trusted domain. Now you can hit the validate button. After a few
second you will be asked if you want to validate the incoming trust as
well. For the you have to use the admin user and must provide the admin
password.

.. _configure_ipa_client:

Configure IPA client
--------------------

To allow sssd to look for users in trusted domains

   subdomains_provider = ipa

has to be added to the domain section in sssd.conf. Additionally you
might want to add 'subdomain_homedir = /home/%d/%u' or similar to define
home directories for users from trusted domains.

To evaluate data from the PAC
(http://tools.ietf.org/html/draft-brezak-win2k-krb-authz-01) the PAC
responder must be started as well. To do this add 'pac' to the services
list to the sssd section in sssd.conf, e.g.

   services = nss, pam, ssh, pac

Currently the PAC is mainly used to add the remote user to additional
groups of the IPA domain.

.. _allowing_individual_access_with_.k5login:

Allowing individual access with .k5login
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If only a few users from a trusted domain shall be allowed to access the
client or if users from the trusted domain shall access the client as a
user from the IPA domain a .k5login (please note the dot as the first
character of the name) file can be used. For the first case the Kerberos
principal name of the user from the trusted domain
(username@TRUSTED.DOMAIN) has to be put into the .k5login file in the
home directory of the trusted user. For the second case the same content
has to be put into the .k5login file in the home directory of the IPA
user.

.. _allowing_global_access_for_a_trusted_domain:

Allowing global access for a trusted domain
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If all users from a trusted domain should be allowed to access the
client the .k5login approach will not scale. Here the following line can
be added to the section for the local realm in /etc/krb5.conf

| ``  auth_to_local = RULE:[1:$1@$0](^.*@TRUSTED.DOMAIN$)s/@TRUSTED.DOMAIN/@trusted.domain/``
| ``  auth_to_local = DEFAULT``

See 'info krb5-admin "Configuration Files" "krb5.conf" "realms
(krb5.conf)"' for more details and examples for auth_to_local.

.. _testing_with_ssh:

Testing with ssh
----------------

A GSSAPI aware Windows ssh client must be installed on the windows
server. I used the putty from Quest http://rc.quest.com/topics/putty/,
but recently GSSAPI support was also added to the "standard" putty
http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html. If you
now log on to the windows server as the test use abc and use putty to
connect with GSSAPI to the FreeIPA server it should just work without
asking for a password.

.. _configuring_putty_for_sso:

Configuring Putty for SSO
~~~~~~~~~~~~~~~~~~~~~~~~~

#. In *Connection - Data*, set Auto-login username to "user@ad.realm".
   Be cautious, this field is case sensitive. To configure
   *Administrator* user in AD domain *ad.test*, configure the field to
   *Administrator@ad.test*
#. In *Connection - SSH - Auth - GSSAPI*, make sure that *Allow GSSAPI
   credential delegation* checkbox is checked
#. In *Session*, set your FreeIPA managed machine *Host name*, save the
   session and connect

.. _adding_remote_users_to_ipa_groups:

Adding remote users to IPA groups
---------------------------------

Users of trusted domains can be added to groups of the IPA domain in two
steps. First an "external" group has to be created to hold the
identifiers of remote objects. Then is group can be added to a group of
the IPA domain.

.. _creating_an_external_group_and_adding_objects:

Creating an external group and adding objects
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To create an external group the '--external' option was added to 'ipa
group-add'

   ipa group-add --desc="External test group" --external ext_test

To add remote objects to the external group from the command line the
SID of the object must be known:

   ipa group-add-member ext_test --external
   S-1-5-21-2324474119-2878384365-2573063092-513

.. _adding_an_external_group_to_an_ipa_group:

Adding an external group to an IPA group
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

External groups can a added like local groups to other groups:

   ipa group-add-member --groups=ext_test local_ipa_group

If now the the KDC receives a TGS request from a trusted domain, i.e a
user from the trusted domain wants to access a service from the IPA
domain, it will extract all SIDs from the PAC in the request. If one or
more of these SIDs are members of external groups and the external
groups are members of IPA groups the SIDs of the IPA groups will be
added to the PAC before sending the service ticket back.

The remote machine will now send the service ticket to the IPA client
where the requested service is running. If the PAC responder is
configured on this client (see `Configure IPA
client <#Configure_IPA_client>`__) the remote user is added to IPA
groups on the client.

.. _freeipa_user_on_windows_desktop:

FreeIPA user on Windows Desktop
-------------------------------

An FreeIPA user can log in to a Windows Desktop from the trusted domain.
The domain part of the user name must be the REALM of the IPA domain,
e.g. 'IPA.TEST\ipauser'.

.. _testing_file_server_cifs_access:

Testing File-Server (CIFS) access
---------------------------------

Please note, although the following step can be done on the IPA server
as on any IPA client, it is not recommended to run a file-serve in the
IPA server.

.. _server_side_ipa_client:

Server side (IPA client)
~~~~~~~~~~~~~~~~~~~~~~~~

One or more share have to be created in the samba configuration by
either adding them to /etc/smb.conf or by using 'net conf addshare' for
registry based configurations. (To add a share on the IPA server for
quick testing use 'net conf addshare test /tmp', please do not forget to
call 'net conf delshare test' after testing).

Since samba isn't very flexible in searching for local user names SSSD
has to be configured to use fully qualified names like
SHORTDOMAINNAME\username instead of the default
username@LONG.DOMAIN.NAME. The following regular expression must be
added to the appropriate domain section or to the sssd section

   re_expression = (?P[^\\]*?)\\?(?P[^\\]+$)

The following regular expression can be used to support both types of
fully qualified names at the same time

   re_expression =
   (((?P[^\\]+)\\(?P.+$))|((?P[^@]+)@(?P.+$))|(^(?P[^@\\]+)$))

Please note that a very recent version of sssd is needed (currently only
available in the ipa-devel repository) to allow the short (NetBIOS)
domain names to be used.

.. _client_side_windows:

Client side (Windows)
~~~~~~~~~~~~~~~~~~~~~

To access the share on the IPA client either

   net use \* \\\ipa.client\sharename

or use 'Map network drive...' available e.g. with a right-click on the
Computer object in the Windows explorer.

FAQ
---

Section listing quick gotchas to help you setup a trust.

Q1> Why do I get the following error when running ipa trust-add
--type=ad

"ipa: ERROR: Cannot find specified domain or server name"

A: Because your IPA server can't see the keberos or LDAP records that
tell it where the target DC is. To troubleshoot which records your
system is having an issue with. Add the following log level = 11 to
/usr/share/ipa/smb.conf.empty. When you check /var/log/httpd/error_log
you will see which SVR records IPA is having an issue resolving.

If per chance your records are fine. It could be the case you just
edited your /etc/resolv.conf file and simply restarting the IPA stack
will resolve your issue. (restart using ipactl restart )

`Category:Obsolete <Category:Obsolete>`__
