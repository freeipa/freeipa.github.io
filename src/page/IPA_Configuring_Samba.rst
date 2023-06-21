IPA_Configuring_Samba
=====================

Overview
========

This document describes the procedure to install and configure Samba for
the integrated environment with IPA.



Installing Samba
================

Install Samba 4. The installation directory will be referred to as
SAMBA_HOME. Replace any occurence of SAMBA_HOME in this document with
the actual path.



Configuring Samba
=================

Create SAMBA_HOME/etc/smb.conf:

::

   [globals]
           netbios name     = samba1
           workgroup        = SAMBADOMAIN
           realm            = SAMBA.EXAMPLE.COM
           server role      = domain controller
           system:anonymous = yes
           sid generator    = backend

   [netlogon]
           path = SAMBA_HOME/var/locks/sysvol/samba.example.com/scripts
           read only = no

   [sysvol]
           path = SAMBA_HOME/var/locks/sysvol
           read only = no



Provisioning Samba Backend
==========================

Execute the following command to provision Samba backend:

::

   % cd SAMBA_HOME
   % export PYTHONPATH=SAMBA_HOME/lib64/python2.6/site-packages
   % share/setup/provision --server-role="domain controller" \
    --domain=SAMBADOMAIN --realm=SAMBA.EXAMPLE.COM \
    --adminpass=Secret123 \
    --ldap-backend-type=fedora-ds \
    --root=root --ldapadminpass=Secret123 \
    --host-name=buildsamba02 --host-ip=127.0.0.1 \
    --slapd-path=/usr/sbin/ns-slapd --setup-ds-path=/usr/sbin/setup-ds.pl



Starting DS Instance
====================

::

   % cd SAMBA_HOME
   % private/ldap/slapd-samba4/start-slapd



Enabling DS Change Log
======================

Enable DS Change Log:

::

   % ldapmodify -H ldapi://%2Fusr%2Flocal%2Fsamba%2Fprivate%2Fldap%2Fldapi -x -D "cn=Manager,dc=samba,dc=example,dc=com" -w Secret123
   dn: cn=Retro Changelog Plugin,cn=plugins,cn=config
   changetype: modify
   replace: nsslapd-pluginEnabled
   nsslapd-pluginEnabled: on
   -

Restart DS:

::

   % cd SAMBA_HOME
   % private/ldap/slapd-samba4/stop-slapd
   % private/ldap/slapd-samba4/start-slapd



Installing Syncback Module
==========================

Copy syncback.so into SAMBA_HOME/modules/ldb directory.

Edit private/sam.ldb:

::

   % cd SAMBA_HOME
   % bin/ldbedit -H private/sam.ldb -b ""

Search for @MODULES entry, add the syncback module at the beginning of
the @LIST as follows:

::

   dn: @MODULES
   @LIST: syncback,resolve_oids,rootdse,lazy_commit,paged_results,ranged_results,anr,serve
    r_sort,asq,extended_dn_store,extended_dn_in,rdn_name,objectclass,descriptor,a
    cl,samldb,password_hash,operational,kludge_acl,schema_load,instancetype,exten
    ded_dn_out_fds,show_deleted,new_partition,partition
   distinguishedName: @MODULES



Starting Samba
==============

::

   % cd SAMBA_HOME
   % sbin/samba -i -M single



Enabling Clear Text Password
============================

Check the current password policy:

::

   % ldapsearch -x -D "cn=Administrator,cn=Users,dc=samba,dc=example,dc=com" \
    -w Secret123 -b "dc=samba,dc=example,dc=com" -s base pwdProperties
   dn: dc=samba,dc=example,dc=com
   pwdProperties: 1

Enable storing clear text password:

::

   % ldapmodify -x -D "cn=Administrator,cn=Users,dc=samba,dc=example,dc=com" \
    -w Secret123
   dn: dc=samba,dc=example,dc=com
   changetype: modify
   replace: pwdProperties
   pwdProperties: 17

`Category:Obsolete <Category:Obsolete>`__