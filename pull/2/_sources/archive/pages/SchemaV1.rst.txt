.. _ipa_v1.0_schema_and_dit_layout:

IPA v1.0 Schema and DIT layout
==============================

.. _dit_layout:

DIT Layout
----------

-  suffix: dc=com (objectclass: pilotObject)

   -  cn=system

      -  cn=kerberos (objectclass: krbContainer)

         -  ... realm entries - krbRealmContainer

      -  cn=IPA

         -  ... IPA config entries (e.g. API, GUI etc.)

   -  dc=realm (objectClass: domain, ipaRealm)

      -  cn=people (objectclass: nsContainer)

         -  ... people entries (objectclass: inetOrgPerson,
            posixAccount, krbPrincipalAux, sambaSamAccount, inetUser)

      -  cn=groups (objectclass: nsContainer)

         -  ... group entries (objectclass: groupOfUniqueNames,
            posixGroup, sambaGroupMapping)

The suffix entry uses the pilotObject objectclass in order to allow
identification of the IPA service like so:

-  uniqueIdentifier: IPA
-  info: IPA V1.0

cn=system contains configuration that is global to the system, including
IPA configuration itself.

cn=kerberos contains kerberos configuration using schema as specified by
XXX

cn=IPA contains IPA specific configuration (possibly with our own custom
schema TBD)

dc=realm contains all entries for the organization, domain (RFC2247) is
used to allow the dc attribute specifying the most significant portion
of the realm name that is also the RDN.

The ipaRealm objectclass enables the discovery of the realm container
with the network data using a simple search. A client can search for
objectclass=ipaRealm and discover the basedn for its searches.

(QUESTION: If we were to add this custom ipaRealm schema anyway, why not
make it something that replaces the pilotObject above and allows
discovery data like this to go there? We could add all sorts of stuff,
like place to add people, place to add computers etc.).

.. _people_schema:

People Schema
-------------

People use:

-  posixAccount (RFC2307)
-  krbPrincipalAux (XXX)
-  inetOrgPerson (RFC2798)
-  sambaSamAccount (samba)
-  inetUser

.. _groups_schema:

Groups Schema
-------------

Groups use:

-  posixGroup (RFC2307)
-  groupOfUniqueNames(RFC4519)
-  sambaGroupMapping(Samba)
