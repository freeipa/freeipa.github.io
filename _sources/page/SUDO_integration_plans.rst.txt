SUDO_integration_plans
======================



SUDO Intergation
================

This page outlines our plans regarding SUDO integration.



Current Situation
-----------------

Currently SUDO utility can be configured to use SUDO rules defined in
the LDAP server. Here is a high level diagram:

.. figure:: SUDO_Current.png
   :alt: SUDO_Current.png

   SUDO_Current.png

The description of the current LDAP support in SUDO can be read
`here <http://www.sudo.ws/sudo/sudoers.ldap.man.html>`__. Any Directory
Server can load the described schema and become a provider of the
centrally managed SUDO rules for the SUDO clients. The problem with such
solution is that the data in the SUDO rules is disjoint from the other
data stored in the Directory Server. User accounts is the example. This
causes an additional management burden for the administrators. IPA sees
goal in reducing this overhead and providing an integrated way of
manging all the objects stored in the IPA. IPA plans to provide an
integrated solution to manage SUDO rules centrally and in the integrated
way. This plan consists of the two phases.



Phase One
---------

On the first phase the IPA server will introduce a new schema that will
allow tight integration between the internal objects and the SUDO rules.
With the help of the compatibility plugin (that is a part of IPA) the
SUDO rules in the internal representation will be translated into the
representation expected by the SUDO clients on the fly. IPA already uses
this approach for netgroups and to allow compatibility between different
versions of the RFC 2307 schemata.

.. figure:: SUDO_Phase1.png
   :alt: SUDO_Phase1.png

   SUDO_Phase1.png

For this phase the following development work needs to be done:

-  Design the new schema for the SUDO. The following page is dedicated
   to this effort: `FreeIPAv2:SUDO Schema
   Design <FreeIPAv2:SUDO_Schema_Design>`__
-  Create management plugin to manage new objects in the schema
-  Expose new functionality via UI/CLI and describe it in the man pages
   and project documentation.



Phase Two
---------

On the second phase the SUDO utility will be integrated with the SSSD.
SUDO is currently implementing pluggable interface that this effort will
take advantage of. The plugin to the SUDO will operate under the same
principles as the NSS or PAM modules. The SSSD will be able to get the
SUDO rules in their native form and cache them on in its local storage
of offline use cases.

.. figure:: SUDO_Phase2.png
   :alt: SUDO_Phase2.png

   SUDO_Phase2.png

For this phase the following development work needs to be done:

-  Create a more detailed design page
-  SUDO needs to release a version that provides pluggable interface
-  Create a plugin into SUDO
-  Create a SUDO responder daemon as a part of the SSSD
-  Enhance IPA back end to fetch SUDO rules and cache them locally