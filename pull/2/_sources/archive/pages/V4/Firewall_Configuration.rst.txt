Overview
--------

The FreeIPA server installer goes through a lot of effort to setup a
variety of services to get the server fully functional after running a
single command. Unfortunately, the server installation script has not
ever attempted to create netfilter rules.

`FirewallD <https://fedoraproject.org/wiki/FirewallD>`__ is a relatively
new service that is a front-end for netfilter. It has a rich concept of
zones, services, and interfaces that makes configurations easy to setup
and administer. For our purposes, there are two important features.
First, FirewallD has a DBus interface that allows reliable reloading and
manipulation without interfering with existing connections or other user
policies. Second, FirewallD uses standardized XML policy files. The
greatest problem with modifying iptables-save files is that it is
non-trivial to parse them or properly insert rules. (There's also a
great variety in where and how users store their iptables-save files,
complicating both the FreeIPA integration UI and implementation.)

By using FirewallD, we will be able to offer users the server
install-time option of enabling a FreeIPA policy that covers all of the
constituent services.

Since a properly configured firewall is necessary for FreeIPA clients
and replicas to operate, it makes more sense to configure FirewallD by
default. Therefore, the firewall configuration will automatically
proceed during ipa-server-install unless a user passes the
"--no-firewall" option.



Use Cases
---------

Samantha is installing FreeIPA server on a Linux system that has
NetworkManager and FirewallD (with both running). She wants to get the
server fully running using the ipa-server-install script, including
firewall configuration. Since ipa-server-install will configure
FirewallD by default, she doesn't have to pass any extra configuration
options.

Dan is installing FreeIPA on a Linux system that has NetworkManager and
FirewallD. His server is connected to a network that has some portions
which are trusted and others that are not. He doesn't want the FreeIPA
exposed to the untrusted portions and needs to setup more complex rules
(called rich-rules in FirewallD). He passes the "--no-firewall" option
to ipa-server-install. The installation program detects that FirewallD
is active and prints out the FirewallD services needed and leaves the
implementation to Dan.

Rebecca is installing FreeIPA on a system without FirewallD.
Ipa-server-install detects that FirewallD is not running and prints out
a message similar to the current situation, which informs the user of
the needed protocols and ports.

Design
------

The ipa-server-install script will get a new option *--no-firewall*. If
this option is not passed, ipa-server-install will attempt to configure
FirewallD, and if unsuccessful will fall-back to printing generic
information about which protocols and ports should be allowed.

Example: ipa-server-install --setup-dns --forwarder=192.168.0.2
--forwarder=192.168.0.3 --no-firewall

Implementation
--------------

-  New Python dependencies: dbus module and lxml package. These are
   commonly found on most distributions in standard/minimal installs,
   including on both RHEL 7 (beta) and Fedora 20.

-  Service dependencies: NetworkManager.service and firewalld.service.
   These are optional dependencies. If only NM is missing,
   ipa-server-install will configure as much of FirewallD as possible,
   but the user will have to run a command after installation (to
   determine the proper FirewallD zone). If FirewallD is not present,
   freeipa-server-install will fall back into similar behavior where it
   prints out information on what firewall protocols and ports are
   needed but leaves the implementation up to the user.

-  One downside to FirewallD is that there is currently no DBus
   interface to modify the persistent firewall configuration. Therefore,
   it is necessary to write (or edit) the FirewallD zone configuration
   XML files, which is why lxml is needed. Generally, most users will be
   using one of the default, built-in zones ("public" is the default).
   In that case, we will have to either copy or preferably serialize the
   running zone configuration to a XML file in /etc/firewalld/zones/,
   including the FreeIPA rules. On the other hand, if the user is
   running a custom zone then the XML file will already exist in
   /etc/firewalld/zones/, and it only needs to be modified to include
   the FreeIPA rules.

-  This firewall option will integrate properly with the existing
   FirewallD configuration, including detecting already enabled services
   that FreeIPA needs. Any customized FirewallD policy will have a
   customized XML zone file in /etc/firewalld/services/, and that will
   be easy to modify safely using lxml. Uncustomized zones can be copied
   from /usr/lib/firewalld/services/ into /etc/firewalld/zones/ and
   modified from there.

-  Firewall configuration will include robust exception handling and
   logging. Modifications will never be left in a partial state. In the
   event of an unrecoverable error, it will fall back to printing the
   traditional message about which protocols and ports are needed.

-  The firewall configuration module takes an explicit list of services
   to enable. That way any disabled services aren't mistakeningly
   allowed through the firewall. Additionally, it also makes it easy to
   add new services to the firewall during upgrades.

-  Code: ipaserver/install/firewallutils.py is new and
   ipaserver/install/tools/ipa-server-install will be modified.

-  Documentation: manpages, user guide.

.. _dbus_method_overview:

DBus Method Overview
~~~~~~~~~~~~~~~~~~~~

Firewall configuration uses NetworkManager and FirewallD DBus methods.
Interaction with DBus includes a bus address and a method call. Here is
an overview of the addresses and methods used.

The object path

The format is

-  Task

   -  bus
   -  method, args
   -  Description

NetworkManager
^^^^^^^^^^^^^^

-  Enumerate network interfaces.

   -  org.freedesktop.NetworkManager
   -  GetDevices()
   -  ipa-server-install detects or prompts the user for the proper IP
      address to use during installation. FirewallD operates on
      interfaces, not addresses, so it's necessary to determine which
      interface has the proper IP address.

-  Get the IP address of each network interface.

   -  org.freedesktop.NetworkManager.Device
   -  Get(), Ip4Address and IP6Address
   -  This continues from the previous task to actually get the address
      of each interface.

FirewallD
^^^^^^^^^

-  Find the zone that has the interface obtained from NetworkManager.

   -  org.fedoraproject.FirewallD1
   -  getZoneOfInterface(), <>
   -  FirewallD configuration happens to zones. This finds the zone
      associated with the correct interface.

-  Find the default zone if the interface is not attached to a defined
   zone.

   -  org.fedoraproject.FirewallD1
   -  getDefaultZone()
   -  Default FirewallD configurations do not attach interfaces to any
      zone. Instead, there is a default zone, which includes all
      interfaces. Fallback to the default zone if the interface isn't
      explicitly attached to another zone.

-  Verify that all services are available that are needed.

   -  org.fedoraproject.FirewallD1
   -  listServices()
   -  FirewallD comes with all of the necessary service definitions for
      FreeIPA. They should always be present, but the situation may
      arise where a user or FirewallD packager may have deleted them. If
      any services are missing, they will need to be created with XML
      files in /etc/firewalld/services/.

-  Reload FirewallD after configuration.

   -  org.fedoraproject.FirewallD1
   -  reload()
   -  After modifying the persistent FirewallD configuration, it is
      necessary to issue a reload call to make the configuration active.



Feature Management
------------------

CLI
~~~

There are no dedicated commands. This only adds the "--no-firewall"
option to ipa-server-install.



Major configuration options and enablement
==========================================

Any commands to enable/disable the feature or turn on/off its parts?

"--no-firewall" will stop all firewall configuration.

Replication
-----------

No impact.

.. _restore_and_uninstall:

Restore and Uninstall
---------------------

Firewall configuration will either modify or create a zone file in
/etc/firewalld/zones/. If a file was modified, the original will be
saved in /var/lib/ipa/sysrestore/ for restore. Otherwise, the zone file
can be removed. Note that a user may have made subsequent modifications
to the zone policy. If any user modifications are detected, only the
FreeIPA firewall configuration will be removed.



Updates and Upgrades
--------------------

If new FreeIPA services are available during upgrades (like DRM and
TPS), the firewall configuration module allows those new services to be
added easily.

Dependencies
------------

Python package lxml and module dbus. Most distributions already include
them even in minimal installs.



External Impact
---------------

None



Backup and Restore
------------------

None



Test Plan
---------

I will need to investigate the CI environment, but here are the basic
tests that I'm considering at this point.

-  lxml, dbus, or both Python modules missing.
-  Authorization denials to DBus system bus.
-  Missing NetworkManager, FirewallD, or both services.
-  Invalid, missing, or inaccessible /etc/firewalld/zones/*.

-  Fedora 19+ and RHEL 7+ platforms.
-  Python 2.7.

-  Partial FirewallD policy configuration.

Test scenarios that will be transformed to test cases for FreeIPA
Continuous Integration during implementation or review phase.



RFE Author
==========

Justin Brown <justin.brown ON fandingo PERIOD org>
