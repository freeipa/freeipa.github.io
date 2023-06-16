Overview
========

There are several methods to share the password across IPA and Samba:

-  sharing the clear text password via LDB module,
-  sharing the clear text password via LDAP backend,
-  sharing the encryption keys.

Regardless of the method, to ensure Kerberos interoperability each
system must be configured to generate encryption keys that are supported
by both applications.

.. _sharing_clear_text_password:

Sharing Clear Text Password
===========================

Sharing the clear text password means that the clear text password will
be distributed between IPA and Samba so each can create any type of
encryption keys or password hashes because it has the clear text
password.

.. _using_ldb_module:

Using LDB Module
----------------

In this method an LDB Module will intercept the user password before
being encrypted, then the Sync Agent will use it to set the password of
the corresponding IPA user.

.. _from_ipa_to_samba:

From IPA to Samba
~~~~~~~~~~~~~~~~~

When the password of an IPA user is modified the clear text password
will be recorded in the DS change log records. The Change Log Monitor
can retrieve it and pass it to the Sync Agent, which in turn will use it
to set the password of the corresponding Samba user.

.. figure:: Ipa3-password-synchronization1.png
   :alt: Ipa3-password-synchronization1.png

   Ipa3-password-synchronization1.png

.. _from_samba_to_ipa:

From Samba to IPA
~~~~~~~~~~~~~~~~~

When the password of a Samba user is modified, Samba will compute the
encryption keys before storing it in the DS, so the clear text password
will not be not available in the DS change log records. The Syncback
Module will intercept the operation and capture the clear text password,
send it to the Sync Agent which will use it to set the password of the
corresponding IPA user.

.. figure:: Ipa3-password-synchronization2.png
   :alt: Ipa3-password-synchronization2.png

   Ipa3-password-synchronization2.png

.. _using_ldap_backend:

Using LDAP Backend
------------------

In this method Samba will store the user password as clear text in the
LDAP backend, then the Sync Agent will use it to set the password of the
corresponding IPA user.

.. _sharing_encryption_keys:

Sharing Encryption Keys
=======================

Sharing the encryption keys means that both IPA and Samba will generate
the encryption keys and password hashes needed by both applications.

.. _from_ipa_to_samba_1:

From IPA to Samba
-----------------

When the password of an IPA user is modified the IPA plugin must also
generate the Samba keys and store it in the DIT. The Change Log Monitor
will pick up the Samba keys and pass it to the Sync Agent, which then
will store the keys directly into the corresponding Samba user object in
Samba's DS backend.

.. figure:: Ipa3-password-synchronization3.png
   :alt: Ipa3-password-synchronization3.png

   Ipa3-password-synchronization3.png

.. _from_samba_to_ipa_1:

From Samba to IPA
-----------------

When the password of a Samba user is modified, the Syncback Module could
do either one of the following:

-  extract Samba keys and reuse it as IPA keys,
-  if that's not sufficient, the module could intercept the clear text
   password and generate the IPA keys.

Then the module will pass IPA keys to the Sync Agent, which then will
store it in the corresponding IPA user object.

.. figure:: Ipa3-password-synchronization4.png
   :alt: Ipa3-password-synchronization4.png

   Ipa3-password-synchronization4.png

`Category:Obsolete <Category:Obsolete>`__
