.. _about_domain_levels:

About domain levels
-------------------

Domain level indicates that server is capable of doing certain
operations. Domain levels allows to migrate to a never version of
freeIPA and activate new features when all servers are migrated and
compatible with that particular feature. The domain level has to be
increased manually, it is not raised during upgrade.

Details on `design page <V4/Domain_Levels>`__

.. _current_domain_level:

Current domain level
~~~~~~~~~~~~~~~~~~~~

To get value of current domain level, please execute the following
command:

| ``$ ipa ``\ **``domainlevel-get``**
| ``-----------------------``
| ``Current domain level: 0``
| ``-----------------------``

.. _increase_domain_level:

Increase domain level
~~~~~~~~~~~~~~~~~~~~~

To increase domain level, please use the following command:

| ``$ ipa ``\ **``domainlevel-set``**\ `` 1``
| ``-----------------------``
| ``Current domain level: 1``
| ``-----------------------``

.. _current_domain_levels:

Current domain levels
---------------------

+--------------+-----------------------+--------------------------+
| Domain level | Introduced in version | Features                 |
+==============+=======================+==========================+
| 0            | 4.2                   | compatible domain level  |
|              |                       | with older IPA versions  |
+--------------+-----------------------+--------------------------+
| 1            | 4.3                   | `Replica                 |
|              |                       | promotion <Domain_Level  |
|              |                       | s#Replica_promotion>`__, |
|              |                       | `Topology management     |
|              |                       | plug                     |
|              |                       | in <Domain_Levels#Topolo |
|              |                       | gy_management_plugin>`__ |
+--------------+-----------------------+--------------------------+

Features
--------

.. _replica_promotion:

Replica promotion
~~~~~~~~~~~~~~~~~

Details on `design page <V4/Replica_Promotion>`__

Howto install replica: `release notes
4.3.0 <Releases/4.3.0#Replica_installation>`__

.. _topology_management_plugin:

Topology management plugin
~~~~~~~~~~~~~~~~~~~~~~~~~~

Details on `design page <V4/Manage_replication_topology>`__
