Schema_Compatibility_Plug-in_Design
===================================



Design Overview
===============

The Schema Compatibility plugin's aim is to synthesize a modified view
of the contents of one part of the directory and to make that view
visible in another part of the directory. It does this by precomputing
the contents of its entries at startup-time and refreshing its entries
as needed.



Map Cache
---------

The map cache keeps a dynamically-constructed group of sets of entries
in memory, grouped by the name of their top-level container, and for
each set of entries maintains a copy of the set's releative
distinguished name. The map cache can be used to quickly answer whether
or not a particular search request crosses into the portion of the DIT
which is "served" by the data in the cache.



Internal Representation
----------------------------------------------------------------------------------------------

At the topmost level, the map cache is a table. Each entry in the table
contains the name of a top-level container entry and a table of sets.

Each set in a container's table of sets contains the relative
distinguished name (RDN) of the set, a linked list of map entries, and a
set of indexes into the list. Each set can also hold a data pointer on
behalf of the backend.

Each item in a set's list of entries contains the entry's relative
distinguished name (relative to the set, which is itself relative to the
group-of-sets container), the entry's whole distinguished name, a unique
identifier (which, currently, stores the NDN of the directory server
entry which was used to create this entry) and a data pointer which is
kept on behalf of the backend (which, currently, stores a Slapi_DN for
the directory server entry which was used to create this entry and the
full Slapi_Entry for the generated entry).

The set indexes its entry list using an entry's unique identifier and
its relative distinguished name.



Back End
--------

The backend interface module sets up, populates, and maintains the map
cache. At startup time, it configures the map cache with the list of
groups of sets of entries, and populates the sets with initial data.
Using postoperation plugin hooks, the backend interface also notes when
entries are added, modified, renamed (modrdn'd), or deleted from the
directory server. It uses this information to create or destroy sets in
the map cache, and to add, remove, or update entries in the map cache's
sets, thereby ensuring that the synthetic data in the map cache is
always up to date with respect to the current contents of the directory
server.

The backend interface reads the configuration it should use for the map
cache from its configuration area in the directory server. Beneath the
plugin's entry, the backend checks for entries with these attributes:

-  schema-compat-search-base
-  schema-compat-search-filter
-  schema-compat-container-group
-  schema-compat-container-rdn
-  schema-compat-entry-rdn
-  schema-compat-entry-attribute
-  schema-compat-check-access

The backend then instructs the map cache to prepare to hold a set of
entries in the given container group (or container groups) with the
given subcontainer RDN name (or names), and then performs a subtree
search under the specified base (or bases, if there's more than one
"schema-compat-search-base" value) for entries which match the provided
filter ("schema-compat-search-filter").

For entry found, a new entry is generated and "added" to the
subcontainer, using the format specifier stored in the
"schema-compat-entry-rdn" and "schema-compat-entry-attribute" attributes
to construct the RDN and attribute values for the entry in the set.

Should one of the directory server entries which was used to construct
one or more entries be modified or removed, the corresponding entries in
every applicable container are updated or removed. Likewise, if an entry
is added to the directory server which would correspond to an entry in a
container, entries are created in the corresponding container.



Specifying Entry Contents
-------------------------

The "schema-compat-entry-rdn" specifier resembles an RPM format
specifier, and can include the values of multiple attributes in any part
of the specifier. The backend composes the string using the attribute
values stored in the directory server entry, using the format specifier
as a guide, and names the resulting entry using the subcontainer's name
and the generated RDN. Attributes specified using values of the
"schema-compat-entry-attribute" attribute are then added. If the
resulting entry fails schema checks, it is automatically given the
"extensibleObject" object class.

An example specification for the "schema-compat-entry-rdn" for a user's
entry could look something like this:

`` uid=%{uid}``

The syntax borrows from RPM's syntax, which in turn borrows from shell
syntax, to allow the specification of alternate values to be used when
the directory server entry doesn't include a "uid" attribute. Additional
operators include "#", "##", "%", "%%", "/", "//", which operate in ways
similar to their shell counterparts (with one notable exception being
that patterns for the "/" operator can not currently be anchored to the
beginning or end of the string).

After the RDN is determined, attributes can be added. Most often, this
will be done either by specifying a specific value (typically for the
entry's object classes) or an existing attribute, like so:

| `` objectclass=posixAccount``
| `` cn=%{cn}``

A format specifier can actually be interpreted in two ways: it can be
interpreted as a single value, or it can be interpreted as providing a
list of values. When the format specifier is being interpreted as a
single value, any reference to an attribute value which does not also
specify an alternate value will cause the directory server entry to be
ignored if the referenced attribute has no value defined for that entry,
or contains multiple values. In the above example, the RDN is evaluated
in such a manner.

For all other attributes, if an attribute is multi-valued, the resulting
entry will have a corresponding number of values.

A pair of built-in "function"s are available for importing values from
other entries.

A function-like invocation expects a comma-separated list of
double-quoted arguments. Any arguments which contain a double-quote need
to escape the double-quote using a '\' character. Naturally the '\' this
character itself also needs to be escaped whenever it appears.



Implemented Functions
----------------------------------------------------------------------------------------------

first(EXPRESSION[,DEFAULT])
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Evaluates EXPRESSION, and if one or more values is available, provides
the first value. If no values result, then DEFAULT is evaluated as an
expression and the result is provided.

match(EXPRESSION,PATTERN[,DEFAULT])
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Selects the value of EXPRESSION which matches the globbing pattern
PATTERN. If no values match, and a DEFAULT was specified, the DEFAULT is
produced, otherwise an error occurs.

| ``dn: cn=group``
| ``member: bob``
| ``member: dave``

============================== =====================================
%match("%{member}","b*")       bob
%match("%{member}","d*")       dave
%match("%{member}","e*")       FAILS
%match("%{member}","e*","jim") jim
%match("%{member}","*","jim")  jim (when a single value is required)
%match("%{member}","*","jim")  bob,dave (when a list is acceptable)
============================== =====================================

regmatch(EXPRESSION,PATTERN[,DEFAULT])
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Selects the value of EXPRESSION which matches the extended regular
expression PATTERN. If no values match, and a DEFAULT was specified, the
DEFAULT is produced.

| ``dn: cn=group``
| ``member: bob``
| ``member: dave``

+-------------------------------------+---------------------------------------+
| %regmatch("%{member}","^b.*")       | bob                                   |
+-------------------------------------+---------------------------------------+
| %regmatch("%{member}","^d.*")       | dave                                  |
+-------------------------------------+---------------------------------------+
| %regmatch("%{member}","e")          | dave                                  |
+-------------------------------------+---------------------------------------+
| %regmatch("%{member}","^e")         | FAILS                                 |
+-------------------------------------+---------------------------------------+
| %regmatch("%{member}","^e.*","jim") | jim                                   |
+-------------------------------------+---------------------------------------+
| %regmatch("%{member}",".*","jim")   | jim (when a single value is required) |
+-------------------------------------+---------------------------------------+
| %regmatch("%{member}",".*","jim")   | bob,dave (when a list is acceptable   |
+-------------------------------------+---------------------------------------+

regsub(EXPRESSION,PATTERN,TEMPLATE[,DEFAULT])
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Selects the value of EXPRESSION which matches the extended regular
expression PATTERN and uses TEMPLATE to construct the output. If no
values match, and a DEFAULT was specified, the DEFAULT is produced,
otherwise an error occurs. The template is used to construct a result
using the n'th substring from the matched value by using the sequence
"%n" in the template.

| ``dn: cn=group``
| ``member: bob``
| ``member: dave``

====================================== =====
%regsub("%{member}","o","%0")          bob
%regsub("%{member}","o","%1")          
%regsub("%{member}","^o","%0")         FAILS
%regsub("%{member}","^d(.).*","%1")    a
%regsub("%{member}","^(.*)e","t%1y")   tdavy
%regsub("%{member}","^e","%1")         FAILS
%regsub("%{member}","^e.*","%1","jim") jim
====================================== =====

deref(THISATTRIBUTE,THATATTRIBUTE)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Creates a separated list of the values of THATATTRIBUTE for directory
entries named by this entry's THISATTRIBUTE.

| ``dn: cn=group``
| ``member: uid=bob``
| ``member: uid=pete``
| ``-``
| ``dn: uid=bob``
| ``uid: bob``
| ``-``
| ``dn: uid=pete``
| ``uid: pete``

====================== ======================================
%deref("member","foo") FAIL (when a single value is required)
%deref("member","foo") (when a list is acceptable)
%deref("member","uid") FAIL (when a single value is required)
%deref("member","uid") bob,pete (when a list is acceptable)
====================== ======================================

referred(SET,THATATTRIBUTE,THATOTHERATTRIBUTE)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Creates a separated list of the values of THATOTHERATTRIBUTE for
directory entries which have entries in the current group in the named
SET and which also have this entry's name as a value for THATATTRIBUTE.
Will fail if used in a context where a single value is required.

| ``dn: cn=group``
| ``-``
| ``dn: uid=bob``
| ``uid: bob``
| ``memberOf: cn=group``
| ``-``
| ``dn: uid=pete``
| ``uid: pete``
| ``memberOf: cn=group``

+----------------------------------+----------------------------------+
| %refer                           | FAIL (when a single value is     |
| red("cn=users","memberof","foo") | required)                        |
+----------------------------------+----------------------------------+
| %refer                           | (when a list is acceptable)      |
| red("cn=users","memberof","foo") |                                  |
+----------------------------------+----------------------------------+
| %refer                           | FAIL (when a single value is     |
| red("cn=users","memberof","uid") | required)                        |
+----------------------------------+----------------------------------+
| %refer                           | bob,pete (when a list is         |
| red("cn=users","memberof","uid") | acceptable)                      |
+----------------------------------+----------------------------------+

merge(SEPARATOR,EXPRESSION[,...])
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Evaluates and then creates a list using multiple expressions which can
evaluate to either single values or lists.

| ``dn: cn=group``
| ``membername: jim``
| ``member: uid=bob``
| ``member: uid=pete``
| ``-``
| ``dn: uid=bob``
| ``uid: bob``
| ``-``
| ``dn: uid=pete``
| ``uid: pete``

======================================================== ============
%merge(",","%{membername}","%deref(\"member\",\"uid\")") jim,bob,pete
======================================================== ============

| 