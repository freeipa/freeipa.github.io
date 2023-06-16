Overview
========

Certificate Mapping Rule objects, as defined in
`V4/Automatic_Certificate_Request_Generation#Mapping_data_to_fields <V4/Automatic_Certificate_Request_Generation#Mapping_data_to_fields>`__,
need a mechanism for defining how to use the data stored in IPA objects
to construct the configuration strings that are passed to the CSR
generation helper. This document aims to elaborate on the features
required for this mapping, and describe the current implementation as
well as possible alternatives.

Requirements
============

There are two different types of mapping rules used in CSR data
formatting:

Data rules
   Describe how to format an individual data item
Syntax rules
   Describe how to combine data rules in order to create a complete
   certificate field

.. _data_rules:

Data rules
----------

-  Include data attribute values
-  Include literal values
-  Include values requested from the user via a prompt
-  For certutil: replace commas with a different separator character (;
   or +) within a DN-type subject alternative name
-  For certutil: shell quoting of parameters to prevent injection
   attacks
-  Suppress rendering of a rule if necessary data is missing

.. _syntax_rules:

Syntax rules
------------

-  Distinguish between generated config files and command-line arguments
-  For openssl: produce and reference new config file sections
-  For openssl: place config value in root or extensions section as
   appropriate
-  Combine several values for same field (e.g. comma-separated list)
-  Suppress rendering of a rule if none of the data rules it contains
   are going to be rendered

.. _planned_implementation:

Planned implementation
======================

We will use Jinja2 for templating. Data rules and syntax rules will both
be snippets of Jinja2 markup. The formatting process will be as follows:

#. Syntax rules will be rendered using Jinja2. Data rules (rule text,
   not rendered) will be passed as the ``datarules`` attribute.
#. Rendered syntax rules will be processed by the Formatter class for
   the selected CSR generation helper. The formatter combines these
   partial rules into a full template for the config.
#. The template will be rendered using Jinja2. Several objects from the
   IPA server (discussed below) will be available in the context for
   this rendering.
#. The final rendered template will be returned to the caller, labeled
   with its function (e.g. a command line or a config file)

.. _data_available_during_final_render:

Data available during final render
----------------------------------

``subject``
   dict of data from the IPA object corresponding to the subject of this
   certificate
``config``
   dict of IPA configuration data (result of the config_show API call)

.. _special_features:

Special features
----------------

Some less-than-obvious features will be required to support generating
the needed configs. Some of them will require extensions to the Jinja2
language, others just require specific usage of the tools that Jinja2
already provides.

``{% section %}{% endsection %}``
   This tag will be introduced to make the included block into a new
   section, as for an openssl config file.
Easy escaping
   Some tags, like ``{% section %}``, are only meaningful during the
   second (full-template) render. During the first render they will be
   implemented to just insert the same tag in the output.
Location specifier
   Rules must be able to provide guidance to the formatter about where
   they belong in the config file (for example, whether a rule is an
   extension or goes in the base ``[req]`` section). This is
   accomplished by setting a template variable in the syntax rule, for
   example ``{% set location = "req" %}``

Example
-------

Example data rules:

| ``email={{subject.email}}``
| ``O={{config.ipacertificatesubjectbase}}\nCN={{subject.username}}``

Example syntax rule:

``subjectAltName=@{% section %}{{datarules|join('\n')}}{% endsection %}``

Example composed config template:

| ``[ req ]``
| ``prompt = no``
| ``encrypt_key = no``
| ````
| ``distinguished_name = {% section %}O={{config.ipacertificatesubjectbase}}``
| ``CN={{subject.username}}{% endsection %}``
| ````
| ``req_extensions = exts``
| ````
| ``[ exts ]``
| ``subjectAltName=@{% section %}email={{subject.email}}{% endsection %}``

.. _current_status:

Current status
==============

The code `currently under
review <https://www.redhat.com/archives/freeipa-devel/2016-July/msg00462.html>`__
implements most of the features described above, with some exceptions:

.. _macros_not_tags:

Macros, not tags
----------------

It was convenient to use jinja2 macros to implement some of the
`#Special features <#Special_features>`__ instead of adding new tags
which requires extending the parser. So, instead of
``{% section %}{% endsection %}``, one has to use
``{% call section() %}{% endcall %}`` to enclose the contents of an
openssl config section. Similarly, the features that allow suppressing
rules with no data are implemented using jinja2 macros. If the macros
turn out to be problematic, we may want to move to using tags after all.

.. _rule_suppression:

Rule suppression
----------------

It is important that the final output does not contain
partially-constructed strings, reference empty sections, or provide
command-line flags missing their arguments. So, we must be able to
prevent rendering an entire rule, when some parts of it can not be
rendered due to missing data. That is currently implemented using three
macros: syntaxrule, datarule, and datafield.

-  syntaxrule wraps the contents of a syntax rule and renders it only
   when at least one of the included datarules will be rendered.
-  datarule wraps the contents of a data rule and renders it only when
   all of the included datafields contain data. It informs the enclosing
   syntaxrule whether the data rule will be rendered.
-  datafield wraps an individual data item. If the wrapped value is
   empty, the enclosing datarule will not be rendered.

The syntaxrule and datarule macros are applied automatically by the
framework code, so users should only be concerned about datafield. All
data items in the data rules must be marked as such using this macro,
for example:

``email:{{ipa.datafield(subject.mail.0)|quote}}``

.. _alternatives_considered:

Alternatives considered
=======================

.. _template_languages:

Template languages
------------------

Several possible tools were considered and tested for implementing these
relationships before settling on jinja2:

-  For prototyping only: built-in python code. This could take advantage
   of the ability to query the API from within FreeIPA. However, it
   would be unsafe to allow administrators to add new mappings that run
   arbitrary code, so this does not satisfy the goal of giving
   administrators the ability to define their own mappings.
-  Jinja2. Nice because it doesn't reinvent the wheel by defining a new
   syntax, but would add a dependency to FreeIPA. Would also probably
   need to use its sandboxing features to prevent becoming a vector for
   arbitrary code execution.
-  Custom syntax, perhaps similar to `NIS format
   specifiers <https://git.fedorahosted.org/cgit/slapi-nis.git/plain/doc/format-specifiers.txt>`__.
   Would only need to define syntax for what we need, but a whole
   language would need to be defined and implemented.

.. _data_interpolation:

Data interpolation
------------------

`This blog
post <http://blog.benjaminlipton.com/2016/07/19/csr-generation-templating.html>`__
(`archived
here <V4/Automatic_Certificate_Request_Generation/Thinking_About_Templating_Post>`__)
analyzes some alternative ways of using jinja2 to format rules, besides
the current option of substituting data rules into syntax rules.

.. _rule_suppression_1:

Rule suppression
----------------

In retrospect, the definitions of the syntaxrule, datarule, and
datafield macros are difficult to understand and modify, in part because
it is difficult to avoid introducing unintended whitespace into the
produced output. As a result of this, the code has a couple of minor
bugs where sections are produced that should not be, which are difficult
to fix because of the brittleness of the macros. We may want to consider
an alternative implementation, such as:

-  It may be possible to write a more flexible implementation as a
   jinja2 tag rather than macros. Although not necessarily easier to
   understand, a parser extension might be able to handle the
   relationship between syntax rules, data rules, data fields, and
   openssl sections better because it has access to the internal AST of
   the template.
-  Instead of simply rendering things to see if they produce output, we
   could explicitly tag data rules with the data items they depend on.
   Then we could automatically insert ``{% if %}`` statements into the
   code to suppress things when those data items are unavailable. Of
   course, if the tagged data items were incorrect the suppression would
   not work correctly.
-  Finally, if we were interpolating user data during each render rather
   than just the final one (`this
   solution <http://blog.benjaminlipton.com/2016/07/19/csr-generation-templating.html#two-pass-data-interpolation>`__
   for example) we could easily drop any rules that didn't produce
   output. However, then we would be at much higher risk of template
   injection attacks.
