\__NOTOC_\_

Overview
========

In ticket `#2184 <https://fedorahosted.org/freeipa/ticket/2184>`__ a new
option --pac_type was introduced to ipa service-mod to individually set
the PAC type for each service ticket. The allowed values are:

-  MS-PAC, the PAC used by Microsoft and specified in
   http://msdn.microsoft.com/en-us/library/cc237917.aspx
-  PAD, Posix Authorization Data, currently work in progress, draft can
   be found at http://tools.ietf.org/html/draft-ietf-krb-wg-pad-01
-  NONE, do not add any authorization data to the ticket
-  all other values are ignored

If the attribute is not set the default values specified in the
ipakrbauthzdata attribute of the ipaConfig object is used. If this is
missing as well, which e.g. is the case when no value is selected in the
Web UI or no value is given with 'ipa config-mod --pac-type=', 'NONE' is
assumed, i.e. not authorization data is added to the ticket.

If the attribute has multiple values all mentioned authorization data
blobs are added to the ticket. If one of the values is 'NONE' no data is
added. The FreeIPA CLI and Web UI take care that 'NONE' cannot be set
with any other value, but it might still be added manually with
ldapmodify.

.. _use_cases10i:

Use Cases
=========

-  For services which do not use the authorization data from the PAC the
   PAC type can be set to 'NONE'. This will keep the tickets smaller and
   avoids unneeded load on the FreeIPA server during the PAC processing.

-  For services which can only handle Kerberos tickets up to a maximal
   size, e.g. NFS, see
   `#3263 <https://fedorahosted.org/freeipa/ticket/3263>`__, the PAC
   type should be set to 'NONE'. The authorization data in the PAC can
   become quite large and can make the Kerberos ticket grow larger than
   the service can handle

-  For services which cannot handle the default PAC type but need a
   different one. (This is for future versions since currently only the
   MS-PAC is supported)

Design
======

Since the UI/CLI changes were already handled by
`#2184 <https://fedorahosted.org/freeipa/ticket/2184>`__ no additional
user interaction is needed here.

For the needed changes to the IPA KDC backend the changes done to fix
`#3263 <https://fedorahosted.org/freeipa/ticket/3263>`__ can be used as
a starting point.

Implementation
==============

In `#3263 <https://fedorahosted.org/freeipa/ticket/3263>`__ the NFS
service got a hardcoded default of 'NONE'. This implicit default should
be removed and replaced with the scheme described here. For newly
created NFS services the CLI and wth Web UI should set ipakrbauthzdata
to 'NONE' if not specified otherwise. During upgrade the valie
'nfs:NONE' will be added to the global ipakrbauthzdata attribute.

To read ipakrbauthzdata together with the other attributes of the
service principal it should be added to std_principal_attrs defined in
ipa_kdb_principals.c. Then the read value can be added to struct
ipadb_e_data in ipadb_parse_ldap_entry() to make it available in the
struct krb5_db_entry of the service principal.

Besides the plain types 'MS-PAC', 'PAD' and 'NONE' the new code must
also handle the 'service:TYPE' style values of the global authorization
data configuration. If the first component for the service principal
matches the 'service' identifier the corresponding type should be use of
there is no specific authorization data type is set for the service
principal. As the plain option the 'service:TYPE' style can be used to
multiple times to set more than one type but if one of the types is
'NONE' no authorization data is added.

All string comparisons are done case-sensitive.

.. _evaluation_logic:

Evaluation logic
----------------

-  If the service entry has ipakrbauthzdata attributes, those are used

   -  If one of the values is 'NONE' no authorization data is added
   -  If the values are 'MS-PAC' or 'PAD' the corresponding
      authorization data is added, respectively.
   -  All other values are ignored.

-  If the global configuration object has ipakrbauthzdata attributes,
   those are used as default for all services without the
   ipakrbauthzdata attribute

   -  If there are service specific default values 'service:TYPE' which
      matches the given service, only those are considered and the TYPE
      is evaluated according to the rules above
   -  For services without matching service specific default values the
      general default values are evaluated according to the rules above

-  For services where none of the above matches, no authorization data
   is added

.. _feature_managment:

Feature Managment
=================

UI
~~

UI related changes were added by ticket
`#2184 <https://fedorahosted.org/freeipa/ticket/2184>`__. Some fixes are
requested in `#3403 <https://fedorahosted.org/freeipa/ticket/3403>`__
and `#3404 <https://fedorahosted.org/freeipa/ticket/3404>`__.

Additionally 'service:TYPE' style entries should be allowed in the glbal
configuration (it does not make sense for services and hosts). See
`#3484 <https://fedorahosted.org/freeipa/ticket/3484>`__.

CLI
~~~

CLI related changes were added by ticket
`#2184 <https://fedorahosted.org/freeipa/ticket/2184>`__. Some fixes are
requested in `#3403 <https://fedorahosted.org/freeipa/ticket/3403>`__.

Additionally 'service:TYPE' style entries should be allowed in the
global configuration (it does not make sense for services and hosts).
See `#3484 <https://fedorahosted.org/freeipa/ticket/3484>`__.



Major configuration options and enablement
==========================================

This feature does not need any options and is enabled by default. The
behaviour is controlled by the value of the ipaKrbAuthzData of the
service object and the ipaConfig object.

Replication
===========

Changes only affect the KDC IPA backend. If not all server and replicas
are updated with this feature KDCs which are not updated might still
return Kerberos tickets with the default PAC type although there is a
different one mentioned in the service object.



Updates and Upgrades
====================

Setting the default for ipaKrbAuthzData in the ipaConfig object to
'MS-PAC' during upgrades is already handled.

As mentioned in the 'Implementation' section for all NFS service
principals the ipaKrbAuthzData must be set to 'NONE' if it is not
already set.

Dependencies
============

This feature does not add any new dependencies.



External Impact
===============

The changes will only touch the KDC IPA backend, no external impact is
expected.

.. _how_to_test10:

How to test
===========

Unfortunately there is not utility to check the presence of a PAC in a
ticket but because of the size of the PAC the size/size-change of the
credential cache can be used to see if a PAC is present in a ticket or
not.

After kinit the typical size of the credential cache file is about 500
bytes if the TGT does not contain a PAC and about 1200 bytes with a PAC.

A service ticket can be added to the credential cache file with the kvno
utility

::

   kvno host/fully.qualified.host.name@REALM.NAME

If the service ticket has a PAC an increase of the size of the
credential cache of 1000 bytes or more can be expected. Without the
difference will be about 450-500 bytes.

.. _testing_updates:

Testing updates
---------------

It should be tested that during updates the global ipaKrbAuthzData
attribute will get the value 'nfs:NONE'. i.e 'ipa config-show' will list
'nfs:NONE' in 'Default PAC types' after upgrades.



RFE Author
==========

`Sumit Bose <User:Sbose>`__
