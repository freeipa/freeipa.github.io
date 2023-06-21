Server_Roles
============

Overview
--------

The implementation of a `dynamic
management <V4/Manage_replication_topology_4_4>`__ of replication
topology uncovered a need for an additional API for querying for a
presence and status of various services (DNS, CA, KRA) on individual
nodes. FreeIPA holds a list of configured services per master in the
``cn=masters,cn=ipa,cn=etc,$SUFFIX`` subtree.

This is however more of an implementation detail than an information
readily consumable by administrators, since there is many-to-one mapping
between systemd services and a high-level functionality provided by
them, e.g. a FreeIPA DNS server functionality is provided by 2 systemd
services set up by ``ipa-dns-install``.

That's why this document introduces the concept of server roles as an
user-facing abstraction providing high-level information about services
running on particular FreeIPA master.



Use Cases
---------

-  as an administrator I want a simple and quick way to determine which
   FreeIPA master hosts Certification Authority/DNS server/Trust
   Controller, etc. I also want to be able to quickly determine which CA
   master manages certificate renewals
-  I also want to see this information when managing my topology through
   WebUI using `Topology
   graph <V4/Manage_replication_topology_4_4#Topology_graph>`__
-  as an administrator, I want to be able to migrate CA renewal master
   to a different server with a single command.

Design
------



Server Roles
----------------------------------------------------------------------------------------------

The systemd-service -> server role mapping will be done entirely by the
python code which will consume existing service entries and map them to
a list of human-readable server roles which will be displayed to a user
with sufficient privilege level, e.g.:

+---------------------+-----------------------------------------------+
| Role                | Services                                      |
+=====================+===============================================+
| CA                  | CA                                            |
+---------------------+-----------------------------------------------+
| DNS server          | DNS, DNSKeySync                               |
+---------------------+-----------------------------------------------+
| Key recovery agent  | KRA                                           |
+---------------------+-----------------------------------------------+
| AD trust controller | ADTRUST, CIFS service principal is a member   |
|                     | of "adtrust agents" sysaccounts group         |
+---------------------+-----------------------------------------------+
| AD trust agent      | host is a member of "adtrust agents"          |
|                     | sysaccounts group                             |
+---------------------+-----------------------------------------------+

Additional implicit ``IPA master`` role will be defined which will group
services which have to be running on each IPA master to provide the most
basic functionality (access to LDAP backend, Kerberos authentication,
HTTP server). This role will not be exposed in the UI.

Each role object will exist in these three states:

+------------+--------------------------------------------------------+
| State      | meaning                                                |
+============+========================================================+
| available  | the FreeIPA (sub)package providing the role was        |
|            | installed on the master                                |
+------------+--------------------------------------------------------+
| configured | all services comprising the role were added to the     |
|            | entry by installer                                     |
+------------+--------------------------------------------------------+
| enabled    | the service entries are marked as enabled in the LDAP  |
+------------+--------------------------------------------------------+

There is currently no plan to implement manipulation service entries by
adding/removing a server role. This feature may be considered in the
future when we have the ability to perform tasks on a local system based
on the information present in LDAP backend. The checks for role
availability will also be implemented later during modularization
efforts. The initial implementation will thus only display only
enabled/configured roles.



Server Attributes
----------------------------------------------------------------------------------------------

Each server may have one or more attributes described a more specific
functionality provided by one or more masters fulfilling the role: e. g.
among multiple CA masters, there is one which is responsible for
certificate renewals and one master is responsible for genration of
CRLs. The renewal master functionality is expressed by setting
``ipaConfigString=caRenewalMaster`` attribute in the CA LDAP entry of
the corresponding server container, so this can be considered a more
specific attribute of a general CA server role.

CRL master functionality is currently not expressed in the LDAP service
container and can be inferred only from examination of Dogtag
configuration files. Since e.g. search for CRL master would require
accessing local filesystems on all masters and examining Dogtag
configuration, the "CRL master" attribute will not be added until there
is an access to consumable LDAP attribute(s).

An API will be provided to set the server attribute to True/False on one
or more masters. However, this will be currently supported only for
server attributes which are configurable purely in LDAP (e.g. CA renewal
master). Attributes which require also local configuration of the host
system (e.g. `DNSSec
keymaster <Howto/DNSSEC#Migrate_DNSSEC_master_to_another_IPA_server>`__
) will be read-only.

In the case that the attribute is singular, i.e. can be set only on a
single master in the topology, the original master will have the
attribute unset and a warning will be issued to the user. The code will
also check whether the accepting server already provides the required
role, e.g. setting 'CA renewal master' attribute on a server without CA
role will raise an error.

Implementation
----------------------------------------------------------------------------------------------

The feature will be implemented in the following steps:

-  A object hierarchy abstracting the server role/attribute concepts.
   These objects will

   -  store the service list->role mapping and ``ipaConfigString``
      value->server attribute mapping
   -  convert these mappings into LDAP search filters used when looking
      up the role/attribute and its status
   -  query the search results to determine whether the role is
      enabled/disabled
   -  perform check whether the package providing the role is installed
      on the master, e.g. by testing for presence of files provided by
      the package

-  injecting the interface to existing ``server-{show,find}`` and
   ``config-{show,mod}`` commands to utilize the new functionality
-  new command (``server-mod``) which will be used to edit attributes of
   specified server. This command will be also used by `DNS
   Location <http://www.freeipa.org/page/V4/DNS_Location_Mechanism>`__
   to set the location of IPA master.
-  new plugin which will display the status of specified role on
   different masters and also a list of DNS records associated with the
   specific role. The latter information will be consumed during
   creation of
   `location-specific <http://www.freeipa.org/page/V4/DNS_Location_Mechanism>`__
   DNS records for different services.
-  new commands (``serverrole-{show,find}``) to list the status of
   server roles on a IPA master and to implement substring search among
   the roles.
-  reimplementation of current code querying for service presence using
   the server role API. This includes e.g. service checking code in
   installers and check for last remaining services and renewal master
   migration in
   `server_del <V4/Manage_replication_topology_4_4#server_del>`__ API
   method.



Feature Management
------------------



Web UI~

WebUI will feature a new ``IPA server -> Topology -> Server roles``
section, which will display a list of all server roles. Each role will
in turn display all server on which it is available, configured, and
enabled. Additionally, the enabled roles (and role attributes) shall be
displayed among the details about the IPA master accessed from
``IPA server -> Topology -> IPA servers`` tab.

Additionally, ``IPA server -> Configuration`` will displayed list of
role attributes along the masters on which the attribute is enabled. A
sufficiently privileged used should be able to change the attribute to a
different master(s).

The topology graph introduced in FreeIPA 4.3 will display the roles of
each master after clicking the node. This may be implemented later
during 4.5 development timeframe.

CLI



Enhanced commands
^^^^^^^^^^^^^^^^^

``server-show``
   The command will print out the list of enabled roles on the master.

``server-find``
   new option ``--servrole`` will enable searching servers having the
   specified role(s) enabled.

::

   ipa server-find --servrole="DNS server" --servrole "CA server"

``config-show``
   the command will display the list of IPA masters and CA servers. CA
   renewal master will also be printed out.

``dnsconfig-show``
   the command will display DNSSec keymaster and list of DNS servers

``trustconfig-show``
   the command will display list of AD trust controllers and agents

``vaultconfig-show``
   the command will display list of KRA servers

``config-mod``
   the command will be enhanced by the ability to set CA renewal master
   to other CA server while unsetting this attribute on the original
   master:

::

   ipa config-mod --ca-renewal-master-server=server1.example.com



New Commands
^^^^^^^^^^^^

``server-role-show FQDN "ROLE_NAME"``
   show the status of role ``"ROLE_NAME"`` of IPA master ``FQDN``

``server-role-find --server FQDN --role "ROLE_NAME" --status "enabled|configured|absent"``
   search for role with ``"substring"`` in name on master ``FQDN`` and
   display its status. When no FQDN and role name are specifed, will
   return status of all recognized roles on all servers. ``--status``
   option can be optionally used to filter the result by role status.

Upgrade
-------

Since there are no changes to LDAP structure/schema, no special upgrade
procedure is necessary.



How to Test
-----------

-  see all roles active on a master:

::

   # ipa server-show ipasrv1.example.com
     Server name: ipasrv1.example.com
     Managed suffixes: domain, ca
     Min domain level: 0
     Max domain level: 1
     Enabled Roles: AD Trust Controller, CA server, DNS server, KRA server

-  find all DNS servers

::

   # ipa server-find --servrole "DNS server"
   --------------------
   2 IPA servers matched
   --------------------
     Server name: ipasrv1.example.com
     Managed suffixes: domain, ca
     Min domain level: 0
     Max domain level: 1
     Enabled Roles: AD Trust Controller, CA server, DNS server, KRA server

     Server name: ipasrv3.example.com
     Managed suffixes: domain
     Min domain level: 0
     Max domain level: 1
     Enabled Roles: DNS server
   ----------------------------
   Number of entries returned 2
   ----------------------------

-  find a CA renewal master

::

   # ipa config-show | grep "CA renewal master"
   IPA CA renewal master: ipasrv1.example.com

-  find DNSSec key master

::

   # ipa dnsconfig-show  | grep 'DNSSec key master'
   IPA DNSSec key master: ipasrv3.example.com

-  switch CA master "ipasrv2.example.com" to a renewal master

::

   # ipa config-mod --ca-renewal-master-server ipasrv2.example.com
    Maximum username length: 32
     Home directory base: /home
     Default shell: /bin/sh
     Default users group: ipausers
     Default e-mail domain: example.com
     Search time limit: 2
     Search size limit: 100
     ...
     IPA CA renewal master: ipasrv2.example.com

-  try to switch CA renewal master to a server without CA role:

::

   # ipa config-mod --renewal-master ipasrv3.example.com
   ipa: ERROR: 'ipasrv3.example.com' cannot be set as CA renewal master: Role 'CA' not configured/enabled

-  show the status of 'DNS server' role on server ipasrv4.example.com
   which lacks freeipa-server-dns subpackage

::

   # ipa server-role-show ipasrv4.example.com --role 'DNS server'
     Server: ipasrv4.example.com
     Role name: DNS server
     Role status: absent

-  configure DNS on ipasrv4.example.com using ``ipa-dns-install`` and
   check the 'DNS server' role status

::

   # ipa server-role-show ipasrv4.example.com --role 'DNS server'
     Server: ipasrv4.example.com
     Role name: DNS server
     Role status: enabled

-  show the status of 'DNS sevrer' (typo) role on DNS server
   ipasrv3.example.com

::

   # ipa serverrole-show ipasrv4.example.com --role 'DNS sevrer'
   ipa: ERROR: 'DNS sevrer': role not found

-  search for status of all roles on ipasrv1.example.com

::

   # ipa server-role-find --server ipasrv1.example.com
   --------------------
   5 IPA server roles matched
   --------------------
   Role name: AD Trust Controller
   Server: ipasrv1.example.com
   Role status: absent

   Role name: CA server
   Server: ipasrv1.example.com
   Role status: enabled
    
   ...

-  search for all servers which are not AD trust controller

::

   # ipa server-role-find --role "AD trust controller" --status "absent"
   --------------------
   1 IPA server role matched
   --------------------
   Role name: AD Trust Controller
   Server: ipasrv2.example.com
   Role status: absent

   Role name: AD Trust Controller
   Server: ipasrv3.example.com
   Role status: absent
   ...



Test Plan
---------

-  basic CRUD tests for the new commands
-  existing CI tests for component installation (DNS(SEC), CA, KRA) can
   test whether the corresponding role is added to the server.

`Server Roles V4.4 test plan <V4/Server_Roles/Test_Plan>`__