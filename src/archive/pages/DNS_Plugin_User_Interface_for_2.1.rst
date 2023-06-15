Summary
-------

DNS plugin User Interface should provide an easy-to-use interface to
manage DNS zones in the IPA domain. The UI must cope with large zones
with thousands of DNS records where each record may contain several
record types (A, AAAA, ...) at the same time.

.. _current_state:

Current State
-------------

User Interface in 2.0 is logically divided to 3 views:

.. figure:: Ipa-dnszone-find.png
   :alt: ipa-dnszone-find.png

   ipa-dnszone-find.png

The first one shows DNS zones managed by IPA. Administrator may choose
the zone to view/edit.

.. figure:: Ipa-dnszone-show.png
   :alt: ipa-dnszone-show.png

   ipa-dnszone-show.png

The chozen zone can be modified and *DNS Resource Records* can be
managed via the link on the left.

.. figure:: Ipa-dnsrecord-find.png
   :alt: ipa-dnsrecord-find.png

   ipa-dnsrecord-find.png

This is the interface where the most of the actual DNS related work
happens. All DNS records are listed there and Administrator can
add/remove records for all supported record types.

Unfortunately, there are several issues connected with this page:

-  Record **foo** in the example on the third screen shot has actually 3
   *A* address, but only one is shown in the UI
-  Record **foo** has also *AAAA* address, but it's not shown at all

See the underlying LDAP object for this resource record:

| `` dn: idnsname=foo,idnsname=example.com,cn=dns,$SUFFIX``
| `` idnsname: foo``
| `` arecord: 10.0.0.1``
| `` arecord: 10.0.0.2``
| `` arecord: 10.0.0.3``
| `` aaaarecord: fc00::1``
| `` objectclass: top``
| `` objectclass: idnsrecord``

This LDAP object structure should be reflected in the DNS UI in 2.1
better.

.. _new_ui_proposal:

New UI Proposal
---------------

Highlights:

-  DNS zone modification screen may remain without many changes
-  DNS record management must support multiple values for one record
   type (e.g. more A values)
-  DNS record management must support multiple records types for one
   record (e.g. support A, AAAA, LOC... for one record)
-  UI shall have an ability to safely handle a large zone file -
   paginated lists in the future, search support
-  UI shall have an ability to link a record to a Host (if managed by
   IPA) - this is important for records with associated A and AAAA
   record types

.. _dns_resource_records_view:

DNS Resource Records View
~~~~~~~~~~~~~~~~~~~~~~~~~

The main control element should be a table will all records for the
chosen DNS zone, **one line per each DNS resource record**. The actual
editing should happen in a new window with **all record types** for
chosen record.

The record table should be sorted by the name of the record,. Every name
should be a link to the **Resource Record Details** view. Administrator
may view/edit/add more values in this view.

To provide some useful information to the Administrator, the **most
common** record type values for each record could be added to the table,
namely A, AAAA and CNAME along with a sign if there is more than one
value for given record types. The Administrator gets all record type
values for the record in Resource Record Details view.

.. table:: Hypothetical DNS resource record table for example.com

   =========== ============ ======= ================ ========== =======
   Record Name A            AAAA    CNAME            More Types Host
   =========== ============ ======= ================ ========== =======
   @                                                 [+]        
   foo         10.0.0.1 [+] fc00::1                             host[+]
   foo2        10.0.0.2                              [+]        host
   foo3                     fc00::2                             host
   foo-cname                        foo.example.com.            
   ns          10.16.78.57                                      host
   \                                                            
   =========== ============ ======= ================ ========== =======

The above table shows a basic concept for the DNS resource record table
for the hypothetical *example.com* zone based on the above description.

Blue text indicates a link to DNS Resource Record Details page with all
associated record types and its values.

Green text indicates a link to host associated with a given record. This
is applicable for records with A/AAAA record types present. [+] mark is
shown when there are multiple Hosts associated with this record. For
example when there are multiple 3 *A* values, up to 3 Hosts may be
associated with the record.

When there are more values for a record type (e.g. multiple *A* record
types for record *foo1*), a [+] mark is shown next to the given record
type.

When an uncommon record type (e.g. *LOC*) record is associated with the
record, a [+] mark is placed to column **More Types**.

.. _dns_resource_records_details_view:

DNS Resource Records Details View
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This window should provide a simple way to actually view/edit/add new
record types for given DNS record. Each record type should should be in
an own section named after the record type.

.. _more_ideas:

More Ideas
~~~~~~~~~~

-  Order DNS resource record table by **data**, i.e. values of *A*,
   *AAAA* or *CNAME*\ s record types of the zone records. The list would
   have to be limited as there can be many record values in the zone.
-  Allow **smart table** which would allow Administrator to sort by data
   in chosen table column - Name (default), *A*, *AAAA* or *CNAME*

References
----------

Currently there exists several open DNS systems with web interfaces to
inspire this effort.

-  `Webmin <http://webmin.com/demo.html>`__: web interface module for
   Bind
-  `Poweradmin <https://www.poweradmin.org>`__: web interface for
   PowerDNS
