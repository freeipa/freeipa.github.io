Overview
--------

Having installed schema in one file and schema updates in other file(s)
is error prone and causes issues as
`#3398 <https://fedorahosted.org/freeipa/ticket/3398>`__.

We should instead base updates on the schema files and have the updater
validate current DS schema with the LDIF file and amend the schema.



Use Cases
---------

Upgrade IPA from a previous version. The upgrade and the upgraded IPA
work without problems.

The behavior does not change, except for content of logs and arguments
of an internal tool.

Design
------

We use a process that Simo suggested:

-  Download schema from server
-  Parse the schema files and check if each attribute and objectclass is
   present and in the correct form.
-  If any attribute is missing, we add it
-  If any attribute has been changed, we change it
-  Same for object classes.

``python-ldap``'s convenience classes are be used to make the comparing
easier.

The X-ORIGIN tag of any added or changed attribute is set to the current
IPA version. (This is partly for technical reasons: ``python-ldap``
doesn't parse X-ORIGIN).

Implementation
--------------

No additional requirements or changes discovered during the
implementation phase.



Feature Management
------------------

UI
~~

N/A

CLI
~~~

See the "configuration options and enablement" section for info on
manual upgrades.



Major configuration options and enablement
------------------------------------------

The ``ipa-ldap-updater`` command will grow two more options:

-  ``--schema``: Also update the LDAP schema. If no ``--schema-file`` is
   specified, update to the built-in IPA schema.
-  ``--schema-file=FILE.ldif``: Specify a schema file. May be used
   multiple times. Implies ``--schema``.

In ``--upgrade`` mode, ``--schema`` is assumed.

Note that the ``ipa-ldap-updater`` command is not intended for end
users.

Replication
-----------

N/A



Updates and Upgrades
--------------------

Please read this whole document for information on this feature's impact
on upgrades :)

Dependencies
------------

This will add a dependency on ``python-ldap``'s schema parser. We
already depend on ``python-ldap`` (though mostly through our wrapper).



External Impact
---------------

N/A



Backup and Restore
------------------

N/A



Test Plan
---------

Existing upgrade tests should be enough; functionality should be
unchanged.
