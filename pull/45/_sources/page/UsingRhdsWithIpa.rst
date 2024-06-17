UsingRhdsWithIpa
================



Installing Red Hat Directory Server
===================================

You should follow the documented procedures for installing Red Hat
Directory Server. These are available online at
` <http://www.redhat.com/docs/manuals/dir-server/>`__\ http://www.redhat.com/docs/manuals/dir-server/



Using Red Hat Directory Server with IPA
=======================================



Schema Requirements
-------------------

IPA uses the standard Red Hat Directory Server (RHDS) schema, combined
with the MIT Kerberos schema.



DIT Requirements
----------------

A number of rules apply to how the Directory Information Tree (DIT)
should be structured. These are probably more important than the schema
employed, unless at some point you want to relocate entries. This is
because RHDS does not support subtree renames, and complex scripting may
be required for a successful migration if this is not taken into
account.



DIT Assumptions
----------------------------------------------------------------------------------------------

The freeIPA DIT assumes the following:

The Suffix (BaseDN) is determined by splitting the realm name into
domain components. For example:

::

   EXAMPLE.COM -> dc=example,dc=com

The DIT then uses the following structure:

.. figure:: IPA-DIT.png
   :alt: IPA DIT structure

   IPA DIT structure

To gain an in-depth view of the schema and DIT used by freeIPA, the most
efficient approach would be to install freeIPA on a test Fedora 7
installation. This is very straight forward and could be completed in 10
- 15 minutes. The freeIPA installation provides a 389 Directory Server
instance that you can analyze and use as a basis for RHDS.



Examining the DIT
----------------------------------------------------------------------------------------------

**To examine the DIT, run the following command:**

::

   ldapsearch -x -b dc=example,dc=com

**or, if you have a kerberos ticket:**

::

   ldapsearch -Y GSSAPI -b dc=example,dc=com

Alternatively, you can install the gq package to examine both the DIT
and the schema.



Downloading and Installing freeIPA
==================================

Refer to the `Downloads <http://www.freeipa.org/page/Downloads>`__ page
for instructions on how to download the freeIPA packages.

Refer to the `Installation and Deployment
Guide <http://www.freeipa.com/page/InstallAndDeploy>`__ for instructions
on how to install freeIPA.