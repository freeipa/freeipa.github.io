Samba_4_Provisioning_External_LDAP_Server
=========================================

Overview
========

Samba can be configured to use an LDAP server (389 DS or OpenLDAP) as
its backend database. Currently the provisioning tool always creates a
new (internal) LDAP server, it cannot use an existing (external) LDAP
server.

Internal LDAP server has the following advantages:

-  Simplicity. The provisioning tool creates and configures the LDAP
   server, no manual configuration needed.
-  Automation. Samba test can create and destroy LDAP servers
   automatically.
-  Security. All LDAP server files can be stored in a private directory.
-  Exclusivity. The LDAP server will not be shared so there will be no
   interference or conflicts with other applications.
-  Efficiency. The communication will be done via Unix domain socket
   (LDAPI).

On the other hand, external LDAP server has the following advantages:

-  Configurability. The LDAP administrator has full control of the
   configuration and management.
-  Troubleshooting. It's easier to isolate a problem by running the LDAP
   server separately from Samba tests.
-  Distribution. Samba and the LDAP server can run on different
   machines. The communication will not be as efficient as LDAPI, but it
   can still be as secure as other LDAP client-server communication done
   via network.
-  Integration. Some applications may want to use the same LDAP server.
   The current IPA design doesn't require this, but in the future the
   design could change.

There are some changes that need to be make in order to support external
LDAP server.



Current Code
============

The provisioning tool is located in these files:

-  source4/setup/provision.
-  source4/scripting/python/samba/provision.py
-  source4/scripting/python/samba/provisionbackend.py
-  source4/scripting/python/samba/provisionexceptions.py

To provision Samba with DS:

::

   % setup/provision --realm=EXAMPLE.COM --domain=EXAMPLE \
   --host-name=samba --host-ip=127.0.0.1 \
   --adminpass=Secret123 --root=root --server-role="domain controller" \
   --ldapadminpass=Secret123 --ldap-backend-type=fedora-ds \
   --slapd-path=/usr/sbin/ns-slapd --setup-ds-path=/usr/sbin/setup-ds.pl

The tool will perform the following operations:

-  Create internal LDAP server:

   -  Create a new LDAP server in the private directory.
   -  Configure Samba schema.
   -  Configure domain, configuration, schema, and Samba partitions.
   -  Configure SASL mappings.
   -  Load Samba admin user for SASL authentication.
   -  Configure referential integrity.
   -  Configure attribute indexing.
   -  DS only:

      -  Configure DNA for SID allocation.

   -  OpenLDAP only:

      -  Configure ACL in slapd.conf.

-  Start LDAP server.
-  Setup Samba:

   -  Setup share database.
   -  Setup secrets database.
   -  Setup registry database.
   -  Setup privileges database.
   -  Setup idmap database.
   -  Setup SAM database.
   -  DS only:

      -  Configure ACL in the base entry of each partition.

-  Stop LDAP server.



Proposed Solution
=================

The provisioning tool should be split into 2 different actions:

-  Creating an empty LDAP server with Samba configuration.
-  Setting up Samba on an LDAP server that has been configured for
   Samba.

A new optional parameter (--ldap-action) will be added to the
provisioning tool to determine which action will be executed. By default
the tool will create an internal LDAP server and setup Samba to use this
LDAP server. This is the same as the original behavior.

If you need to use an external LDAP server, you can either configure the
LDAP server manually, or use the provisioning tool to create an initial
LDAP server which you can customize later on. Then you need to run the
provisioning tool again to setup Samba using the LDAP server that you
have created earlier.



Creating LDAP Server
--------------------

Creating LDAP server will execute the following operations:

-  Create a new LDAP server in the specified directory.
-  Configure Samba schema.
-  Configure domain, configuration, schema, and Samba partitions.
-  Configure SASL mappings.
-  Load Samba admin user for SASL authentication.
-  Configure referential integrity.
-  Configure attribute indexing.
-  DS only:

   -  Configure DNA for SID allocation.

-  OpenLDAP only:

   -  Configure ACL in slapd.conf.
   -  Configure root password in each partition. See note below.

On OpenLDAP each partition has its own root user. Only the root user can
remove the base entry in each partition. This is necessary for setting
up Samba on an existing LDAP server. To enable root user, the root
password must be specified in the slapd.conf.

A new optional parameter (--ldap-dir) will be added to the provisioning
tool to specify the LDAP server directory. The default value is
SAMBA_HOME/private/ldap.



Setting up Samba
----------------

Setting up Samba will execute the following operations:

-  Remove existing entries in domain, configuration, and schema
   partitions.
-  Setup share database.
-  Setup secrets database.
-  Setup registry database.
-  Setup privileges database.
-  Setup idmap database.
-  Setup SAM database.
-  DS only:

   -  Set the new domain SID in DNA configuration.
   -  Configure ACL in the base entry of each partition.

A new optional parameter (--ldap-uri) will be added to the provisioning
tool to specify the URI of the external LDAP server. By default the
value is ldapi://. The default path to socket is /private/ldap/ldapi.
Note that the file separator "/" should be encoded as %2F.

The LDAP admin password (--ldapadminpass) must be specified explicitly,
so the tool can access the LDAP server. Otherwise it will generate a
random password.

Examples
========



Configuring Samba with Internal LDAP Server
-------------------------------------------

::

   % setup/provision --realm=EXAMPLE.COM --domain=EXAMPLE \
   --host-name=samba --host-ip=127.0.0.1 \
   --adminpass=Secret123 --root=root --server-role="domain controller" \
   --ldapadminpass=Secret123 --ldap-backend-type=fedora-ds \
   --slapd-path=/usr/sbin/ns-slapd --setup-ds-path=/usr/sbin/setup-ds.pl



Configuring Samba with External LDAP Server
-------------------------------------------

Create a new LDAP server on /root/Samba/fedora-ds using the following
command:

::

   % setup/provision --realm=EXAMPLE.COM --domain=EXAMPLE \
   --host-name=samba --host-ip=127.0.0.1 \
   --adminpass=Secret123 --root=root --server-role="domain controller" \
   --ldapadminpass=Secret123 --ldap-backend-type=fedora-ds \
   --ldap-action=create --ldap-dir=/root/Samba/fedora-ds

Make sure the LDAP server is running. Then setup Samba using the new
LDAP server by specifying the LDAP URI:

::

   % setup/provision --realm=EXAMPLE.COM --domain=EXAMPLE \
   --host-name=samba --host-ip=127.0.0.1 \
   --adminpass=Secret123 --root=root --server-role="domain controller" \
   --ldapadminpass=Secret123 --ldap-backend-type=fedora-ds \
   --ldap-action=setup --ldap-uri=ldapi://%2Froot%2FSamba%2Ffedora-ds%2Fldapi

`Category:Obsolete <Category:Obsolete>`__