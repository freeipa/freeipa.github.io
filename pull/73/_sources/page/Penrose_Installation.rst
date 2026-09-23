Penrose_Installation
====================

Prerequisites
=============

::

   % yum install ant java-1.6.0-openjdk-devel

Installation
============

::

   % git clone git://github.com/endisd/penrose-server.git
   % cd penrose-server
   % git checkout -b development origin/development
   % ant clean dist install

::

   % cd /opt/vd-server-2.0/bin
   % ./vd-service.sh install
   % chmod +x /etc/init.d/vd-server

`Category:Obsolete <Category:Obsolete>`__