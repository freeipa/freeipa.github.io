Archived from:
http://blog.benjaminlipton.com/2016/07/19/csr-generation-templating.html

Background
==========

I am working on a project
(`ticket <https://fedorahosted.org/freeipa/ticket/4899>`__,
`design <http://www.freeipa.org/page/V4/Automatic_Certificate_Request_Generation>`__)
to simplify creating certificates in FreeIPA. Currently administrators
must generate a Certificate Signing Request (CSR) matching the type of
certificate they wish to issue. They submit this CSR to FreeIPA using
the ``ipa`` ``cert-request`` command, and if all the specified fields
match the data FreeIPA has about the certificate subject, a cert will be
issued. This seems a bit silly; if FreeIPA has this information already,
can't it automatically generate a CSR with the correct data?

However, different certificate applications require different data, so
the Certificate Profile (a concept from the Dogtag CA that specifies the
fields in the cert, constraints on their values, and how the final
values should be constructed) needs to contain information on how to
transform the data in FreeIPA into the fields of the certificate.
Further, different administrators may want to use different tools to
manage their private keys, so we must be able to communicate these
certificate field values back in formats understood by different
utilities such as openssl and certutil. Those tools will be responsible
for generating the actual CSR from the provided configuration.

As suggested in the `Mapping Rules
design <http://www.freeipa.org/page/V4/Automatic_Certificate_Request_Generation/Mapping_Rules>`__,
the first implementation of this system used python to implement the
low-level formatting rules, such as *return the user's email address,
prefixed by the string 'email:'*. However, it is a goal of this project
to allow new rules to be added at runtime, so these rules must be
text-based rather than part of the code. This post will try to imagine
what the rules would look like if implemented using the
`Jinja2 <http://jinja2.pocoo.org/>`__ templating language.

Requirements
------------

We must at a minimum be able to generate two different types of
configuration, the openssl config file:

::

   [ req ]
   prompt = no
   encrypt_key = no

   distinguished_name = dn
   req_extensions = exts

   [ dn ]
   O=DOMAIN.EXAMPLE.COM
   CN=user

   [ exts ]
   subjectAltName=@SAN

   [ SAN ]
   email=user@example.com
   dirName=SANdn

   [ SANdn ]
   1.DC=com
   2.DC=example
   CN=users
   UID=user

and the certutil command line:

::

   certutil -R -a -s "CN=user,O=DOMAIN.EXAMPLE.COM" --extSAN "email:user@example.com,dn:UID=user;CN=users;DC=example;DC=com"

Some interesting things to note about these formats:

-  The contents of an extension can be constructed from multiple
   sources, such as an email address and a distinguished name.
-  The openssl format is hierarchical. Some parameters, such as
   ``req_extensions`` and ``dirName`` always refer to the name of a new
   config section. Others can optionally refer to a config section using
   an ``@``.
-  In openssl, the certificate subject is created under the ``[req]``
   section, while extensions are created under their own section.
-  Openssl has a quirky way of denoting distinguished names. They are
   ordered from least to most specific (opposite how LDAP presents
   them). And if two AVAs have the same attribute type, they must be
   prefixed with different strings ending in ``.`` (or ``:`` or ``,``)
   as the config file format will otherwise discard all but one.
-  Certutil is also a bit quirky about distinguished names in the
   Subject Alt Name extension. Because the argument to the extSAN flag
   is comma-delimited, the components of the DN must be separated using
   a different delimiter, like a semicolon.

Implementations
===============

.. _two_pass_data_interpolation:

Two-pass data interpolation
---------------------------

::

   ((user data -> data rules) -> syntax rules) -> output

One way we can approach constructing one extension from multiple sources
it to use two sets of rules - one rule for each data item that provides
a value for the extension, and one rule specifying the name and syntax
of the extension as a whole. We evaulate the data rules first, then feed
the values produced into the associated syntax rules to get the final
output for that extension. Finally, the extension output is passed to
the formatter, to produce the final output. We wish to express the data
and syntax rules using the templating language, but the formatters (one
for each CSR generation tool) will be implemented as python classes.

So how do we represent openssl sections in this scheme? The formatter
needs to accept input in a (very limited) markup language, which defines
where the sections are, what goes into them, and perhaps whether a given
line should be placed under ``[req]`` or ``[exts]``. Even with the
features of the formatter markup very limited, it would still be
possible for a user to accidentally or intentionally inject some markup
that would make it impossible to generate a certificate for them. So,
some kind of escaping is also needed, but it would be jinja2 template
markup escaping, not the HTML escaping that jinja2 already supports.

Example data rules:

::

   email={{subject.email}}

::

   O={{config.ipacertificatesubjectbase}}\nCN={{subject.username}}

Example syntax rules:

::

   --extSAN {{values|join(',')}}

::

   subjectAltName=@{{{% endraw %}{% raw %}'{% section %}'}}{{values|join('\n')}}{{{% endraw %}{% raw %}'{% endsection %}'}}

That's a lot of braces! We have to escape the ``section`` and
``endsection`` tags sequences so they will appear verbatim in the final
template, producing something like:

::

   subjectAltName=@{% section %}email={{subject.email}}
   URI={{subject.inetuserhttpurl}}{% endsection %}

If we used a different type of markup for the user data interpolation
and for denoting sections, the escaping would not be necessary; however,
we would still need to preprocess the ``values`` to escape any jinja2
markup that comes from the user data, and we would still have two types
of markup being used in parallel.

Note, too, that the ``section`` tag does not exist yet in jinja2; it
would need to be implemented as an extension.

.. _two_pass_template_interpolation:

Two-pass template interpolation
-------------------------------

::

   (user data -> (data rules -> syntax rules)) -> output

Alternatively, we can do the substitution on the templates themselves
before interpolating user data, building up one big template that we
then render with the data from the database. This is safer because the
user-specified data never gets interpreted as a template, so we don't
have to worry about escaping the user data or limiting the features of
the template language. On the other hand, this may be challenging for
the rule writer, because one must keep in mind the number of times a
given rule will be run through the templating engine to get the escaping
correct. Data rules will be used as templates only once (consuming user
data) but syntax rules will be used as templates once to incorporate the
data rules into the templates, and then again when the user data is
included. Thus, any template tags relating to the processing of user
data (such as, I imagine, ones for specifying openssl sections) need to
be escaped.

Surprisingly, this hardly changes the way the rules are written! All of
the example rules given above would still be valid, but the ``values``
would be the data rules themselves rather than data rules with
interpolated user data. And of course, the ``values`` would not be
escaped beforehand.

.. _template_based_hierarchical_rules:

Template-based hierarchical rules
---------------------------------

::

   user data -> collected rules -> output

One way to get away from escaping and multiple evaluations is to
redesign the template so that the order of its elements no longer
matters. That is, the hierarchical relationships between data items,
certificate extensions, and the CSR as a whole could be encoded using
jinja2 tags. It's probably easiest to explain this idea with an example:

::

   {% group req %}
   {% entry req %}extensions={% group exts %}{% endentry %}
   {% entry req %}distinguished_name={% group subjectDN %}{% endentry %}
   {% entry subjectDN %}O={{config.ipacertificatesubjectbase}}\nCN={{subject.username}}{% endentry %}
   {% entry exts %}subjectAltName=@{% group SAN %}{% endentry %}
   {% entry SAN %}email={{subject.email}}{% endentry %}
   {% entry SAN %}URI={{subject.inetuserhttpurl}}{% endentry %}

The config for certutil would be quite similar:

::

   certutil -R -a {% group opts %}
   {% entry opts %}-s {% group subjectDN %}{% endentry %}
   {% entry opts %}--extSAN {% group SAN %}{% endentry %}
   {% entry subjectDN %}CN={{subject.username}},O={{config.ipacertificatesubjectbase}}{% endentry %}
   {% entry SAN %}email:{{subject.email}}{% endentry %}
   {% entry SAN %}uri:{{subject.inetuserhttpurl}}{% endentry %}

Each CSR generation helper would have its own notion of "groups," which
would be implemented as jinja2 extensions. The entries of a group would
be collected and inserted into the group in whatever way was appropriate
for that helper. Each line of these templates would be either a cert
mapping rule referenced in the cert profile, or something built into the
formatter for the CSR generation helper. There would be no distinction
between data rules and syntax rules, and the order that rules appeared
in the template would be irrelevant because the tags specified the
hierarchy.

This approach has some downsides, too:

#. Section names are now specified in the rules, which means there could
   be conflicts between different rules, and that a rule can only ever
   be included in a particular section. If two sections need the same
   data, two different rules are needed.
#. Some types of groups are formatted differently from others (e.g. in
   certutil, ``opts`` is space-separated, while ``SAN`` is
   comma-separated. It's not entirely clear where this should be
   encoded, and how.

Concern #1 is probably an ok tradeoff, as it's not clear how broadly
reusable rules will be anyway. However, #2 would need to be addressed in
any actual implementation.

.. _formatter_based_hierarchical_rules:

Formatter-based hierarchical rules
----------------------------------

::

   user data -> low-level rule -> formatting code -> group objects
   group objects -> higher-level rule -> formatting code -> group objects
   ...
   group objects -> top-level rule -> output

Instead of linking rules together into a hierarchy using tags, leaving
it to the templating engine to interpret that structure, we could encode
the structure in the rule entities themselves and use multiple
evaluations to handle the hierarchy in the formatter, before the data
even gets to the templating engine. Each rule would be stored with the
name of the group within which it should be rendered, as well as the
names of any groups that the rule includes. For example, to adapt the
rule ``{% entry exts %}subjectAltName=@{% group SAN %}{% endentry %}``
to this schema, we would say that it is an element of the "exts" group,
and provides the "SAN" group. By linking up group elements to group
providers, we construct a tree of rules.

The formatter would evaluate these rules beginning at the leaves and
passing the results of child nodes into variables in the parent node
templates. The formatter is responsible for determining what exactly
gets passed into the parent node, such as an object representing an
openssl config section, or just a list of formatted strings. Parent
nodes decide how to present the passed objects, such as by
comma-separating the strings or referencing the name of the section.
This addresses concern #2 from the previous approach, because the tools
of the jinja2 language are now available for expressing how to format
the results of groups of rules.

Example leaf rules:

::

   group: SAN
   template: email={{subject.email}}

::

   group: subjectDN
   template: O={{config.ipacertificatesubjectbase}}\nCN={{subject.username}}

Example parent rules:

::

   group: opts
   groupProvided: SAN
   template: --extSAN {{ SAN|join(',') }}

::

   group: exts
   groupProvided: SAN
   template: subjectAltName=@{{ SAN.section_name }}

This has several advantages over the two-pass interpolation approaches:

#. Profiles are simpler to configure, because they just contain a list
   of references to rules rather than a structured list of groups of
   rules.
#. Profiles are also simpler to implement, with no sub-objects in the
   database.
#. It's no longer necessary to pay attention to escaping when writing
   rules. Each rule is used as a template exactly once, and complex
   structures are handled by the formatter code rather than template
   tags so tags don't need to be passed along.
#. User data is never used as a template, which reduces the attack
   surface.

However, there are also some potential concerns:

#. Whether the openssl and certutil hierarchies for rules are compatible
   (i.e. can the parent group can be listed in the mapping rule or must
   it be in the transformation rule?)
#. Are there any instances where something needs to be a group but can't
   be its own openssl section? How would we convey this to the openssl
   formatter?
#. Conversely, are there cases where we would want to be able to create
   a section without creating a new rule? For example, a DN in a subject
   alternative name needs to be its own section. Do we then need rules
   just for filling out parts of that DN?

Conclusions
===========

Although hierarchical rules seem like an interesting solution to avoid
escaping and simplify the configuration in the cert profile itself, I
think the interpolation approaches are easier to understand and explain,
which is valuable for this already unexpectedly-complex feature.

Even though it is a little counter-intuitive, I lean towards the
template interpolation solution rather than the straightforward data
interpolation one because it doesn't incorporate user data until the
last step. This would make it incompatible with the existing
python-based rules, but those are going to be replaced anyway, and in
fact they may be vulnerable to injection attacks as well. Escaping of
tags that are to be interpreted by the formatter will still be
inconvenient, but we may be able to provide extensions to the template
language to make that easier.
