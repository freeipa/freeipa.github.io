Penrose_Installation
====================

Prerequisites
=============

.. code-block:: text

   % yum install ant java-1.6.0-openjdk-devel

Installation
============

.. code-block:: text

   % git clone git://github.com/endisd/penrose-server.git
   % cd penrose-server
   % git checkout -b development origin/development
   % ant clean dist install

.. code-block:: text

   % cd /opt/vd-server-2.0/bin
   % ./vd-service.sh install
   % chmod +x /etc/init.d/vd-server

`Category:Obsolete <Category:Obsolete>`__