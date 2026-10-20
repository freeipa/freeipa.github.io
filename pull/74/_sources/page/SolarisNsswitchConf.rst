SolarisNsswitchConf
===================

The following is a sample /etc/nsswitch.conf file for Solaris IPA
clients.

Back to `Configuring UNIX Clients <ConfiguringUnixClients>`__

::

   #
   # /etc/nsswitch.conf:
   #


   passwd:     files ldap nis
   group:      files ldap nis

   # You must also set up the /etc/resolv.conf file for DNS name
   # server lookup.  See resolv.conf(4).
   hosts:      files dns

   # Note that IPv4 addresses are searched for in all of the ipnodes databases
   # before searching the hosts databases.
   ipnodes:   files dns

   networks:   files nis
   protocols:  files nis
   rpc:        files nis
   ethers:     files nis
   netmasks:   files nis
   bootparams: files
   publickey:  files
   # At present there isn't a 'files' backend for netgroup;  the system will 
   #   figure it out pretty quickly, and won't use netgroups at all.
   netgroup:   files nis
   automount:  files ldap nis
   aliases:    files
   services:   files nis
   printers:   user files

   auth_attr:  files nis
   prof_attr:  files nis
   project:    files nis