Overview
========

For legacy applications, FreeIPA maintains in the domain DIT a set of
entries conforming AD schema. Those entries are kept in memory map and
managed by *Schema compat plugins*. The in-memory maps are protected by
a lock that acts basically like a backend lock. Those entries are not
indexed so a search can hit a performance issue. In addition several
deadlock
`scenario <https://www.freeipa.org/page/V4_slapi_nis_locking>`__ exist
if to process a request we need to access pages in real domain entries.

It was decided

-  to move in-memory maps into real backend entries to benefit from
   indexing and avoid deadlock scenario.
-  to move the AD conforming entries into a dedicated and separated DS
   instance (running with AD schema): **secondary instance**
-  The mapping is **no more running on the primary instance** instead it
   runs in the separated DS instance

The AD dedicated instance contains **only** AD like entries. Regular
(posix) FreeIPA entries are limited to the main FreeIPA instance.

In the rest of the document, AD instance will also be called *secondary*
or *RFC2307* or *AD* instance. The main FreeIPA instance will also be
called *primary* or *posix* or *main* instance.

In the rest of the document entries on the primary instance are also
called *primary entries* or *original entries*. The entries on the
secondary instance are also called *secondary entries* or *mapped
entries* or *duplicated twin entries*.

Design
======

.. _initial_situation:

Initial situation
-----------------

A FreeIPA instance contains two main backends: **UserRoot** and
**ipaca**.

**ipaca** contains the certificate information and is not impacted by
this design.

containers
~~~~~~~~~~

**userRoot** DIT has multiple containers: **users**, **groups**,
**hosts**, **computers**, **sudo rules**, **hbac rules**... and a
special container **compat**. All containers are stored in LDAP DB files
on disk **except compat** that is stored in memory tables (aka *maps*).
A user entry (under **users** container) is duplicated and transformed
into a user entry under the **compat** container. It is the same for
**groups** and **computers**.

.. figure:: GC_slapi_nis_img_current_situation.png
   :alt: GC_slapi_nis_img_current_situation.png

   GC_slapi_nis_img_current_situation.png

The regular containers contain entries conforming posix schema. Those
containers are replicated on all servers in a FreeIPA topology. The
*compat* container contains *AD* like entries and conforms *AD schema*.
The entries in that container **are not** replicated to the others
servers, instead each server manage its own set of mapped entries.

.. _schema_compat_container:

Schema Compat container
~~~~~~~~~~~~~~~~~~~~~~~

The **compat** container stores entries into in-memory maps

The entries can be retrieved with LDAP search. LDAP search request can
lookup the original entry (i.e.
uid=my_user,cn=users,cn=accounts,dc=domain) as well as its twin
(duplicate/transform) entry (i.e.
uid=my_user,cn=users,cn=compat,dc=domain).

The mapped entries are provisioned (from on disk containers users,
groups and computers) at server startup. Then a LDAP write operation
that updates a user (or groups or computer) into the disk DB, is also
processed by **Schema Compat** plugin (aka slapi-nis) that
duplicate/transform the updated entries and stores the resulting entries
into a 'in-memory' map. To do this transformation, Schema Compat plugin
apply a **Formating definition** specific to type of entries (users,
groups or computers) that is mapped

.. figure:: GC_slapi_nis_img_schema_compat.png
   :alt: GC_slapi_nis_img_schema_compat.png

   GC_slapi_nis_img_schema_compat.png

.. _formating_keyword_deref:

Formating keyword: deref
~~~~~~~~~~~~~~~~~~~~~~~~

A **Formating definition** contains a set `format
specifiers <https://www.freeipa.org/page/FreeIPAv2:Schema_Compatibility_Plug-in_Design#Back_End>`__.
The specifier *schema-compat-entry-attribute* describes with *keywords*
how to create the values of the *compat* entries. The
`deref <https://www.freeipa.org/page/FreeIPAv2:Schema_Compatibility_Plug-in_Design#deref.28THISATTRIBUTE.2CTHATATTRIBUTE.29>`__
keyword allows to retrieve values from the entries in the DB.

In the example below, a new developer (dev1) is added to a group
(engineering). The group *cn=engineering,* is updated. Schema compat
catches the update and updates the compat group *cn=engineering,*.
Schema compat uses the *groups formating definition* in order to updates
the attribute *memberUid* in the compat group. The *memberUid* requires
to dereference the *member* value of the group *cn=engineering,*. It
refers to user *dev1*. Schema compat does **an internal search** to
retrieve that user. Then it adds the *uid* value of that user as value
of *memberUid*. In short **deref keyword requires lookup in
users/groups/computer that are in the DB**.

.. figure:: GC_slapi_nis_img_SC_deref.png
   :alt: GC_slapi_nis_img_SC_deref.png

   GC_slapi_nis_img_SC_deref.png

.. _primary_instance:

Primary instance
----------------

Because of performance (no index) and robustness (deadlocks) concerns,
the Freeipa DS instance is separated into two DS instances: primary and
secondary.

The primary contains all FreeIPA posix entries

.. figure:: GC_slapi_nis_img_sec_inst_overview.png
   :alt: GC_slapi_nis_img_sec_inst_overview.png

   GC_slapi_nis_img_sec_inst_overview.png

.. _compat_chained_backend:

Compat chained backend
~~~~~~~~~~~~~~~~~~~~~~

The compat containers on the primary instance are chained to the
secondary instance.

All operations are replayed on the secondary instance. Write operation
will be rejected because compat backend on secondary is in read-only.
(TBD should also the compat backend be in read-only on the primary
instance ?)

Read operations are replayed on secondary instance. Authentication on
secondary instance TBD. ACI evaluation is done locally
(*nsCheckLocalACI*) on the primary instance because access control
mostly rely on group membership and we do not want to provision
groups/users on secondary instance.

.. _secondary_instance:

Secondary instance
------------------

A secondary instance will run with RFC 2307 schema. This instance will
store on disk DB entries (conforming RFC 2307 schema). These entries are
duplicate/transformed from entries that are in the primary instance
(posix schema). Only few containers entries (users, groups and
computers) from the primary instance are stored in the secondary
instance.

.. _rfc2307_map_plugin:

RFC2307 map plugin
~~~~~~~~~~~~~~~~~~

The original entries, that are updated on the primary instance, are
retrieved with a sync-repl lookup. There is **one sync-repl** instance
**per** container (in the figure above only *users* container is
showed). So on the secondary instance there will be at least **3
sync-repl** instance (for users, groups and computers).

A sync-repl instance runs in a RFC2307 map plugin, so several RFC2307
map plugin will run with different config entries (similar to
uiduniqueness plugin). The type of the plugin is *TBD*. At plugin
startup, it spawn a dedicated thread that connects to the primary
instance.

Authentication on the primary instance is TBD.

The plugin config entry contains the *base* search, *filter* and
*cookie* of the sync repl. A possibility is to keep those info into a
specific entry into the mapped container (like a RUV entry).

The config entry also contains the `formating
specifiers <https://www.freeipa.org/page/FreeIPAv2:Schema_Compatibility_Plug-in_Design#Back_End>`__.
The lookup entries are processed with the formating specifiers and then
stored locally.

Sync_repl retrieves a full entry (not an update), so each time an
primary entry is updated, the secondary entry is **deleted** and
**added**. That means that secondary entries (for example in
*compat_users* containers) are **leafs**. This limitation is to reduce
the complexity of evaluating MODs of an entry. An updated primary
entries, triggers a DEL and a ADD of its secondary entry.

Sync_repl **is not synchronized** with the LDAP update on the primary
instance. That means the secondary entry will be updated **after** the
update of the primary entry. If a client application updates a **primary
entry** and then immediately does LDAP search on primary instance of the
compat_user entry. The LDAP search will following the chained backend
and retrieve the entry from the secondary instance. There is a
possibility that this entry **does not reflect** the update done on the
primary instance. Compare to `current
status <https://www.freeipa.org/page/V4/chained_compat_tree#Schema_Compat_container>`__
where the updates are atomic, with a secondary instance **updates of the
primary and secondary entry are not longer atomic**

RFC2307 map plugin spawn a dedicated thread that

-  connects/bind to the primary instance. If connection/authentication
   fails it iterates
-  It retrieves *users_cookie* from a specific entry under compat_users
   container. If there is a **cookie** it does a sync_repl **refresh and
   persist with cookie**. If there is **no cookie** it does a sync_repl
   '''refresh and persist without cookie.
-  from each retrieved entry, it applies the formatting.
-  when `Sync Info intermediate
   message <http://www.port389.org/docs/389ds/design/content-synchronization-plugin.html#sync-info-message>`__
   is received (end of refresh), it sets a flag that the **compat_users
   container is initialized**
-  Then the following retrieved entries are formatted and do (requires
   it is a **leaf**).

   -  If the changetype is delete, it DEL the entry in the compat_users
      container
   -  if the changetype is add or modify, then if the entry did not
      exist in the compat_user container it ADD the entry, else it DEL
      and ADD the entry
   -  if the changetype is MODRDN, then it DEL the source entry and ADD
      the destination entry

-  Under the same transaction it updates the **users_cookie** that is
   kind of RUV

.. _compat_backend:

compat backend
~~~~~~~~~~~~~~

The compat backend on the secondary instance is read-only. Internal
updates to that backend will need the flag
*SLAPI_OP_FLAG_BYPASS_REFERRALS*.

.. _formating_keyword_deref_1:

Formating keyword deref
~~~~~~~~~~~~~~~~~~~~~~~

The formating **deref** keyword is an expensive keyword. It was already
an expensive keyword when Schema compat plugin was running on primary
instance. It will also be expensive with the proposed design.

In the figure below, we can see the RFC2307 map plugin that reads the
update of posix *grp1*. According to the formating definitions (*dn:
cn=groups,cn=rfc_map,cn=plugins,cn=config*), it duplicates and
transforms the original posix group into a AD group. To generate the
*memberUID* value of the AD group, it needs to dereference the members
of the original posix group. Those members are posix entries: *user_1*
and *user_2*.

In the current implementation, it is done with internal searches on a
local backend.

With the new implementation, the secondary instance (AD instance) does
not contain locally the posix entries. So the internal search will
retrieve them with a chained suffix to the primary (posix) instance.

.. figure:: GC_slapi_nis_img_sec_inst_deref.png
   :alt: GC_slapi_nis_img_sec_inst_deref.png

   GC_slapi_nis_img_sec_inst_deref.png

Improving deref keyword (for example `adding member to a large static
group <https://bugzilla.redhat.com/show_bug.cgi?id=1364144>`__) is
beyond the scope of this design.

This design is in progress and the following lines are not stable. It is
just a set of ideas

Majors issues

-  updates of the primary and secondary entry are not longer atomic.
-  compat entries are leafs
-  The primary instance contains typical FreeIPA master information.
   This reflect posix entries, using a posix schema
-  The secondary instance runs with a AD schema
-  regular replication is not possible between primary and secondary
-  Secondary instance will contain Gobal Catalog entries that are pulled
   from primary instance using sync_repl. Source tree is TDB, target
   tree is a specify backend 'dc=global catalog'
-  Secondary instance will contain compat entries taken from
   'dc=compat,' from the primary instance.
-  There is two options, fill it with plugins running on primary (but
   this would mean slapi-nis running on primary doing internal update on
   chained suffix rather than managing a in-memory map)
-  on secondary instance, compat can be read-only backend and rfc-map
   plugin should use the appropriate flag
-  provision the remote 'dc=compat,' with a sync_repl (prefered)
-  change the name 'primary/secondary' into posix/rfc2307 instance
