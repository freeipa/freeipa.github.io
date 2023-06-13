Prerequisites
=============

::

   % yum install bind

Configuration
=============

If your DNS server is using chroot, make sure the following files are
placed in the appropriate location. Create zone data file
/var/named/domain1.com.zone.db:

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

   _kerberos               IN TXT DOMAIN1.COM

   _kerberos._tcp          IN SRV 0 100 88         ipa1
   _kerberos._udp          IN SRV 0 100 88         ipa1
   _kerberos-master._tcp   IN SRV 0 100 88         ipa1
   _kerberos-master._udp   IN SRV 0 100 88         ipa1
   _kpasswd._tcp           IN SRV 0 100 464        ipa1
   _kpasswd._udp           IN SRV 0 100 464        ipa1

   _ntp._udp               IN SRV 0 100 123        ipa1

Create reverse mapping file /var/named/192.168.1.rev:

::

   $ORIGIN 1.168.192.in-addr.arpa.
   $TTL    1W
   @                       IN SOA  domain1.com. root.domain1.com. (
                                   01              ; serial
                                   2D              ; refresh
                                   4H              ; retry
                                   6W              ; expiry
                                   1W )            ; minimum

                           IN NS                   dns1.domain1.com.

   100                     IN PTR                  dns1.domain1.com.
   101                     IN PTR                  ipa1.domain1.com.
   102                     IN PTR                  ipa2.domain1.com.
   103                     IN PTR                  client1.domain1.com.

Set file ownership:

::

   % chown named.named /var/named/domain1.com.zone.db
   % chown named.named /var/named/192.168.1.rev

Edit /etc/named.conf:

::

   options {
           #listen-on port 53 { 127.0.0.1; };
           #listen-on-v6 port 53 { ::1; };
           #allow-query     { localhost; };
           ...
   };

   zone "domain1.com." IN {
           type master;
           file "domain1.com.zone.db";
   };

   zone "1.168.192.in-addr.arpa." IN {
           type master;
           file "192.168.1.rev";
   };

Restart DNS:

::

   % service named restart

Verification
============

::

   % dig _kerberos.example.com TXT @localhost
   ...
   ;; ANSWER SECTION:
   _kerberos.example.com.  86400   IN  TXT "EXAMPLE.COM"
   ...

::

   % dig _ldap._tcp.example.com SRV @localhost
   ...
   ;; ANSWER SECTION:
   _ldap._tcp.example.com. 86400   IN  SRV 0 100 389 ipa.example.com.
   ...

::

   % nslookup ipa.example.com
   % nslookup ipa-client.example.com

   % nslookup 192.168.1.100
   % nslookup 192.168.1.101

References
==========

-  `DNS <http://freeipa.org/page/InstallAndDeploy#DNS>`__
-  `Implementing FreeIPA in a mixed
   Environment <http://www.freeipa.org/page/Implementing_FreeIPA_in_a_mixed_Environment_(Windows/Linux)_-_Step_by_step>`__

`Category:Obsolete <Category:Obsolete>`__
