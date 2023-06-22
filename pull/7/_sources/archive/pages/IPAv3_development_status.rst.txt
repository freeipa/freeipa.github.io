Overview
========

The main task we see in front of us as a next step after finishing the
IPA v2 development work is allowing IPA and AD to coexist in natural and
trusted way in the customer environment. This page is dedicated to
tracking this effort.

Concept
=======

For IPA to become integrated and trusted by AD, IPA needs to be able to
pretend as if it is an AD domain controller. This can be accomplished by
integrating Samba 4 and IPA. The two components need to be able to
operate on the same data and share the KDC. The following page gives
deeper into the drivers and high level architecture of the proposed
solution: `"IPA and AD" integration. <IPA_and_AD>`__

Effort
======

.. _system_integration:

System Integration
------------------

-  `Architecture <Obsolete:IPAv3_Architecture>`__

.. _backend_synchronization:

Backend Synchronization
-----------------------

-  `Design <Obsolete:IPAv3_Synchronization_Design>`__
-  `Task List <Obsolete:IPAv3_Synchronization_Task_List>`__
-  `Technical Notes <Obsolete:IPAv3_Technical_Notes>`__

.. _kerberos_integration:

Kerberos Integration
--------------------

The Kerberos integration effort is tracked here:

-  `Project
   Page <http://k5wiki.kerberos.org/wiki/Projects/Samba4_Port>`__
-  `Attempt to itemize tasks for the
   effort <http://k5wiki.kerberos.org/wiki/Task-List_for_Samba4_Port_(Andrew_Bartlett)>`__

.. _new_dns_interface:

New DNS Interface
-----------------

-  `Design Page (done in V2) <V2/DNS_Interface_Design>`__

`Category:Obsolete <Category:Obsolete>`__
