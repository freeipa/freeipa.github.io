Synchronization_Agent
=====================

Overview
========

Synchronization agent is responsible for synchronizing IPA and Samba
DIT's. The synchronization process is described in `this
page <Obsolete:IPAv3_Synchronization_Process>`__.



Management Operations
=====================

The following methods can be used to manage the synchronization process.

initialize()
------------

This method initializes IPA and Samba directories.



start() and stop()
------------------

This method enable/disable the Change Log Monitor and Syncback Module.



Synchronization Operations
==========================

The following methods can be invoked by the Change Log Manager or
Syncback Module to execute the synchronization process.



ipaObjectAdded(dn, attributes)
------------------------------

This method should be invoked when an IPA object is added. It will
create the corresponding Samba object.



ipaObjectModified(dn, modifications)
------------------------------------

This method should be invoked when an IPA object is modified. It will
update the corresponding Samba object.



ipaObjectRenamed(dn, newRdn, deleteOldRdn)
------------------------------------------

This method should be invoked when an IPA object is renamed. It will
rename the corresponding Samba object.

ipaObjectDeleted(dn)
--------------------

This method should be invoked when an IPA object is deleted. It will
remove the corresponding Samba object.



sambaObjectAdded(dn, attributes)
--------------------------------

This method should be invoked when an Samba object is added. It will
create the corresponding IPA object.



sambaObjectModified(dn, modifications)
--------------------------------------

This method should be invoked when an Samba object is modified. It will
update the corresponding IPA object.



sambaObjectRenamed(dn, newRdn, deleteOldRdn)
--------------------------------------------

This method should be invoked when an Samba object is renamed. It will
rename the corresponding IPA object.

sambaObjectDeleted(dn)
----------------------

This method should be invoked when an Samba object is deleted. It will
remove the corresponding IPA object.



Error Recovery Operations
=========================

The following methods can be invoked to recover from errors.

ipaSyncAll()
------------

This method will synchronize all IPA objects into Samba.

sambaSyncAll()
--------------

This method will synchronize all Samba objects into IPA.

ipaSyncObject(dn)
-----------------

This method will synchronize a single IPA object into Samba.

sambaSyncObject(dn)
-------------------

This method will synchronize a single Samba object into IPA.

`Category:Obsolete <Category:Obsolete>`__