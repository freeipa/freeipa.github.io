Automatic_Certificate_Request_Generation
========================================

Overview
========

It is currently difficult to create a correct CSR to request a
certificate from FreeIPA, especially when using uncommon certificate
profiles. The user must coax tools such as openssl into generating a CSR
that will pass validation by FreeIPA under the selected certificate
profile. For profiles besides caIPAserviceCert, the user interface
provides no guidance as to the correct commands or configs to be used
for this task. This situation is especially disappointing considering
that in many cases, nearly all of the information that should be
included in the certificate is already available in LDAP (and in fact
FreeIPA reads it from there during validation).

This design aims to improve this situation by automatically generating
certificate field values from LDAP data where possible. In this model,
the FreeIPA certificate profile will be augmented to contain a means of
mapping from LDAP attributes to certificate fields. Additionally, in
cases where the required data is not in LDAP we can still use the fields
and constraints specified in the profile to request data from the user,
validate the responses, and include the data in the correct place in the
CSR.

`This blog
post <https://blog-ftweedal.rhcloud.com/2015/11/freeipa-pki-current-plans-and-a-future-vision/>`__
outlines a vision for how this process could look in the long term.
However, for the time being we will start with some smaller changes that
should still make things easier for administrators.



Use Cases
=========



TLS server authentication
-------------------------

Service (or host?) certificate used for authenticating during TLS
handshake. Has the id-kp-serverAuth extended key usage enabled.



TLS client authentication
-------------------------

User certificate used for client authentication during a TLS handshake
with mutual authentication. Has the id-kp-clientAuth extended key usage
enabled.



S/MIME User Signing Certificates
--------------------------------

User certificate with the id-kp-emailProtection extended key usage
enabled. Must include an rfc822Name (email address) in the Subject
Alternate Name extension.



Smart Card Authentication
-------------------------

This is a user certificate as well, but (as discussed
`here <https://blog-nkinder.rhcloud.com/?p=184>`__) may be intended for
authentication only, not encryption, and therefore have the key and data
encipherment usage types disabled. Significantly, all signing operations
in this use case are performed by the smart card. Thus the user
interface generating the fields in CSR will need to interact with the
card to get the completed CSR with signature.



Key Escrow
----------

A convenient web-based UI for cert generation would be to generate a
keypair on the server, and then download it to the client along with the
signed cert. This may even be a good practice; for encryption it is
generally a good idea to maintain a copy of the private key so that data
can still be retrieved if the main copy is lost. Such certs should not
be used for non-repudiation, however.



Cert requiring a novel extension
--------------------------------

An administrator has a need for user certificates containing an
extension never before used in FreeIPA. They need the value of this
extension to be constructed form several fields in the user object. (As
a made-up example, say it should be the user's street address, city,
state, and postal code fields, separated by \| characters.) They should
be able to add a profile for these certificates and have the new field
be automatically populated when they make requests with that profile.

IECUserRoles
------------

User certificate with added User Roles extension, OID 1.2.840.10070.8.1,
for RBAC. This field is currently not understood by FreeIPA or Dogtag
but is simply passed along from the CSR to the certificate. This is an
example where user-provided data will be needed until additional
components are written to store the User Roles data in LDAP.

Design
======

Components:

-  New configuration language to specify mapping from stored data to
   certificate field values
-  Library to read mapping rules, get data for a principal, format it
   into a script that builds a CSR for a particular certificate profile
-  New UI to collect any unspecified values from user and generate CSR



Mapping data to fields
----------------------

Goals: Each field in the certificate profile can be generated from the
data stored by FreeIPA plus supplementary user input. However, the
mapping can sometimes be nontrivial, and change in ways that are
unanticipated by the FreeIPA developers. Thus, the profile must have
control over:

#. Which fields to set, how they should be encoded in the cert, and
   constraints on the values.
#. Which of all the relevant object data to include in a field. For
   example, Subject Alternate Names can come from email addresses, LDAP
   object names, DNS names, and various other sources. A certificate
   issuer might want to include only some of the possible SAN types
   available for a particular principal in a particular type of cert.
#. How object data is combined to get the field value. For example, the
   deployment may need the components of the Subject field to go in a
   specific order. Or, a profile may introduce a new certificate field
   that is formatted differently from those previously used; this
   flexibility would allow the profile to derive this field from the
   data without requiring code changes.

As described in the `certificate profiles
design <V4/Certificate_Profiles#Design>`__, a certificate profile
consists of a Dogtag profile (defining rules for verifying a CSR and
transforming it into a cert) as well as some additional data stored
within FreeIPA.

-  Requirement 1 is already provided by Dogtag certificate profiles, so
   we don't need to do much there.
-  Requirement 2 is the FreeIPA component of the profile. For each field
   (required field or extension) in the certificate profile, the CSR
   generation library will invoke a "mapping rule" that defines how to
   add that field using the chosen helper, plus one or more rules that
   define how FreeIPA object fields can be used to fill in the field
   value. A collection of mapping rules for all known fields and
   extensions will be provided with the FreeIPA client.
-  Requirement 3 means giving administrators/profile creators the
   ability to define their own mapping rules. This allows the CSR
   generation feature to handle advanced use cases that are not yet
   known, without requiring a code release.

Configuration
----------------------------------------------------------------------------------------------

The mapping rules will be specified via configuration files in JSON
format. For the older design, in which mappings were stored in the IPA
database, see
`V4/Automatic_Certificate_Request_Generation/Schema <V4/Automatic_Certificate_Request_Generation/Schema>`__.

There are two types of configuration files meant to be accessible to
users: Cert profile configurations contain a JSON array of:

-  Cert mapping rule (JSON object) with fields:

   -  "syntax": name of "syntax" cert mapping rule - name of field and
      how values are combined
   -  "data": JSON array of names of "data" cert mapping rules - defines
      values to include in field

Cert mapping rule configurations contain a JSON object with fields:

-  "rules": JSON array of rules with different formats for different
   helper utilities. Each is a JSON object with fields:

   -  "helper": Target CSR generator (e.g. openssl)
   -  "template": Transformation template (see
      `V4/Automatic_Certificate_Request_Generation/Mapping_Rules <V4/Automatic_Certificate_Request_Generation/Mapping_Rules>`__)
   -  "options": JSON object of key-value pairs altering formatting
      behavior for a specific helper

-  "options": JSON object of key-value pairs altering formatting
   behavior for all helpers

Example
----------------------------------------------------------------------------------------------

A profile for a user cert could have the following configuration:

``userCert.json``

::

   [
       {
           "syntax": "syntaxSubject",
           "data": [
               "dataUsernameCN",
               "dataSubjectBase"
           ]
       },
       {
           "syntax": "syntaxSAN",
           "data": [
               "dataEmail"
           ]
       }
   ]

Then, the definitions of a couple of these cert mapping rules (see
`V4/Automatic_Certificate_Request_Generation/Mapping_Rules <V4/Automatic_Certificate_Request_Generation/Mapping_Rules>`__
for discussion of the template syntax):

``syntaxSubject.json``

::

   {
     "rules": [
       {
         "helper": "openssl",
         "template": "distinguished_name = {% call openssl.section() %}{{ datarules|reverse|join('\n') }}{% endcall %}"
       },
       {
         "helper": "certutil",
         "template": "-s {{ datarules|join(',') }}"
       }
     ],
     "options": {
       "required": true,
       "data_source_combinator": "and"
     }
   }

``dataUsernameCN.json``

::

   {
     "rules": [
       {
         "helper": "openssl",
         "template": "CN={{subject.uid.0}}"
       },
       {
         "helper": "certutil",
         "template": "CN={{subject.uid.0|quote}}"
       }
     ],
     "options": {
       "data_source": "subject.uid.0"
     }
   }



Certificate data formatting
---------------------------

A new library will allow users to generate a script that will build a
correct CSR. The parameters to the library call will be:

-  Certificate principal
-  Certificate profile
-  Target CSR generation helper

For each cert mapping rule in the chosen certificate profile, the
process will look up the transformation rule matching the targeted
generator. It will use the templates in these rules to format data from
the principal's object in IPA into field values formatted to be accepted
by that generator, and then use knowledge of the generator to construct
a full command line or config file to generate the certificate.

For example, a request targeting openssl would produce a script which
uses a config file like the following to generate the csr with the
``openssl`` command:

| ``[ req ]``
| ``prompt = no``
| ``encrypt_key = no``
| ``distinguished_name = dn``
| ``req_extensions = exts``
| ``[ dn ]``
| ``O=DOMAIN.EXAMPLE.COM``
| ``CN=user``
| ``[ exts ]``
| ``subjectAltName=@SAN``
| ``[ SAN ]``
| ``email=user@example.com``
| ``dirName=SANdn``
| ``[ SANdn ]``
| ``1.DC=com``
| ``2.DC=example``
| ``CN=users``
| ``UID=user``

The "req" section is required and defines parameters for the openssl req
command. The "dn" and "exts" sections contain components of the
distinguished name and x509v3 certificate extensions, respectively.
Those sections can be named anything and those names are referenced by
the distinguished_name and req_extensions parameters in the "req"
section.

In contrast, a request targeting certutil would produce a command line
like:

``certutil -R -a -s "CN=user,O=DOMAIN.EXAMPLE.COM" --extSAN "email:user@example.com,dn:UID=user;CN=users;DC=example;DC=com"``

Permissions
----------------------------------------------------------------------------------------------

There are no additional permissions required for this functionality. A
principal will only be able to request data via this method that they
would otherwise be able to read. If the mappings for the profile specify
data to which the requesting principal does not have access, those
fields will be left blank unless they have the "required" option set.



Certificate request workflows
-----------------------------



FreeIPA command-line client
----------------------------------------------------------------------------------------------

#. User runs command to request cert with autogenerated CSR
#. Command-line client requests principal object from server
#. Client prompts for user input for each profile field defined as
   user-specified
#. Client builds a script incorporating server and user data
#. Client runs script, passing data to helper library or program (such
   as openssl or NSS)
#. Helper generates private key and CSR
#. Client submits CSR to server
#. IPA server validates CSR against data in LDAP
#. IPA server forwards CSR to Dogtag, which issues cert
#. Cert is returned to the client
#. Client presents private key and cert to user



FreeIPA Web UI~

This flow works similarly to the command-line client, except that the
web browser is not able to generate the CSR automatically (although this
feature existed historically, support for it appears to be declining
dramatically). So, the browser presents the config file and/or command
line to the user, who runs the helper manually and enters the CSR back
into the browser.

Certmonger
----------------------------------------------------------------------------------------------

There are two workflows that could be implemented here, depending on
whether we want certmonger to prompt for more information or just take
everything on the command line.

**Option 1** for collecting CSR data:

#. User runs getcert command passing in alternate profile (-T flag)
#. Getcert synchronously requests CSR data from IPA commandline
#. IPA prompts for user input for each profile field defined as
   user-specified
#. Getcert adds certmonger request including server-generated and
   user-specified data

**Option 2** for collecting CSR data:

#. User runs getcert command passing in alternate profile (-T flag) and
   any user-specified certificate fields
#. Getcert adds certmonger request including user-specified data
#. Certmonger asynchronously requests CSR data from IPA commandline
#. Certmonger adds IPA-generated data to saved request

In either case, certmonger now proceeds to request certificate as in the
other cases.

Implementation
==============

Mapping rules: see
`V4/Automatic_Certificate_Request_Generation/Mapping_Rules <V4/Automatic_Certificate_Request_Generation/Mapping_Rules>`__



Feature Management
==================

UI



Cert mapping rule management UI~

In the initial prototype, rules will be added or modified by modifying
config files and no UI is available.



Cert profile management UI~

In the initial prototype, mappings for a profile will be added or
modified by modifying config files and no UI is available.



Certificate request UI~

The UI for issuing a new certificate should be updated according to the
workflow described in the `Design section <#Design>`__. Once the
principal and profile are specified, it should query the server for the
CSR data and prompt the user for any missing information. It should then
provide the user with the exact config file/command to run to generate
the CSR to enter.

CLI



Cert mapping rule management UI~

In the initial prototype, rules will be added or modified by modifying
the config files on the client.



Cert profile management UI~

In the initial prototype, mappings for a profile will be added or
modified by modifying the config files on the client.



Certificate request UI~

``ipa cert-get-requestdata``

This is a new method within the FreeIPA CLI that gathers the data needed
to construct a certificate request, in the format appropriate for the
specified helper program or library.

Request parameters:

``--principal=PRINCIPAL``
   Principal to be the subject of cert
``--profile-id=STR``
   Certificate profile to request
``--format=STR``
   Output format for CSR data (e.g. "openssl" for openssl config file)

Response parameters:

``files``
   Contents of config files needed to generate the request
``commands``
   Command lines to run to generate the request
``user-specified``
   Fields in the profile that can not be automatically filled and must
   be requested from the user

``ipa cert-request``

``--autofill``
   Automatically generate a CSR using server data
``--use=STR``
   Tool to use for building CSR (e.g. openssl)
TBD
   Options for specifying file/NSS database of existing or new key to
   use, where to write cert, key generation type and size, etc.



Configuration
-------------

Profiles and mapping rules will be configured using JSON files in
``/usr/share/ipa/csr/{profiles,rules}``, as described in `Mapping data
to
fields <V4/Automatic_Certificate_Request_Generation#Mapping_data_to_fields>`__.

Upgrade
=======

As this feature is only part of the client, no special considerations
for upgrades are necessary.



How to Use
==========



TLS server authentication
-------------------------

Certmonger:

::

   `` sudo ipa-getcert request ``\ :literal:` -K HTTP/`hostname` -N CN=`hostname`,O=EXAMPLE.COM`

IPA CLI:

`` ipa cert-request ``\ :literal:` --autofill --principal=HTTP/`hostname\``



TLS client authentication
-------------------------

Certmonger:

`` sudo ipa-getcert request ``\ `` -K ${USER} -N CN=${USER},O=EXAMPLE.COM -T caIPAUserCert``

IPA CLI:

`` ipa cert-request ``\ `` --autofill --principal=${USER} --profile-id=caIPAUserCert``



S/MIME User Signing Certificates
--------------------------------

Certmonger:

`` sudo ipa-getcert request ``\ `` -K ${USER} -N CN=${USER},O=EXAMPLE.COM -T caIPAUserCertSMIME``

IPA CLI:

`` ipa cert-request ``\ `` --autofill --principal=${USER} --profile-id=caIPAUserCertSMIME``



Smart Card Authentication
-------------------------

| `` $ ipa cert-get-requestdata --principal=${USER} --profile-id=caIPAUserCert --helper=openssl --out=user.conf  # Something like this, --out flag may be something else``
| `` $ openssl``
| `` OpenSSL> engine dynamic -pre SO_PATH:/usr/lib64/openssl/engines/engine_pkcs11.so -pre ID:pkcs11 -pre LIST_ADD:1 -pre LOAD -pre MODULE_PATH:opensc-pkcs11.so``
| `` OpenSSL> req -engine pkcs11 -new -key ``\ `` -keyform engine -out user.req -text -config user.conf``
| `` $ ipa cert-request user.req --principal=${USER} --profile-id=caIPAUserCert``

Thanks to `Nathan Kinder <https://blog-nkinder.rhcloud.com/?p=184>`__
for guidance on smart card interaction.



Key Escrow
----------

Not directly supported by this design, but any project to add this could
use the ipa cert-get-requestdata API for its CSR generation.



Cert requiring a novel extension
--------------------------------

Certmonger:

`` sudo ipa-getcert request ``\ `` -K ${USER} -N CN=${USER},O=EXAMPLE.COM -T FancyExtensionUserCert``

IPA CLI:

`` ipa cert-request ``\ `` --autofill --principal=${USER} --profile-id=FancyExtensionUserCert``



IECUserRoles
------------

Certmonger:

| `` sudo ipa-getcert request ``\ `` -K ${USER} -N CN=${USER},O=EXAMPLE.COM -T IECUserRoles \``
| ``   -V IECUserRoles=``

IPA CLI:

| `` ipa cert-request ``\ `` --autofill --principal=${USER} --profile-id=IECUserRoles``
| `` ``



Test Plan
=========



Certificate request API
-----------------------

-  Test that ``ipa cert-get-requestdata`` produces data for all included
   profiles.
-  Test that ``ipa cert-get-requestdata --helper=openssl`` output is
   accepted by openssl for all included profiles.
-  Test that ``ipa cert-request --autofill`` generates a certificate for
   all included profiles.
-  Test that ``ipa cert-get-requestdata --profile-id=IECUserRoles``
   outputs IECUserRoles as a user-specified field
-  Test that ``ipa cert-request --autofill --profile-id=IECUserRoles``
   prompts user for IECUserRoles value
-  Test that ``ipa cert-get-requestdata`` and
   ``ipa cert-request --autofill`` return an error on a profile with no
   mapping rules.



Certificate profile management API
----------------------------------

-  Test import of profile with mapping rules
-  Test import of profile referencing nonexistent mapping rule
-  Test import of profile with malformed mapping rules
-  Test import of profile name that already exists
-  Test modification of profile with mapping rules
-  Test export of profile with mapping rules



Alternatives Considered
=======================

Architecture
------------

This was originally to be implemented as the
``ipa cert-get-requestdata`` API call, but the design was changed to a
standalone library that can be used client-side. The standalone
implementation will be easier for users to update, not requiring a
server upgrade to pick up changes to the code. This will be useful if
changes are needed to the syntax passed to the helper command, or to add
new helpers. Since users will want to upgrade their helper utilities, or
even use multiple versions at the same time, this flexibility may be
needed.

It would seem to be most straightforward for the IPA server to submit
data to Dogtag directly instead of sending it back to the client. Most
of the data already comes from the server, and submitting data directly
could hide some of the complexity around supporting multiple helpers.
However, current Dogtag profiles only accept input in the form of a
signed CSR, and the signature must happen on the client side because
that is where the private key is. It is possible to create Dogtag
profiles that include "profile inputs" other than
`CertReqInput <https://git.fedorahosted.org/cgit/pki.git/tree/base/server/cms/src/com/netscape/cms/profile/input/CertReqInput.java>`__,
which is used in the `existing
profiles <https://git.fedorahosted.org/cgit/freeipa.git/tree/install/share/profiles/caIPAserviceCert.cfg#n10>`__
to read data from a CSR. For example,
`GenericInput <https://git.fedorahosted.org/cgit/pki.git/tree/base/server/cms/src/com/netscape/cms/profile/input/GenericInput.java>`__
seems to allow setting of arbitrary fields in the certificate before it
gets signed. This might provide a mechanism for IPA to directly add
additional data to a certificate, allowing the CSR to contain only the
fields that are supposed to be user-specified. However, this requires
much deeper integration with Dogtag and may not fit with the larger
architectural goals for PKI in FreeIPA, so we will not pursue it for
now.

Taking a different tactic, we could have Dogtag request any data that is
missing from the CSR but is required by the certificate profile, either
from LDAP directly or from IPA (as in `the blog
post <https://blog-ftweedal.rhcloud.com/2015/11/freeipa-pki-current-plans-and-a-future-vision/>`__
mentioned earlier). However, as requests to Dogtag are currently all
made with the same identity, there can be no user-specific privilege
checking on the call from Dogtag back to LDAP. Thus until `GSSAPI
requests to Dogtag <https://fedorahosted.org/freeipa/ticket/5011>`__ are
implemented there is a risk that a misconfigured profile could expose
data about a principal this way. Further, the necessary changes to
Dogtag to implement this are too involved for the time allotted to the
current project.



Mapping technique
-----------------

An earlier concept of the mapping between IPA data and configs used for
CSR generation did not include the abstraction of cert mapping rules.
Instead, each cert profile would contain one or more templates
describing how to construct a full configuration for different CSR
generation helpers. This would be a simpler approach because it does't
require a new object type for mapping rules. However, mapping rules
provide a few advantages:

-  A wide variety of mapping rules can be provided with IPA, making it
   easier to create new profiles
-  Mapping rules can be made to support multiple CSR generation helpers,
   in which case profiles that use them automatically support multiple
   helpers as well
-  Formatting of the helper-specific config is moved from template into
   code, which can better handle complex formats and different output
   types (config file, command line)
-  Increased knowledge about the format of each field in the certificate
   may make it easier to autogenerate FreeIPA+Dogtag profiles from a
   single source.