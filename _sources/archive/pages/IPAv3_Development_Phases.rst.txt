Overview
========

This page describes the development phases for the IPA & Samba backend
synchronization.

.. _phase_1:

Phase 1
=======

In the first phase we will have 3 systems: IPA, Samba, and AD. Each
system will have a client, server, and name server. A synchronization
will be set up between IPA and Samba. A trust will be established
between Samba and AD.

.. figure:: Ipa3-phase1.png
   :alt: Ipa3-phase1.png
   :width: 200px

   Ipa3-phase1.png

.. _phase_2:

Phase 2
=======

In the second phase we will set up an additional server in each system.
A replication will be set up between the old and the new server. A
synchronization will be set up between the new IPA server and new Samba
server. The name servers will be configured such that the client can
connect to any of the servers in each system.

.. figure:: Ipa3-phase2.png
   :alt: Ipa3-phase2.png
   :width: 200px

   Ipa3-phase2.png

.. _phase_3:

Phase 3
=======

In the third phase we will combine the backends of each pair of IPA and
Samba servers. We will also move the IPA and Samba servers into a single
machine. The IPA and Samba name servers will also be combined.

.. figure:: Ipa3-phase3.png
   :alt: Ipa3-phase3.png
   :width: 200px

   Ipa3-phase3.png

`Category:Obsolete <Category:Obsolete>`__
