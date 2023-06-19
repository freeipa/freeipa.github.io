Overview
========

This document describes the high-level design of IPA and Samba
integration. The integration is designed such that these applications
represent a single domain/realm by sharing their data.

.. _shared_backend:

Shared Backend
==============

IPA and Samba use a hierarchical data structure to store the data in
their backend database. In LDAP terminology the structure is called
Directory Information Tree (DIT). Both IPA and Samba can use 389
Directory Server (DS) as a backend.

In IPAv3, a single DS instance will be used to store both IPA DIT and
Samba DIT and these DIT's will be synchronized.

.. figure:: Ipa3-shared-backend.png
   :alt: Ipa3-shared-backend.png

   Ipa3-shared-backend.png

The unified DIT is described in `this
page <Obsolete:IPAv3_Unified_DIT>`__. The synchronization process is
described in `this page <Obsolete:IPAv3_Synchronization_Process>`__.

.. _shared_objects:

Shared Objects
==============

Each DIT contains many objects, but only some of them need to be shared
between these applications.

Shared objects include:

-  users
-  groups
-  hosts

Each shared object in one application will have a corresponding object
in the other application representing the same entity. The objects are
linked using the GUID.

.. figure:: Ipa3-shared-objects.png
   :alt: Ipa3-shared-objects.png

   Ipa3-shared-objects.png

The object mapping is described in `this
page <Obsolete:IPAv3_DIT_Mapping>`__.

.. _shared_attributes:

Shared Attributes
=================

Each object contains many attributes, but only some of them need to be
shared between these applications.

.. figure:: Ipa3-shared-attributes.png
   :alt: Ipa3-shared-attributes.png

   Ipa3-shared-attributes.png

The attribute mapping is described in these pages:

-  `User Attribute Mapping <Obsolete:IPAv3_User_Attribute_Mapping>`__
-  `Group Attribute Mapping <Obsolete:IPAv3_Group_Attribute_Mapping>`__

`Category:Obsolete <Category:Obsolete>`__
