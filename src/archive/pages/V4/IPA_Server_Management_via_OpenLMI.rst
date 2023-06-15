Overview
========

Current (as of 4.1) deployment and management of the FreeIPA server
cluster does not render itself well for automatic unattended operations.
The deployment sequence of primary and replicas installation assumes
physical presence of an admin at every step. This page is dedicated to
making the deployment and life-cycle management of a group of FreeIPA
servers in a single domain to be more manageable. The goal is to make it
more automated when initiated by admin, script tools like Puppet, Chef,
Salt or via provisioning systems like Foreman.



Use Cases
=========

Use cases are best covered in the `respective mail
thread <https://lists.fedorahosted.org/pipermail/openlmi-devel/2014-July/002281.html>`__.

Architecture
============

.. figure:: OpenLMI_Architecture.png
   :alt: OpenLMI_Architecture.png

   OpenLMI_Architecture.png

`Link to the original .odg file if you need to make edits is
here <https://drive.google.com/file/d/0B3tfpNCVjJdCNzk3eEtKaF9VeGM/edit?usp=sharing>`__

The diagram shows a general relationships between components of the
solution. In the top there is an administrator that will interact with
either provisioning tools or LMI CLI directly (on the right) or with
already deployed FreeIPA instance on the left.

The bottom part of the diagram shows the components that will be
implemented (or already exist) that will interact with each other on the
the node being managed.

**Lines**

-  **Black** - tool invocation or interactions that are well known and
   already defined and outside of scope of this design
-  **Orange** - calls over CIM protocol
-  **Blue** - Local D-BUS calls on the system
-  **Red** - Policy kit calls (also implemented over D-BUS under the
   hood, but services will use client API that wraps the D-BUS call)
-  **Green** - Interaction with the pure FreeIPA related components
-  **Purple** - Operations against an FreeIPA client or server instance

**Boxes**

-  **Pink** - Core OpenLMI related components, tools, services and
   libraries. This includes the future OpenLMI provider to expose the
   IPA server node management functionality.
-  **Blue** - D-BUS
-  **Salmon** - OpenLMI daemons

   -  Server Role Service is the service that would run on the node and
      would in general specify what the server is dedicated to. This
      component is effectively a registry of KVPs (key-value-pairs) and
      reflects the high level state of the system.
   -  ``ipaserverd`` Service is the new service that would be
      implemented by the FreeIPA project and would constitute the core
      of the IPA node management functionality. It will be in charge of
      session and state management as well as execution of operations
      like: install server, uninstall server, make server CRL generator,
      make server cert tracker, add/remove DNS and other components,
      check state/status of the server, etc.
   -  realmd Service is the existing component to manage the client and
      perform operations to discover domain, join or leave it.

-  **Red** - PolicyKit that will be consulted regarding each operation.
   It is suggested that FreeIPA project would build a set of policy
   definitions that will be packaged as a part of the core PolicyKit
   policies that would define which users or groups of users can perform
   which operations on the node. It will be a mixture of the local users
   like *root* and *openpegasus* to be able to deploy the initial
   instance as well as a predefined in FreeIPA group of admins that have
   the right to do instance management. This group will be added to IPA
   at the installation time or during upgrade to the version where the
   feature would be first implemented.
-  **Orange** - FreeIPA related components. It is expected that the
   FreeIPA tools and scripts like ``ipa-server-install`` and
   ``ipa-replica-manage`` will be refactored
   `#4468 <https://fedorahosted.org/freeipa/ticket/4468>`__ on top of a
   common python library to manage an instance that can be invoked
   directly by the ``ipaserverd`` service. The refactored tools are
   still shown as they will remain for backward compatibility. New tools
   might be developed to implement local command line for the new node
   management functions.
-  **Yellow** - FreeIPA server and client instances. The thick arrow
   shows that a new model would be implemented to be able to promote a
   client into a server
   `#3506 <https://fedorahosted.org/freeipa/ticket/3506>`__,
   `#2888 <https://fedorahosted.org/freeipa/ticket/2888>`__.

.. _scope_of_work:

Scope of work
=============

This section covers the catalog of changes that need to be implemented
to complete the feature

-  **OpenLMI**

   -  Create an OpenLMI model for FreeIPA server node management
   -  Implement an OpenLMI provider that implement this CIM model. The
      provider will use D-BUS calls to communicate with ``ipaserverd``
   -  Design interface that Server Role Server will use to get get high
      level information (KVPs)

-  **Security & Access control**

   -  Create a PolicyKit policy
   -  Integrate PolicyKit check into the ``ipaserverd``
   -  Add access group to FreeIPA (new install and upgrade)
   -  Define initial creation of SSL channel between a client and a
      server. For new systems that is a bit of leap of faith. It can be
      solved by a locally generated on the node self signed cert, by
      placing a cert when the image is created (kickstart) or via
      Puppet. regardless this aspect and security implications should be
      clearly identified and spelled out.

-  **FreeIPA node management**

   -  Define a set of node management interfaces
   -  Create a python library that will implement these interfaces
   -  Refactor existing tools to use new library
   -  Provide additional tools or options as needed
   -  Hook this API into the ``ipaserverd`` service. The ``ipaserverd``
      may need to run the processes asynchronously so that it is able to
      return status calls when installing FreeIPA server for example.

-  **Core FreeIPA changes**

   -  Implement client promotion to replica capability
   -  Move topology into replicate tree
   -  Create a plugin to manage cluster over OpenLMI python client
   -  Create IPA CLI to manage the deployment (add replica, remove
      replica, move CRL generation, add dns etc.)
   -  Create UI to display IPA topology and allow to easily manage the
      instances and their relations via this interface
   -  Create ACIs and roles for these management capabilities

-  **Log Collection**

   -  Create a command to prepare a file with a snippet of log from a
      particular log file for the specified time interval (most likely
      will be a separate OpenLMI provider not shown on the diagram)
   -  Implement an SCP kind of mechanism to fetch this file
   -  Implement a way to display content of the fetched file from IPA UI
      and CLI
   -  Add ability to attach this file to a UI plugin of the external
      case management system

These items in turn might require individual child design pages.

Implementation
==============

Any additional requirements or changes discovered during the
implementation phase.



Feature Management
==================

UI
~~

How the feature will be manged via the UI

CLI
~~~

Overview of the CLI commands



Major configuration options and enablement
==========================================

Any configuration options? Any commands to enable/disable the feature or
turn on/off its parts?

Replication
===========

Any impact on replication?



Updates and Upgrades
====================

Any impact on updates and upgrades?

Dependencies
============

Any new package and library dependencies.



External Impact
===============

Impact on other development teams and components



Backup and Restore
==================

Any files or configuration that needs to be taken care of in backup or
restore procedure.



Test Plan
=========

Test scenarios that will be transformed to test cases for FreeIPA
Continuous Integration during implementation or review phase.



RFE Author
==========

Author/contact person for the feature
