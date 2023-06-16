Some decisions made before FreeIPA is deployed and adopted are very hard
to be fixed later, if not impossible. This article therefore digs in the
most important decisions needed for a successful deployment.

Infrastructure
--------------

DNS
~~~

`DNS <DNS>`__ is deliberately listed first as DNS plays an important
role in identity management functionality, especially
`Kerberos <Kerberos>`__.

Domain
^^^^^^

FreeIPA should always have own primary domain, e.g. ``example.com`` or
``ipa.example.com`` which should not be shared with other
`Kerberos <Kerberos>`__ based identity management system as otherwise
there will be collisions on Kerberos system level. For example, if both
FreeIPA and Active Directory use the same domain, `trusts <trusts>`__
will be never possible, as well as automatic client server discovery via
`DNS <DNS>`__ SRV records.

Client machines do not need to be in the same domain as FreeIPA servers.
For example, FreeIPA may be a domain ``ipa.example.com`` and clients in
domain ``clients.example.com``, there just need to be a clear mapping
between DNS domain and Kerberos realm. Standard method to create the
mapping are
```_kerberos`` <https://web.mit.edu/kerberos/krb5-1.15/doc/admin/realm_config.html#mapping-hostnames-onto-kerberos-realms>`__
TXT DNS records. (FreeIPA DNS adds these automatically.)

.. _considerations_for_active_directory_integration:

Considerations for Active Directory integration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Active Directory domain is a complex system. It includes logically
structured set of resources (machines, users, services, ...) which
belong to potentially multiple DNS domains. Multiple DNS domains can be
part of the same AD domain (where AD domain is by definition same as AD
Kerberos realm). Multiple AD domains can be combined into a forest. The
very first AD domain created in the forest is called **forest root
domain**. Upper-cased name of the primary DNS domain of the AD domain is
used as AD Kerberos realm name.

IPA domain is a similarly complex system. It includes logically
structured set of resources (machines, users, services, ...) which
belong to potentially multiple DNS domains. Unlike Active Directory, we
have a single IPA domain/realm per deployment and for Active Directory
this single IPA domain looks like a separate Active Directory forest.
Active Directory considers primary DNS domain used as a basis for
FreeIPA Kerberos realm to be a forest root domain for FreeIPA domain
(like forest root domain for Active Directory).

FreeIPA domain can be placed in **any** DNS domain which does not
directly overlap with any domain in Active Directory forest. It could
be, for example, ``ipa.example.com``, if this DNS zone is not occupied
by any other AD domain in the same forest. It could be
``ipa.ad.example.com`` too, it could be ``example``\ **``.net``** as
well -- as long as there are no overlaps on the same DNS zone level.

The trust between two Active Directory forests is always established as
a trust between forest root domains of those forests. If FreeIPA domain
uses ``ipa.ad.example.com`` as the primary DNS zone, then we would be
saying about establishing forest trust between Active Directory forest
``ad.example.com`` and FreeIPA domain ``ipa.ad.example.com``. If there
are multiple DNS zones belonging to IPA domain, it is recommended to
place ```_kerberos`` TXT
records <https://web.mit.edu/kerberos/krb5-1.15/doc/admin/realm_config.html#mapping-hostnames-onto-kerberos-realms>`__
pointing to the FreeIPA realm name in each of them for proper discovery
of network resources by FreeIPA clients.

.. _dns_server:

DNS server
^^^^^^^^^^

FreeIPA domain may be either served from an integrated `DNS <DNS>`__
service or an external name service. A FreeIPA domain delegated to the
integrated `DNS <DNS>`__ service is a recommended approach.

When using external name server, identity management functionality or
`trusts <trusts>`__ will be possible, however the configuration will be
much more difficult and error prone. Full list of benefits of using the
integrated DNS service can be found in the `DNS
article <DNS#Benefits_of_integrated_DNS>`__.

.. _kerberos_realm_name:

Kerberos realm name
~~~~~~~~~~~~~~~~~~~

When you are installing first FreeIPA server you are always defining a
Kerberos realm name for this installation. When selecting the realm name
please follow these rules:

-  The realm name must not conflict with any other existing Kerberos
   realm name (e.g. name used by Active Directory).
-  The realm name should be upper-case (``EXAMPLE.COM``) version of
   primary DNS domain name (``example.com``).
-  FreeIPA clients from multiple distinct DNS domains
   (``example``\ **``.com``**, ``example``\ **``.net``**,
   ``example``\ **``.org``**) can be joined to single Kerberos realm
   (``EXAMPLE``\ **``.COM``**)
-  One FreeIPA installation always represents single Kerberos realm.

.. _multi_site_deployment_awareness:

Multi-site deployment awareness
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

FreeIPA servers and clients may be distributed in various geographical
locations. `DNS Location mechanism <V4/DNS_Location_Mechanism>`__ allows
to split topology into separated areas called *locations* (mapping to
geographical areas). Clients using DNS SRV records (like SSSD) in a
particular location then use the servers from the same location as the
first (the closest FreeIPA servers) and servers from different locations
as the backup (remote FreeIPA servers). For more details please refer to
`FreeIPA DNS Locations HowTo document <Howto/IPA_locations>`__.

When planning deployment, it is important to keep in mind that DNS
locations feature in FreeIPA require at least one DNS server in each
location. If it is necessary, multiple DNS views on external DNS servers
can be used instead of deploying DNS server for each location but such
setups are typically less resilient.

PKI
~~~

When `PKI <PKI>`__ service is configured, FreeIPA hosts and services may
obtain signed certificates from FreeIPA CA. Certificates can then be
used for authentication or secure authentication in configured services.

Currently, there are 3 types how FreeIPA can blend in a existent
certificate environment:

#. No blending - FreeIPA is simply installed with own `PKI <PKI>`__
   service and a self-signed CA certificate
#. External CA - FreeIPA is installed with own `PKI <PKI>`__, but it's
   CA certificate is signed with external certificate authority (Active
   Directory Certificate Service or other custom Certificate Service).
   This requires the external CA to allow sub CAs.
#. `CA-less installation <V3/CA-less_install>`__ - FreeIPA does not
   configure own CA, but uses signed host certificates from external CA.

Servers/Replicas
~~~~~~~~~~~~~~~~

.. _freeipa_server_exclusivity:

FreeIPA server exclusivity
^^^^^^^^^^^^^^^^^^^^^^^^^^

It is highly recommended to avoid deploying other applications or
services on the FreeIPA server virtual machine/bare metal machine, from
several primary reasons:

-  Performance reasons - FreeIPA server can be exhaustive to the
   plaform, especially when the number of LDAP objects is high
-  Stability - FreeIPA is integrated in the system and if 3rd party
   application changes configuration or services FreeIPA depends on,
   FreeIPA can break
-  It is easier to migrate the FreeIPA server to newer platform
   (affecting for example RHEL-6 and RHEL-7 deployments)

.. _number_of_servers:

Number of servers
^^^^^^^^^^^^^^^^^

FreeIPA runs in a replicated multi-master environment. The number of
servers depends on several factors:

-  How many entries are in the system?
-  How many different geographically dispersed datacenters you have?
-  How active are applications and clients regarding authentications and
   LDAP lookups.

Generally it is recommended to have at least 2-3 replicas in each
datacenter. There should be at least one replica in each datacenter with
additional FreeIPA services like `PKI <PKI>`__ or `DNS <DNS>`__ if used.
Note that it is not recommended to have more than 4 replication
agreements per replica. Following example demonstrated the recommended
infrastructure:

.. figure:: Topology-16.png
   :alt: Deployment example with 16 FreeIPA servers
   :width: 300px

Deployment example with 16 FreeIPA servers

.. figure:: Topology-12.png
   :alt: Deployment example with 12 FreeIPA servers
   :width: 300px

Deployment example with 12 FreeIPA servers

Clients
~~~~~~~

Every client should have at least 2 `DNS <DNS>`__ servers configured in
``/etc/resolv.conf`` for resiliency. Update resolv.conf and *DHCPd*
configuration accordingly.

Enrolling each client using ipa-client-install requires access to port
443 (HTTPS) on IPA master. This is because once enrolled, client uploads
own SSH keys and performs few more operations. IPA CLI also uses the
same port to communicate to IPA master. Thus, it is required to have
access to HTTPS (443) from a client side.

.. _disaster_recovery:

Disaster recovery
-----------------

Please refer to `Backup and Restore <Backup_and_Restore>`__ article.

.. _active_directory_integration:

Active Directory Integration
----------------------------

In order to be able to configure `trusts <trusts>`__, `DNS <DNS>`__
needs to be configured properly, FreeIPA must have an own primary DNS
domain matching it's `Kerberos <Kerberos>`__ realm name. DNS domain and
realm have to be different from Active Directory DNS domain.

Another important requirement is IPv6 stack. Recommended way for
contemporary networking applications is to only open IPv6 sockets for
listening because IPv4 and IPv6 share the same port range locally.
FreeIPA uses Samba as part of its Active Directory integration and Samba
**requires enabled IPv6 stack** on the machine.

**DO NOT** use ``ipv6.disable=1`` on the kernel commandline: It disables
the whole IPv6 stack and breaks Samba.

If necessary, adding ``ipv6.disable_ipv6=1`` will keep the IPv6 stack
functional but will not assign IPv6 addresses to any of your network
devices except the loopback. This is recommeneded approach for cases
when you don't use IPv6 networking.

Creating and adding following lines to for example
/etc/sysctl.d/ipv6.conf will avoid assigning IPv6 addresses to a
specific network interface:

| `` net.ipv6.conf.all.disable_ipv6 = 1``
| `` # Disabling "all" does not apply to interfaces that are already "up" when sysctl settings are applied. ``
| `` net.ipv6.conf.``\ ``.disable_ipv6 = 1``
| `` # Interface lo must have IPv6 enabled``
| `` net.ipv6.conf.lo.disable_ipv6 = 0``

where *interface0* is your specialized interface. Note that all we are
requiring is that IPv6 stack is enabled at the kernel level and this is
recommended way to develop networking applications for a long time
already.

Migration
---------

FreeIPA can already `migrate <Howto/Migration>`__ from a general LDAP
server or NIS. It cannot, however, automatic migration from a pure
Kerberos solution or from other FreeIPA deployment (see tickets
`#3656 <https://fedorahosted.org/freeipa/ticket/3656>`__ and
`#4285 <https://fedorahosted.org/freeipa/ticket/4285>`__).

.. _extending_freeipa:

Extending FreeIPA
-----------------

Both FreeIPA schema, CLI and `Web UI <Web_UI>`__ can be extended.
`Directory Server <Directory_Server>`__ schema needs to be extended
manually on one server via LDAP manipulation tools. On the other hand,
both CLI and Web UI can be extended with plugins shipped together with
vanilla FreeIPA packages. See `Documentation <Documentation>`__ for
additional resources on how to write the extensions.
