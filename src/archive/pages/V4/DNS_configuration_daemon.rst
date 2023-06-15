Overview
--------

FreeIPA uses for all DNS subsystem related operations a BIND plugin
bind-dyndb-ldap. That plugin configures BIND using data from LDAP DB and
synchronizes . Due this purpose bind-dyndb-ldap heavily uses internal
BIND API and BIND hacks to be able to do all requirements. The more
features FreeIPA contains the harder is maintain this plugin and keep up
to date with newer versions of BIND DNS server. Basically we stuck with
adding new features due plugin maintainability.

However configuration of BIND server can be extracted from
bind-dyndb-ldap plugin and moved to separated BIND configuration daemon
which will modify directly named.conf and uses \``rndc`\` utils to
adding/removing and reloading zones and configuration. With
configuration daemon the bind-dyndb-plugin should serve only as
synchronization mechanism for records in zones.

-  See `bind-dyndb-ldap code
   analysis <https://fedorahosted.org/bind-dyndb-ldap/wiki/Maintainability>`__
   how BIND internals are hacked
-  bind-dynbd-ldap page with some ideas about `alternative
   approaches <https://fedorahosted.org/bind-dyndb-ldap/wiki/SecondGeneration/Ideas>`__



Use Cases
---------

List of all DNS operation that are done by FreeIPA and have effect to
BIND:

.. _synchronization_of_dns_records:

Synchronization of DNS records
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  copying DNS records from to LDAP DB to BIND
-  copying DNS records from BIND to LDAP DB
-  creating reverse records from A records (*dnszone-mod
   --allow-sync-ptr=BOOL; dnsconfig-mod --allow-sync-ptr=BOOL*)

.. _managing_master_zones:

Managing master zones
~~~~~~~~~~~~~~~~~~~~~

-  creating new zones in BIND (source LDAP DB)
-  removing zones from BIND (added by FreeIPA)
-  enable/disable zone (*dnszone-{enable|disable}*)
-  setting default TTL per zone (*dnszone-mod --default-ttl=INT*)
-  setting forwarders per zone (*dnszone-mod --forwarders=STR*)
-  setting forward-policy per zone (*dnszone-mod --forward-policy=ENUM*)
-  setting update-policy (*dnszone-mod --update-policy=STR*)
-  setting allow-query per zone (*dnszone-mod --allow-query=STR*)
-  setting allow-transfer per zone (*dnszone-mod --allow-transfer=STR*)
-  enabling DNSSEC inline signing per zone (*dnszone-mod --dnssec=BOOL*)
-  setting NSEC3PARAM record (*dnszone-mod --nsec3param-rec=STR*)

.. _managing_forward_zones:

Managing forward zones
~~~~~~~~~~~~~~~~~~~~~~

-  creating new zones in BIND (source LDAP DB)
-  removing zones from BIND (added by FreeIPA)
-  enable/disable zone (*dnsforwardzone-{enable|disable}*)
-  set forwarders (*dnsforwardzone-mod --forwarder=STR*)
-  set forward-policy (*dnsforwardzone-mod --forward-policy=ENUM*)

.. _bind_configuration_global_and_per_dns_server:

BIND configuration (global and per DNS server)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  set global forward policy (*dnsconfig-mod --forward-policy=ENUM*)
-  set global forwarders (*dnsconfig-mod --forwarder=STR*)

.. _bind_configuration_per_server:

BIND configuration (per server)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  SOA mname (authoritative server) override (*dnsserver-mod
   --soa-mname-override=DNSNAME*)

.. _ipa_dns_locations:

IPA DNS Locations
~~~~~~~~~~~~~~~~~

-  CNAME record mappings per server, generated dynamically in BIND

DNSSEC
~~~~~~

-  There is no need for special care, BIND does inline zone signing
   automatically when configured. There is quite big code implemented on
   bind-dyndb-ldap side but it is caused by using BIND internal API
   otherwise it can be replace by one line in named.conf

Design
------

TBD

Implementation
--------------

TBD



Feature Management
------------------

UI
~~

Same as now, no changes expected (*ipa dns\** commands).

CLI
~~~

Same as now, no changes expected (*ipa dns\** commands).

Configuration
~~~~~~~~~~~~~

TBD

Upgrade
-------

TBD



How to Use
----------

No changes to current usage of DNS features.



Test Plan
---------

TBD
