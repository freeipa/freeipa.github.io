\__NOTOC_\_

.. _centralized_identity_management_and_authentication_for_linux:

CENTRALIZED IDENTITY MANAGEMENT AND AUTHENTICATION FOR LINUX
------------------------------------------------------------

FreeIPA is an integrated identity and authentication solution for
Linux/UNIX networked environments. A FreeIPA server provides centralized
authentication, authorization and account information by storing data
about user, groups, hosts and other objects necessary to manage the
security aspects of a network of computers.

OVERVIEW
--------

FreeIPA allows Linux administrators to centrally manage identity,
authentication and access control aspects of Linux and UNIX systems by
providing simple to install and use command line and web based
management tools. FreeIPA is built on top of well known Open Source
components and standard protocols with a very strong focus on ease of
management and automation of installation and configuration tasks.

BENEFITS
--------

-  Allows all your users to access all the machines with the same
   credentials and security settings
-  Access personal files transparently from any machine in an
   authenticated and secure way
-  Use advanced grouping mechanism to restrict network access to
   services and files only to specific users
-  Centrally manage security mechanism like passwords, SSH Public Keys,
   SUDO rules, Keytabs, Access Control Rules
-  Delegate selected administrative tasks to other power users

FEATURES
--------

-  Centralized authentication via Kerberos or LDAP
-  Identity management:

   -  users, groups, hosts, host groups, netgroups, services, SELinux
      users

-  Manageability:

   -  Pluggable and extensible framework for WebUI/CLI
   -  Rich Command Line and Web-based user interface

-  Simple installation scripts for server and client
-  Flexible administrative model that allows delegation
-  Certificate provisioning and automatic renewal
-  User self-service and password reset
-  Seamless integration into Active Directory Environment via
   cross-realm Kerberos trust or user synchronization
-  Centrally defined Host Based Access Control
-  Up to twenty servers with multi master replication and flexible
   replication topology
-  Interoperability with a broad set of Linux and UNIX clients
-  Integrated DNS server
-  Centrally-managed SUDO rules, SSH public keys, and SELinux user
   mappings
-  Group-based password policies
-  Serving sets of automount maps to different clients
-  Automatic management of private user groups
-  Can act as a NIS server for legacy systems
-  Painless password migration

COMPONENTS
----------

-  `LDAP Server <http://directory.fedoraproject.org/wiki/Main_Page>`__ -
   based on the 389 project (LDAP)
-  `KDC <http://k5wiki.kerberos.org/wiki/Main_Page>`__ - based on MIT
   Kerberos implementation
-  `PKI <http://pki.fedoraproject.org/wiki/PKI_Main_Page>`__ based on
   Dogtag project
-  `Samba <http://www.samba.org/>`__ libraries for Active Directory
   integration
-  DNS Server based on `BIND <https://www.isc.org/software/bind>`__ and
   the `Bind-DynDB-LDAP <https://fedorahosted.org/bind-dyndb-ldap>`__
   plugin

CAPABILITIES
------------

-  Multiple FreeIPA servers can easily be configured in a FreeIPA Domain
   in order to provide redundancy and scalability.
-  The 389 Directory Server is the main data store and provides a full
   multi-master LDAPv3 directory infrastructure.
-  Single-Sign-on authentication is provided via the MIT Kerberos KDC.
   Authentication capabilities are augmented by an integrated
   Certificate Authority based on the Dogtag project. Optionally Domain
   Names can be managed using the integrated ISC Bind server.
-  Security aspects related to access control, delegation of
   administration tasks and other network administration tasks can be
   fully centralized and managed via the Web UI or the ipa Command Line
   tool.

CONTRIBUTING
------------

FreeIPA is an open source project. We are always open to ideas,
comments, use cases, deployment scenarios, enhancement requests and
patches. For information see `CONTRIBUTE <Contribute>`__ page.
