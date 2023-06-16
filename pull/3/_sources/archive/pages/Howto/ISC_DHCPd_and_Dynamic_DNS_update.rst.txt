Making FreeIPA based `DNS <DNS>`__ work with DDNS and dhcpd server
requires a couple of steps

A typical dhcpd.conf with working DDNS - **without FreeIPA** will have a
section looking something like the following

.. code:: bash

   # this file includes definition of DDNS_UPDATE key
   include "/etc/dhcp/ddns.key";

   # replace xxxxxxx as appropriate
   zone xxxxxxx.com. {
     primary 127.0.0.1;
     key DDNS_UPDATE;
   }

   # replace 2.0.10 with your reverse ip
   zone 2.0.10.in-addr.arpa. {
     primary 127.0.0.1;
     key DDNS_UPDATE;
   }

The corresponding part of named.conf will look like this (not complete
file, this is after the options section):

.. code:: bash

   # note that this file is identical to /etc/dhcp/ddns.key
   # not experimented with pointing to same file which
   # could be tricky if you are chroot
   include "/var/named/named.keys";

   zone "xxxxxxx.com" {
       type master;
       file "masters/db.xxxxxxx.com";
       allow-update { key DDNS_UPDATE; };
   };

   zone "2.0.10.in-addr.arpa" {
       type master;
       file "masters/rev.2.0.10";
       allow-update { key DDNS_UPDATE; };
   };

Assuming you started with a working DDNS setup your files will look
something similar. Once you install FreeIPA this will no longer work.
This is because the database is now moved to inside the
`LDAP <Directory_Server>`__ server. The zone section is now deleted and
moved into `LDAP <Directory_Server>`__.

You can update your zone definition inside FreeIPA and add

.. code:: bash

   grant DDNS_UPDATE wildcard * ANY;

to the zone definition.

`Category:How to <Category:How_to>`__
