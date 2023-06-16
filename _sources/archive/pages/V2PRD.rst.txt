Summary
=======

The freeIPA project should release a follow on freeIPAv2 within six
months of the release of freeIPAv1 that will:

-  Address barriers to v1 usage
-  Provide v2 Identity functionality (machine and service identity)
-  Provide initial Policy and Audit functionality

.. _general_overview:

General overview
================

IPAv1
-----

-  Main use cases solved

   #. Authenticate user to Linux/Unix using Kerberos/LDAP instead of NIS
   #. Manage Linux/Unix user identity centrally and more easily (GUI)
   #. Enable basic synch with AD and roadmap to a more robust synch

-  Compelling reason to use

   #. Compliance is forcing organizations off of NIS
   #. Efficiency is forcing organizations to a better identity
      management solution
   #. Too expensive to maintain an LDAP/Kerberos implementation
      themselves

IPAv2
-----

-  Main use cases solved

   -  User Identity

      -  Same as IPAv1
      -  With v2, solution has matured to include wider kerberos
         support, better Windows synch, SPML integration

   -  Machine authentication and identity

      -  Identify and group machines (and virtual instances)
      -  Allow new machines to join IPA and gain an identity
      -  Easily provide each machine with a kerberos principal and
         certificate
      -  Use above to authenticate machines and provide trust

   -  Service authentication and identity

      -  Identify and group applications for the purpose of applying
         policy to them

   -  Acces Control

      -  Set and enforce policy of which users can access which apps on
         which machines.
      -  Enable a centrally managed sudo configuration to scope
         administrative control.
      -  Audit user keystrokes

   -  Machine policy

      -  Centrally describe which machines should have which compliance
         policy
      -  Centrally audit compliance with the policy

   -  Audit

      -  Centrally collect and audit (i) security events (ii) all logs
         (iii) every keystroke by a selected group of users and or
         machines

   -  Manage the above easily
   -  Library

      -  Make a library to provide access control and authentication
         decisions to new applications (do early in project -- need to
         support text file)

-  Compelling reason to use

   #. Compliance and efficiency push to have a better access control
      solution for the Linux/Unix world
   #. Compliance and efficiency motivate to centrally manage
      administrator delegation and sudo configuration
   #. Compliance pushes to monitor machine lockdown configuration
   #. Compliance pushes to easily store and report on centrally the
      keystroke of admins
   #. Security and compliance push to centrally understand/analyze audit
      events

Glossary
========

-  Compliant System Configuration = Describing how a system should be
   configured to comply with organizational or governmental policy
-  System Lockdown = Actually locking down a system for security and
   compliance
-  System Configuration Audit = Auditing a system to see if its
   configuration is in conformance with the compliant system
   configuration
-  System Event Audit = An event
-  Services = Running code to which you can connect (locally or across
   the network)
-  Applications = Code running locally
-  Access policy = Policy describing the rules by which an authenticated
   user can connect to a resource

.. _detailed_requirements:

Detailed Requirements
=====================

Deployment
----------

-  [1] Need to have an offline/local support for laptops and other
   disconnected operation
-  [2] Should work well over a VPN
-  [3] Make it easy to migrate from NIS to IPA
-  [4] Easy to migrate from LDAP to IPA
-  [5] Make a library to provide access control and authentication
   decisions to new applications

   -  [5.1] Do early in schedule
   -  [5.2] Need to support text file

-  [6] Support IPv6

.. _user_identity_and_user_authentication:

User Identity and User Authentication
-------------------------------------

-  [7] Enhanced provisioning system integration

   -  [7.1] Accept an SPML feed to integrate with provisioning solutions

-  [8] Enhanced AD integration

   -  [8.1] Allow synch of any AD attributes from multiple domains
   -  [8.2] Allow arbitrary set of attributes to be synched

-  [9] Enhanced kerberos support. Kerberos should work, be easy to use,
   easy to configure for the following:

   -  [9.1]CIFS
   -  [9.2]Apache
   -  [9.3]cups
   -  [9.4]imap cyrus-imapd, dovecot,
   -  [9.5]mta (for smtp) -- sendmail, postfix, exim -- include support
      for email aliases.
   -  [9.6]Jboss
   -  [9.7]Autofs integration (store maps in IPA)

-  [10] PGP integration

   -  [10.1] Enable simple storage of PGP public keys into directory

-  [11] Include basic NIS Server functionaliy in IPA

   -  [11.1] NIS client can leverage NIS Server
   -  [11.2] NIS Server leverage IPA directory backend and user store

-  [83] Improved password aging and password policies.

.. _machine_identity_and_authentication:

Machine Identity and Authentication
-----------------------------------

-  [12] Identify machines and virtual machines uniquely

   -  [12.1] Assign a kerberos principal to the machine/vm.
   -  [12.2] Kerberos machine/vm principal name will be administrator
      assigned
   -  [12.3] Kerberos machine/vm principal will default to the hostname
   -  [12.4] Generate a certificate for the machine/vm

-  [13] It must be possible and easy to change the machine/vm name

   -  [13.1]Provide a tool to change the machine principal name when a
      virtual machine is copied

-  [14]For v2, machine principal doesn't expire. v3, code to self renew
-  [15] Allow machines to join IPA and gain a unique identity and find
   their policy.

   -  [15.1] Upon join, certificate is generated and deployed to the
      machine
   -  [15.2] Code will automatically renew certificate before it expires
      --- (Question of how applications behave when this happens)
   -  [15.3] Upon join, make it easy to integrate into the existing
      network (network settings, policy, printers)

-  [15.4] Make it possible for a machine without an identity to join IPA
   and work with it
-  [15.5] Allow a machine to leave the realm, removing the identity from
   IPA and destroying any certificates / keytabs. May include bootable
   CD to allow removal from realm and secure deletion of all storage.
-  [16] Capture attributes about the machine

   -  Laptop or not
   -  IP address
   -  Hardware information / inventory (from smolt, factor or dmidecode)
   -  Identify operating system on the machine

-  [17] Enable grouping of machines
-  [18] Secure DNS updates from the client
-  [19] DNS integration

   -  [19.1] Include a DNS server
   -  [19.2] Store DNS integration in LDAP
   -  [19.3] Management console provides way to add entries (advanced
      config)

-  [20]DHCP integration

   -  [20.1] Include a DHCP server,
   -  [20.2] Store information in LDAP

-  [21] Enable identification of printers
-  [22] ??? Secure Attention Key

.. _service_identity:

Service Identity
----------------

-  [23] Uniquely identify services and applications using kerberos
-  [24] Enable grouping of services and applications
-  [25] Machine kerberos principals should not be used for services by
   default
-  [26] Service principals should be easily generatable and useable
-  [27] Option to automatically create service principal when service is
   setup
-  [28] Should be easy to give the same service principal to multiple
   services on different machines (cluster use case)
-  [29] Just one certificate for the machine not one for each service

.. _certificate_system_integration:

Certificate System Integration
------------------------------

-  [30] Include a certificate system as part of default server install
   and config
-  [31] Utilize certificate system to gain server certs for each IPA
   server
-  [31] Enable smooth end user certificate enrollment and provisioning
   to a smart card
-  [32] NOT FOR THIS VERSION. Allow the organization to not use the
   included certificate system but have IPA call out to a different CA

.. _home_directory_encryption_integration:

Home Directory Encryption Integration
-------------------------------------

-  [33] Centrally backup encryption keys for encryption of home
   directory
-  [34] Enable administrator of sufficient authority to retrieve keys

Policy
------

-  [35] Policy should be supported on Linux, Unix, Windows, MacOS.
-  [36] Policy should be stated once in a high-level, platform neutral
   way and then translated to platform specific controls.
-  [37]Policy language should be declarative and analyzable.
-  [38]Policy language should be standards-based if at all possible.
-  [39] Policy should be enforceable by applications that are not part
   of IPA (i.e., IPA Policy should serve as a platform)
-  [40] Policy decisions should be obtainable from a language-neutral
   source
-  [41] Platform specific policy should be possible.
-  [42] Policy controls should initially target OS but be capable of
   controlling applications.
-  [43] If possible use an already existing, prevalent method of
   specifying policy
-  [44] Question: Should we build an authorization engine that is
   pluggable to support different representations of policy.

.. _centrally_manage_access_control_policy:

Centrally manage Access control policy
--------------------------------------

-  [45] Don't focus in v2 on modifying configuration files
-  [46] Focus on providing solutions where services call out to IPA for
   authorization
-  [47] Set and enforce policy of which users can access which services
   on which machines.
-  [48] Create an authorization plugin for PAM that calls out to IPA for
   these services:

   -  GDM
   -  SSH
   -  FTP
   -  Login

-  [49] Create an authorization plugin for Apache that calls out to IPA.
-  [50] Create a plugin for the JBoss authorization framework that calls
   out to IPA for decisions.
-  [51]Provide a library that allows others to implement code to call
   out to IPA for authorization
-  [52] Enable authorization policy for the following to be centrally
   managed by IPA

   -  IMAP
   -  SMTP
   -  iptables
   -  ipsec

-  [53] Set and enforce who can run which application
-  [54] Manage netgroups on server and client enforce netgroup access
-  [55] Access control should use the following information when making
   decisions:

   -  User identity
   -  User roles (and current role)
   -  Time of day / date
   -  Network location / topology
   -  System identity

.. _administrative_delegation_and_scoping:

Administrative Delegation and Scoping
-------------------------------------

-  [56] Modify sudo so it calls out to IPA for authentication and
   authorization
-  [57] Enable reduced scope root accounts
-  [58] Allow delegation of a subset of administrator privileges to
   users / roles / applications.
-  [59] Controls should include limiting of access to applications /
   tools to controlling the editing of configuration files or data.
-  [60] Administrative delegation for v2 should include OS privilege,
   database privilege (MySQL), and JBOSS privilege.
-  [61] Post installation, if required by enterprise policy, change the
   root password on a device to a preconfigured one found during the
   configuration phase (or possibly randomized). The device is now
   "owned" by the administrative domain.
-  [62] Enable migration of existing sudo config into IPA

.. _centrally_manage_selinux_policy:

Centrally manage SELinux policy
-------------------------------

-  [63] PERHAPS NOT IN V2. Take SELinux policy for an application and
   deploy it to box that has that application
-  [64] PERHAPS NOT IN V2. Map and SELinux policies to groups of
   machines and deliver the policy to the right machine.

.. _system_configuration_and_lockdown:

System Configuration and Lockdown
---------------------------------

-  [65] Centrally map a required configuration policy to a group of
   machines
-  [66] Analyze compliance of the system to that policy
-  [67] Alert when configuration is not in compliance and what the
   particulars are
-  [68] Enable reporting on the above
-  [69] IPA must accept policy check in XCCDF format from NIST
-  [70] Client accept policy check in XCCDF format?

Audit
-----

-  [71] Centrally collect and audit the following (configuration
   settings which)

   -  [71.1]security events
   -  [71.2]all logs
   -  [71.3]command logging
   -  [71.4]every keystroke by a selected group of users

-  [72] Receive syslog events centrally and securely

   -  [72.1] Create a secure tunnel using SSL
   -  [72.2] Create a secure tunnel using GSS-API

-  [73] Configure audit subsystems centrally
-  [75] Collect audit subsystem events centrally

   -  [75.1] Create a secure tunnel using SSL
   -  [75.2] Create a secure tunnel using GSS-API

-  [76] Enable storage of syslog and audit data in a SQL database
-  [77] Enable reporting on syslog and audit events
-  [77] Handle disconnected machines. Configuraiton to:

   -  [77.1]Stop machine function
   -  [77.2]Cache and forward log events

-  [78] Audit change of machine and service and user principals should
   be timestamped/associated with a period
-  [79] IPA should be able to control which users, groups, machines will
   have key stroke logging enabled

.. _security_of_system:

Security of System
------------------

-  [80] To modify data in IPA, user/process needs to be authenticated
   and authorized
-  [81] Secure the communication between central management store and
   machines
-  [82] Secure the machine machine communication
