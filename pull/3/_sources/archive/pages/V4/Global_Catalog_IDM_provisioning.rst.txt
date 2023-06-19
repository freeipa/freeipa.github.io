Overview
--------

Global Catalog is a special service that exposes information about
objects in FreeIPA domain in a way expected by the clients enrolled to
Active Directory environments. Active Directory domain controllers and
other systems use Global Catalog to search across the potentially
multi-domain forest for the information about users, groups, machines,
and other types of objects.

The information stored in the Global Catalog represents a subset of the
information available at the specific domain controllers in Active
Directory. As such, it is only accessible for read-only purposes. Any
change to the information shall come through the domain controllers
responsible for a domain in question.

The information stored in the Global Catalog are provisioned from a
Freeipa LDAP instance (aka *primary* instance). The Global Catalog is
available with an other LDAP instance known as *AD* instance. *primary*
and *secondary* instances do not share the same schema, so there is
required transformation of the provisioned LDAP entry to conform Global
Catalog needs.

This document describes the provisioning from *primary* instance and the
transformation of entries on the *AD* instance.

.. _use_case:

Use Case
--------

Design
------

FreeIPA Global Catalog service is a read-only LDAP service available
over port 3268. This service runs on a *AD* LDAP instance that use a
schema compatible with Active Directory LDAP schema. Entries in *AD*
instance are provisioned from a *primary* freeipa ldap instance, that
runs a different schema. So provisioned entries are crafted to conform
the *AD* instance schema.

The consequence of running different schema is that it is **not
possible** to use regular multimaster replication between the *primary*
and *AD* LDAP instances. In fact multimaster replication, replicates the
schema. The provisioning relies `content
synchronization <http://www.port389.org/docs/389ds/design/content-synchronization-plugin.html>`__
(RFC4533) supported by 389-ds since 1.3.5.

The LDAP entries exposed on Global Catalog represents a subset of the
information available at *primary* instance and also adaptation of the
attributes/objectclass to conforms the *AD* instance. The transformation
(remove/add/modification of attributes/objectclass) will be done by
plugins running on *AD* instance. These plugins will do the same jobs as
`slapi-nis <https://git.fedorahosted.org/cgit/slapi-nis.git/tree/doc/format-specifiers.txt>`__

Provisioning
~~~~~~~~~~~~

Provisioning relies on `content
synchronization <http://www.port389.org/docs/389ds/design/content-synchronization-plugin.html>`__
(RFC4533). This mechanism is triggered when a client sends a *SEARCH*
request with a *Sync request control*. The synchronization is done via a
dedicated client thread, that will run **RefreshandPersist** mode (see
`RefreshAndPersist <https://tools.ietf.org/html/rfc4533#page-16>`__)
with or without cookie.

.. _content_synchronization_protocol_elements:

Content synchronization protocol elements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _sync_request_control:

Sync Request Control
''''''''''''''''''''

This control is used in the search request to trigger a content
synchronization

| ``   syncRequestValue ::= SEQUENCE {``
| ``         mode ENUMERATED {``
| ``             -- 0 unused``
| ``             refreshOnly       (1),``
| ``             -- 2 reserved``
| ``             refreshAndPersist (3)``
| ``         },``
| ``         cookie     syncCookie OPTIONAL,``
| ``         reloadHint BOOLEAN DEFAULT FALSE``
| ``     }``

.. _sync_info_message:

Sync Info Message
'''''''''''''''''

| ``  syncInfoValue ::= CHOICE {``
| ``         newcookie      [0] syncCookie,``
| ``         refreshDelete  [1] SEQUENCE {``
| ``             cookie         syncCookie OPTIONAL,``
| ``             refreshDone    BOOLEAN DEFAULT TRUE``
| ``         },``
| ``         refreshPresent [2] SEQUENCE {``
| ``             cookie         syncCookie OPTIONAL,``
| ``             refreshDone    BOOLEAN DEFAULT TRUE``
| ``         },``
| ``         syncIdSet      [3] SEQUENCE {``
| ``             cookie         syncCookie OPTIONAL,``
| ``             refreshDeletes BOOLEAN DEFAULT FALSE,``
| ``             syncUUIDs      SET OF syncUUID``
| ``         }``
| ``     }``

In *RefreshAndPersist* mode, the refresh stage does **not** end with a
SearchResultDone but with **refreshDone** (refreshDelete or
refreshPresent) and a **newcookie**.

In persist stage, when multiple entries are deleted, it uses several
sync info messages with **refreshDeletes=TRUE** and the set of syncUUIDs
deleted.

.. _sync_state_control:

Sync state control
''''''''''''''''''

| ``     syncStateValue ::= SEQUENCE {``
| ``         state ENUMERATED {``
| ``             present (0),``
| ``             add (1),``
| ``             modify (2),``
| ``             delete (3)``
| ``         },``
| ``         entryUUID syncUUID,``
| ``         cookie    syncCookie OPTIONAL``
| ``     }``

In persist stage, for each updated entry/reference is send in a
SearchResultEntry/SearchResultReference message with sync state control.
The control contains the entryUUID. For *ADD* entry, state is **add**.
For *MOD/MODDN* entry, state is **modify**. For *DEL* entry, state is
**delete**.

Note: in this implementation we assume we will not receive
SearchResultReference message.

.. _refresh_stage:

Refresh stage
^^^^^^^^^^^^^

During `refresh
stage <http://www.port389.org/docs/389ds/design/content-synchronization-plugin.html#refresh-only-cookie-present>`__,
the synchronization client thread will receive ldap messages containing
entries with sync state control. The control state is either **Present**
or **Delete**. A set of **Present** entry messages is known as *Present
phase*. A set of **Delete** entry message is known as **Delete** phase.
XXX Not clear if phases are clearly separated or we can receive
Present-Delete-Present for example XXX.

If no cookie is provided (*initial content*), only **Present** sync
state control are received. Then the **primary** server will end the
Refresh phase sending an **intermediate** message to switch to
persistent phase.

If a cookie is provided (*content update*), the primary server starts
with **Delete** phase sending all entries deleted since the cookie
timestamp. They are send with intermediate message(s) (tag
LDAP_TAG_SYNC_ID_SET) XXsync_send_deleted_entriesXX. Then the server
enters in a **Present** phase sending all entries that have been
added/modified/moddn since the cookie timestamp. They are send with
LDAP_RES_SEARCH_ENTRY msg with sync control.

.. _persistant_stage:

Persistant stage
^^^^^^^^^^^^^^^^

Transformations
~~~~~~~~~~~~~~~
