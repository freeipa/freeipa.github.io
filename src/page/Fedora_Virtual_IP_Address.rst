Fedora_Virtual_IP_Address
=========================



Host File
=========

.. code-block:: text

   192.168.1.100 ipa.example.com ipa
   192.168.1.200 samba.example.com samba



Virtual Address
===============

Select an existing network interface (e.g. eth0). Create a new network
interface as its child:

.. code-block:: text

   % cd /etc/sysconfig/network-scripts
   % cp ifcfg-eth0 ifcfg-eth0:0

Edit ifcfg-eth0:0:

.. code-block:: text

   DEVICE=eth0:0
   IPADDR=<new IP address>
   # BOOTPROTO=dhcp

Restart networking service:

.. code-block:: text

   % service network restart

`Category:Obsolete <Category:Obsolete>`__