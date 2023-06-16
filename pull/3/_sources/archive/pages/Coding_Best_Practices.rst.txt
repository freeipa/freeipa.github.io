Long-term ongoing `refactoring <V4/Refactorings>`__ means that some
practices you see in FreeIPA's existing code are considered obsolete.
They should not be used in new code, and old code should be updated when
they are encountered.

Here is a list of these, with examples of the old and new way of doing
things.

Current
=======

.. _decorator_based_plugin_registration:

Decorator-based plugin registration
-----------------------------------

To allow registering plugins to non-default API objects, plugin
registration changed from:

| ``from ipalib import api``
| ``...``
| ``class myobj_mod(LDAPUpdate): ...``
| ``api.register(myobj_mod)``

to:

| ``from ipalib.plugable import Registry``
| ``register = Registry()``
| ``...``
| ``@register()``
| ``class myobj_mod(LDAPUpdate): ...``

Note that it's necessary to name the decorator ``register``.

.. _split_long_translatable_strings:

Split long translatable strings
-------------------------------

Long translatable strings, such as plugin modules' ``__doc__``, should
be split up by paragraph, so that if something is changed/added, only
the affected part needs to be re-translated. Example:

| ``__doc__ = _("""``
| ``Foo Plugin``
| ``""") + _("""``
| ``This plugin allows management of enterprise foos and bars.``
| ``""") + _("""``
| ``Here is a second paragraph.``
| ``"""")``

Note that the strings should *only* be split when they are changed,
because after a split all parts have to be re-translated.

.. _do_not_use_star_imports:

Do not use star imports
-----------------------

Star imports make it more difficult to see where names come from, and
names newly added to the imported module can unintentionally shadow
variables elsewhere.

Instead of

| ``from baseldap import *``
| ``class tank_create(LDAPCreate):``

use

| ``from ipalib.plugins import baseldap``
| ``class tank_create(baseldap.LDAPCreate):``

or

| ``from ipalib.plugins.baseldap import LDAPCreate``
| ``class tank_create(LDAPCreate):``

See https://fedorahosted.org/freeipa/ticket/2653, and PEP8

.. _use_the_admintool_framework_for_installationmanagement_scripts:

Use the admintool framework for installation/management scripts
---------------------------------------------------------------

There is a framework that standardizes common tasks such as argument
parsing, logging, error handling, interactive prompting. Use it.

For a simple example, see the implementation of
``ipa-server-certinstall``.

Related design concerning the CLI changes: `V3/Logging and
output <V3/Logging_and_output>`__

Finished
========

For these items, all code in IPA was updated and, if applicable, the old
way of doing things was disabled.

.. _ldapentry_dict_like_api:

LDAPEntry dict-like API
-----------------------

See
`HowTo/Migrate_your_code_to_the_new_LDAP_API <HowTo/Migrate_your_code_to_the_new_LDAP_API>`__

A LDAPEntry no longer unpacks to a tuple:

| ``# Old code``
| ``dn, attrs = entry``
| ``values = attrs[attrname]``

Instead, the entry is a dict-like object:

| ``# New code``
| ``dn = entry.dn``
| ``values = entry[attrname]``

To highlight errors, a LDAPEntry currently cannot be iterated.
