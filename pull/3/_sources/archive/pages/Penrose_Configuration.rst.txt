Configuration
=============

::

   % cd /opt/vd-server-2.0
   % cp -r samples/ipa partitions

Configure the hostname, suffix, and credentials in these files:

-  /opt/vd-server-2.0/partitions/ipa/DIR-INF/connections.xml
-  /opt/vd-server-2.0/partitions/ipa/DIR-INF/sources.xml

Start Penrose:

::

   % service vd-server start

`Category:Obsolete <Category:Obsolete>`__
