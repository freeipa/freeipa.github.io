Overview
--------

Ticket `#4074 <https://fedorahosted.org/freeipa/ticket/4074>`__; also
see the `-devel
thread <http://www.redhat.com/archives/freeipa-devel/2013-December/msg00063.html>`__

The permission target filter will become multi-valued.

"Type" permissions, such as most default permissions, will use
objectclass target filters instead of wildcard targets.

This is a change to `Permissions_V2 <V3/Permissions_V2>`__ that will be
implemented in the same release.

An additional virtual attribute, ``extratargetfilter`` (``--filter`` in
the CLI), will show/update filters that are not implied by ``--user`` or
--memberof.



Use Cases
---------

It is now possible to use multiple --filter and --memberof options,
possibly in combination with -type:

| ``$ ipa permission-add foo --type user --filter '(sn=Smith)' --filter '(givenname=John)' --memberof editors --right read``
| ``----------------------``
| ``Added permission "foo"``
| ``----------------------``
| ``  Permission name: foo``
| ``  Granted rights: read``
| ``  Bind rule type: permission``
| ``  Subtree: cn=users,cn=accounts,dc=idm,dc=lab,dc=eng,dc=brq,dc=redhat,dc=com``
| ``  Extra target filter: (givenname=John), (sn=Smith)``
| ``  Member of group: editors``
| ``  Type: user``

The --type and --memberof options create filters that can be viewed with
the --all option:

| ``$ ipa permission-show foo --all``
| `` dn: cn=foo,cn=permissions,cn=pbac,dc=idm,dc=lab,dc=eng,dc=brq,dc=redhat,dc=com``
| `` Permission name: foo``
| `` Granted rights: read``
| `` Bind rule type: permission``
| `` Subtree: cn=users,cn=accounts,dc=idm,dc=lab,dc=eng,dc=brq,dc=redhat,dc=com``
| `` Extra target filter: (givenname=John), (sn=Smith)``
| `` Raw target filter: (sn=Smith), (givenname=John), (memberOf=cn=editors,cn=groups,cn=accounts,dc=idm,dc=lab,dc=eng,dc=brq,dc=redhat,dc=com), (objectclass=posixaccount)``
| `` Member of group: editors``
| `` Type: user``
| `` ipapermissiontype: V2, SYSTEM``
| `` objectclass: ipapermission, top, groupofnames, ipapermissionv2``

Design
------

.. _multi_valued_ipapermtargetfilter:

Multi-valued ipapermtargetfilter
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``ipapermtargetfilter`` attribute, and its corresponding
``rawfilter`` option, will become multi-valued. When more than one value
is present, they all need to apply -- they will be joined by ``(& )`` to
create the ACI.

.. _multivalued___memberof:

Multivalued --memberof
~~~~~~~~~~~~~~~~~~~~~~

Currently the ``--memberof`` option of ``permission-mod`` sets the
targetfilter to ``(memberof=``\ *``groupname``*\ ``)``.

The option will become multi-valued, and it will no longer conflict with
the ``--filter`` option.

When ``--memberof`` is specified, the ``permission-mod`` command will
remove any existing ``(memberof=...)`` filter(s) that correspond co
concrete groups, but leave any other filters; then add any additional
filter(s) specified by the ``--memberof``, ``--type``, ``--filter``
options.

The ``permission-add`` and ``permission-find`` commands will only add
the memberof filter to any filter(s) specified by other options.

On output, memberof filter(s) matching existing group name(s) will cause
corresponding memberof output items.

=== --type sets (objectclass=...) targetfilter ===

Currently the ``--type`` option sets the ACI location to the appropriate
container DN, and the target to a wildcard DN:
*``uid_attr``*\ ``=*,``\ *``container_dn``*.

Instead of setting the target, the option will now set the target filter
to ``(objectclass=...)`` (or possibly, multiple such filters).

Similarly to ``--memberof``, ``permission-mod``'s ``--type`` will any
existing ``(objectclass=...)`` filter(s) corresponding to a pre-existing
type.

On output, if the ACI location matches an eligible object type, and
proper objectclass filters are present, a corresponding type will be
reported.

.. _canonical_objectclasses_for_filter:

Canonical objectclasses for filter
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Each IPA object type that can be used for the ``--type`` option will be
assigned an object class that will be used for the filter. These will
initially be:

user
   posixaccount
group
   ipausergroup or posixgroup
host
   ipahost
service
   ipaservice
hostgroup
   ipahostgroup
netgroup
   ipanisnetgroup
dnsrecord
   idnsrecord

These will be declared in their respective plugin classes. The existence
of this declaration will make the type usable in a permission (in
contrast with the current situation, where a list of types is hardcoded
in the permission & ACI plugins).

.. _raw_targetfilter_vs._extratargetfilter:

Raw targetfilter vs. extratargetfilter
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In CLI, ``ipapermtargetfilter`` will be accessible as ``--rawfilter``.
Setting the option affects the type & memberof virtual attributes. On
output, the value will only be present if ``--all`` or ``--raw`` is
specified.

Another multivalued option, ``extratargetfilter`` (CLI name: ``filter``)
will only list the target filters that are not implied by the
``memberof`` and ``type`` virtual attributes. When setting this option,
these implied filters will be preserved.

Implementation
--------------

Additional requirements or changes discovered during the implementation
phase were merged into this document.



Feature Management
------------------

UI
~~

The necessary UI design and changes should be done as part of
`V3/Permissions V2 <V3/Permissions_V2>`__.

CLI
~~~

Permission ``--memberof`` and ``--filter`` options will now accept
multiple values.



Updates and Upgrades
--------------------

This change will be implemented in the same release as `V3/Permissions
V2 <V3/Permissions_V2>`__. See that design for update concerns.

Dependencies
------------

No new package and library dependencies.



External Impact
---------------

Externally, this is a part of `V3/Permissions V2 <V3/Permissions_V2>`__.



RFE Author
----------

`Petr Viktorin <User:Pviktorin>`__
