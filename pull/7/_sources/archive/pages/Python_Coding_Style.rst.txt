This page outlines standards for `Python <http://python.org/>`__ code
contributed to freeIPA v2 and beyond.

.. _python_version:

Python version
==============

In the FreeIPA master branch and branches >= 3.0.0, we require at least
Python 2.7. This permits the use of set literals, dict and set
comprehensions, and `many other
features <https://docs.python.org/dev/whatsnew/2.7.html>`__.

.. _python_coding_style:

Python coding style
===================

Although Python's syntax enforces quite a lot of stylistic consistency,
there is still much wiggle room, so a style guide will help keep us
productive and happy.

Fortunately, thanks to the "Benevolent Dictator For Life" (Guido van
Rossum, the author of Python), there is only one coding style for
Python, which is defined in PEP-8 and PEP-257:

-  `PEP 8: Style Guide for Python
   Code <http://www.python.org/dev/peps/pep-0008>`__
-  `PEP 257: Docstring
   Conventions <http://www.python.org/dev/peps/pep-0257>`__

.. _python_unit_testing:

Python unit testing
===================

All the Python unit testing is being done using *nose*:

-  `nose: a discovery-based unittest
   extension <http://somethingaboutorange.com/mrl/projects/nose/>`__

nose was chosen because it allows unit tests to be written very quickly
and in a quite natural and flexible manner.

All the unit tests should be located in the ``tests/`` directory in the
source tree.

.. _python_api_documentation:

Python API documentation
========================

We are generating API documentation using the *Epydoc* tool:

-  `Epydoc: Automatic API Documentation Generation for
   Python <http://epydoc.sourceforge.net>`__

The Epydoc generated documentation is an excellent way to learn the v2
code base, particularly because all the cross-references are
hyper-linked. Up-to-date documentation is always available on
freeipa.org:

-  `freeIPA Python API
   documentation <http://freeipa.org/developer-docs/>`__
-  `Tutorial for Plugin
   Authors <http://freeipa.org/developer-docs/ipalib-module.html>`__

.. _python_internationalized_i18n_strings:

Python Internationalized (i18n) Strings
=======================================

If the string will be internationalized (e.g. is marked with \_()) and
it has more than one format substitution you **MUST** use *named* format
specifiers, not positional format specifiers.

Here is an example of incorrect usage with positional substitution:

`` _('item %s has %s value') % (name, value)``

Here is the correct usage using named substitution:

`` _('item %(name)s has %(value)s value') % {'name':name, 'value':value}``

-or-

`` _('item %(name)s has %(value)s value') % dict(name=name, value=value}``

Why does this matter? Word ordering is locale dependent. Translators
need the flexibility to reorder the words in the string. If you use
positional format substitutions the translator can't reorder the
wording. However, if you use named format substitutions the translator
has the freedom to reorder the wording. Try to pick names for the for
format specifiers which will provide hints to the translator as to
meaning of the substitution.

.. _python_docstrings:

Python docstrings
=================

We are using *reStructuredText* markup in the docstrings, which you can
read about here:

-  `reStructuredText: Markup Syntax and Parser Component of
   Docutils <http://docutils.sourceforge.net/rst.html>`__

There is considerable example code included in the docstrings. All these
examples are automatically tested using the Python doctest module:

-  `doctest: Test interactive Python
   examples <http://docs.python.org/library/doctest.html>`__

We are relying on nose to automatically discover all the code examples
and then to test them with doctest. This is done using the
``--with-doctest`` option, like this:

``nosetests --with-doctest``

.. _unused_dummy_variables:

Unused (dummy) variables
========================

We want to avoid unused variables in code as much as possible. In case
you need to use a variable that has no further use (iteration, tuple
unpacking) then this variable must start with an underscore character
('_'). The Pylint check for unused variables has been enabled and such
dummy variables must match the regular expression '_.+' otherwise they
are reported as errors.

Example:

| ``name, _weight, age = human['John']``
| ``print(name, age)``
