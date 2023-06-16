Overview
--------

**Old behavior:**

-  --name-server address was used as SOA MNAME attribute and the NS
   record pointing to this server address was created in zone apex. For
   relative/inside zone server name, --ip-address option was required,
   which created an additional A record.
-  User has to write an admin email (SOA RNAME) during zone creation.

We should use the replicas with installed DNS as NS servers in zone
apex. The hostname of NS records has to be A/AAAA resolvable. Those
replicas have already configured A/AAAA records in IPA managed DNS and
they are reachable (otherwise there is bigger problem than
miss-configured DNS).

This makes ``--ip-address option`` in ``dnszone-add`` command
deprecated.

Param ``--name-server`` is autofilled.

Param ``--admin-email`` is autofilled with default value **hostmaster**.

Required by ticket:
`#4149 <https://fedorahosted.org/freeipa/ticket/4149>`__.

Hits tickets: `#3344 <https://fedorahosted.org/freeipa/ticket/3344>`__,
`#3343 <https://fedorahosted.org/freeipa/ticket/3343>`__



Use Cases
---------

-  User wants to create zone/reverse zone

   -  uses option ``--ip-address``
   -  uses option ``--name-server``
   -  uses option ``--admin-email``
   -  uses none of options listed above

-  User wants to install server (replica) with DNS
-  User wants to remove replica with DNS

Design
------

-  No schema changes required
-  API changes:

   -  ``dnszone-add`` automatically creates NS records in zone apex with
      all IPA DNS server hostnames
   -  DNS zone's param ``--admin-email`` (SOA RNAME) is autofilled with
      relative value: **hostmaster**, bind server appends the zone
      suffix automatically. Admin email validator allows to add a
      relative domain name.
   -  DNS zone's param ``--name-server`` is not used to create NS
      records anymore
   -  dnszone-add's param ``--ip-address`` is deprecated, value is
      ignored, param is presented in API due to compability with older
      clients



Feature Management
------------------

CLI
~~~

.. _dnszone_add:

dnszone-add
^^^^^^^^^^^

All hostnames of DNS capable replicas are inserted into the **zone apex
NS record**. User can modify those records using the
``dnsrecord-mod zone @ --ns-rec=hostname`` command.

If a server/replica where the command is executed has installed DNS,
**SOA MNAME** attribute is filled with the replica hostname (in absolute
form). Otherwise (the replica where the command is executed is not has
no DNS installed) the first DNS replica's hostname returned from LDAP
(lexicographic order) is used as SOA MNAME.

The SOA RNAME (admin email) attribute has default autofilled value
**hostmaster**. The default value is relative, bind will append zone
automatically.

Options:

-  ``--ip-address`` option is deprecated, value is ignored. If this
   option occurs in options, server will respond with warning:
   **"ip-address option is deprecated. Value will be ignored. "**

-  ``--admin-email`` option fill SOA MNAME. Validation/normalization is
   less restrictive and allows to add relative names.

-  ``--name-server`` option is not required, if user specify the
   name-server value, the value is used as SOA MNAME, no additional NS
   record of this value is added (only ipa servers are added as NS
   records). Server returns warning message: **"semantic of
   '--name-server' option was changed: the option is used only for
   setting up the SOA MNAME attribute.\nTo edit NS record(s) in zone
   apex, use command dnsrecord-mod [zone] @ --ns-rec=nameserver'."**

User specified nameserver is verified, if A/AAAA record(s) could be
resolved, otherwise exception is raised. To suppress this validation
option --force can be used.

**NOTE:** If user really needs to have SOA MNAME inside the zone, first,
the zone with default nameserver value has to be created, then A/AAAA
record of nameserver has to be added, and finally user can change SOA
MNAME with ``dnszone-mod --name-server=nameserver_in_zone`` command.
Second approach is use the --force option, and create A/AAAA record for
nameserver later.

.. _dnszone_mod:

dnszone-mod
^^^^^^^^^^^

Options:

-  ``--admin-email`` option: no changes
-  ``--name-server`` option: same behavior as in ``dnszone-add`` command

.. _ipa_server_installation_with_dns:

IPA server installation (with DNS)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Commands: ``ipa-server-install (ipa-replica-install) --setup-dns`` or
``ipa-setup-dns`` add replica hostname to each NS record in IPA managed
DNS zones.

.. _ipa_server_uninstallation_with_dns:

IPA server uninstallation (with DNS)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The IPA server hostname is removed from all NS records in IPA managed
DNS zones (including records outside of the zone apex).

UI
~~

-  Hide ``--ip-address`` field in adding new zone dialog
-  Hide ``--admin-email`` field in adding new zone dialog
-  Hide ``--name-server`` field in adding new zone dialog
-  Hide ``--force`` checkbox in adding new zone dialog



External Impact
---------------

Update documentation..



Test Plan
---------

.. _adding_zone:

Adding zone
~~~~~~~~~~~

-  IPA servers with DNS: ipa-dns1.example.com, ipa-dns2.example.com)
-  IPA servers without DNS: ipa.example.com

.. _add_zone_on_server_with_dns:

Add zone on server with DNS
^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Command: ``dnszone-add zone.test.``
-  Assumption: command executed on *ipa-dns2* server
-  Result: zone is created with values

   -  nsrecord: [ipa-dns1.example.com., ipa-dns2.example.com.]
   -  idnssoarname: hostmaster
   -  idnssoamname: ipa-dns2.example.com.

.. _add_zone_on_server_without_dns:

Add zone on server without DNS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Command: ``dnszone-add zone.test.``
-  Assumption: command executed on *ipa.example.com* server (non DNS)
-  Result: zone is created with values

   -  nsrecord: [ipa-dns1.example.com., ipa-dns2.example.com.]
   -  idnssoarname: hostmaster
   -  idnssoamname: ipa-dns1.example.com. # First nameserver is used

.. _add_zone_with_unresolvable_nameserver:

Add zone with unresolvable nameserver
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Command:
   ``dnszone-add zone.test. --name-server unresolvable.address.test.``
-  Result: exception raised

   -  Exception: NotFound: Nameserver unresolvable.address.test. does
      not have a corresponding A/AAAA record

.. _add_zone_with_resolvable_nameserver:

Add zone with resolvable nameserver
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Command:
   ``dnszone-add zone.test. --name-server resolvable.nameserver.test.``
-  Result: zone is created with values and warning

   -  nsrecord: [ipa-dns1.example.com., ipa-dns2.example.com.]
   -  idnssoarname: hostmaster
   -  idnssoamname: resolvable.nameserver.test.

.. _add_zone_with_relative_nameserver:

Add zone with relative nameserver
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Command: ``dnszone-add zone.test. --name-server relative``
-  Result: exception raised

   -  Exception: NotFound: Nameserver relative.zone.test. does not have
      a corresponding A/AAAA record

.. _add_zone_with_relative_nameserver_with___force_option:

Add zone with relative nameserver with --force option
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Command: ``dnszone-add zone.test. --name-server relative --force``
-  Result: zone is created with values and warning

   -  nsrecord: [ipa-dns1.example.com., ipa-dns2.example.com.]
   -  idnssoarname: hostmaster
   -  idnssoamname: relative
   -  warning: "'--name-server' is used only for setting up the SOA
      MNAME attribute.\nTo edit NS record(s) in zone apex, use command
      'dnsrecord-mod [zone] @ --ns-rec=nameserver'."

.. _add_zone_with_relative_nameserver_and_ip_address_old_client:

Add zone with relative nameserver and ip-address (old client)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Command:
   ``dnszone-add zone.test. --name-server relative --ip-address 172.16.70.1``
-  Result: exception raised

   -  Exception: NotFound: Nameserver relative.zone.test. does not have
      a corresponding A/AAAA record

-  Note: raise error: no such option --ip-address on new clients

.. _add_zone_with_relative_nameserver_and_ip_address_and___force_old_client:

Add zone with relative nameserver and ip-address and --force (old client)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Command:
   ``dnszone-add zone.test. --name-server relative --ip-address 172.16.70.1 --force``
-  Result: zone is created with values and warning

   -  nsrecord: [ipa-dns1.example.com., ipa-dns2.example.com.]
   -  idnssoarname: hostmaster
   -  idnssoamname: relative
   -  warning: "'--name-server' is used only for setting up the SOA
      MNAME attribute.\nTo edit NS record(s) in zone apex, use command
      'dnsrecord-mod [zone] @ --ns-rec=nameserver'."
   -  warning: "ip-address option is deprecated. Value will be ignored.
      "

-  Note: raise error: no such option --ip-address on new clients

.. _add_zone_with_relative_admin_email:

Add zone with relative admin-email
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Command: ``dnszone-add zone.test. --admin-email it-department``
-  Assumption: zone is created on *ipa-dns1.example.com.* server
-  Result: zone is created with values

   -  nsrecord: [ipa-dns1.example.com., ipa-dns2.example.com.]
   -  idnssoarname: it-department
   -  idnssoamname: ipa-dns1.example.com.

.. _modifying_zone:

Modifying zone
~~~~~~~~~~~~~~

-  Zone *zone.test.* exists with values:

   -  nsrecord: [ipa-dns1.example.com., ipa-dns2.example.com.]
   -  idnssoarname: hostmaster
   -  idnssoamname: ipa-dns1.example.com.

.. _modify_zone_with_unresolvable_nameserver:

Modify zone with unresolvable nameserver
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Command:
   ``dnszone-mod zone.test. --name-server unresolvable.address.test.``
-  Result: exception raised

   -  Exception: NotFound: Nameserver unresolvable.address.test. does
      not have a corresponding A/AAAA record

.. _modify_zone_with_resolvable_nameserver:

Modify zone with resolvable nameserver
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Command:
   ``dnszone-mod zone.test. --name-server resolvable.nameserver.test.``
-  Result: zone is modified with values

   -  nsrecord: [ipa-dns1.example.com., ipa-dns2.example.com.]
   -  idnssoarname: hostmaster
   -  idnssoamname: resolvable.nameserver.test.

.. _modify_zone_with_relative_nameserver_with_a_record_in_zone:

Modify zone with relative nameserver with A record in zone
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Command:
   ``dnszone-mod zone.test. --name-server relative-with-A-rec-in-zone``
-  Result: zone is modified with values

   -  nsrecord: [ipa-dns1.example.com., ipa-dns2.example.com.]
   -  idnssoarname: hostmaster
   -  idnssoamname: relative-with-A-rec-in-zone

.. _modify_zone_with_relative_nameserver_no_aaaaa_record_in_zone_with___force_option:

Modify zone with relative nameserver (no A/AAAA record in zone) with --force option
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Command: ``dnszone-mod zone.test. --name-server relative --force``
-  Result: zone is created with values and warning

   -  nsrecord: [ipa-dns1.example.com., ipa-dns2.example.com.]
   -  idnssoarname: hostmaster
   -  idnssoamname: relative
   -  warning: "'--name-server' is used only for setting up the SOA
      MNAME attribute.\nTo edit NS record(s) in zone apex, use command
      'dnsrecord-mod [zone] @ --ns-rec=nameserver'."

.. _modify_zone_with_relative_admin_email:

Modify zone with relative admin-email
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Command: ``dnszone-mod zone.test. --admin-email it-department2``
-  Result: zone is created with values

   -  nsrecord: [ipa-dns1.example.com., ipa-dns2.example.com.]
   -  idnssoarname: it-department2
   -  idnssoamname: ipa-dns1.example.com.

.. _install_serverreplica:

Install server/replica
~~~~~~~~~~~~~~~~~~~~~~

-  Command: ``ipa-server-install --setup-dns`` or
   ``ipa-replica-install --setup-dns`` or ``ipa-dns-install``
-  Result: Installed replica hostname is appended to every IPA managed
   DNS zone apex as nameserver

.. _remove_replica:

Remove replica
~~~~~~~~~~~~~~

-  Command: ``ipa-replica-manage del replica.example.com``
-  Assumption: replica is with DNS installation
-  Result: replica hostname is removed from every NS record in IPA
   managed domain (including records outside zone apex)



RFE Author
----------

`mbasti <User:Mbasti>`__ 14:00 16 September 2014 (CEST)

`Category:FreeIPA V4 Test Plan <Category:FreeIPA_V4_Test_Plan>`__
`Category:FreeIPA Test Plan <Category:FreeIPA_Test_Plan>`__
