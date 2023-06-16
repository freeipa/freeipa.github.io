Overview
========

Currently the provisioning code for different backends are clumped
together in a single Python class. It uses a number of if-then
statements to control the path of the code. To improve manageability the
code should be refactored into several classes utilizing more
object-oriented techniques.

.. _class_hierarchy:

Class Hierarchy
===============

::

   ProvisionBackend
   + LDBBackend
   + ExistingBackend
   + LDAPBackend
     + FDSBackend
     + OpenLDAPBackend

ProvisionBackend
================

The ProvisionBackend is the base class for all Samba backend classes. It
defines some methods that should be overridden by the subclasses:

-  init() - initializing backend
-  start() - starting backend
-  shutdown() - shutting down backend
-  post_setup() - post-setup configuration

LDBBackend
==========

The LDBBackend is the default TDB backend.

ExistingBackend
===============

The ExistingBackend is used with an existing LDAP server.

LDAPBackend
===========

The LDAPBackend is used with a new LDAP server. It defines a method that
should be overridden by the subclasses:

-  provision() - backend specific provisioning steps

FDSBackend
==========

The FDSBackend is used with 389 DS server.

OpenLDAPBackend
===============

The OpenLDAPBackend is used with OpenLDAP server.

Patches
=======

-  `s4:provision - Added initial implementation of FDSBackend and
   OpenLDAPBackend <http://gitweb.samba.org/?p=samba.git;a=commit;h=fbc5696e38754b6014875c231edd5f56479e134b>`__
-  `s4:provision - Added start() method in
   LDAPBackend <http://gitweb.samba.org/?p=samba.git;a=commit;h=be766a384173bb02c5306e5884d1228973fe5dd7>`__
-  `s4:provision - Moved provision_xxx_backend() into backend-specific
   provision()
   method <http://gitweb.samba.org/?p=samba.git;a=commit;h=ba12eb99a04671197b92c998d72c09fd5c23c5da>`__
-  `s4:provision - Added setup() method in
   LDAPBackend <http://gitweb.samba.org/?p=samba.git;a=commit;h=1564067fbc8490bcef5523db1d7e997dca00f0bf>`__
-  `s4:provision - Added constructors for FDSBackend and
   OpenLDAPBackend <http://gitweb.samba.org/?p=samba.git;a=commit;h=55bb60a5db559a06a05a1be6633d92b8f6555c08>`__
-  `s4:provision - Added LDBBackend and
   ExistingBackend <http://gitweb.samba.org/?p=samba.git;a=commit;h=f3bc54a8f1a405bfd8886bd46a1c2ca1b47acae7>`__

`Category:Obsolete <Category:Obsolete>`__
