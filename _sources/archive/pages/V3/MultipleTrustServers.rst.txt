\__NOTOC_\_

Overview
========

Ticket `#2189 <https://fedorahosted.org/freeipa/ticket/2189>`__;

Each FreeIPA server in the realm has potential to serve as domain
controller in the cross-forest realm trust. This page outlines design
for implementing multiple servers support in FreeIPA.

.. _use_cases10g:

Use Cases
=========

Once ``ipa-adtrust-install`` ran on the FreeIPA server, the server can
handle requests from trusted domains by means of Samba project's
``smbd`` and ``winbindd`` daemons.

Hosts in FreeIPA realm may be enrolled against specific FreeIPA replica
server. User from trusted domain can access these hosts and their
identities will be resolved against the replica. However, if replica
server does not have trust support configured, these identities will not
be processed since running ``winbindd`` process is required to contact
the trusted domain's domain controllers and Global Catalog servers.

Domain controllers are advertised to clients via SRV records in DNS.
Since replica servers may be arranged in a specific topology, adding new
domain controller would need to respect the topology design. It means
priority/weight of the domain controller compared to other domain
controllers should be adjustable. Prime use case for this is branch
office deployments.

Design
======

-  Each domain controller uses separate identity and service key to talk
   to FreeIPA LDAP server. The identity is tied to the server hostname.

-  Service principal is ``cifs/hostname@REALM``, identified in LDAP as
   ``krbprincipalname=cifs/hostname@REALM``.

-  All identities are members of
   ``cn=adtrust agents``,\ ``cn=sysaccounts``,\ ``cn=etc``,\ ``$SUFFIX``.
   Thus, all replica servers can see what other servers are providing
   domain controller service.

-  Replica server only becomes domain controller when
   ``ipa-adtrust-install`` utility was executed on it. It means all DC
   setup is delivered via the ``ipa-adtrust-install``.

-  ``ipa-adtrust-install`` should be able to detect other DCs by looking
   at existing identities as members of the
   ``cn=adtrust agents``,\ ``cn=sysaccounts``,\ ``cn=etc``,\ ``$SUFFIX``
   tree and modify list of SRV records under ``_msdcs`` and default site
   configuration if DNS is controlled by FreeIPA.

-  Domain Controller priority/weight can be modified at run time since
   it only affects SRV records in the DNS (if FreeIPA controls the DNS).
   Normal ``ipa dnsrecord-mod`` commands should be used for this
   purpose, operating on SRV records for ``_msdcs`` and default site
   configuration.

-  There are trust properties that are global for the realm. Some of
   them are modifiable, some not. Thus, ``ipa trustconfig-show`` and
   ``ipa trustconfig-mod`` should reflect both global and local settings
   (realm-wise and DC-wise).

-  Following properties of the trust are global for the realm:

   -  NetBIOS domain name (read-only, affects existing trusts)
   -  Domain name (read-only, affects existing trusts)
   -  Domain GUID (read-only, informational)
   -  Additional domain suffixes exposed to the trusted party, handled
      as black list against global list of additional domains associated
      with our or transitive realm, read/write
   -  Fallback primary group (read-write)

-  Following properties of the trust are per Domain Controller:

   -  priority of the DC and GC services (read-write, DNS SRV record)

Details on ``ipa trustconfig`` commands design are available at
http://www.freeipa.org/page/V3/Trust_config_command Details on
additional domain suffixes handling are available at
http://www.freeipa.org/page/V3/Domain_suffixes

Implementation
==============

Implementation-wise there are three parts:

-  ``ipa-adtrust-install``:

   -  Gather list of CIFS services that are also members of
      ``cn=adtrust agents`` and add SRV records for them to \_msdcs in
      ipaserver/install/adtrustinstance.py:ADTrustInstance::__add_dns_service_records().
   -  Once Global Catalog Server implementation will be ready, configure
      its use as one of ``ADTrustInstance`` setup steps.

-  ``IPA framework``:

   -  Add display and modification of trust properties as
      ``ipa trustconfig-*`` commands, including listing DC and GC
      servers.
   -  NOTE: there is no need to modify trust creation procedure since it
      appears that AD DC assumes LSA CreateTrustedDomainEx2 call comes
      from the DC (in Windows the UI to establish trust is only on DC)
      and therefore does not do DNS discovery during validation step.
      Since smbd running on the same host that 'ipa trust-add' runs on
      will contact the same host's LDAP server, there is no issue in
      replication at this stage.

-  ``IPA Web UI``

   -  Add support for ``ipa trustconfig-*`` to Web UI.



Major configuration options and enablement
==========================================

No additional options to ``ipa-adtrust-install``

Replication
===========

All data is already in a replicated namespace.



Updates and Upgrades
====================

No changes to schema, no need to run anything additional on updates or
upgrades

Dependencies
============

No additional dependencies beyond AD trusts support



External Impact
===============

Once ``ipa-adtrust-install`` install ran on replica, the replica will be
advertised via ``_msdcs`` SRV namespace as a domain controller.



RFE author
==========

`Alexander Bokovoy <User:ab>`__
