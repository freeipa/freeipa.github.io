Samba_4_Replication
===================

Overview
========

This page describes how to setup Samba replica using Fedora DS as a
backend.

This document uses the following environment:

-  Domain name: domain1.com
-  Samba master: samba2.domain1.com
-  Samba replica: samba2.domain1.com

Installation
============

See `Samba Installation <Obsolete:Samba_4_Installation>`__.

Configuration
=============

Create /usr/local/samba/etc/smb.conf for the replica:

::

   [globals]
           netbios name    = samba2
           ...



Provisioning Fedora DS Backend
==============================

Setup Fedora DS instance for the replica:

::

   % cd samba/source4
   % setup/provision-backend --realm=DOMAIN1.COM --domain=DOMAIN1 --server-role='domain controller' \
   --ldap-admin-pass=Secret123 --ldap-backend-type=fedora-ds

Edit /usr/local/samba/private/ldap/fedorads.inf:

::

   [General]
   FullMachineName         = samba2.domain1.com
   SuiteSpotUserID         = nobody
   SuiteSpotGroup          = nobody
   ServerRoot              = /usr/local/samba/private/ldap

   ConfigDirectoryLdapURL  = ldap://samba2.domain1.com
   ConfigDirectoryAdminID  = admin
   ConfigDirectoryAdminPwd = Secret123

   AdminDomain             = domain1.com

   [slapd]
   ServerPort              = 390
   ServerIdentifier        = samba
   Suffix                  = DC=domain1,DC=com

   RootDN                  = cn=Directory Manager
   RootDNPwd               = Secret123

   ldapifilepath           = /usr/local/samba/private/ldap/ldapi

   start_server            = 0
   install_full_schema     = 0

   SchemaFile              = /usr/local/samba/private/ldap/99_ad.ldif
   ConfigFile              = /usr/local/samba/private/ldap/fedorads-partitions.ldif

   inst_dir                = /usr/local/samba/private/ldap/slapd-samba
   config_dir              = /usr/local/samba/private/ldap/slapd-samba
   schema_dir              = /usr/local/samba/private/ldap/slapd-samba/schema
   lock_dir                = /usr/local/samba/private/ldap/slapd-samba/lock
   log_dir                 = /usr/local/samba/private/ldap/slapd-samba/logs
   run_dir                 = /usr/local/samba/private/ldap/slapd-samba/logs
   db_dir                  = /usr/local/samba/private/ldap/slapd-samba/db
   bak_dir                 = /usr/local/samba/private/ldap/slapd-samba/bak
   tmp_dir                 = /usr/local/samba/private/ldap/slapd-samba/tmp
   ldif_dir                = /usr/local/samba/private/ldap/slapd-samba/ldif
   cert_dir                = /usr/local/samba/private/ldap/slapd-samba

Edit /usr/local/samba/private/ldap/99_ad.ldif, replace
1.3.6.1.4.1.1466.115.121.1.44 with 1.3.6.1.4.1.1466.115.121.1.26.

::

   % cd /usr/local/samba/private/ldap
   % /usr/sbin/setup-ds.pl --file=fedorads.inf



Starting Fedora DS
==================

::

   % cd /usr/local/samba/private/ldap
   % slapd-samba/start-slapd



Configuring Multi-Master Replication
====================================

Samba uses 3 databases in Fedora DS. They require separate replication
agreements.

::

   % yum install perl-LDAP

Download `mmr.pl <Media:mmr.txt>`__ script to configure MMR:

::

   % mmr.pl \
   --host1 samba1.domain1.com --host2 samba2.domain1.com --port 390 \
   --host1_id 1 --host2_id 2 \
   --binddn 'cn=Directory Manager' \
   --bindpw Secret123 \
   --repmanpw Secret123 \
   --base dc=domain1,dc=com \
   --create

   % mmr.pl \
   --host1 samba1.domain1.com --host2 samba2.domain1.com --port 390 \
   --host1_id 1 --host2_id 2 \
   --binddn 'cn=Directory Manager' \
   --bindpw Secret123 \
   --repmanpw Secret123 \
   --base cn=Configuration,dc=domain1,dc=com \
   --create

   % mmr.pl \
   --host1 samba1.domain1.com --host2 samba2.domain1.com --port 390 \
   --host1_id 1 --host2_id 2 \
   --binddn 'cn=Directory Manager' \
   --bindpw Secret123 \
   --repmanpw Secret123 \
   --base cn=Schema,cn=Configuration,dc=domain1,dc=com \
   --create



Provisioning Samba
==================

::

   % setup/provision --realm=DOMAIN1.COM --domain=DOMAIN1 \
   --adminpass=Secret123 \
   --ldap-backend-type=fedora-ds \
   --ldap-backend=ldapi:///usr/local/samba/private/ldap/ldapi \
   --partitions-only

::

   Server Role:    domain controller
   Hostname:       samba2
   NetBIOS Domain: DOMAIN1
   DNS Domain:     domain1.com
   DOMAIN SID:     S-1-5-21-3010954269-3145692404-1112636010
   Admin password: Secret123



Joining Samba Domain
====================

::

   % cd /usr/local/samba/bin
   % net join DOMAIN1 BDC -U Administrator --password=Secret123

::

   Joined domain DOMAIN1 (S-1-5-21-1030068324-2126043060-2085863383)

Generate UUID:

::

   % uuidgen

Create a file containing the following entry:

::

   dn: CN=NTDS Settings,CN=SAMBA2,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=domain1,DC=com
   objectClass: top
   objectClass: applicationSettings
   objectClass: nTDSDSA
   cn: NTDS Settings
   options: 1
   showInAdvancedViewOnly: TRUE
   systemFlags: 33554432
   dMDLocation: CN=Schema,CN=Configuration,DC=domain1,DC=com
   invocationId: <UUID>
   msDS-Behavior-Version: 2

Add the entry to Samba master:

::

   % cd /usr/local/samba/bin
   % ./ldbadd -H ldap://samba1.domain1.com -p -U Administrator --password=Secret123 <file>



Starting Samba Replica
======================

::

   % cd /usr/local/samba/sbin
   % ./samba -i -M single -d 3



Enable Change Log
=================

Copy changelog schema into
/usr/local/schema/private/ldap/slapd-samba/schema.

Enable Retro Changelog plugin on replica:

::

   % ldapmodify -h samba2.domain1.com -p 390 -x -D "cn=Directory Manager" -w Secret123
   dn: cn=Retro Changelog Plugin,cn=plugins,cn=config
   changetype: modify
   replace: nsslapd-pluginEnabled
   nsslapd-pluginEnabled: on
   -

Restart DS:

::

   % cd /usr/local/samba/private/ldap/slapd-samba
   % stop-slapd
   % start-slapd

::

   % ldapsearch -h samba2.domain1.com -p 390 -x -D "cn=Directory Manager" -w Secret123 -b "cn=changelog"

DNS
===

The DNS needs to be configured such that it points to both master and
replica. So if the master fails, the client will be able to find the
replica automatically.

::

   $ORIGIN domain1.com.
   $TTL 1W
   @               IN SOA  domain1.com. root.domain1.com. (
                                   2009070913   ; serial
                                   2D           ; refresh
                                   4H           ; retry
                                   6W           ; expiry
                                   1W )         ; minimum
                   IN NS   dns2

                   IN A    192.168.1.101
                   IN A    192.168.1.102

   dns2            IN A    192.168.1.100
   samba1          IN A    192.168.1.101
   samba2          IN A    192.168.1.102

   gc._msdcs       IN CNAME        samba1
   ff3b280e-6caa-11de-ab0a-e44b8f038cdc._msdcs     IN CNAME        samba1

   _gc._tcp        IN SRV 0 100 3268       samba1
   _gc._tcp.Default-First-Site-Name._sites IN SRV 0 100 3268       samba1

   _ldap._tcp.gc._msdcs    IN SRV 0 100 389        samba1
   _ldap._tcp.Default-First-Site-Name._sites.gc._msdcs     IN SRV 0 100 389 samba1

   _ldap._tcp              IN SRV 0 100 389        samba1
   _ldap._tcp              IN SRV 0 100 389        samba2

   _ldap._tcp.dc._msdcs    IN SRV 0 100 389        samba1
   _ldap._tcp.dc._msdcs    IN SRV 0 100 389        samba2

   _ldap._tcp.pdc._msdcs   IN SRV 0 100 389        samba1

   _ldap._tcp.ff3b2587-6caa-11de-ab0a-e44b8f038cdc IN SRV 0 100 389        samba1
   _ldap._tcp.ff3b2587-6caa-11de-ab0a-e44b8f038cdc IN SRV 0 100 389        samba2

   _ldap._tcp.ff3b2587-6caa-11de-ab0a-e44b8f038cdc.domains._msdcs          IN SRV 0 100 389 samba1
   _ldap._tcp.ff3b2587-6caa-11de-ab0a-e44b8f038cdc.domains._msdcs          IN SRV 0 100 389 samba2

   _ldap._tcp.Default-First-Site-Name._sites               IN SRV 0 100 389 samba1
   _ldap._tcp.Default-First-Site-Name._sites               IN SRV 0 100 389 samba2

   _ldap._tcp.Default-First-Site-Name._sites.dc._msdcs     IN SRV 0 100 389 samba1
   _ldap._tcp.Default-First-Site-Name._sites.dc._msdcs     IN SRV 0 100 389 samba2

   _kerberos._tcp          IN SRV 0 100 88         samba1
   _kerberos._tcp          IN SRV 0 100 88         samba2

   _kerberos._tcp.dc._msdcs        IN SRV 0 100 88 samba1
   _kerberos._tcp.dc._msdcs        IN SRV 0 100 88 samba2

   _kerberos._tcp.Default-First-Site-Name._sites   IN SRV 0 100 88 samba1
   _kerberos._tcp.Default-First-Site-Name._sites   IN SRV 0 100 88 samba2

   _kerberos._tcp.Default-First-Site-Name._sites.dc._msdcs IN SRV 0 100 88 samba1
   _kerberos._tcp.Default-First-Site-Name._sites.dc._msdcs IN SRV 0 100 88 samba2

   _kerberos._udp          IN SRV 0 100 88         samba1
   _kerberos._udp          IN SRV 0 100 88         samba2

   _kerberos-master._tcp           IN SRV 0 100 88         samba1
   _kerberos-master._tcp           IN SRV 0 100 88         samba2

   _kerberos-master._udp           IN SRV 0 100 88         samba1
   _kerberos-master._udp           IN SRV 0 100 88         samba2

   _kpasswd._tcp           IN SRV 0 100 464        samba1
   _kpasswd._tcp           IN SRV 0 100 464        samba2

   _kpasswd._udp           IN SRV 0 100 464        samba1
   _kpasswd._udp           IN SRV 0 100 464        samba2

   _kerberos               IN TXT  DOMAIN1.COM

See also `DNS <Obsolete:Samba_4_DNS>`__.

References
==========

-  [http://technet.microsoft.com/en-us/library/cc755994(WS.10).aspx How
   Active Directory Replication Topology Works]
-  `Setting up and testing Active Directory
   failover <http://www.improve.dk/blog/2008/03/02/setting-up-and-testing-active-directory-failover>`__
-  `Backup Domain
   Control <http://us1.samba.org/samba/docs/man/Samba-HOWTO-Collection/samba-bdc.html>`__
-  `Flexible Single Master of
   Operation <http://en.wikipedia.org/wiki/FSMO>`__
-  `Using Ntdsutil.exe to transfer or seize FSMO roles to a domain
   controller <http://support.microsoft.com/kb/255504>`__
-  [http://msdn.microsoft.com/en-us/library/cc964399(PROT.13).aspx
   Windows Server Protocols]
-  `Configuring Replication from the Command
   Line <http://www.centos.org/docs/5/html/CDS/ag/8.0/Managing_Replication-Configuring-Replication-cmd.html>`__
-  `Core Server Configuration Attributes Reference -
   cn=changelog5 <http://www.centos.org/docs/5/html/CDS/cli/8.0/Configuration_Command_File_Reference-Core_Server_Configuration_Reference-Core_Server_Configuration_Attributes_Reference.html#Configuration_Command_File_Reference-Core_Server_Configuration_Attributes_Reference-cnchangelog5>`__

`Category:Obsolete <Category:Obsolete>`__