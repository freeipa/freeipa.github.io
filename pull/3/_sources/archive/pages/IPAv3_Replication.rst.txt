.. _prepare_replica:

Prepare Replica
===============

Execute the following on the master:

::

   % ipa-replica-prepare ipa2.domain1.com

::

   % scp /var/lib/ipa/replica-info-ipa2.domain1.com.gpg root@ipa2.domain1.com:/var/lib/ipa/

.. _configure_replica:

Configure Replica
=================

Execute the following on the replica:

::

   % ipa-replica-install /var/lib/ipa/replica-info-ipa2.domain1.com.gpg

If you need to uninstall replica:

::

   % ipa-server-install --uninstall

.. _enable_change_log:

Enable Change Log
=================

Enable Retro Changelog plugin on replica:

::

   % ldapmodify -h ipa2.domain1.com -p 389 -x -D "cn=Directory Manager" -w Secret123
   dn: cn=Retro Changelog Plugin,cn=plugins,cn=config
   changetype: modify
   replace: nsslapd-pluginEnabled
   nsslapd-pluginEnabled: on
   -

Restart DS:

::

   % service dirsrv restart

::

   % ldapsearch -h ipa2.domain1.com -p 389 -x -D "cn=Directory Manager" -w Secret123 -b "cn=changelog"

DNS
===

Edit zone data file /var/named/domain1.com.zone.db:

::

   $ORIGIN domain1.com.
   $TTL    1W
   @                       IN SOA  domain1.com. root.domain1.com. (
                                   01              ; serial
                                   2D              ; refresh
                                   4H              ; retry
                                   6W              ; expiry
                                   1W )            ; minimum

                           IN NS                   dns1

   dns1                    IN A                    192.168.1.100
   ipa1                    IN A                    192.168.1.101
   ipa2                    IN A                    192.168.1.102
   client1                 IN A                    192.168.1.103

   _ldap._tcp              IN SRV 0 100 389        ipa1
   _ldap._tcp              IN SRV 0 100 389        ipa2

   _kerberos               IN TXT DOMAIN1.COM

   _kerberos._tcp          IN SRV 0 100 88         ipa1
   _kerberos._tcp          IN SRV 0 100 88         ipa2

   _kerberos._udp          IN SRV 0 100 88         ipa1
   _kerberos._udp          IN SRV 0 100 88         ipa2

   _kerberos-master._tcp   IN SRV 0 100 88         ipa1
   _kerberos-master._tcp   IN SRV 0 100 88         ipa2

   _kerberos-master._udp   IN SRV 0 100 88         ipa1
   _kerberos-master._udp   IN SRV 0 100 88         ipa2

   _kpasswd._tcp           IN SRV 0 100 464        ipa1
   _kpasswd._tcp           IN SRV 0 100 464        ipa2

   _kpasswd._udp           IN SRV 0 100 464        ipa1
   _kpasswd._udp           IN SRV 0 100 464        ipa2

   _ntp._udp               IN SRV 0 100 123        ipa1
   _ntp._udp               IN SRV 0 100 123        ipa2

See also `DNS <Obsolete:IPAv3_DNS>`__.

References
==========

-  `Setting up Multi-Master
   Replication <http://www.freeipa.org/page/InstallAndDeploy#Setting_up_Multi-Master_Replication>`__

`Category:Obsolete <Category:Obsolete>`__
