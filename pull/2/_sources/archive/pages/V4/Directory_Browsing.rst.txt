Overview
========

The IPA Web UI needs to provide a way to browse the entire content of
IPA directory (see `ticket
#981 <https://fedorahosted.org/freeipa/ticket/981>`__). The current
mechanism has some issues:

-  The UI doesn't show the entire directory. Due to DS restrictions on
   plain LDAP search, currently the UI will only show the first 100
   entries. Simply increasing or removing the limit would be impractical
   because the UI may have to handle a large amount of data and also it
   might degrade server performance.
-  The UI doesn't provide a way to sort the entries. The entries needs
   to be sorted based on the entire directory, not based on the entries
   currently displayed in the UI.

There are several options to address these issues:

#. Using Simple Paged Results to retrieve entries
#. Using Simple Paged Results to retrieve primary keys
#. Virtual List View
#. Hybrid solution

.. _option_1_using_simple_paged_results_to_retrieve_entries:

Option #1: Using Simple Paged Results to retrieve entries
=========================================================

Simple Paged Results allows the UI to retrieve all entries in multiple
pages. It also supports server-side sorting. The problem is that the
pages are returned sequentially, so the UI will only able to provide a
'Next' button. It will not be able to go to a previous or specific page
without re-searching from the beginning. This option also requires the
server to maintain an open connection to the DS while the entries are
being retrieved (see `ticket
#215 <https://fedorahosted.org/freeipa/ticket/215>`__).

This is how it will work:

#. The admin opens the first page of the Users list page in the UI.
#. The UI issues ipa user-find --offset=0 --count=100
   --orderby=uid,cn,email.
#. IPA server creates a session and uses simple paged results to
   retrieve the first 100 from the DS.
#. IPA server returns the results to the UI but it keeps the DS
   connection open.
#. The UI shows the results of the first page.
#. The admin clicks Next.
#. The UI issues ipa user-find --offset=100 --count=100.
#. IPA server users the same connection to retrieve the next 100 users.
#. The UI shows the results of the second page.

.. _option_2_using_simple_paged_results_to_retrieve_primary_keys:

Option #2: Using Simple Paged Results to retrieve primary keys
==============================================================

The UI could also use Simple Paged Results to retrieve all primary keys
(see `ticket #1262 <https://fedorahosted.org/freeipa/ticket/1262>`__).
Since it has the full list, the UI can go to any page directly. The UI
can then retrieve the additional data (e.g. givenName, sn) for the
entries that are going to be displayed in the current page. This option
does not require server session.

This is how it will work:

#. The admin opens the first page of the Users list page in the UI.
#. The UI issues ipa user-find --primary-keys --orderby=uid,cn,email.
#. The server uses simple paged results to retrieve all primary keys
   from the DS.
#. The UI constructs a batch command to retrieve the attributes of the
   first 100 users. The batch command consists of ipa user-show
   operations.
#. The server executes the batch command and return the results to the
   UI.
#. The UI shows the results of the first page.
#. The admin opens a different page (it doesn't have to be sequential).
#. The UI calculates the offset, then constructs a batch command to
   retrieve the attributes of the users in that page.
#. The server executes the batch command and return the results to the
   UI.
#. The UI shows the results of that page.

A complete list of primary keys will be smaller than a complete list of
entries with the attributes. However, for a large directory it might
still be a problem.

.. _option_3_virtual_list_view_vlv:

Option #3: Virtual List View (VLV)
==================================

VLV allows IPA server to retrieve any page of the search result
directly. The UI will only need to send the offset and the size for the
current page. The problem is that the VLV index has to be created ahead
of time and it has a fixed base, scope, filter, and sort order. Only
search requests that match this configuration will benefit from VLV.
This option does not require server session.

This is how it will work:

#. The admin opens the first page of the Users list page in the UI.
#. The UI issues ipa user-find --offset=0 --count=100
   --orderby=uid,cn,email.
#. IPA server uses VLV to retrieve the users from the DS based on the
   requested offset and count.
#. IPA server returns the results to the UI.
#. The UI shows the results of the first page.
#. The admin opens a different page (it doesn't have to be sequential).
#. The UI calculates the offset and issues ipa user-find command.
#. IPA server uses VLV to retrieve the users from the DS based on the
   requested offset and count.
#. IPA server returns the results to the UI.
#. The UI shows the results of that page.

.. _option_4_hybrid_solution:

Option #4: Hybrid solution
==========================

The DS can prepare a few standard VLV indexes, for example "Users sorted
by UID", "Users sorted by email". The UI can use them to handle the most
common use cases: browsing without filter and sorted by one attribute.

For less common use cases, the UI can use one of the earlier solutions
using Simple Paged Results.

When the admin opens the list page, the UI determines which type of
operation it's going to execute.

.. _configuring_vlv:

Configuring VLV
===============

Configure DS:

::

   ldapadd -x -D "cn=Directory Manager" -w Secret123 << EOF
   dn: cn=Users cn=users cn=accounts dc=example dc=com,cn=userRoot,cn=ldbm database,cn=plugins,cn=config
   objectClass: top
   objectClass: vlvSearch
   cn: Users cn=users cn=accounts dc=example dc=com
   vlvBase: cn=users,cn=accounts,dc=example,dc=com
   vlvScope: 1
   vlvFilter: (objectclass=*)

   dn: cn=by UID cn=users cn=accounts dc=example dc=com,cn=Users cn=users cn=accounts dc=example dc=com,
     cn=userRoot,cn=ldbm database,cn=plugins,cn=config
   objectClass: top
   objectClass: vlvIndex
   cn: by UID cn=users cn=accounts dc=example dc=com
   vlvSort: uid givenName sn
   EOF

Stop DS:

::

   service dirsrv stop EXAMPLE-COM

Generate indexes:

::

   /var/lib/dirsrv/scripts-EXAMPLE-COM/vlvindex -n userRoot \
   -T "by UID cn=users cn=accounts dc=example dc=com"

Start DS:

::

   service dirsrv start EXAMPLE-COM

.. _using_vlv:

Using VLV
=========

::

   ldapsearch -x -D "cn=Directory Manager" -w Secret123 -b "cn=users,cn=accounts,dc=example,dc=com" -s one \
   -E \!vlv=0/100/1/0 -E \!sss=uid/givenName/sn \
   "(objectclass=*)" dn givenName sn

.. _using_simple_paged_results:

Using Simple Paged Results
==========================

::

   ldapsearch -x -D "cn=Directory Manager" -w Secret123 -b "cn=users,cn=accounts,dc=example,dc=com" -s one \
   -E \!pr=100 -E \!sss=uid/givenName/sn \
   "(objectclass=*)" dn givenName sn

References
==========

-  `Simple Paged
   Results <http://directory.fedoraproject.org/wiki/Simple_Paged_Results_Design>`__
-  `Creating Browsing (VLV)
   Indexes <http://docs.redhat.com/docs/en-US/Red_Hat_Directory_Server/8.2/html/Administration_Guide/Creating_Indexes-Creating_VLV_Indexes.html>`__
-  `LDAP Extensions for Scrolling View Browsing of Search
   Results <http://tools.ietf.org/html/draft-ietf-ldapext-ldapv3-vlv-09>`__
