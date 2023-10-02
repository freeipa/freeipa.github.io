Development_Status
==================

*This page and all pages it links to are a work in progress. Please note
that any content may be changed as the development progresses.*



IPA v2 Development Progress
---------------------------

Design
------



IPA Client
----------------------------------------------------------------------------------------------

-  `IPA Client Design Overview <FreeIPAv2:IPA_Client_Design_Overview>`__
-  `Service Controller
   Daemon <FreeIPAv2:SSSD/Service_Controller_Daemon>`__



IPA Server
----------------------------------------------------------------------------------------------



System Management
^^^^^^^^^^^^^^^^^

-  `Access control <FreeIPAv2:Access_Control>`__

Kerberos
^^^^^^^^

-  `Automatic Ticket Renewal <FreeIPAv2:Automatic_Ticket_Renewal>`__ -
   stable version



CA Integration
^^^^^^^^^^^^^^

-  `Certificate Management <FreeIPAv2:Certificate_Management>`__ -
   stable version

Policy
^^^^^^

THIS EFFORT IS DEFERRED

-  `Overall Design of Policy Related
   Components <FreeIPAv2:Overall_Design_of_Policy_Related_Components>`__
-  `Integration with SELinux <FreeIPAv2:Integration_with_SELinux>`__

SUDO
^^^^

-  `SUDO integration plans <FreeIPAv2:SUDO_integration_plans>`__ -
   Description of how we plan to implement centralized SUDO management
   using LDAP rather than policies.
-  `SUDO Schema Design <FreeIPAv2:SUDO_Schema_Design>`__



Back End Concepts
^^^^^^^^^^^^^^^^^

-  `Concepts and Objects <FreeIPAv2:Concepts_and_Objects>`__ -
   Description of the new concepts (stable version)
-  `DS Design Summary <FreeIPAv2:DS_Design_Summary>`__ - base objects
   (stable version)
-  `DS Design Summary 2 <FreeIPAv2:DS_Design_Summary_2>`__ - policy
   objects (stable version)
-  `Schema for loading and
   processing <FreeIPAv2:Schema_for_loading_and_processing>`__ - based
   on the design pages above



Plugin framework
^^^^^^^^^^^^^^^^

-  `Tutorial for Plugin
   Authors <http://freeipa.org/developer-docs/ipalib-module.html>`__
-  `Python API documentation <http://freeipa.org/developer-docs/>`__ -
   Generated automatically from source code using
   `epydoc <http://epydoc.sourceforge.net/>`__.



DNS integration
^^^^^^^^^^^^^^^

-  `DNS Integration Design <FreeIPAv2:DNS_Integration_Design>`__
-  `Dynamic updates with
   GSS-TSIG <FreeIPAv2:Dynamic_updates_with_GSS-TSIG>`__
-  `DNS Location Discovery <FreeIPAv2:DNS_Location_Discovery>`__ (DRAFT)



389 Directory Server
^^^^^^^^^^^^^^^^^^^^

-  `Schema Compatibility Plug-in
   Design <FreeIPAv2:Schema_Compatibility_Plug-in_Design>`__



Default configurations for IPA components
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  `DNA plugin default
   configuration <FreeIPAv2:DNA_plugin_default_configuration>`__



IPA Server Reference Pages
----------------------------------------------------------------------------------------------

-  `Schema Available in DS in IPA
   v1 <http://freeipa.org/static/IPAV1Available.html>`__



User Interface Design
---------------------

Usability
----------------------------------------------------------------------------------------------

-  `Usability Testing
   Materials <FreeIPAv2:Usability_Testing_Materials>`__
-  `Usability Testing Results <FreeIPAv2:Usability_Testing_Results>`__



Design
----------------------------------------------------------------------------------------------

-  `WebUI-2-1 <FreeIPAv2:WebUI-2-1>`__

Testing
----------------------------------------------------------------------------------------------

-  `UI Unit Tests <FreeIPAv2:UI_Unit_Tests>`__
-  `Selenium <FreeIPAv2:Selenium>`__



Current progress
----------------

We initially began development of the new v2 components outside of our
main repository. These have since been re-integrated for the server and
the new client code will remain separate.

To checkout the code, use the ``git clone``\ *``repository_url``*.
Please note that these repositories changing rapidly and may not always
be buildable.



IPA Server
----------------------------------------------------------------------------------------------

The server using the new XML-RPC framework. This is the same repository
as the V1 code. All work for V2 has been done in the master branch.

-  `Git repository <http://git.fedorahosted.org/git/freeipa.git>`__:
   ```git://git.fedorahosted.org/git/freeipa.git`` <git://git.fedorahosted.org/git/freeipa.git>`__



Test Repository
^^^^^^^^^^^^^^^

With the release of alpha 1 we have created a yum repository that
contains the Fedora 11 x86 and x86_64 binaries. To use this repository
retrieve the file:

`freeipa-devel.repo <http://freeipa.org/downloads/freeipa-devel.repo>`__



IPA Client
----------------------------------------------------------------------------------------------

The client components.

SSSD:

-  `Git repository <http://git.fedorahosted.org/git/sssd.git>`__:
   ```git://git.fedorahosted.org/sssd.git`` <git://git.fedorahosted.org/sssd.git>`__



DNS integration
----------------------------------------------------------------------------------------------



Dynamic loading of DLZ drivers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is a patch allowing BIND to dynamically load DLZ drivers. Without
the patch, drivers need to be compiled into BIND. We aim to get this
feature upstream as soon as possible.

Code will be available soon.



IPA LDAP driver
^^^^^^^^^^^^^^^

Code will be available soon.



BIND DLZ write-back support patch
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

No code available at the moment.

Documentation
-------------

The documentation is still work in progress. Here is the progress so
far:

-  `FreeIPA
   Guide <https://docs.fedoraproject.org/en-US/Fedora/15/html/FreeIPA_Guide/index.html>`__

   *Provides detailed information about IPA, the technologies with which
   it works, and some of the terminology used to describe it. It also
   provides high-level design information for both the IPA client and
   server.*



Developer Documentation
-----------------------

-  `Where and how SSL is used in IPA <FreeIPAv2:SSLUsage>`__
-  `Command-Line tools overview <FreeIPAv2:CLI_Overview>`__
-  `NIS compatibility plugin <FreeIPAv2:NIS_Compatibility>`__
-  `Delegation <FreeIPAv2:Delegation>`__
-  `Machine enrollment <FreeIPAv2:Machine_join>`__
-  `Certificate_Authority <FreeIPAv2:Certificate_Authority>`__
-  `Command-line documentation
   requirements <FreeIPAv2:CommandDocumentation>`__
-  `Configuration files <FreeIPAv2:Config_Files>`__



Documentation Repository
------------------------

-  `Git doc repository for v2.1 and
   later <http://git.fedorahosted.org/git/?p=docs/freeipa-guide.git>`__

   *The repository contains all of the XML and PNG files used to build
   the entire documentation set, using ``publican`` and DocBook XML.
   This is part of the Fedora Documentation Project.*

-  `Git repository for 2.0 and
   older <http://git.fedorahosted.org/git/?p=ipadocs.git>`__

   ''This contains the git repository is an archive for post-1.0 and 2.0
   documentation.