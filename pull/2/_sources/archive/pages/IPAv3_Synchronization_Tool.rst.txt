Overview
========

Synchronization tool is a collection of applications that will be used
to synchronize IPA and Samba backends so they contain identical data.
The synchronization process is described in `this
page <Obsolete:IPAv3_Synchronization_Process>`__.

.. figure:: Ipa3-synchronization-tool.png
   :alt: Ipa3-synchronization-tool.png

   Ipa3-synchronization-tool.png

.. _change_log_monitor:

Change Log Monitor
==================

The Change Log Monitor monitors changes occuring in DS then invoke the
Synchronization Agent to process the update.

.. _syncback_module:

Syncback Module
===============

The Syncback Module intercepts update requests coming into Samba just
like DS plugin then invoke the Synchronization Agent to process the
update. See also `this page <Obsolete:Samba_Syncback_Module>`__.

.. _synchronization_agent:

Synchronization Agent
=====================

The Synchronization Agent receives the notification about changes in one
tree from either Change Log Monitor or Syncback Module, transform the
object using the Mapping Engine, then updates the corresponding object
in the other tree. See also `this
page <Obsolete:Synchronization_Agent‏‎>`__.

.. _mapping_engine:

Mapping Engine
==============

The Mapping Engine transforms objects from one format into another using
a set of mapping rules. See also `this
page <Obsolete:Mapping_Engine>`__.

.. _synchronization_manager:

Synchronization Manager
=======================

The Synchronization Manager provides a user interface (command-line or
graphical) for maintaining the synchronization process which includes
initializing the backends, starting/stopping synchronization, managing
mapping rules, and resolving errors. See also `this
page <Obsolete:Synchronization_Manager>`__.

`Category:Obsolete <Category:Obsolete>`__
