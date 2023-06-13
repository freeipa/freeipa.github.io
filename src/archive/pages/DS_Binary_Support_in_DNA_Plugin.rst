\__TOC_\_

Overview
========

Currently the DNA plugin doesn't support binaries, so new parameters
need to be added to specify the output format, byte-ordering, and
bit-length. The binary prefix needs to be specified in base-64 encoding.

Configuration
=============

::

   dn: cn=Object SIDs,cn=Distributed Numeric Assignment Plugin,cn=plugins,cn=config
   objectClass: top
   objectClass: extensibleObject
   cn: Object SIDs
   dnaType: objectSid
   dnaMaxValue: 1000
   dnaMagicRegen: 0
   dnaFilter: (|(objectClass=user)(objectClass=group))
   dnaScope: dc=domain1,dc=com
   dnaNextValue: 500
   dnaSharedCfgDn: cn=Object SIDs,ou=Ranges,dc=domain1,dc=com
   # Prefix is domain SID in binary format (base-64)
   dnaPrefix:: AQUAAAAAAAUVAAAAAdL5iaonuvzYomz/
   # SID is binary
   dnaFormat: binary
   # RID is using little-endian
   dnaByteOrdering: little-endian
   # RID is 16-bit long
   dnaBitLength: 16

   dn: cn=Object SIDs,ou=Ranges,dc=domain1,dc=com
   objectClass: top
   objectClass: nsContainer
   cn: Object SIDs

Parameters:

-  dnaPrefix: string \| binary in base-64
-  dnaFormat: "normal" \| "binary". Default: "normal"
-  dnaByteOrdering: "big-endian" \| "little-endian". Default:
   "big-endian"
-  dnaBitLength: integer. Default: 8

.. _proposed_changes:

Proposed Changes
================

Definitions
-----------

::

   #define DNA_FORMAT        "dnaFormat"
   #define DNA_FORMAT_NORMAL 0
   #define DNA_FORMAT_BINARY 1

   #define DNA_BYTE_ORDERING "dnaByteOrdering"
   #define DNA_BIG_ENDIAN    0
   #define DNA_LITTLE_ENDIAN 1

   #define DNA_BIT_LENGTH    "dnaBitLength"

   struct configEntry {
       char *prefix;
       int format;
       int byteOrdering;
       int bitLength;
   }

.. _parsing_configuration:

Parsing Configuration
---------------------

dna_parse_config_entry():

::

   rc = slapi_entry_attr_find(e, DNA_PREFIX, &attribute);
   rc = slapi_attr_first_value(attribute, &value);
   entry->prefix = slapi_value_get_berval(value);

   value = slapi_entry_attr_get_charptr(e, DNA_FORMAT);
   if (!strcmp(value, "normal")) {
       entry->format = DNA_FORMAT_NORMAL;

   } else if (!strcmp(value, "binary")) {
       entry->format = DNA_FORMAT_BINARY;
   }

   value = slapi_entry_attr_get_charptr(e, DNA_BYTE_ORDERING);
   if (!strcmp(value, "big-endian")) {
       entry->byteOrdering = DNA_BIG_ENDIAN;

   } else if (!strcmp(value, "little-endian")) {
       entry->byteOrdering = DNA_LITTLE_ENDIAN;
   }

   entry->bitLength = slapi_entry_attr_get_int(e, DNA_BIT_LENGTH);

.. _finding_first_free_value:

Finding First Free Value
------------------------

dna_first_free_value():

::

   int prefix_length = slapi_value_get_length(entry->prefix);

   char *new_value;

   if (config_entry->prefix) {

       if (entry->format == DNA_FORMAT_NORMAL) {
           strcpy(new_value, config_entry->prefix);
           strcat(new_value, value);

       } else if (entry->format == DNA_FORMAT_BINARY) {
           new_value = ... allocate array ...
           ... copy prefix to array ...

           if (entry->byteOrdering == DNA_BIG_ENDIAN) {
               ... copy value to array ...
           } else if (entry->byteOrdering == DNA_LITTLE_ENDIAN) {
               ... copy value to array ...
           }
       }

   } else
       strcpy(new_value, value);

References
==========

-  `Distributed Numeric Assignment Plug-in
   Attributes <http://www.redhat.com/docs/manuals/dir-server/8.1/cli/dna-attributes.html>`__
-  `DNA Plugin <http://directory.fedoraproject.org/wiki/DNA_Plugin>`__

`Category:Obsolete <Category:Obsolete>`__
