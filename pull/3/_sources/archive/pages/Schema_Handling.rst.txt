When developing new features it may be necessary to extended the LDAP
schema on the server. This operation is delicate, what follows are best
practices and requirements to handle schema changes in the FreeIPA
project.

.. _prior_art:

Prior Art
---------

We generally try to use existing standard schema if at all available, so
the first step when any new schema is needed is to search the literature
and find out if anything exists and is a good fit for the task at hand.
If prior schema exist but is not sufficient we also prefer to use what
is available and then only extend it with additional auxiliary
objectclasses and attributes. If no schema exists or existing schema is
an odd fit, then new schema can be considered.

.. _general_considerations:

General Considerations
----------------------

FreeIPA is a distributed system, where multiple servers act on the same
data at the same time, this introduces additional constraints on the
kind of operations that can be performed on a server. Extreme care
should be used when defining and introducing new schema. The author (And
reviewers) always need to keep in mind how all servers will operate
during a transition phase in which some servers have the new code and
schema and some servers don't. Although schema is propagated via
replication that only happens after a new server send new changes to an
older one. For some amount of time two servers may run at different
schema revisions. This need to be accounted for. Race conditions may
happen when objects are updated at the same time on two different
servers, there are techniques that can be used to make sure updates
between two servers do not stomp on each other, if in doubt open a
discussion on the mailing list about how to best handle updates for the
specific case at hand.

.. _choosing_attributes_and_objectclasses:

Choosing attributes and objectclasses
-------------------------------------

Creating new schema requires considereing a number of factors.

.. _naming_conventions:

Naming Conventions
~~~~~~~~~~~~~~~~~~

If the schema is very FreeIPA specific we try to namespace attributes
and objectclasses by using the prefix 'ipa' in fron of attribute and
objectclass names. If the schema is of general use and nothing
standardized exist then other prefixes may be considered. We always
advice to use meaningful prefixes nonetheless to avoid potential future
name clashes with other popular schema.

OIDs
~~~~

The FreeIPA project have been assigned its own OID space under the
original 389ds OID space. The FreeIPA toplevel OID is:
2.16.840.1.113730.3.8 If you plan to create new attributes and
objectclasses please announce that on the development list and ask for
assignment, with a full schema description if available. If you haven't
worked out the details of each new object but you know exactly how many
you will need, you can get an allotment reserved too.

.. _objectclasses_and_attribute_types:

ObjectClasses and Attribute types
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Please chose the most appropriate type for attributes, it is important
to make sure you think through each attribute syntax, and equality, as
well as how using these attributes will impact performances and whether
indexes should be created. For objectclasses it is important to decide
whether the class is structural or auxiliary, and which of the
attributes should be always required (MUST) and which should be optional
(MAY). In case of doubt open a discussion on the development mailing
list.

.. _changing_existing_schema:

Changing existing Schema
------------------------

In rare cases it may be necessary to modify an existing objectclass or
an attribute to add more capabilities. We stgrongly recommend to avoid
changes if at all possible, and allow only "safe" changes.

For objectclasses we allow changing from auxiliary to structural only,
not the other way around. For attributes, an attribute can only go from
MUST to MAY, not the other way around. Attributes cannot be dropped, and
new attributes can only be added as MAY. For attributes, changes in
syntax are allowed only if the new syntax is a complete superset of the
previous one. Changes in equality matches are allowed. A SINGLE-VALUE
attribute can become multivalued but not the other way around.

.. _technical_details:

Technical Details
-----------------

To change the schema, edit (or add) an appropriate
``install/share/*.ldif`` file in the FreeIPA source. When creating a new
file, it needs to be added to ``install/share/Makefile.am`` and to
``IPA_SCHEMA_FILES`` in ``ipaserver/install/dsinstance.py``. See
`Contribute/Code <Contribute/Code>`__ for how to submit your change.

Background
----------

A good place to learn about LDAP schema concepts is `"LDAP for Rocket
Scientists" at zytrax.com <http://www.zytrax.com/books/ldap/ch3/>`__.
