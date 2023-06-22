Migrate_your_code_to_the_new_LDAP_API
=====================================

A new LDAP API has been introduced to the framework in FreeIPA 3.2 (see
`V3/LDAP code <V3/LDAP_code>`__). Compatibility with the old legacy API
has been kept in place but is currently in the process of being
deprecated. Any attempt to use the old API will cause an exception to be
raised in FreeIPA 4.0. The old API will be completely removed in FreeIPA
4.1.

The new API uses custom dictionary-like objects to represent entries
instead of ``(dn, entry_attrs)`` tuples.

Adding an entry:

| ``# Before``
| ``entry_attrs = dict(...)``
| ``ldap.add_entry(DN(...), entry_attrs)``
| ``# After``
| ``entry_attrs = dict(...)``
| ``entry = ldap.make_entry(DN(...), entry_attrs)``
| ``ldap.add_entry(entry)``

Modifying an entry:

| ``# Before``
| ``dn, entry_attrs = ldap.get_entry(...)``
| ``entry_attrs[...] = ...``
| ``ldap.update_entry(dn, entry_attrs)``
| ``# After``
| ``entry = ldap.get_entry(...)``
| ``entry[...] = ...``
| ``ldap.update_entry(entry)``

Searching entries:

| ``# Before``
| ``result, truncated = ldap.find_entries(...)``
| ``for dn, entry_attrs in result:``
| ``    print dn``
| ``# After``
| ``result, truncated = ldap.find_entries(...)``
| ``for entry in result:``
| ``    print entry.dn``

Details
-------

In FreeIPA 3.3 and below, iteration over an entry yields dn and the
entry itself. This preserves backwards compatibility with the
python-ldap style:

``dn, entry_attrs = ldap.get_entry(...)``

In FreeIPA 4.0, iterating over an entry will raise an exception, so this
use can be detected and fixed.

In FreeIPA 4.1, entries complete the move to a dict-like interface.
Iteration will yield attribute names.