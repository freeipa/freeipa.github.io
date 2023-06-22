Introduction
------------

Trusts Services against `Active
Directory <http://en.wikipedia.org/wiki/Active_Directory>`__ servers are
provided through integration with `Samba <http://samba.org>`__
components.

.. _integrating_linux_systems_into_active_directory:

Integrating Linux systems into Active Directory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

See `Dmitri Pal <User:dpal>`__'s talk on
`devconf.cz <http://devconf.cz>`__ on the subject of Active Directory
Trusts. Sometimes, using FreeIPA trust with AD is codenamed as "Indirect
integration with AD" because Linux systems are talking mostly to FreeIPA
instead of directly talking to AD.

It dives into various options how to integrate classic POSIX and Active
Directory worlds together
(`slides <media:Devconf2013-linux-ad-integration-options.pdf>`__),
compares them and explains advantages and disadvantages of each:

{{#ev:youtube|cS6EJ1L7fRI}}

RHEL blog contains `more guidance on when to use FreeIPA trust with
AD <http://rhelblog.redhat.com/2015/05/27/direct-or-indirect-that-is-the-question/>`__.
TextPlease note that FreeIPA is known under name "IdM" in the RHEL
world.

Documentation
-------------

Active Directory domain is a complex system. It includes logically
structured set of resources (machines, users, services, ...) which
belong to potentially multiple DNS domains. Multiple DNS domains can be
part of the same AD domain. Multiple AD domains can be combined into a
forest. The very first AD domain created in the forest is called
**forest root domain**. The primary DNS domain of the AD domain is used
as a basis for Kerberos realm.

IPA domain is a similarly complex system. It includes logically
structured set of resources (machines, users, services, ...) which
belong to potentially multiple DNS domains. Unlike Active Directory, we
have a single IPA domain per deployment and for Active Directory this
single IPA domain looks like a separate Active Directory forest. Active
Directory considers DNS domain used as a basis for IPA Kerberos realm to
be a forest root domain for IPA domain (like forest root domain for
Active Directory).

IPA domain can be placed in **any** DNS domain which does not directly
overlap with any domain in Active Directory forest. It could be, for
example, ``ipa.example.com``, if this DNS zone is not occupied by any
other AD domain in the same forest. It could be ``ipa.ad.example.com``
too, it could be ``ipa-example.com`` as well -- as long as there are no
overlaps on the same DNS zone level.

The trust between two Active Directory forests is always established as
a trust between forest root domains of those forests. If IPA domain uses
``ipa.ad.example.com`` as the primary DNS zone, then we would be saying
about establishing forest trust between Active Directory forest
``ad.example.com`` and IPA domain ``ipa.ad.example.com``. If there are
multiple DNS zones belonging to IPA domain, it is recommended to place
appropriate service and TXT records pointing to the primary IPA domain
in each of them for proper discovery of network resources by IPA
clients.

We have improved documentation about trust to AD in Red Hat Enteprise
Linux 7.2 and you can find comprehensive coverage of the feature in the
`corresponding chapter of the Windows Integration
Guide <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Windows_Integration_Guide/active-directory-trust.html>`__

Designs
~~~~~~~

-  `Introducing Active Directory Trust Feature <IPAv3_AD_trust>`__
-  `Serving legacy clients for AD
   Trusts <V3/Serving_legacy_clients_for_trusts>`__
-  `Using POSIX attributes defined in
   AD <V3/Use_posix_attributes_defined_in_AD>`__
-  `Configurable SID Blacklists <V3/Configurable_SID_Blacklists>`__
-  `trust-config command <V3/Trust_config_command>`__
-  `Global Catalog Support <V3/Trust_GC_support>`__

HOWTOs
~~~~~~

Configuration
^^^^^^^^^^^^^

-  `How to set up trusts <Active_Directory_trust_setup>`__
-  `How to prepare a test AD instance for
   testing <Setting_up_Active_Directory_domain_for_testing_purposes>`__

Troubleshooting
^^^^^^^^^^^^^^^

-  `Inspecting the PAC <Howto/Inspecting_the_PAC>`__
-  `Debugging and troubleshooting
   SSSD <https://docs.pagure.org/SSSD.sssd/users/troubleshooting.html>`__

Blogs
~~~~~

-  `Red Hat blogs on Identity
   Management <http://rhelblog.redhat.com/tag/identity-management/>`__
