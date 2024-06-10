DNS
===

FreeIPA DNS integration allows administrator to manage and serve DNS
records in a domain using the same CLI or Web UI as when managing
identities and policies. At the same time, administrator can benefit
from the tight DNS integration in FreeIPA management framework and have
configuration changes in FreeIPA server covered by automatic DNS updates
(see next chapters for more detailed list of benefits).



Initial Considerations
----------------------

The DNS component in FreeIPA was designed and built about several basic
assumptions and goals that should be always considered when assessing
enhancements or other requests to this component.

Assumptions
----------------------------------------------------------------------------------------------

-  DNS is hard to manage and lot of admins who want to deploy FreeIPA
   would have difficulties setting up DNS properly.
-  DNS is central to have a decent Kerberos experience.
-  Single-master DNS is error prone, especially for inexperienced
   admins.


Goals `Category:Goals`_.
----------------------------------------------------------------------------------------------

-  Provide an integrated DNS server which can be used to ease FreeIPA
   deployment ("get you going").

   -  Provide ability to standup and tear down replicas without caring
      for the special "master" DNS server.

-  Provide an alternative option for users with existing DNS
   infrastructure: Provide means for integrating FreeIPA with existing
   DNS infrastructure.
-  Goal is **NOT** to provide general-purpose DNS server. Features
   beyond easing FreeIPA deployment and maintenance are explicitly out
   of scope.



Benefits of integrated DNS
--------------------------

DNS component in FreeIPA is optional and user may choose to manage all
DNS records manually in other third party DNS server.

Please consider the following benefits of integrated DNS in FreeIPA
before enrolling a custom DNS solution:

-  Clients can be configured to automatically run DNS updates
   (*nsupdate*) when their IP address changes and thus keeping its DNS
   record up-to-date. DNS zones can be configured to synchronize
   client's reverse (PTR) record along with the forward (A, AAAA) DNS
   record.
-  FreeIPA domain has automatically maintained LDAP and Kerberos SRV
   records allowing an easy autodiscovery in FreeIPA clients
-  FreeIPA domain has automatically maintained Microsoft Windows service
   records required for `Trusts <Trusts>`__ configuration.
-  Automatic management of *ipa-ca.example.com* DNS record keeping
   forward (A, AAAA) records to all FreeIPA servers with a Certificate
   Authority configured. This DNS record is used in all certificates
   issued by FreeIPA as a general point to obtain certificate validation
   either via OCSP responder or CRL.

Caveats
-------

Caveats applicable to DNS apply as usual. It is extremely hard to change
DNS domain in existing installations so it is better to think ahead.

Most importantly, **do not shadow or hijack other DNS names!** You
should only use names which are delegated to you by the parent domain.

For example, if your company *Example, Inc.* bought domain
``example.com.`` you can use any domain in this sub-tree, e.g.
``whatever.example.com.``.

**Not respecting this rule will cause problems sooner or later!** (This
caveat includes inventing your own top-level domain like ``int.``)

-  Generally you will have problems with DNSSEC validation. This
   situation will be detected as domain hijacking.
-  Even without DNSSEC, you will have problems if the same name is used
   by multiple parties at the same time, especially when new top-level
   domains are delegated or during company mergers.



Internal-only domains
----------------------------------------------------------------------------------------------

It is perfectly fine to configure certain DNS zones to respond only to
clients in certain subnets or to apply other kinds of access control.
For internal names you can use arbitrary sub-domain in a DNS sub-tree
you own, e.g. ``int.example.com.``. **Always respect rules from the
previous section.**



DNS views / split-horizon DNS
----------------------------------------------------------------------------------------------

General advice about DNS views is **do not use them** because views make
DNS deployment harder to maintain and security benefits are questionable
(when compared with ACL).

Problems:

-  Using one name for multiple different machines (e.g. public vs.
   internal) is confusing.
-  DNS caching on clients causes problems for machines roaming between
   different DNS views.
-  DNSSEC deployment is harder to maintain when views are involved.

Technically it is much cleaner to put all internal names in a sub-domain
like ``int.example.com.`` (while ``example.com.`` is the public-facing
domain) and restrict access to this sub-domain using ACL as described in
the previous section.

Architecture
------------

The DNS integration is based on the
`bind-dyndb-ldap <https://fedorahosted.org/bind-dyndb-ldap/>`__ project,
which enhances BIND name server to be able to use FreeIPA server LDAP
instance as a data backend (data are stored in *cn=dns* entry, using
`schema defined by
bind-dyndb-ldap <http://git.fedorahosted.org/cgit/bind-dyndb-ldap.git/tree/doc/schema>`__.

Security
--------

FreeIPA LDAP directory information tree is by default accessible to any
user in the network, or (if anonymous search is disabled) to any
authenticated user. As DNS data are often considered as sensitive and as
having access to *cn=dns* tree would be basically equal to being able to
run zone transfer to all FreeIPA managed DNS zones, contents of this
tree in LDAP are hidden by default.

Only the following users have read access to the DNS tree:

-  bind-dyndb-ldap service principal itself
-  Members of the *admins* group
-  Users with *Read DNS Entries* attributes
-  Users with per-zone permission have read access to the permitted zone
   (these permissions can be created with *dnszone-add-permission*
   command)

Debugging
---------

When there is a suspicion that the DNS component is not behaving
correctly, standard system log (*/var/log/messages* or system journal)
can be consulted if there are any errors logged by BIND.

If the error is more subtle, BIND configuration (*/etc/named.conf*) can
be updated to produce a more detailed log. Standard `BIND
documentation <https://bind9.readthedocs.io/en/v9.18.18/reference.html#configuration-file-named-conf>`__
can be consulted for help.

Most common problems are caused by mis-configuration. Please see
`bind-dyndb-ldap documentation
page <https://fedorahosted.org/bind-dyndb-ldap/#Documentationandsupport>`__
and `FreeIPA troubleshooting DNS page <Troubleshooting#DNS_Issues>`__.



Bug reporting
-------------

Please follow `instructions published by bind-dyndb-ldap
project <https://fedorahosted.org/bind-dyndb-ldap/wiki/BugReporting>`__.



Additional Documentation
------------------------

-  `bind-dyndb-ldap project
   pages <https://fedorahosted.org/bind-dyndb-ldap/>`__

   -  `Maintainability analysis affecting the design
      goals <https://fedorahosted.org/bind-dyndb-ldap/wiki/Maintainability>`__

:ref:`Category:Goals <Category:Goals>`__