Keytab_Retrieval
================

Overview
--------

Currently, once a Kerberos key has been created it is not possible to
retrieve it from the `KDC <Kerberos>`__. The only option is to generate
a new key. However it is suboptimal when multiple machines (e.g. a
cluster) need to share the same key (high availability/load balancing
purposes). In these cases distributing the key needs to be performed
completely through out of band/custom channels.

A mechanism to be able to retrieve an existing key from the
`KDC <Kerberos>`__ (subject to access control) resolves the *secure
distribution channel* problem. Leaving to the cluster members only the
problem of retrieving the key when a new one is created or an old one
rotated.

Additionally, the current ``setkeytab`` extended operation is suboptimal
as it creates keys on the client system. This has 2 bad side effects:

#. Password policies can't be applied because the server never sees the
   plain text
#. Keys for algorithms unknown to the client but known to the server
   cannot be generated (useful when admins workstations request keys on
   behalf of other systems).



Use Cases
---------



A load balancing cluster of HTTP server that allow GSSAPI/Krb5 negotiation
----------------------------------------------------------------------------------------------

The load balancing servers need to be able to share the same Service
Principal Name (SPN) for ``HTTP/public-http-server-name@REALM`` and key
so that whichever cluster member a client contact can decrypt the ticket
and authenticate the client.

This may be coupled with throwaway machines, where for example the
number of load balancer cluster members increase dynamically with the
traffic, so new machines are added to the pool dynamically.

In this case, assuming provisioning of individual machines credentials
is taken care of by the provisioning software, each machine can be given
the right to fetch the key for the common SPN, so that they can obtain
the necessary key material via an authenticated request to the IPA
server.

Design
------

The solution is to extend the password management plugin in FreeIPA to
also allow retrieving existing keys, and subject retrieval to **strict
access control**.

One of the problems of designing access control is that we cannot rely
just on making the ``KrbPrincipalKey`` attribute readable. Because that
attribute is actually encrypted and needs to be decrypted before a
keytab can be returned. It would also mean the attribute can be
explicitly fetched and brute force attacks run on it.

Because we need to process a request we need to create a new extended
operation similar to the ``setkeytab`` extended operation already
available.



New Extended operation
----------------------------------------------------------------------------------------------

New Extended operation OID: **2.16.840.1.113730.3.8.10.5**

The new extended operation allows to create a new keytab or get an
existing one and aim to supplant the current setkeytab extended
operation, allowing to eventually turn off the old one.

The following is the psudo-ASN.1 code definition for the extended
operation payload:

| ``KeytabGetRequest ::= CHOICE {``
| ``    newkey s     [0] NewKeys,``
| ``    curkey s     [0] CurrentKeys,``
| ``    reply        [2] Reply``
| ``}``
| ``NewKeys ::= SEQUENCE {``
| ``    serviceIdentity [0] OCTET STRING,``
| ``    enctypes        [1] SEQUENCE OF Int16,``
| ``    password        [2] OCTET STRING OPTIONAL``
| ``}``

| ``CurrentKeys ::= SEQUENCE {``
| ``    serviceIdentity [0] OCTET STRING``
| ``}``

If the getNew attribute is true a new keytab is being requested. In this
case a password may be provided or not. If not one is generated
randomly. In case a password is provided it is subject to password
policy checking as per policies defined on the entry for which a keytab
is being requested. A list of enctypes is always necessary in input when
a new keytab is requested. However the list is filtered though the
allowable enctypes list and if nothing is left the operation is refused.

If the getNew attribute is false, then the existing key is being
requested. In this case password and enctypes MUST NOT be set.

| ``Reply ::= SEQUENCE {``
| ``    new_kvno        Int32``
| ``    keys            SEQUENCE OF KrbKey,``
| ``}``

| ``KrbKey ::= SEQUENCE {``
| ``    key       [0] EncryptionKey,``
| ``    salt      [1] KrbSalt OPTIONAL,``
| ``    s2kparams [2] OCTET STRING OPTIONAL,``
| ``}``

| ``EncryptionKey ::= SEQUENCE {``
| ``    keytype   [0] Int32,``
| ``    keyvalue  [1] OCTET STRING``
| ``}``

| ``KrbSalt ::= SEQUENCE {``
| ``    type      [0] Int32,``
| ``    salt      [1] OCTET STRING``
| ``}``

The reply actually is very similar to the current ``setkeytab`` request
format.

A kvno is returned, followed by a sequence of ``KrbKeys``. Each
``KrbKey`` is constituted of an ``EncryptionKey`` buffer which includes
an integer that indicated the encryption type used and an octet string
that holds the actual key material. Optionally a ``KrbSalt`` is added
again indicating type and optionally a value.

This is the information actually needed by a client to be able to write
out a keytab after receiving the reply.



Access Control
----------------------------------------------------------------------------------------------

It would be nice, at this point to be able to have a way to express
access control related to actions taken by extended operations rather
than just direct access to attributes and to relate this access to
actors and targets.

The actors are the users attempting the operation as authenticated by
the `Directory Server <Directory_Server>`__. The targets are the objects
that hold the information. What is missing is a way to describe
permissions that tie a specific extended operation to them.

For this a new schema is necessary, based on a nice feature that is
available in LDAP - *sub-types*.



New Schema
----------------------------------------------------------------------------------------------

Attributes:

| ``IPA_OID.11.51 NAME 'ipaAllowedToPerform'``
| ``              DESC 'DNs allowed to perform an operation'``
| ``              SUP distinguishedName X-ORIGIN 'IPA-v3')``
| ``IPA_OID.11.52 NAME 'ipaProtectedOperation'``
| ``              DESC 'Operation to be protected'``
| ``              EQUALITY caseIgnoreMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15{128} )``

Objectclasses:

| ``IPA_OID.12.22 NAME 'ipaAllowedOperations'``
| ``              SUP top AUXILIARY``
| ``              DESC 'Class to apply access controls to arbitrary operations'``
| ``              MAY ( ipaAllowedToPerform $ ipaProtectedOperation ) X-ORIGIN 'IPA v3')``

This schema allows to add the ``ipaAllowedToPerform`` attribute to an
object, with a sub-type that indicates what special operation we want to
allow. The DN in the value indicates who is allowed to perform the
operation. The ``ipaProtectedOperation`` attribute is "virtual" and is
only ever used in ACI instructions. An extended plugin that want to
check if an operation is possible will check if operating on the
``ipaProtectedOperation;sub-type`` attribute is allowed but that
operation will never actually be performed. However if it were nothing
would really happen, a useless attribute may end up being added to an
object, but that wouldn't change the security properties of the
operation.



New ACIs
----------------------------------------------------------------------------------------------

The extended operation uses 2 named sub-types: read_keys/write_keys. The
read_keys sub-type identify the ability to retrieve a key, while
write_keys allows someone to create a new key (from a password or a
randomly generated one).

An example ACI rule to allow retrieval is this:

``aci: (targetattr="ipaProtectedOperation;read_keys")(version 3.0; acl "Users allowed to retrieve keytab keys"; allow(read) userattr="ipaAllowedToPerform;read_keys#USERDN";)``

For this ACI to have effect an attribute needs to be added to a target
service entry like this:

| ``dn: HTTP/www.example.com@EXAMPLE.COM,cn=services,cn=accounts,dc=example,dc=com``
| ``changetype: modify``
| ``add: objectclass``
| ``objectclass: ipaAllowedOperations``
| ``-``
| ``add: ipaAllowedToPerform;read_key``
| ``ipaAllowedToPerform;read_key: fqdn=clustermember1.example.com,cn=computers,cn=accounts,dc=example,dc=com``
| ``ipaAllowedToPerform;read_key: fqdn=clustermember2.example.com,cn=computers,cn=accounts,dc=example,dc=com``
| ``ipaAllowedToPerform;read_key: fqdn=clustermember3.example.com,cn=computers,cn=accounts,dc=example,dc=com``

With this ACI and attributes in place clustermember1.example.com,
clustermember2.example.com and clustermember3.example.com hosts can
retrieve an existing keytab for the service HTTP on the www.example.com
host.

`V4/Keytab Retrieval Management <V4/Keytab_Retrieval_Management>`__
design page describes administration interface for setting the
ipaAllowedToPerform attribute. CLI equivalent for the LDIF above is:

``ipa service-allow-retrieve-keytab HTTP/www.example.com --hosts={clustermember1.example.com,clustermember2.example.com,clustermember3.example.com}``



Compatibility with older FreeIPA servers
----------------------------------------------------------------------------------------------

``ipa-getkeytab`` falls back to the old extended operation for fetching
new keys when an old server does not have the new extended operation.

Implementation
--------------

The old setkeytab operation was used in conjunction with the
``managedBy`` attribute to allow to set keytabs by other entities. For
example the host keytab is allowed, by default to request arbitrary
services keys on the same hosts via the ``managedBy`` attribute.

In order to preserve this feature an additional ACI has been provided:

``aci: (targetattr="ipaProtectedOperation;write_keys")(version 3.0; acl "Entities are allowed to rekey managed entries"; allow(write) userattr="managedby#USERDN";)``



Feature Management
------------------

UI

N/A.

CLI

``ipa-getkeytab`` has a new ``-r`` switch:

``  -r, --retrieve                                           Retrieve current keys without changing them``



How to Test
-----------



Use Case: A load balancing cluster of HTTP server that allow GSSAPI/Krb5 negotiation (TBD)
----------------------------------------------------------------------------------------------

#. Install FreeIPA server with DNS on a host, e.g. with hostname
   ``server.example.test``
#. Enroll FreeIPA clients ``client1.example.test`` and
   ``client2.example.test``
#. Create DNS A record ``client.example.test`` that has 2 forward
   addresses of ``client1.example.test`` and ``client2.example.test``
#. Add a new host ``client.example.test`` - there will be no client
   enrolled to it:

      ``ipa host-add client.example.test``

#. Add a new service HTTP/client.example.test:

      ``ipa service-add HTTP/client.example.test``

#. Allow ``client1.example.test`` and ``client2.example.test`` to read
   ``client.example.test`` Kerberos key by configuring
   ``ipaAllowedToPerform;read_key`` attribute following the example in
   `New ACIs <#New_ACIs>`__ section.

      ``ipa service-allow-retrieve-keytab HTTP/client.example.test --hosts={client1.example.test,client2.example.test}``

#. On both ``client1.example.test`` and ``client2.example.test`` read
   the keytab for ``client.example.test``

      ``ipa-getkeytab -r -s server.example.test -p HTTP/client.example.test -k /etc/httpd/conf/client.keytab``

#. Configure Apache with mod_auth_kerb on both clients and secure it
   with Kerberos
#. With any FreeIPA user with valid Kerberos ticket, try to access web
   server on ``client.example.test``. It should work fine whether
   forwarded to ``client1.example.test`` or ``client2.example.test``

`Category:FreeIPA V4 Test Plan <Category:FreeIPA_V4_Test_Plan>`__
`Category:FreeIPA Test Plan <Category:FreeIPA_Test_Plan>`__