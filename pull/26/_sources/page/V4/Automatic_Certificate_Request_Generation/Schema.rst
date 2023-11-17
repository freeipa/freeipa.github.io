Schema
======

This page is an **obsolete previous version** of the storage components
of the `Automatic Certificate Request
Generation <V4/Automatic_Certificate_Request_Generation>`__ design. We
begin with an informal overview of the design, followed by an effort to
flatten the informal schema description into a valid LDAP schema.

Note: OIDs are taken from the space immediately after the other
certificate profiles schema in
``install/share/60certificate-profiles.ldif``.

Overview
========

Cert mapping rule objects:

-  Name
-  Description
-  Transformation rule (multi-value):

   -  Target CSR generator (e.g. openssl)
   -  Transformation template (see
      `V4/Automatic_Certificate_Request_Generation/Mapping_Rules <V4/Automatic_Certificate_Request_Generation/Mapping_Rules>`__)

Additions to cert profile objects:

-  Cert mapping rule (multi-value):

   -  DN of cert mapping rule for syntax and name of field
   -  DN of cert mapping rule for field value (multi-value)

Example
-------

A profile for an S/MIME user cert should have the following cert mapping
rules:

-  Cert mapping rule

   -  NameRule: "subjectName"
   -  ValueRule: "defaultOrganizationDN"

-  Cert mapping rule

   -  NameRule: "subjectAltName"
   -  ValueRule: "qualifiedEmailAddress"
   -  ValueRule: "qualifiedDirName"

Then, the definitions of these cert mapping rules (transformations
currently given using `NIS function specifier
syntax <https://git.fedorahosted.org/cgit/slapi-nis.git/plain/doc/format-specifiers.txt>`__
but see
`V4/Automatic_Certificate_Request_Generation/Mapping_Rules <V4/Automatic_Certificate_Request_Generation/Mapping_Rules>`__
for discussion of alternatives):

-  

   -  Name: subjectName
   -  Description: Subject Name field of certificate.
   -  Transformation rule

      -  Helper: openssl
      -  Template: "distinguished_name=%{values}"

   -  Transformation rule

      -  Helper: certutil
      -  Template: "-s %{values}"

-  

   -  Name: defaultOrganizationDN
   -  Description: Principal name as CN, with default O field appended.
   -  Transformation rule

      -  Helper: openssl
      -  Template: "CN=%{cn},O=EXAMPLE.COM"

   -  Transformation rule

      -  Helper: certutil
      -  Template: "CN=%{cn},O=EXAMPLE.COM"

-  

   -  Name: subjectAltName
   -  Description: Subject Alternate Name extension.
   -  Transformation rule

      -  Helper: openssl
      -  Template: "subjectAltName=%merge(',', %{values})"

   -  Transformation rule

      -  Helper: certutil
      -  Template: "--extSAN=%merge(',', %{values})"

-  

   -  Name: qualifiedEmailAddress
   -  Description: Principal's email address, with tag to mark it as an
      email.
   -  Transformation rule

      -  Helper: openssl
      -  Template: "email:%{email}"

   -  Transformation rule

      -  Helper: certutil
      -  Template: "rfc822:%{email}"

-  

   -  Name: qualifiedDirName
   -  Description: Principal's DN, with tag to mark it as a dirName.
   -  Transformation rule

      -  Helper: openssl
      -  Template: "dirName:%opensslSection(%{dn})"

   -  Transformation rule

      -  Helper: certutil
      -  Template: "dn:%merge(',', %{dn})"

Note: ``%opensslSection`` is a hypothetical openssl-specific directive
that would create a new config file section for the components of
``%{dn}`` and place a reference to that section in the template output.



Certificate Profiles
====================

The objects in this section will be considered conceptually part of the
certificate profile, and represent the way that specific profile is
configured to build a certificate request. These objects will be
imported and exported along with the certificate profile.



CertProfile additions
---------------------

In each CertProfile, we can now list the fields that need to be mapped
into the certificate request.

-  Cert field mapping (multi-value): reference to CertFieldMappingRule

::

   attributeTypes: (2.16.840.1.113730.3.8.21.1.9 NAME 'ipaCertFieldMapping' DESC 'Reference to ipaCertFieldMappingRule: Ruleset describing how to construct a certificate field' SUP distinguishedName EQUALITY distinguishedNameMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 X-ORIGIN 'IPA v4.5' )
   objectClasses: (2.16.840.1.113730.3.8.21.2.1 NAME 'ipaCertProfile' SUP top STRUCTURAL MUST ( cn $ description $ ipaCertProfileStoreIssued ) MAY ipaCertFieldMapping X-ORIGIN 'IPA v4.2' )

CertFieldMappingRule
--------------------

This object represents the way a particular field should be constructed
in CSRs for this certificate profile.

-  Name
-  Cert syntax mapping: DN of CertMappingRuleset for syntax and name of
   field
-  Cert data mapping (multi-value): DN of CertMappingRuleset for field
   value

::

   attributeTypes: (2.16.840.1.113730.3.8.21.1.10 NAME 'ipaCertSyntaxMapping' DESC 'Reference to ipaCertMappingRuleset: How to format the specification for this field' SUP distinguishedName EQUALITY distinguishedNameMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 SINGLE-VALUE X-ORIGIN 'IPA v4.5' )
   attributeTypes: (2.16.840.1.113730.3.8.21.1.11 NAME 'ipaCertDataMapping' DESC 'Reference to ipaCertMappingRuleset: How to map data into field values' SUP distinguishedName EQUALITY distinguishedNameMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 X-ORIGIN 'IPA v4.5' )
   objectClasses: (2.16.840.1.113730.3.8.21.2.4 NAME 'ipaCertFieldMappingRule' SUP top STRUCTURAL MUST ( cn $ ipaCertSyntaxMapping $ ipaCertDataMapping ) X-ORIGIN 'IPA v4.5' )



Mapping Rules
=============

The objects in this section conceptually make up the "mapping rules."
They are built into FreeIPA (or, once the ability to do so is added,
created by an administrator).

Each rule for transforming data to config syntax must be associated with
the specific helper that can consume that syntax. There are two possible
approaches to representing this association. Option B, which uses
attribute tagging to record the helper, has one fewer objectClass and
stores the data in fewer objects. However, it limits future
extensibility because the rule is stored in text string rather than
having its own entry. This means that if additional metadata is required
(as a possible example, one might want to record whether a string
represents a command-line option or a config file line) it will need to
be encoded in the text of the rule. It is difficult to determine whether
this is a serious problem.



Option A
--------

In this option, the association to helpers is accomplished by placing
each rule in a separate CertTransformationRule object that records both
the rule and the helper that it belongs to. A CertMappingRuleset
references several of these CertTransformationRules, as seen in the
following diagram: |CertMappingSchemaA.dot.png|

CertMappingRuleset
----------------------------------------------------------------------------------------------

This object represents the ways a data item might need to be formatted
to achieve a particular CSR result, for all the different CSR generation
helpers.

-  Name
-  Description
-  Transformation rule (multi-value): reference to
   CertTransformationRule

::

   attributeTypes: (2.16.840.1.113730.3.8.21.1.12 NAME 'ipaCertTransformation' DESC 'Reference to ipaCertTransformationRule: How a data item should be mapped for a particular helper' SUP distinguishedName EQUALITY distinguishedNameMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.12 X-ORIGIN 'IPA v4.5' )
   objectClasses: (2.16.840.1.113730.3.8.21.2.5 NAME 'ipaCertMappingRuleset' SUP top STRUCTURAL MUST ( cn $ description $ ipaCertTransformation ) X-ORIGIN 'IPA v4.5' )

CertTransformationRule
----------------------------------------------------------------------------------------------

This object represents a particular way of transforming data, for a
particular CSR generation helper.

-  Name
-  Target CSR generator (e.g. openssl)
-  Transformation template (see
   `V4/Automatic_Certificate_Request_Generation/Mapping_Rules <V4/Automatic_Certificate_Request_Generation/Mapping_Rules>`__)

::

   attributeTypes: (2.16.840.1.113730.3.8.21.1.13 NAME 'ipaCertTransformationTemplate' DESC 'How to transform a specific data item' EQUALITY caseExactMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 SINGLE-VALUE X-ORIGIN 'IPA v4.5' )
   attributeTypes: (2.16.840.1.113730.3.8.21.1.14 NAME 'ipaCertTransformationHelper' DESC 'Helper to which this transformation is targeted' EQUALITY caseExactMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 SINGLE-VALUE X-ORIGIN 'IPA v4.5' )
   objectClasses: (2.16.840.1.113730.3.8.21.2.6 NAME 'ipaCertTransformationRule' SUP top STRUCTURAL MUST ( cn $ ipaCertTransformationTemplate $ ipaCertTransformationHelper ) X-ORIGIN 'IPA v4.5' )



Option B
--------

In this option, the association of rules to helpers is accomplished
using LDAP attribute tagging. A CertMappingRuleset has a multi-valued
attribute to store the rules, and each such attribute is tagged with the
helper to which the rule belongs. |CertMappingSchemaB.dot.png|



CertMappingRuleset
----------------------------------------------------------------------------------------------

This object represents the ways a data item might need to be formatted
to achieve a particular CSR result, for all the different CSR generation
helpers.

-  Name
-  Description
-  Transformation rule (multi-value): Rule for transforming data into
   helper syntax. Must provide the helper name as attribute tag.

::

   attributeTypes: (2.16.840.1.113730.3.8.21.1.12 NAME 'ipaCertTransformation' DESC 'How to transform a specific item for a specific helper' EQUALITY caseExactMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 X-ORIGIN 'IPA v4.5' )
   objectClasses: (2.16.840.1.113730.3.8.21.2.5 NAME 'ipaCertMappingRuleset' SUP top STRUCTURAL MUST ( cn $ description $ ipaCertTransformation ) X-ORIGIN 'IPA v4.5' )

.. |CertMappingSchemaA.dot.png| image:: CertMappingSchemaA.dot.png
.. |CertMappingSchemaB.dot.png| image:: CertMappingSchemaB.dot.png