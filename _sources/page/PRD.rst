PRD
===

Summary
=======

freeIPA2.0 will focus on making IPA as usable as possible by other
projects. This will include enabling IPA to manage machine and service
identity and creating a plug-in architecture for IPA.

The goal is to release a version of freeIPA that will:

-  Address barriers to v1 usage
-  Provide v2 Identity functionality (machine and service identity)
-  Provide initial Policy and Audit functionality (dropped, except for
   Host Based Access Control)



General Overview
================



Main Use Cases Solved by IPAv1
------------------------------

-  Manage Linux/Unix user identities, groups, and passwords centrally
   and more easily (CLI,GUI)
-  Authenticate users to Linux/Unix using Kerberos/LDAP instead of NIS
-  Easily install Kerberos, LDAP, NTP
-  Enable basic synchronization with AD (with 1.1)



Main Use Cases for IPAv2
------------------------

-  User Identity Management (based on functionality implemented in v1)
-  Machine identity

   -  Enrollment of the new machines

      -  As a result of the enrollment machine principal must be created
         and machine credentials provisioned to the machine
      -  Machine credentials can be keytab and/or machine certificate.

   -  Machine authentication

      -  Machines coming on the network and requesting services within
         the IPA realm shall be authenticated against that realm
      -  Machine authentication credentials shall be used to provide
         mutual authentication/trust, encryption, and SSO capabilities
         for the services and applications requesting resources and
         accessing other services within the same IPA realm

-  Machine Management

   -  Allow management of individual machines, groups of machines and
      virtual instances
   -  Allow centralized management of different kinds of machine
      policies

-  Policy Management (dropped)

   -  Identify and group applications for the purpose of applying policy
      to them
   -  Create an integrated policy delivery mechanism
   -  Centrally describe and define compliance policies for the machines

-  Access Control

   -  Set and enforce a policy that specifies which users can access
      which applications on which machines.
   -  Enable central management of pam login access controls
   -  Enable a centrally managed SUDO configuration to scope
      administrative control. (dropped)
   -  Enable central management of SELinux based Role Based Access
      Control (dropped)
   -  Enable central management of SELinux policy (dropped)

-  Audit (dropped)

   -  Centrally collect (by a selected group of users and/or machines)
      and store:

      -  (i) security events
      -  (ii) all logs
      -  (iii) every keystroke

   -  Centrally audit compliance with the policies
   -  Centrally manage what log events are collected from what groups of
      machines
   -  Manage collected logs easily
   -  Provide reporting tools
   -  Provide external interfaces to analyze and relate collected data

-  External Interfaces (dropped)

   -  As a part of the client create an interface to provide access
      control decisions and authentication to new applications (do early
      in the project)



Compelling Reason to Use
========================

-  Compliance and efficiency are forcing organizations to move off NIS
   and pushing them to use a better identity management and access
   control solution for the Linux/Unix world
-  Efficiency is forcing organizations to use a better identity
   management solution
-  Too expensive to maintain own custom LDAP/Kerberos implementation
-  Have been using services that "assume a security mechanism" and wish
   to secure connections with kerberos or PKI
-  Compliance and efficiency motivate to centrally manage administrator
   delegation

Glossary
========

| Compliant System Configuration : Describing how a system should be
  configured to comply with organizational or governmental policy
| System Lockdown : Actually locking down a system for security and
  compliance
| System Configuration Audit : Auditing a system to see if its
  configuration is in conformance with the compliant system
  configuration
| System Event Audit : An event
| Services : Running code to which you can connect (locally or across
  the network)
| Applications : Code running locally
| Access policy : Policy describing the rules by which an authenticated
  user can connect to a resource



High Level Requirements
=======================

-  1. Machine Identity and Authentication
-  2. Services Identity
-  3. Extensible (Plug-in) Architecture / Framework
-  4. Kerberos
-  5. Certificate and Registration Authority Integration
-  6. Policy (dropped)
-  7. IPA client
-  8. Migration and Interoperability
-  9. Audit (dropped)
-  10. Security of the System
-  11. FreeRADIUS plugin (dropped)
-  12. Active Directory Integration
-  13. User Identity and Administration Enhancements
-  14. Web UI/CLI
-  15. Quality, Performance and Documentation



Detailed Requirements
=====================



1. Machine Identity and Authentication
--------------------------------------

**Note:** Term "Machine" below means is either a physical host or a
guest image in virtual machine.

-  [1.1] Integrate DNS server into the IPA server **(Planned)**

   -  [1.1.1] Store DNS information in the DS **(Planned)**
   -  [1.1.2] Allow IPA clients to automatically discover IPA servers
      (using DNS configuration) **(Planned)**
   -  [1.1.3] Allow management of the DNS entries through the central
      IPA management console **(Planned)**
   -  [1.1.4] Continue to allow IPA to function with an external DNS
      server **(Planned)**

-  [1.2] Policy on an IPA server shall determine the rules of the
   enrollment for the new machines. Options include:

   -  [1.2.1] The machine shall automatically be registered in IPA and
      configured with settings that were pre-initialized **(Planned)**
   -  [1.2.2] An Administrator is required to manually authenticate to
      the IPA server and initialize the configuration settings
      **(Planned)**

-  [1.3] When machine joins IPA realm the following operations shall be
   performed: **(Planned)**

   -  [1.3.1] A unique and permanent identifier (machine GUID) shall be
      set for each machine. **(Planned)**
   -  [1.3.2] Assign a kerberos principal to the machine. **(Planned)**
   -  [1.3.3] Kerberos machine principal name will be administrator
      assigned or automatically set **(Planned)**
   -  [1.3.4] Kerberos machine principal will default to the hostname
      **(Planned)**
   -  [1.3.5] Capture attributes about the machine **(Planned to some
      extent)**

      -  [1.3.5.1] Hostname
      -  [1.3.5.2] Identify operating system on the machine
      -  [1.3.5.3] In the case of administrator enrollment, the ID of
         the administrator

   -  [1.3.6] Generate and provision keytab for machine authentication
      **(Planned)**
   -  [1.3.7] Generate and provision machine certificate for the
      machine/VM to be used by applications and services that require
      PKI authentication **(Planned)**
   -  [1.3.8] Integrate machine into the existing network by downloading
      and applying policies related to the machine (network settings,
      policy, printers) **(NOT Planned)**
   -  [1.3.9] Allow specifying a “policy profile” for a machine during
      enrollment. This will automatically place machine into
      corresponding machine groups. (Low Level – can be deferred)
      **(Deferred)**

-  [1.4] Enable grouping of machines **(Planned)**

   -  [1.4.1] Define and apply different kinds of policies to different
      groups of machines **(NOT Planned)**
   -  [1.4.2] Define policies for applications running on the machines
      and apply these policies to groups or individual machines **(NOT
      Planned)**
   -  [1.4.3] Allow creation of the high level collections of policies
      (for example: “Manager Laptop”, “Developer Desktop”, “Web Server”
      etc.) to simplify deployment of the system (Low Level – can be
      deferred) *(NOT Planned)*'

-  [1.5] Name change and machine cloning scenarios. Provide a tool to:

   -  [1.5.1] Change the machine name **(No tool planned. The machine
      would have to be re-enrolled after rename)**
   -  [1.5.2] Change the machine kerberos principal name when a virtual
      machine is copied or migrated **(No specific work planned. This is
      solved via re-enrollment use case)**

-  [1.6] Renewal

   -  [1.6.1] Automatically renew kerberos credential according to the
      centrally managed renewal policies **(Planned - do it yourself
      instructions)**
   -  [1.6.2] Automatically renew and provision certificates before
      their expiration according to the centrally managed policy
      **(Planned)**

-  [1.7] Allow a machine to leave the realm, de-activating the identity
   from IPA and destroying/revoking any certificates/keytabs.
   **(Planned)**

   -  [1.7.1] The task of de-activating a machine from a realm should
      not require access (physical or network) to that client machine
      **(Planned)**
   -  [1.7.2] Allow machine to be de-activated from IPA realm through
      the IPA client software **(Planned)**
   -  [1.7.3] Allow machine to be re-enrolled into the IPA realm.
      **(Planned)**

-  [1.8] Update LDAP schema to support machine identity and related
   policies **(Planned - but only for identity part)**
-  [1.9] Maintain the identity of the machine or virtual machine after
   an upgrade of the OS including a major upgrade for example from 4 to
   5. **(Planned)**



2. Services Identity
--------------------

-  [2.1] Services Identity and Credentials

   -  [2.1.1] Allow different ways of defining services, the credentials
      they require for authentication and policies that control their
      operation:

      -  [2.1.1.1] Specify services through IPA client. In this case the
         provisioning of the service should happen automatically
         (subject to policy validation on server) (low priority)
      -  [2.1.1.2] Specify in the GUI or command line on the server
         side. In this case client side software should detect (not
         necessarily immediately and automatically) and download
         application (service) related policies and credentials.

   -  [2.1.2] Automatically create/provision/renew/revoke the
      credentials required for the services and applications to function
      according to the policies defined for the services and machines

      -  [2.1.2.1] default /etc Machine kerberos principals shall not be
         used for services by default (except for SSHD which will
         leverage machine kerberos principal) services should have it's
         own keytab
      -  [2.1.2.2] In IPA v2 allow only Kerberos or PKI authentication
         for the services
      -  [2.1.2.3] It should be possible to issue one certificate for
         each service running on the machine/VM

-  [2.2] Service Management

   -  [2.2.1] Allow unique identification of services and applications
   -  [2.2.2] Provide means to define and associate policies or groups
      of polices to a collection (group) of applications/services
      running on a machine or a group of machines. **(NOT Planned)**
   -  [2.2.3] Implement an option to automatically create service
      principal and/or certs when a new service is setup (later than
      machine join)
   -  [2.2.4] Should be easy to give the same service principal to
      multiple services on different machines (cluster use case)
   -  [2.2.5] Service principals and/or service certs should be easily
      to generate and manage

**The following service related work is currently planned:**

-  **Creating service via UI as in v1**
-  **Provisioning service keytab (ipa-getkeytab) - same as in v1**
-  **Provisioning certificates for services. Certs will be stored in the
   service record**
-  **The UI will be modified to be more user friendly as a part of the
   whole UI rework effort.**
-  **The services will be associated to the hosts they run on base on
   the host name which is a part of the kerberos principal**

**The following service related work is currently not planned:**

-  **Handling cluster use cases**
-  **Sharing of the kerberos principals between services**
-  **No policy management**



3. Extensible (Plug-in) Architecture / Framework
------------------------------------------------

-  [3.1] IPA should be built with an extensible framework that supports
   addition of multiple different features and capabilities as add-ons.
   It is implied that a feature can be represented by a collection of
   packages that by itself constitute plug-ins, modules and scripts of
   different nature.
-  [3.2] The IPA server shall allow following extensibility, server
   restart required:

   -  [3.2.1] Modification of LDAP Schema
   -  [3.2.2] DS Server plug-in – plug-in into the LDAP server itself
   -  [3.2.3] Plug-in into XML-RPC interface
   -  [3.2.4] Plug-in into GUI

-  [3.3] Constraints for Extensible Framework

   -  [3.3.1] Features should be able to provide schema that is picked
      up by the GUI
   -  [3.3.2] Existing entries should be dynamically updated with
      default values where required by the new Schema
   -  [3.3.3] Features shall be able to be installed into IPA easily
   -  [3.3.4] The GUI may be restarted to reflect new plug in UI
      elements
   -  [3.3.5] It shall be possible to deploy feature elements (plug-ins)
      in an asymmetric fashion on different IPA servers. For example, a
      schema for a feature is deployed on all IPA replica instances but
      the UI plug-in and XML-RPC service will only run on a single IPA
      sever.

-  [3.4] Rework schema to support extensible architecture
-  [3.5] Plugins shall be self identifying
-  [3.6] It should be possible to list/activate/deactivate plugins from
   the GUI
-  [3.7] It should be possible to resolve inter-plugin dependencies
-  [3.8] Need to provide a way for customers to define scripts to be run
   when certain actions are performed.

| **This work is core and planned.**
| **3.6 is deferred (if not dropped)**

4. Kerberos
-----------

**No work planned other than consulting.**

-  [4.1] Kerberos Functionality

   -  [4.1.1] Kerberos shall work and be easy to use - **We are not
      going to do any modfications to KDC relative to v1.**
   -  [4.1.2] It shall be easy to configure Kerberos for the following:
      **Current implementation of the services provides enough. We will
      consult other teams on how to kerberise their products.**

      -  [4.1.2.1] CIFS
      -  [4.1.2.2] Apache
      -  [4.1.2.3] CUPS
      -  [4.1.2.4] Imap Cyrus-imapd, Dovecot,
      -  [4.1.2.5] mta (for smtp) -- sendmail, postfix, exim -- include
         support for email aliases.
      -  [4.1.2.6] Jboss
      -  [4.1.2.7] NFSv3 and NFSv4

   -  [4.1.4] Allow Kerberos principals to be known by multiple aliases.
      For example, clients do not have to know a mail or web server's
      fully qualified domain name in order to authenticate. An alias or
      short name should be sufficient. **Requires protocol changes and
      standards work - deferred.**

-  [4.2] Kerberos Integration (might be deferred)

   -  [4.2.1] Currently much of the Kerberos Service (KDC) configuration
      information is stored in files on the local file system. This
      makes replication problematic. Kerberos configuration shall be
      stored in the Directory so it can be replicated throughout the IPA
      infrastructure **Requires changes to KDC. We are not planning to
      implement these changes in v2. It is big effort - deferred.**
   -  [4.2.2] Improve the DS schema to store Kerberos related
      information (user info & config info) **Requires changes to KDC.
      We are not planning to implement these changes in v2. It is big
      effort - deferred.**



5. Certificate and Registration Authority Integration
-----------------------------------------------------

-  [5.1] Include a certificate authority and registration authority as
   part of default server installation and configuration

   -  [5.1.1] No other Certificate system components will be included in
      this release
   -  [5.1.2] No key escrow DRM
   -  [5.1.3] No token management or token key smart card components
   -  [5.1.4] No OCSP responder

-  [5.2] The included CA will automatically publish CRLs

   -  [5.2.1] All certs issued will include the CRL location

-  [5.3] Certificate Authority will be used to issue the machine and
   service specific certs
-  [5.4] In this version, there will only be support for machine and
   service certificates. There will be no support for user certificates
   for authentication, signing, encryption
-  [5.5] It should be possible to store Certificates and key material in
   either flat files or an NSS database

| **Planned. Accomplished mostly by the CS team.**
| **No plans to support CRL checking by the client components that
  connect to the server using SSL (XML-RPC, audit). We are considering
  mechanism to dirstibute cert needed for this using DS and LDAP
  lookup.**

6. Policy
---------

**THE WHOLE SECTION IS DROPPED.**

-  [6.1] Policy management and definition

   -  [6.1.1] Policy language should be declarative and analyzable.
   -  [6.1.2] Policy should be stated once in a high-level, platform
      neutral way and then translated to platform specific controls.
   -  [6.1.3] Policy language should be standards-based if at all
      possible.
   -  [6.1.4] It may be acceptable to use an already existing, prevalent
      method of specifying policy.
   -  [6.1.5] If an application already has policy definitions it may be
      possible to manage those definitions in IPA. Over time it is
      desired that these 3rd party policy definitions will be rewritten
      as described above.
   -  [6.1.6] Allow the same centrally managed policy to be presented in
      different formats to different applications (policies might be
      stored in a generic format but then translated into different
      formats by application specific plug-ins or converters).
   -  [6.1.7] IPA policy storage and management engines shall be
      extensible so that new policies can be dropped in, managed
      centrally and delivered through existing IPA channels to the
      requesting application.
   -  [6.1.8] Policies (access control policies) shall (among others)
      include limiting of access to applications / tools to controlling
      the editing of configuration files or data.
   -  [6.1.9] The policy types supported by the IPA system may include:

      -  Sudoers files
      -  SELinux policy files

-  [6.2] The policy management and policy delivery mechanisms shall be
   optimized to reduce:

   -  [6.2.1] Number of long lived network connections (if any).
   -  [6.2.2] Policy download network traffic (what to cache and how
      frequently to refresh).
   -  [6.2.3] Number of client requests and round trips to determine
      which policies apply to the machine and need to be downloaded or
      refreshed.

-  [6.3] Policy enforcement

   -  [6.3.1] Policy should be enforceable by applications that are not
      part of IPA (i.e., IPA Policy should serve as a platform).
   -  [6.3.2] Policy decisions should be obtainable from a
      language-neutral source and if possible through standard
      interface.
   -  [6.3.3] Policy controls should initially target machine policies
      but be capable of controlling service applications.
   -  [6.3.4] Policy enforcement decision shall be based at least on the
      following factors:

      -  User identity
      -  User role
      -  System/Machine identity
      -  Application/service requesting identity check
      -  Resource/service/application being accessed
      -  Time of day/date
      -  Network location/topology (inside/outside firewall)

-  [6.4] Access Control

   -  [6.4.1] Create mechanisms to deliver access control policies to
      the applications. The policies can be delivered in different ways:

      -  [6.4.1.1] Directly via API. Application calls API to perform
         authorization checks. It is implementation specific whether it
         is an API that makes the application a direct client of the IPA
         server itself or a local API that allows application to connect
         to local IPA service (IPA client component) that abstracts the
         IPA server capabilities (provide early in the project cycle)
      -  [6.4.1.2] Via application specific configuration files

   -  [6.4.2] Following applications shall receive access control
      information (policies) from IPA system. The preferred channel (API
      vs. File) is application specific and shall be determined based on
      the application needs and capabilities. Applications are:

      -  [6.4.2.1] PAM for needs of GDM, SSH, FTP, Login
      -  [6.4.2.2] Apache to perform authorization checks while users
         access web resources
      -  [6.4.2.3] JBoss to perform authorization checks while users
         access web resources
      -  [6.4.2.4] IMAP
      -  [6.4.2.5] SMTP
      -  [6.4.2.6] iptables
      -  [6.4.2.7] ipsec
      -  [6.4.2.8] SUDO – for access control and authentication
      -  [6.4.2.9] SELinux
      -  [6.4.2.10] AMQP

   -  [6.4.3] Enable reduced scope root accounts. If SELinux can support
      this functionality.

-  [6.5] Delegation. If SELinux can support this functionality **(We
   plan administrative delegation within the DS using ACIs)**

   -  [6.5.1] Allow delegation of a subset of administrator privileges
      to users / roles / applications.
   -  [6.5.2] Administrative delegation shall include at least OS
      privilege, database privilege (MySQL), and JBOSS privilege.

-  [6.6] Need meta group. Group which groups up groups of users,
   machines, services, hosts - We might not do this if it deems not the
   solution **(Not a solution)**
-  [6.7] The design of the IPA shall allow for other types of the
   authentication (2FA, 3FA etc.) be used. Vendors should be able to
   build their strong authentication solutions and integrate into the
   realm without conflicting with IPA. It should not only be allowed but
   allow vendors to provide password hiding capability – user never
   knows his password he just uses token or smart card. This means that
   these solutions should be able to inject themselves into the
   authentication sequence, intercept provided credential (OTP for
   example) and replay kerberos password (that it stores in its
   database) back to IPA as if user provided password. The impact on the
   already existing products shall be minimized (do not change
   interfaces, try to be compatible with what already exists). The
   alternative types of the authentication shall be allowed in the
   following scenarios:

   -  Machine enrollment and keytab provisioning
   -  User authentication into the box via PAM
   -  User authentication via SSH
   -  Apache authentication to Web site

This is a design requirement. The actual implementation will be
deferred. **(Was considered but no hooks will be implemented.)**

-  [6.8] There should be a way to centrally control preventing user
   logging into the machine (server side enforcement). This would
   require buy in from Kerberos community and might be deferred. **(Not
   possible)**
-  [6.9] Implement centralized management of the roles for the needs of
   the different applications including but not limited to:

   -  [6.9.1] MRG (AMQP)
   -  [6.9.2] Virt
   -  [6.9.3] SELinux
   -  [6.9.4] Policy Kit

-  [6.10] Enable existing SUDO configuration files to be loaded into IPA
   server



7. IPA client
-------------

-  [7.1] IPA client is a separately installable package that allows
   machines to be integrated into the IPA domain. **(Planned)**
-  [7.2] IPA client shall be supported on **(subject to change)**:
   **(Current scope is Linux only.)**

   -  [7.2.1] Red Hat Enterprise Linux 4.8 (32/64)
   -  [7.2.1] Red Hat Enterprise Linux 5.1, 5.2, 5.3 (32/64)
   -  [7.2.2] AIX 5.3
   -  [7.2.3] HP-UX 11i v3
   -  [7.2.4] Solaris 9, 10 (SPARC/x86/x64)
   -  [7.2.5] Suse 10 (32/64)
   -  [7.2.6] Data ONTAP (Netapp's proprietary OS for their Filers) --
      added by kwirth, 7/10

-  [7.3] IPA server must support clients and client configurations
   released in IPA v1 version. **(Planned)**
-  [7.4] IPA client shall be a provider of the IPA services to all
   applications and services running on the machine. **(Planned)**
-  [7.5] IPA client shall provide IPA services to the applications
   regardless whether the machine is connected to the IPA server (server
   is reachable) or machine is offline (server not reachable). There is
   no requirement for the applications to take advantage of the provided
   services. The applications until explicitly mentioned are not outside
   of the scope of the current PRD. **(Planned)**
-  [7.6] IPA must be capable of providing AAA functions to the following
   explicitly listed applications providing time allows to implement a
   corresponding plug-in **(Planned)**

   -  [7.6.1] PAM framework
   -  [7.6.2] NSS (Name Service Switch) Users, groups , netgroups
   -  [7.6.3] SUDO - **(NOT Planned)**
   -  [7.6.4] SELinux - **(NOT Planned)**
   -  [7.6.5] Automount **(Planned + support of different locations)**
   -  [7.6.6] Linux Desktop (GNOME) (Lower priority) - **(NOT Planned)**
   -  [7.6.7] Apache (Lower priority) - **(NOT Planned)**
   -  [7.6.8] JBoss (Lower priority) - **(NOT Planned)**

-  [7.6] IPA client installation and configuration shall support
   following operations: **(Planned)**

   -  [7.6.1] Enroll – make the machine a part of the IPA domain. During
      enrollemen the new certificates and keytab are provisioned. If
      enroll is run twice without the cleanup in between it should
      either: **(Planned. Implementation details might be slightly
      different.)**

      -  [7.6.1.1] Return an error and ask the user to run the
         "Disconnect" operation first.
      -  [7.6.1.2] Assume that this is a "rebuild" operation and cleanup
         and reprovision everything automatically.

   -  [7.6.2] Disconnect – remove all the configuration related to the
      IPA realm. **(Planned)**
   -  [7.6.3] Rebuild – disconnect and then enroll (convinience
      feature). **(Deferred)**
   -  [7.6.4] Uninstall – disconnect and remove software **(Planned)**

-  [7.7] IPA Client Connectivity **(Planned)**

   -  [7.7.1] IPA client configuration shall store information required
      for the client to connect to IPA server and access all its
      services (Kerberos, LDAP, XML-RPC, Policy download, DNS, Audit).
      **(Planned)**
   -  [7.7.2] IPA client software shall automatically discover IPA
      server configuration and topology. Client shall connect to the
      closest (best) server out of the list of available servers. Client
      shall always try to use the closest server it can detect.
      **(Planned)**
   -  [7.7.3] IPA client shall automatically fail over to a different
      IPA server if its “preferred” IPA server is not available.
      **(Planned)**
   -  [7.7.4] IPA client shall effectively determine its online/offline
      status and automatically reconnect to IPA server when client gets
      online. **(Planned)**
   -  [7.7.5] IPA client offline state shall not affect response time of
      the IPA client services to applications. **(Planned)**
   -  [7.7.6] IPA client shall be capable of caching all necessary
      information about user that had logged in at least once to be able
      to provide them services offline. The cached information shall
      have an expiration limit. After the specified period of time the
      cached data shall be discarded. **(Planned)**
   -  [7.7.7] Define and apply IPA centrally managed policies regarding
      offline entities. Create and enforce the policy for how long the
      user offline information is cached (90 days default) and what to
      do if time elapsed (discard). **(NOT Planned.)**
   -  [7.7.8] IPA client shall download and cache policies required for
      operation of the applications/services running on the box. If the
      box goes offline or IPA servers become unreachable the
      applications continue to function without disruptions.\ **(NOT
      Planned)**
   -  [7.7.9] On reconnect to the IPA server, the IPA client shall
      resync/refresh/renew cached credentials, keytabs, certs and other
      downloaded entities (policies and configuration information for
      applications) based on the centrally defined and managed policies.
      **(Partially - no policy or kerberos key refresh)**
   -  [7.7.10] For the system to automatically join IPA realm without
      re-prompting user when IPA server connection is restored the
      password shall be cached on the system. There shall be a policy to
      allow or disallow caching clear text password. **(Similar
      functionality is planned but we do it differently. )**
   -  [7.7.11] In case of outage a lot of users will be trying to
      connect to IPA domain again, authenticate and download policies.
      This might cause a huge load on the server. Create a solution (may
      be just doc) to describe how to reduce the risk of overwhelming
      the server (have more servers...). **(Considered in designs but no
      further action is planned so far)**
   -  [7.7.12] Reduce the amount of data to download or refresh.
      **(Planned)**
   -  [7.7.13] IPA Client shall work correctly whether it is connected
      to the IPA server via local network or VPN. **(Planned)**

-  [7.8] Post installation, if required by enterprise policy, change the
   root password on a device to a pre-configured one found during the
   configuration phase (or possibly randomized). The device is now
   "owned" by the administrative domain. Check that the password is the
   same as the one centrally defined and if not reset. Log the event.
   **(Deferred)**
-  [7.9] IPA client shall automatically renew user TGT according to the
   centrally defined polices. This shall be done to avoid failures of
   the operations that take extensive period of time to run.
   **(Deferred. Not included into the task breakdown. Only the machine
   TGT renewal is planned)**
-  [7.10] On the client and server sides there should be a process that
   would monitor the system and restart components that has crashed.
   **(Planned)**
-  [7.11] Address following bugs: 442680, 454896 (clone of 445754)
   **(Deferred. It is pretty much same as [7.9])**
-  [7.12] IPA Client install shall have an option to enable
   pam_mkhomedir cfg. **(Have not been evaluated but might be doable
   without requiring extra time)**



8. Migration and Interoperability
---------------------------------

-  [8.1] Migration from DS **(Planned)**

   -  [8.1.1] The IPA solution shall provide a method for migrating user
      identities from an existing directory or identity store into the
      IPA servers directory. Document the process. Might require prof
      services. **(Planned)**
   -  [8.1.2] Create a way to migrate user passwords without affecting
      user experience. One of the ides it to use pam_krb5_migrate.
      **(Planned)**
   -  [8.1.3] A standard IPA input format will be defined so if a
      customer wishes to migrate data from a directory that uses
      non-standard schema or layout they will need to export their data
      and map it into this input format. **(Planned)**

-  [8.2] Make it easy to migrate from NIS to IPA **(Planned)**

   -  [8.2.1] Include basic NIS Server functionality in IPA so that NIS
      clients can leverage NIS Server **(Planned)**
   -  [8.2.2] Store NIS information in LDAP back end **(Planned)**
   -  [8.2.3] Allow NIS information to be loaded into LDAP back end from
      the existing NIS configuration files **(Dropped. Customer would
      have to deal with this part himself. Sample script are available
      in the nis plugin package in the documentation directory)**
   -  [8.2.4] Since passwords can't be migrated, provide a way for the
      self-service password reset. There should be a special procedure
      to migrate passwords. **(Planned but done differently)**

-  [8.3] Manage netgroups in IPA server. **(Planned)**

   -  [8.3.1] Include netgroups response capabilities in IPA as part of
      NIS service. **(Planned)**
   -  [8.3.2] Store netgroups information in LDAP back end. May not use
      the standard schema. **(Planned)**
   -  [8.3.3] Allow netgroups information to be loaded into LDAP back
      end from the existing netgroups configuration files. **(This will
      be accomplished using virtual directory product and thus not and
      IPA requirement)**
   -  [8.3.4] split the concept of group of users and posix group.
      https://bugzilla.redhat.com/show_bug.cgi?id=477037 **(Planned)**

-  [8.4] Manage automount maps in IPA server. **(Planned)**

   -  [8.4.1] Store automount information in the back end. **(Planned)**
   -  [8.4.2] Allow automount information to be loaded into back end
      from the existing map files. **(Planned)**
   -  [8.4.3] Have management interface to configure this data.
      **(Planned)**
   -  [8.4.4] Allow different automount maps be served for different
      sets of clients both via LDAP and NIS. **(Planned)**

-  [8.5] Support IPv6. Any new services have to support IPv6. Legacy
   like NIS is not required. **(Nothing specific planned. DS replication
   does not support IPv6 yet.)**

9. Audit
--------

**REMOVED FROM THE SCOPE. IT IS UNCLEAR IF AUDIT PART WOULD BE DEVELOPED
IN THE CONTEXT OF THIS PROJECT OR WE WILL WITH INTEGRATE SOME OTHER
PROJECT.**

-  [9.1] Centrally collect, store and audit different kinds of the audit
   information for example.

   -  Security events
   -  All logs
   -  Command logging
   -  Every keystroke by a selected group of users

-  [9.2] The audit subsystem shall collect, deliver, store and archive
   logs in a way complint with goverment requlations like (PCI, SOX
   etc.).
-  [9.3] Audit information from IPA servers (i.e Kerberos, LDAP,
   Cerficate) must be captured by IPA audit system, aggregated, and
   stored centrally. (gathering kerberos logs may be difficult).
-  [9.4] IPA client side portion of the audit system shall collect
   events from available feeds (Syslog, audit, internal security events,
   keyboard, other external events - whatever feed provider is
   installed).

   -  [9.4.1] Implement feed provider for audit system.
   -  [9.4.2] Implement feed provider for syslog
   -  [9.4.3] Implement feed provider for IPA server and client internal
      events.

-  [9.5] Allow centralized management of the filtering policies on per
   feed type per machine bases.
-  [9.6] Feed providers should be authenticatable
-  [9.7] The audit system shall define, manage, and enforce central
   policies related to the internal event feed (shall include but not
   limited to: events related to machine, service rename as well as
   change of other principals)
-  [9.8] Logs shall be consolidated in the central place(s)
-  [9.9] Future versions may require that Logs shall be signed to
   prevent tampering with them in transit.
-  [9.10] Log data should be archiveable. Those archives must be highly
   compressed and optionally encrypted.
-  [9.11] Audit channels shall be secure (might be SSL; leverage
   credentials available on the machine – keytab, certs).
-  [9.12] Audit subsystem might take advantage of the AMQP technology.
-  [9.13] The audit consolidation topology of the audit subsystem may be
   different from the replication topology.

   -  [9.13.1] The topology shall be changeable
   -  [9.13.2] The configuration shall be stored in the back end and
      replicated in the IPA realm
   -  [9.13.3] In case of disaster an IPA server can become a new
      consolidation point
   -  [9.13.4] Allow consolidating different feeds in different back end
      servers
   -  [9.13.5] Centrally configure the audit subsystem and its policies
      using the IPA administration interface

-  [9.14] Enable storage of the audit data in the external pluggable
   data store that is a SQL database.

   -  [9.14.1] Define and document database schema

-  [9.15] Enable reporting on the audit data
-  [9.16] Allow analysis of logs
-  [9.17] Allow existing reporting tools to bind into the data store
   (i.e. SQL access).
-  [9.18] Handle disconnected machines. Depending upon the policy
   either:

   -  [9.18.1] Cache and forward log events on IPA client
   -  [9.18.2] Stop machine functioning
   -  [9.18.3] Discard audit events

-  [9.19] Audit information in transit shall adhere to existing audit
   information standards as much as possible.



10. Security of the System
--------------------------

-  [10.1] To modify data in IPA, user/process needs to be authenticated
   and authorized
-  [10.2] Secure the communication between central management store and
   machines
-  [10.3] Secure the machine - IPA communication
-  [10.4] It should be possible to run IPA in an insecure fashion
-  [10.5] It should be possible to switch between secure and insecure
   operation
-  [10.6] There should be a way from the IPA UI to disallow anonymous
   reads to the IPA (v2)



11. FreeRADIUS Plug-in
----------------------

**This whole section is deferred to later version.**

-  [11.1] Support freeRADIUS as a plug in server to IPA

   -  [11.1.1] Freeradius data should be centrally managed
   -  [11.1.2] Where possible Freeradius data should be stored and
      replicated in IPA directory store

-  [11.2] FreeRADIUS Auth Methods (Prioritized list)

   -  [11.2.1] SSL VPN Remote access.
   -  [11.2.2] WirelessLAN and VPN
   -  [11.2.3] 802.1x based WLAN.
   -  [11.2.4] 802.1x LAN. (low priority).
   -  [11.2.5] Test with:

      -  [11.2.5.1] Client OS: (in order)vista, xp, mac leopard RHEL 5,
         4, windows 2000,
      -  [11.2.5.2] VPN clients: Cisco VPN client, Windows native VPN
         client, vpnc, openvpn
      -  [11.2.5.1] WLAN and 802.1x client: Microsoft native WLAN
         client, NetworkManager, Cisco Aeronet, Funk/Juniper Odyssey
         client
      -  [11.2.5.1] WLAN Access point: Cisco
      -  [11.2.5.1] VPN concentrator: Cisco VPN 3000 or equivalent,
      -  [11.2.5.1] SSLVPN: Juniper

-  [11.3] Support policy with RADIUS. Enable users to be placed in a
   RADIUS group. Allow or disallow access based on this group.
-  [11.4] Centrally audit
-  [11.5] Usage scenarios for reference:

   -  [11.5.1] VPN Scenario

      -  User connects to Internet
      -  Starts VPN client on client machine (RHEL, Windows, Apple)
      -  Enters username and password
      -  This is sent over IPSec to VPN server at corporate
      -  VPN Server talks RADIUS to freeRADIUS server sending over
         username and password
      -  FreeRADIUS calls out to Directory Server for authentication
      -  Yes/No given back

   -  [11.5.2] SSL VPN Scenario

      -  User connects to Internet. Starts browser.
      -  Clicks on link to SSLVPN
      -  Enters username and password
      -  This is sent over SSL to SSLVPN server at corporate
      -  SSLVPN Server talks RADIUS to freeRADIUS server sending over
         username and password
      -  FreeRADIUS calls out to Directory Server for authentication
      -  Yes/No given back

   -  [11.5.3] WirelessLAN and VPN. (This is Red Hat's usage of WLAN.)

      -  End user connects to Wireless LAN which is secured with WEP key
      -  From here the use case is same as the remote access use case
         above. Getting on the WLAN doesn't get them on the corporate
         network. It just gets corporate Internet access. They have to
         get onto the VPN to get corporate acccess.

   -  [11.5.4] 802.1x based WLAN. This is where WLAN's are going and
      many corporations have them deployed.

      -  End user connects to Wireless LAN which is secured with 802.1x
         and WPA
      -  RADIUS server negotiates secure tunnel using PEAP through
         wireless access point
      -  End user enters username and password
      -  Sent to RADIUS server
      -  Which calls out to DS to validate username and password
      -  Yes/No sent back and based on this WLAN connection (and full
         access to corporate network) given



12. Active Directory Integration
--------------------------------

-  [12.1] Support Kerberos trust between IPA and AD. **(WAS NOT
   EVALUATED - MOVED to V3)**



13. User Identity and Administration Enhancements
-------------------------------------------------

-  [13.1] Policy management UI **(NOT Planned)**

   -  [13.1.1] Policy management privelages should be assignable and
      delegatable **(NOT Planned)**

-  [13.2] Better Password Aging / Password Policy **(Planned)**

   -  [13.2.1] IPA should use Directory Server native password policy
      **(Planned)**
   -  [13.2.2] Allow per group password related policies. **(Planned)**

-  [13.3] IPA v2 will have more flexible delegated admin controls the UI
   should support these **(Planned)**

   -  [13.3.1] Identify, define and implement delegation Use Cases
      **(Planned)**

-  [13.4] Allow Admin Approval step before user is able to gain higher
   privileges **(Deferred)**
-  [13.5] There should be a way to specify UID ranges during the
   installation. In v1 the uid ranges are hard coded. **(Might be done
   if cost is low.)**
-  [13.6] There should be a way to dump a list of all users as policy
   dictates(V2) **(Nothing special is planned. Do LDAP search)**
-  [13.7] Allow setting different password policies per different
   (non-overlapping) groups of users. **(Planned)**



14. Web UI/CLI-----------

-  [14.1] Improve UI/CLI to support pluggable architecture.
   **(Planned)**
-  [14.2] Take advantage of the improved access control rights.
   **(Planned)**
-  [14.3] Provide UI for all management functions. **(Planned)**
-  [14.4] Extend admin UI to allow creation and control of replication
   agreement configuration. **(Deferred)**

   -  [14.4.1.] It should be possible to configure an new IPA server
      through a web wizard **(Deferred)**

-  [14.5] Support double byte characters **(Planned)**
-  [14.6] Localize into Chinese, Japanese, Korean, German, French,
   Spanish **(Planned after release)**



15. Quality, Performance and Documentation
------------------------------------------

-  [15.1] This release should go through a formal QA cycle. The
   engineering team should develop metrics for quality and ensure that
   they are met. **(Planned)**
-  [15.2] Support up to 200 different sites and branches. **(Deferred)**
-  [15.3] Gather statistic for reliability **(Planned)**
-  [15.4] Baseline Performance numbers should be gathered for:
   **(Considering)**

:# User lookup

:# Authentication

-  [15.5] All high-priority docs bugs must be fixed **(Planned)**
-  [15.6] Provide documentation and samples on how to kerberise Apache
   applications **(Considering)**
-  [15.7] Do not introduce performance issues. Make sure that all LDAP
   lookups are optimized. Cache as much as makes sense. **(Planned)**