Documentation
=============



.. _documentation-users:

User Documentation
------------------

.. toctree::
   :maxdepth: 1
   :titlesonly:

   Quick Start Guide </page/Quick_Start_Guide>
   Deployment Recommendations </page/Deployment_Recommendations>
   How-tos </page/HowTos>
   Frequently Asked Questions </page/Frequently_Asked_Questions>



.. _local-documentation-user-guides:

User Guides
~~~~~~~~~~~

Use Red Hat Enteprise Linux documentation:

-  **FreeIPA 3.0.x**:

   -  `Identity Management Guide for RHEL
      6 <https://access.redhat.com/knowledge/docs/en-US/Red_Hat_Enterprise_Linux/6/html/Identity_Management_Guide/index.html>`__
      (`report
      bug <https://bugzilla.redhat.com/enter_bug.cgi?product=Red%20Hat%20Enterprise%20Linux%206&component=doc-Identity_Management_Guide>`__)

-  **FreeIPA 3.3.x and 4.x**:

   - RHEL 8: `Identity Management documentation <https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8#Identity%20Management>`__
   - RHEL 9: `Identity Management documentation <https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9#Identity%20Management>`__
   -  `Linux Domain Identity, Authentication and Policy Guide for RHEL
      7 <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Linux_Domain_Identity_Authentication_and_Policy_Guide/index.html>`__
      (`report
      bug <https://bugzilla.redhat.com/enter_bug.cgi?product=Red%20Hat%20Enterprise%20Linux%207&component=doc-Linux_Domain_Identity_Management_Guide>`__)
   -  `Windows Integration Guide for RHEL
      7 <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Windows_Integration_Guide/index.html>`__
      (`report
      bug <https://bugzilla.redhat.com/enter_bug.cgi?product=Red%20Hat%20Enterprise%20Linux%207&component=doc-Windows_Integration_Guide>`__)
   -  `System-Level Authentication Guide for RHEL
      7 <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System-Level_Authentication_Guide/index.html>`__
      (`report
      bug <https://bugzilla.redhat.com/enter_bug.cgi?product=Red%20Hat%20Enterprise%20Linux%207&component=doc-System_Level_Authentication_Guide>`__)

Upstream user guide :doc:`is not maintained </page/Upstream_User_Guide>` anymore
as all effort is put into the Red Hat Enteprise Linux documentation.
Bugs found in the documentation can be reported in `Red Hat Issues tracker 
<https://issues.redhat.com/secure/CreateIssue!default.jspa>`__ by choosing RHELDOCS project and Documentation component.



By Component
~~~~~~~~~~~~

Learn about FreeIPA by reading information about the particular
components that compose the entire solution.

.. toctree::
   :maxdepth: 1
   :titlesonly:

   Directory Server </page/Directory_Server>
   Kerberos </page/Kerberos>
   PKI </page/PKI>
   DNS </page/DNS>
   Certmonger </page/Certmonger>
   NTP Server </page/NTP_Server>
   Web UI </page/Web_UI>
   Trusts </page/Trusts>
   Client </page/Client>



Additional Resources
~~~~~~~~~~~~~~~~~~~~

-  `FreeIPA Planet <http://planet.freeipa.org>`__: blog/RSS aggregator



Old Resources
~~~~~~~~~~~~~

-  :ref:`FreeIPA 1.2.1 and FreeIPA 2.0.0 deprecated user guides <deprecated-upstream-guides>`
-  :doc:`Old FreeIPA v1.2.1 Documentation Page </page/V1_Documentation>`



Developer Documentation
-----------------------



Design goals and core decisions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. toctree::
   :maxdepth: 1
   :titlesonly:

   Active Directory integration </page/Goals/AD_Integration>
   Scalability </page/Goals/Scalability>
   ACIs </page/Goals/ACIs>



Feature Designs
~~~~~~~~~~~~~~~

This section contains designs that have actually been implemented in
FreeIPA:

.. toctree::
   :maxdepth: 1
   :titlesonly:

   Version 4 Designs </page/V4_Designs>
   Version 4 Minor Enhancements </page/V4_Minor_Enhancements>
   Version 3 Designs </page/V3_Designs>
   Version 3 Minor Enhancements </page/V3_Minor_Enhancements>
   Version 2 Designs </page/V2_Designs>
   Version 1 Designs </page/V1_Designs>



Feature Proposals
~~~~~~~~~~~~~~~~~

This section contains new or historical proposal that have not yet been
approved/implemented or that have been superseded or abandoned.

.. toctree::
   :maxdepth: 1
   :titlesonly:

   Version 4 Proposals </page/V4_Proposals>
   Version 3 Proposals </page/V3_Proposals>
   Version 2 Proposals </page/V2_Proposals>
   Version 1 Proposals </page/V1_Proposals>



.. _local-documentation-developer-resources:

Additional Resources
~~~~~~~~~~~~~~~~~~~~

-  `Tour of FreeIPA 389-ds
   plugins <http://talks.vda.li/talks/2017/freeIPA/tour-of-ipa-389-ds-plugins/>`__
-  `Extending FreeIPA
   Guide <http://abbra.fedorapeople.org/guide.html>`__
-  :doc:`How to add a new attribute to IPA </page/HowTo/Add_a_new_attribute>`
-  :doc:`How to migrate your code to the new LDAP API </page/HowTo/Migrate_your_code_to_the_new_LDAP_API>`
-  :doc:`v2 PRD </page/V2PRD>`: draft requirements for FreeIPA v2
-  :doc:`FreeIPA v2 development documentation </page/V2_development>`:
   Historical development information.
-  `Federated Authentication Utilizing Apache &
   SSSD <https://jdennis.fedorapeople.org/doc/sssd_configuration.pdf>`__



Tester Documentation
--------------------



Test Plans
~~~~~~~~~~

This section contains test plans that have been designed for FreeIPA:

.. toctree::
   :maxdepth: 1
   :titlesonly:

   Version 4 Test Plans </page/V4_Test_Plans>
   Release process </page/Release>



FreeIPA Training Series
-----------------------



FreeIPA 4.5
~~~~~~~~~~~

-  :download:`FreeIPA CA <Freeipa-ca-component.pdf>`
-  :download:`Introduction to LDAP <Freeipa-introduction-to-ldap.pdf>`



FreeIPA 3.3 & SSSD 1.11
~~~~~~~~~~~~~~~~~~~~~~~~~~



FreeIPA server presentations
----------------------------------------------------------------------------------------------

-  `FreeIPA 3.3 Trust features <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/FreeIPA33-trust.pdf>`__
-  `Blending FreeIPA in a Certificate
   Infrastructure <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/FreeIPA33-blending-in-a-certificate-infrastructure.pdf>`__
-  `Extending the FreeIPA
   Server <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/FreeIPA33-extending-freeipa.pdf>`__
-  `AD Trust for Legacy Clients <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/FreeIPA33-legacy-clients.pdf>`__
-  `FreeIPA Client and Server Improvements in
   3.3 <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/FreeIPA33-server-and-client.pdf>`__



SSSD client presentations
----------------------------------------------------------------------------------------------

-  `SSSD Active Directory Improvements in
   1.11 <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/FreeIPA33-sssd-1-11-ad-improvements.pdf>`__
-  `SSSD AD Provider: Access
   Control <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/FreeIPA33-sssd-access-control.pdf>`__
-  `SSSD DNS Improvements in AD
   Environment <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/FreeIPA33-sssd-dns-ad.pdf>`__



FreeIPA 3.0 & SSSD 1.9.2
~~~~~~~~~~~~~~~~~~~~~~~~~~



FreeIPA server presentations
----------------------------------------------------------------------------------------------

-  `FreeIPA Trust Basics <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_Trust_Basics.pdf>`__
   (`ODP <https://github.com/freeipa/freeipa.github.io/raw/main/src/page/Freeipa30_Trust_Basics.odp>`__)
-  `FreeIPA Server/Client Core
   Changes <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_client_server.pdf>`__
   (`ODP <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_client_server.odp>`__)
-  `SSH Public Keys Feature <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_SSH_Public_Keys.pdf>`__
   (`ODP <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_SSH_Public_Keys.odp>`__)
-  `SELinux User Maps Feature <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_SELinuxUserMap.pdf>`__
   (`ODP <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_SELinuxUserMap.odp>`__)
-  `DNS Persistent Search
   Feature <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_DNS_persistent_search.pdf>`__
   (`ODP <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_DNS_persistent_search.odp>`__)
-  `DNS Zone Transfers <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_DNS_zone_transfers.pdf>`__
   (`ODP <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_DNS_zone_transfers.odp>`__)



SSSD client presentations
----------------------------------------------------------------------------------------------

-  `SSSD AD Provider Feature <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_sssd-ad-provider.pdf>`__
   (`ODP <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_sssd-ad-provider.odp>`__)
-  `SSSD AutoFS
   Integration <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_sssd-autofs-integration.pdf>`__
   (`ODP <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_sssd-autofs-integration.odp>`__)
-  `SSSD OpenSSH
   Integration <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_SSSD_OpenSSH_integration.pdf>`__
   (`ODP <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_SSSD_OpenSSH_integration.odp>`__)
-  `SSSD Pre-Seeding Users for First
   Boot <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_sssd-preseed-users.pdf>`__
   (`ODP <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_sssd-preseed-users.odp>`__)
-  `SSSD SUDO Integration
   Feature <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_SSSD_SUDO_Integration.pdf>`__
   (`ODP <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Freeipa30_SSSD_SUDO_Integration.odp>`__)



Public Presentations
--------------------

Red Hat Summit 2025 lighting talks
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These talks were part of the Community Theater at the exhibition floor of the Red Hat Summit 2025. They ran for 20 minutes each, including discussion, and weren't recorded.

`FreeIPA lightning talks at the Summit <https://events.experiences.redhat.com/widget/redhat/sum25/SessionCatalog2025?search=%22FreeIPA%22>`__

Below are the local copies of the slides:

- :download:`Deployment migration <Experiencing_FreeIPA_before_RHEL_IdM__deployment_migration.pdf>`
- :download:`Integration with Keycloak <Experiencing_FreeIPA_before_RHEL_IdM__integration_with_Keycloak.pdf>`
- :download:`Trust between IdM deployments <Experiencing_FreeIPA_before_RHEL_IdM__trust_between_IdM_deployments.pdf>`
- :download:`ansible-freeipa dynamic inventory <Experiencing_FreeIPA_before_RHEL_IdM__dynamic_inventory_in_ansible-freeipa.pdf>`

Here you can find presentations or other materials which FreeIPA team
presented on various public conferences.

-  `Devconf 2018 <http://devconf.cz/cz/2018>`__: `Smart Card in Identity
   Management <https://www.youtube.com/watch?v=LfgwVNDloLo&t=4s&list=PLa1lk-DCoIV1EbYAIdfGzEisokG5bPQ-s&index=74>`__
   (talk on youtube)
-  `SnowCamp.io <http://snowcamp.io/fr/previous-editions/>`__:
   `Authentication using One-Time Password Token and Smart
   Card <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/snowcampio_2FA.pdf>`__
-  FreeIPA presentation at `NYLUG's
   meetup <http://www.meetup.com/nylug-meetings/events/218903375/>`__ in
   January 2014:
   `PDF <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Identity_And_Directories_with_FreeIPA.pdf>`__
-  `Devconf 2013 <http://www.devconf.cz>`__: `Integrating Linux systems
   into Active Directory
   Environment <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Devconf2013-linux-ad-integration-options.pdf>`__
   (`talk on youtube <http://www.youtube.com/watch?v=cS6EJ1L7fRI>`__)
-  `FOSDEM 2013 <https://fosdem.org/2013/>`__ Idm Presentation slides in
   `PDF <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/FOSDEM-Building-IDM.pdf>`__ format
-  `DjangoCon Europe 2013 - Django + Kerberos
   authentication <http://www.roguelynn.com/circus/>`__ with
   `slides <https://speakerdeck.com/roguelynn/introduce-django-to-your-old-friends>`__
   and `video <http://www.youtube.com/watch?v=oerxTvMn-uM>`__ available.
-  `LinuxAlt 2012 <http://www.linuxalt.cz>`__: `Introducing
   FreeIPA <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Mkosek-linuxalt2012.pdf>`__
-  `LinuxCon
   2012 <http://events.linuxfoundation.org/events/linuxcon-europe>`__:
   `FreeIPA hands-on workshop
   session <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/Linuxcon-ipa-hands-on.pdf>`__
-  `Red Hat Summit 2011 <http://www.redhat.com/summit/2011>`__: `FreeIPA
   presentation <http://www.redhat.com/summit/2011/presentations/summit/whats_next/friday/pal_crittenden_f_1100_ipa_overview_rev3.pdf>`__
-  `FOSDEM 2009 <https://archive.fosdem.org/2009/>`__: FreeIPA
   presentation - `PDF <https://github.com/freeipa/freeipa.github.io/blob/main/src/page/FreeIPA-FOSDEM.pdf>`__, `OO.org
   3.0.1 <http://simo.fedorapeople.org/freeipa/FreeIPA-FOSDEM.odp>`__
