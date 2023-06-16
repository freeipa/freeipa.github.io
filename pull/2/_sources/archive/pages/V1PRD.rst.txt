.. _identity_policy_audit_1.0_prd:

Identity, Policy & Audit 1.0 PRD
================================

Overview
========

.. _description_of_the_ipav1_solutions:

Description of the IPAv1 solutions
----------------------------------

-  **High Level Components**

   -  Fedora Linux
   -  Fedora Directory Server
   -  MIT Kerberos 5 integrated with Directory Server
   -  FreeRADIUS using Directory Server backend
   -  Samba 3 using Directory Server backend
   -  Kerberized packages
   -  JBoss kerberized and leveraging the same kerberos ticket
   -  Simple installation and configuration on the servers
   -  Simple configuration on the client
   -  Synch of identity information with AD
   -  Windows kerberos ticket used with Linux apps
   -  Support Solaris, HP-UX,IBM AIX, Suse Linux clients authenticating
      to IPA server
   -  Start of a GUI for user management

-  **Benefits**

   -  Centralized identity management for Linux/Unix world
   -  Centralized authentication point
   -  Plus platform for additional solutions
   -  Work with AD without banking everything on AD
   -  Unified policies,
   -  Unified logging
   -  Reduce cost of admin
   -  Enhanced security and audit

.. _target_users:

Target Users
------------

#. Eval/Demo/Community Member Developer or IT person wants to set up our
   Identity solution or Authentication solution and work with it or play
   with it in their subnet and with a few machines
#. Official IT project to test or deploy the solution with integration
   into the corporate network
#. Enterprise customers who are required to deploy Identity management
   solution for Unix/Linux due to audit requirements resulting from
   financial or privacy regulations. SMBs are a secondary target.

Requirements
============

.. _server_requirements:

Server Requirements
-------------------

.. _req1_create_centralized_identity_server:

[Req1]: Create centralized identity server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  [Req1.1]: The IPA server will be a combination of the following
   technologies

   -  Fedora Linux

      -  Including KRB5 configured to use Fedora Directory Server

   -  Fedora Directory Server
   -  Configuration UI
   -  User management UI

-  [Req1.2]: The IPA Server will consist of the above technologies
   installed and tested on a single server. Although these technologies
   can be installed on separate systems and made to work with differing
   versions, the only configuration that will be supported will be the
   component versions described in the server section of the platform
   requirements, below, installed on a single system.

-  [Req1.3]: It should be possible to deploy multiple IPA servers within
   a single domain/realm. In this case data should be replicated between
   servers as described in the Replication Section below. All Servers
   should offer identical functionality.

-  [Req1.4]: In the case where an IPA server is configured as a read
   only replica, it should be able to accept updates from a client and
   ensure those updates get written to a master.

-  [Req1.5]: The User administration interface should work with existing
   user management tools such as those that utilize libuser.

.. _req2_create_central_authentication_point_ipa_server_install:

[Req2] Create central authentication point IPA server install
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  [Req2.1]: In addition to Kerberos and Directory Server, there will be
   an optional ability to install integrated freeRadius and Samba 3
   servers both leveraging the Directory Server as a backend.

-  [Req2.2] Admin deploys IPA server with Samba and FreeRADIUS, All on
   one box. Can have multiple replicated boxes.
-  [Req2.3] IPSec Remote access. End user connects to corporate from the
   road or from home via VPN

   #. Connects to Internet
   #. Starts VPN client on client machine (RHEL, Windows, Apple)
   #. Enters username and password
   #. This is sent over IPSec to VPN server at corporate
   #. VPN Server talks RADIUS to freeRADIUS server sending over username
      and password
   #. FreeRADIUS calls out to Directory Server for authentication
   #. Yes/No given back

-  [Req2.4] SSL Remote access. End user connects to corporate from the
   road or home via SSLVPN

   #. Connects to Internet. Starts browser.
   #. Clicks on link to SSLVPN
   #. Enters username and password
   #. This is sent over SSL to SSLVPN server at corporate
   #. SSLVPN Server talks RADIUS to freeRADIUS server sending over
      username and password
   #. FreeRADIUS calls out to Directory Server for authentication
   #. Yes/No given back
   #. WirelessLAN and VPN.
   #. End user connects to Wireless LAN which is secured with WEP key
   #. From here the use case is same as the remote access use case
      above. Getting on the WLAN doesn't get them on the corporate
      network. It just gets corporate Internet access. They have to get
      onto the VPN to get corporate acccess.

-  [Req2.5] 802.1x based WLAN. This is where WLAN's are going and many
   corporations have them deployed.

   #. End user connects to Wireless LAN which is secured with 802.1x and
      WPA
   #. RADIUS server negotiates secure tunnel using PEAP through wireless
      access point
   #. End user enters username and password
   #. Sent to RADIUS server
   #. Which calls out to DS to validate username and password
   #. Yes/No sent back and based on this WLAN connection (and full
      access to corporate network) given

-  [Req2.6] 802.1x LAN. Some organizations have deployed. Same as 4 but
   through the 802.1x aware switch and not through the WLAN gateway.
-  [Req2.7] Should work with at least the following

   #. Client OS: Fedora, RHEL 4, 5, Windows 2000, XP, Vista. Mac?
   #. VPN clients: Cisco VPN client, Windows native VPN client, vpnc
   #. WLAN and 802.1x client: NetworkManager, Microsoft native WLAN
      client, Cisco Aeronet, Funk/Juniper Odyssey client
   #. WLAN Access point: Cisco
   #. VPN concentrator: Cisco VPN 3000 or equivalent,
   #. SSLVPN: Juniper

-  [Req2.8]: Supported PAP/CHAP/EAP

   #. Proposal: Only require and test support for PEAP for Wireless
      LANs.
   #. I don't understand the implications for what this means for
      cleartext passwords or not.
   #. I also don't know what the VPN use cases mean for cleartext
      passwords or not
   #. Goal is to do the 80% broad coverage for v1 and leave the 20% for
      later.
   #. Can't require client side certs. Server side certs ok ---- this
      makes PEAP most likely candidate
   #. Discussion of EAP types and details (scroll down):
   #. http://www.cisco.com/en/US/products/hw/wireless/ps4555/products_qanda_item0900aecd801764fa.shtml
   #. Nice argument for PEAP
   #. http://articles.techrepublic.com.com/5100-1035-6148543.html
   #. Microsoft clients support PEAP. At least Vista and XP SP1 natively

   -  http://www.microsoft.com/technet/community/columns/cableguy/cg1202.mspx

-  Can Linux clients support PEAP via NetworkManager. This seems to say
   so

   -  http://grok.lsu.edu/Article.aspx?articleId=1470

-  [Req2.9]: ClearText passwords: If at all possible we will avoid
   cleartext passwords even if that means modifying freeradius.

   #. If we decide that we must support this for some setups it will
      only be optional (admin will have to take positive action to
      enable). We also discussed some options for limiting replication
      of the cleartext passwords to only certain server nodes, but it
      was generally felt that was post 1.0.
   #. Cleartext passwords should not be used unless required by a
      specific feature. i.e. If a customer wishes to deploy an IPA id
      server without freeradius, cleartext passwords are not required.

.. _req3_create_provisioninginitial_configuration_tool_for_ipa_server:

[Req3] Create provisioning/Initial configuration tool for IPA server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  [Req3.1]: Create a server provisioning tool that easily hooks
   together the following

   -  Fedora Directory Server
   -  Kerberos
   -  RADIUS
   -  SAMBA
   -  DNS
   -  Active Directory
   -  NTP
   -  DHCP

-  [Req 3.2] At a minimum the privisioning tool should produce a zone
   file with the service discovery entries that the admin can load on an
   existing DNS server

-  [Req3.3]: The tool should be used to faciliate the initial
   configuration for the following scenarios

   #. Fresh install of new IPA server
   #. Fresh install of new IPA Server with Central Authentication point
      functionality (freeRADIUS and Samba)
   #. Upgrade of IPA Server to IPA Server with Central Authentication
      point functionality
   #. Upgrading from self signed certs to certificates issued by an
      external CA

-  [Req3.4]: During provisioning the tool should offer mechanism to get
   certificates from an existing CA or create self generated
   certificates.

-  [Req3.5]The provisioning tool is the same tool as the server config
   tool (ipa-config).

-  [Req3.6]: Steps an admin needs to take to get system up and running

   -  Goal is for person using the IPA to have their client
      auto-discover the IPA server, or at most type in the realm or name
      to have the client set up and pointed to the central management
      solution.
   -  Install the configuration tool (yum install ipa-config). This will
      pull in all of the required components.
   -  Optionally install central authentication point configuration
      add-ons (yum install ipa-config-radius ipa-config-samba). This
      will pull in the central authentication point required components.
   -  Run ipa-config which will ask a minimum of questions (ideally just
      for a realm name and admin password).
   -  Optionally replace the self-signed certificate with a different
      certificate using ipa-config.
   -  Add service discovery entries into corporate DNS (_ldap and
      \_kerberos standard entries). This process will be documented but
      not automated.
   -  Connect to administrative gui via the web and add users, groups,
      etc.

.. _req4define_default_schemas_for_ipa_server:

[Req4]Define default schemas for IPA server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  [Req4.1]: Additional schema will be required to store user and
   authentication information.
-  [Req4.2]: This Schema should not conflict with the standard schema
   that will ship with the standalone version of Fedora Directory Server
-  [Req4.3] While it is acknowledged that future versions of the IPA
   product will have enhanced data schema to add functionality e.g.
   SAMBA4), care should be taken to minimize migration effort during
   upgrades.
-  [Req4.4] Intial schema should be designed to simplify future
   upgrades.
-  [Req4.5] In cases where this cannot be done, tools should be provided
   to facilitate easy inplace upgrades

.. _req5kerberize_jboss_middleware:

[Req5]Kerberize JBoss Middleware
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  [Req5.1] Add additional kerberos support will to JBoss core
   authentication module that will allow it to accept Kerberos
   credentials submitted by the browser.

-  [Req5.2]In this version, any JBoss generated page that asks users to
   type their names/passwords can be configured to add another option to
   "Login using Kerberos credentials".

-  [Req5.3]If the Kerberos login fails, the user will be redirected to a
   web page that provides the user with browser-specific instructions on
   how to configure the browser to user Kerberos, as well as
   instructions on how to contact the Help Desk.

-  [Req5.4] Support Kerberos credentials submitted by IE6, IE7, FF1.x,
   FF2.x

.. _req6_replication_and_failover_requirements:

[Req6] Replication and Failover Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  [Req6.1] The IPA server should support all the Replication features
   of Fedora Directory Server including 4 way multimaster replication
   and Windows Sync.
-  [Req6.2] Replication will only be tested/support between IPA servers.
   Replication between and IPA server and a non IPA stand alone
   Directory server is not supported.
-  [Req6.3] All entries in the Directory should be replicated,
   replication is not limited to merely identity entries.
-  [Req6.4] A script should be provided that admins can use to set up
   Replication, including MMR. [Karl M]
-  [Req6.5] Documentation should be provided to simplify Windows Active
   Directory integration and synchronization
-  [Req6.6] in addition to synchronising Directory Data the replication
   system should support the ability to synchronise other IPA
   configuration data. e.g. FreeRadius config, kerberos config, etc
-  [Req6.7] IPA servers should be configurable to support the following
   failures

   -  **Local IPA server or connection to local IPA fails:** Clients
      gracefully failover to remote/backup/standby IPA server
   -  **Directory Server on Local IPA server fails:** Clients gracefully
      failover to remote/backup/standby IPA server

.. _req7_migration_requirements:

[Req7] Migration Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  [Req7.1] The IPA server shall provide a method for easily migrating
   user identities from an existing directory or identity store into the
   IPA servers directory
-  [Req7.2]A standard IPA input format will be defined so if a customer
   wishes to migrate data from a directory that uses non-standard schema
   or layout they will need to export their data and map it into this
   input format.
-  [Req7.3] In particular, we will support a migration from

   #. Fedora Directory Server
   #. Kerberos V5 ( This may not be easy or indeed Possible )

-  [Req7.4] To enable password migration the Adminstration UI should
   provide an interface where the user can set their new IPA password.
   This interface will use the users orginal password to authenticate
   the request

.. _req8_server_platform_support:

[Req8] Server Platform Support
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  [Req 8.1] The IPA Server 1.0 should run on

   -  Linux

      -  i386 and x86_64 only, No Itanium, no Power
      -  MIT Kerberos version 1.6
      -  Samba Version 3
      -  FreeRADIUS version 1.1.6
      -  DNS
      -  NTP

   -  Fedora Directory Server

      -  The WindowsSync feature of DS will be tested/supported on AD
         Windows Server 2003, AD Windows Server 2000

.. _ipa_client_requirements:

IPA Client Requirements
-----------------------

.. _req9_client_configuration:

[Req9] Client configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  [Req9.1]Create client config to allow admins to easily join Linux
   systems to an IPA domain.
-  [Req9.2] At a minimum the tools should:

   -  Update client Kerberos configuration files
   -  Update client Directory Server configuration files
   -  Update client Authentication files for system login and sshd
   -  Update client NTP config
   -  If the IPA server has been configured as a DNS server the client
      config tool should configure the appropriate DNS configuration on
      the client.

-  [Req9.3] Steps the user would do for client config tool on Linux

   -  Run a client configuration tool
   -  Click on IPA Centralized Management
   -  Service discovery is attempted

      -  Success: ask user if they would like to use discovered realm.
         Client will be configured to authenticate against realm using
         dns service discovery.
      -  Failure: prompt for the host name / ip of auth server. Client
         will be configured to authenticate against the ream using only
         the provided host (no service discovery).

   -  Optionally the user can visit the Administration UI on the IPA
      server and have it generate sample configuration files (e.g.
      httpd.conf) or have it configure the browser to work with that
      Kerberos Realm

-  [Req9.4] Client Configuration: Other Systems

   -  Other supported client systems (Windows, Solaris, etc.) will not
      have automated configuration.
   -  Documentation on authentication (using standard protocols) will be
      provided.

-  [Req9.5] All platforms listed in the client support section below
   should be tested and supported as clients of the IPA server product

-  [Req9.6] The following client applications should utilize the
   authentication service of the IPA product

   -  System login for Linux - configured via client config tool
   -  Firefox - configured via web page on Administration server
   -  Thunderbird - configuration documentation to be provided
   -  Apache - configured via template from web page on Administration
      server
   -  SSH/SSHD - configured via client config tool
   -  Evolution - configuration documentation to be provided
   -  NFS v4 filesharing - configuration documentation to be provided
   -  CUPS - configuration documentation to be provided

.. _req10_client_support:

[Req10] Client Support
~~~~~~~~~~~~~~~~~~~~~~

-  [Req10.1]The following platform should be supported as clients of the
   IPA product

   -  [Req10.1.1] Linux

      -  RHEL 5 ( i386 and x86_64 )
      -  RHEL 4.5 ( i386 and x86_64 )
      -  RHEL 3 (i386)
      -  RHEL 2.1 (i386)
      -  Suse (Versions 9 & 10) ( i386 and x86_64 )

   -  [Req 10.1.2] Unix and Windows

      -  Solaris 2.6, 7, 8, 9 & 10 ( SPARC )
      -  Solaris 10 x86
      -  AIX (5.1, 5.2, 5.3)
      -  HPUX (11.0, 11i v1, 11i v2) ( PA-RISC or IA 64 )
      -  Mac OSX
      -  Windows 2000, XP, Vista ( i386 only)

.. _req11_windows_interop:

[Req11] Windows interop
-----------------------

-  Windows Platform support falls into 2 categories
-  [Req11.1] AD clients: The windows client will rely on Microsoft
   Active Directory for account information and Authentication services.
   The IPA server will use the Windows Sync functionality to synchronise
   Username, Password and Group information. This scenario covers 3
   separate use cases that will be supported

   -  [Req11.1.1] Win sync between AD and IPA only
   -  [Req11.1.2] Kerberbos trust relationship between AD and IPA only
   -  [Req11.1.3] Both Win sync and Kerberbos trust relationship between
      AD and IPA

-  [Req11.2] IPA clients: The windows client will rely on the IPA server
   for account information and Authentication services. The IPA server
   will act as an NT4 style domain controller. Only NTLM authentication
   will be supported in this release, no Kerberos

.. _req12_security_requirements:

[Req12] Security requirements
-----------------------------

The IPA servers (both masters and replicas) are central repositories of
information on users, groups, and other information. Sensitive
operations like login and file permissions are based on this
information, and therefor will be high-value targets worthy of attack.
Because the IPA servers will be distributed in various parts of an
organization (possibly 1 or more at each office location), a breach in
any one server can compromise the entire system.

The IPA system needs to offer a level of security that will withstand
attacks from inside the organization as well as from outside.

In particular, the IPA server should support these features:

-  [Req 12.1] Use the certificates that were created or installed by the
   initial configuration/provisioning tool to create SSL connections
-  [Req 12.2] Admins should be able to easily configure IPA servers to
   use SSL between all server instances. (QUESTION: other than using SSL
   for LDAP replication and chaining, what other communications do we
   need to protect?)
-  [Req 12.3] Admins should be able to configure the IPA servers to use
   SSL for all sensitive requests (for example, users changing their
   passwords, admins deleting a user) made through the web interface.

#. use kerberos to forward user credentials and do successful binds
   against the LDAP server using these credentials (we should just need
   forward-able tickets to be able to do this).
#. keep each connection to the LDAP server strictly tied to the user it
   is serving (and on behalf of which it is acting). Extreme care on
   security mechanisms to make sure one user can't hijack another user
   connection is required.

-  [Req 12.4] A security review of the IPA server and solution is
   desired

-  [Req 12.5] Create default SELinux profiles for all IPA components.
   Exploiting a security vulnerability in one module will not result in
   the attacker being able to take over the entire machine.

   -  [Req 12.5.1] Under no circumstances should the adminstrator be
      required to turn off SELinux in order to get the IPA services to
      operate

.. _req_13international_support:

[Req 13]International Support
-----------------------------

-  [Req13.1] Version 1.0 of the IPA product will have no special
   international features
-  [Req13.2] For the first release no consideration will be taken for
   any Non-US privacy laws
-  [Req13.3] All text will be in English for this release, including UI
   screens, manuals, online help, log files, etc.
-  [Req13.4] However the GUI should be built with future localization in
   mind -- i.e it should have resource files etc.. to make it
   straightforward to localize it at a future date.

.. _user_interface:

User Interface
--------------

.. _req14_general_ui_requirements:

[Req14] General UI Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The IPA server shall provide an interface for management of Identity
(and in the future, Policy & Audit) information.

-  [Req14.1] A web interface which should be installed and configured
   automatically on every instance of the IPA server
-  [Req14.2] A set of commandline tools that is installed by default on
   each IPA server, and optionally, may be installed on any Fedora
   client system (as decribed in the Platform requirement section).
-  [Req14.3]The commandline tools should offer identical functionality
   as the Web Interface, where appropriate (It is understood that some
   tasks may only be possible in a graphical interface)
-  [Req14.4] Web interface and command lines should use the same API
-  [Req14.5] Third party or custom developed tool must be able to use
   the published API
-  [Req14.6] The GUI for version 1.0 will allow for the creation,
   modification, deletion and discovery of users and groups in the
   Directory Server such that they are provisioned for both Kerberos and
   the Network Switch Service.
-  [Req14.7] In addition, typical administration activities such as
   account inactivation, password changes, setting of password policy,
   and the editing of organizational information for users shall be
   supported.
-  [Req14.8] GUI should be built with extensibility in mind. Should be
   straightforward for an admin to modify the GUI to enable it to see
   custom attributes. As an example, the interface should allow for the
   editing of extended data items as required by the deployment
   including site supplied schema.
-  [Req14.9] Rapid, accurate search must be a fundamental attribute of
   the GUI
-  [Req14.10] Build v1 GUI for identity but keep in mind that we will be
   adding policy, audit, and the delegation of administration abilities
   to it
-  [Req14.11] All connections to the User Interface described above will
   default to use SSL. This applies to both the web and cli clients.
-  [Req14.12] All connections will be Kerberos Authenticated. V1.0 will
   not support anonymous connections
-  [Req14.13] Configuration information for the user interface should be
   stored in the Directory. This will allow future versions of the IPA
   product the flexibility of enforcing the configuration on either the
   server or by the user interface.

.. _req15_administrator_access_control_and_delegation:

[Req15] Administrator Access Control and Delegation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  [Req15.1] Administrators are assigned administrative privileges by
   existing administrators
-  [Req15.2] Initial admins created on server install
-  [Req15.2] A simple mechanism to allow the delegation of
   administrative abilities to individuals and groups of users should be
   incorporated.
-  [Req15.3] It should be possible to limit the scope of administrative
   privileges to specific groups users.

   -  e.g. Administrator A is only allowed to modify Users in the
      Accounting Group. Only Administrator B is allowed to create users
      who are based in France.

.. _req16_creationediting:

[Req16] Creation/Editing
~~~~~~~~~~~~~~~~~~~~~~~~

-  [Req16.1] Suggest user account and dynamic generation of user account
   field values e.g. mail address and login name from full name, uid
   number generation etc.

   -  [Req16.1.1] certain fields require uniqueness, suggestions must be
      valid

-  [Req16.2] Uniqueness checking should be enforced by the user
   interface. i.e. Before creating an entry the interface should search
   to ensure that it does not attempt to create a duplicate entry. This
   may lead to a race condition

   -  [Req16.2.1] A list of predefined attributes that must be unique
      (e.g uid), offer suggestions (e.g. uid) or are mandatory (e.g.
      Manager) will be stored in the directory
   -  [Req16.2.2] No tools will be provided to modify these predefined
      attributes. Customers must manually modify the directory for site
      specific attributes

-  [Req16.3] Template system based on configured objectclasses for users

   -  [Req16.3.1] Configured objectclasses may be site supplied
   -  [Req16.3.2] Dynamic form generation for attributes

-  [Req16.4] Templates for automatic data field filling, some attributes
   have commonly used values - the template system should allow for
   defaults

   -  [Req16.4.1] The default attributes should all have friendly
      display names

-  [Req16.5] The UI will provide no way for admins to modify access
   controls for attributes
-  [Req16.6] If site specific schema has been added manually to the
   directory server the UI should display new fields. Where possible the
   UI should not allow users to view or modify values if they do not
   have sufficient privileges

.. _req17_discovery:

[Req17] Discovery
~~~~~~~~~~~~~~~~~

-  [Req17.1] Require mechanisms to reduce candidate list

   -  [Req17.1.1] e.g. simple search, search in search results, alphabar
      etc.
   -  [Req17.1.2] Partial match searches, name, phone number etc.

-  [Req17.2] Simple selection of tasks pertaining to a target entry
-  [Req17.3] Status indications (inactivated etc.) viewable with results
-  [Req17.4] Configuration of default display attributes per class

[Req18]Deletion
~~~~~~~~~~~~~~~

-  [Req18.1]V1.0 of the IPA server will support 2 modes of account
   removal

   -  [Req18.1.1] Deletion: All entries and attributes related to a user
      is deleted. Upon deletion uids otherwise become available for
      reuse (presents security hazard)
   -  [Req18.1.2] Suspension: Allow inactivation and reactivation of
      accounts according to the directory methods for doing so.

-  [Future Req] V2.0 will add a tombstone feature where certain
   attributes are kept, possibly in a separate directory branch, to
   allow accounts to be resurrected or to prevent uid reuse.

.. _req19_policy:

[Req19] Policy
~~~~~~~~~~~~~~

-  [Req19.1] Only administrators will be able to reset forgotten
   passwords. There will be no user self service password reset for
   forgotten passwords.
-  [Req19.2] The UI should allow administrators the ability to set the
   Password policy, e.g. Password aging, quality (min/max/complexity
   rules for all passwords)

   -  [Req19.2.1] This functionality should match that offered by Fedora
      Directory Server
   -  [Req19.2.2] Password Quality should be enforced regards of what
      mechanism is used to change the password
   -  [Req19.2.3] Descriptive error messages should communicate why
      passwords that do not satisfy policy are rejected [This is a low
      priority requirement]

-  [Req19.3] Notification of impending expiration. If a user has an
   email address in their entry it should be possible to configure the
   IPA server to send notification when passwords are about to expire
-  [Req19.3] If a manager is deleted the manager attribute of
   subordinates should be set to the managers manager

.. _req20_radius_policy:

[Req20] RADIUS Policy
~~~~~~~~~~~~~~~~~~~~~

-  [Req20.1] Allow users to be placed in a group or role that specifies
   RADIUS users. Allow or disallow RADIUS access based on this

.. _req21administrator_management_of_groups:

[Req21]Administrator Management of Groups
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  [Req21.1] Select groups to add a user to
-  [Req21.2] Select users to add to a groupindividually, (V2 may allow
   search results to fill in group)
-  [Req21.3] support static posix groups only for now
-  [Req21.4] Allow the empty group (default schema does not, need
   workaround)
-  [Req21.5] Allow to perform user based operation on a group e.g.
   inactivate a whole group, set shared attributes like street address
   etc.
-  [Req21.6] Allow removal of all users from the group without deleting
   the group

.. _req22non_administrative_user_use:

[Req22]Non Administrative user use
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  [Req22.1] Allow general search facility
-  [Req22.2] Editing of own data (where allowed)
-  [Req22.3] The Web UI should provide a page that will automatically
   configure the users browser for Kerberos use within the IPA domain.
   V1.0 will only provide this functionality for Firefox. IE and Safari
   will require manual configuration.
-  [Req22.4] To enable password migration the Web UI should provide an
   interface where the user can set their new IPA password. This
   interface will use the users orginal password to authenticate the
   request

.. _req23user_self_management:

[Req23]User Self Management
~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  [Req23.1] Users will automatically log in to the web and cli
   interfaces of the IPA server using Kerberos. From there, they will be
   able to manage these aspects of their accounts:
-  [Req23.2] Password change. Users will be able to change their
   passwords. The Web UI will show them the strength of the password as
   they type their new password. Password quality will be enforced as
   defined in the Policy Requirements section
-  [Req23.3] Aditionally users will be able to modify values for whcih
   they have access control rights to view and modify
-  [Req23.4] The product will ship with a predefined list of these, e.g.
   Work phone number, Cell phone number, Personal URL
-  [Req23.5] Admins should be able to manually modify this list and have
   the user UI reflect that modification. e.g A site may prevent users
   from changing their phone number. In the phone number field should
   not be editable. It is acknowledged that if the field is not set
   there is no way for the UI to enforce this requirement. In this case
   the access control is enforced by the server.
-  [Req23.6] Users should also be allowed to modify their password,
   default shell and description fields through commandline tools,
   including

   -  [Req23.6.1] PAM based tools such as *passwd* on all supported Unix
      and Linux platforms
   -  [Req23.6.2] Windows user account management tools on supported
      Windows platforms in the following scenarios

      -  If the Windows system is an IPA server client. i.e. the IPA
         server is using Samba to act as an NT4 style domain controller
      -  If windows sync is installed on an Active Directory domain
         controller.

.. _req24gui_server_policies:

[Req24]GUI Server Policies
~~~~~~~~~~~~~~~~~~~~~~~~~~

In general the GUI is not to entertain general management of the LDAP
server, in fact a goal is to disguise the fact that an LDAP server
exists underneath. However, some items as they pertain to the management
of users and groups are necessarily server wide:

-  [Req24.1] Allow setting of system wide password policy
-  [Req24.2] We will NOT support fine grained password policy in this
   version)

.. _req26documentation_requirements:

[Req26]Documentation Requirements
---------------------------------

Much of the value with the IPA product will come from it's ease of
installation and usage. This will only be achieved through clear,
concise documentation.

-  [Req26.1] Installation and Deployment guide: Describing all steps
   necessary to deploy IPA server including

   -  Using the Config script
   -  integrating with Active Directory

-  [Req26.3] Administration Guide: Describe tasks in administration user
   interface
-  [Req26.4] Users Guide: Describe tasks in user self management
   interface
-  [Req26.5] Client Setup Guide: For each of the client platforms listed
   in the above Platform Support section clear documentation should be
   provided, detailing how to configure the platform to act as a client
   of the IPA product.
-  [Req26.6] Migration Guide: Detailing steps necessary to migrate users
   from existing Directory or Kerberos deployments

.. _notes_on_future_releases:

Notes on Future Releases
========================

Features listed below will not be included in release 1.0 of the IPA
server product

.. _host_management:

Host Management
^^^^^^^^^^^^^^^

-  v1.0 will not provide any mechanism for managing host or server
   entries
-  The initial configuration script will not configure DNS services on
   the IPA server.

.. _netgroups_and_host_based_access_control:

Netgroups and Host Based access control
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A common requirement for access management systems is user
authorization, such as that used for host based access control. This
release will not provide any netgroup management capability. After the
necessary features are added to the supported client platforms an
updated IPA release can support this feature.

.. _windows_file_and_print_services_cifs:

Windows file and print services (CIFS)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The IPA product will not be able to provide authentication services for
Windows file and print services

.. _user_interface_features_not_in_version_1.0:

User Interface features Not in version 1.0
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. Computer management
#. Host based access control i.e. nis netgroups
#. Logging/audit monitoring
#. Inactivation after period of inactivity
#. Contractor feature - refresh user e.g. every 6 months, or inactivated
#. Org Chart
#. Maps
