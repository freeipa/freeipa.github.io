Introduction
------------

The ``ipa-join`` command is used to join a machine to the IPA realm.
What this does is:

-  Create a host entry if one does not exist
-  Create a host/ service principal within the host entry
-  Retrieve a keytab

Setup
-----

ipa-join is not currently integrated into ``ipa-client-install``.
``ipa-client-install`` must be run prior to running ipa-join.

Information used by ipa-join such as the server to connect to is found
in /etc/ipa/default.

The CA certificate used, if needed, is in /etc/ipa/ca.crt and is
retrieved by the IPA client installer.

Options
-------

-  -h hostname: set the FQDN of this host. This is normally the nodename
   value of uname (2)
-  -k keytab: the location of the keytab to write. The default is
   ``/etc/krb5.keytab``
-  -w bindpw: the one-time password to use for bulk enrollment
-  -q: quiet mode, errors only
-  -d: debug mode

.. _authenticated_join:

Authenticated join
------------------

If the user running ``ipa-join`` has kerberos credentials then those are
used to authenticated in order to retrieve a keytab **unless** the user
includes the one-time password on the command-line. If the password is
included then this is treated as a bulk enrollment.

These requests use the XML-RPC API.

Example
~~~~~~~

::

   # kinit admin
   # ipa-join

.. _bulk_enrollment:

Bulk enrollment
---------------

A bulk host is defined as a pre-created host entry that contains a
one-time password. This password allows a user to authenticate over LDAP

.. _example_1:

Example
~~~~~~~

::

   # ipa-join -w secret123

.. _things_to_test:

Things to test
--------------

The assumption for all of these is that the client is already
configured. Whether the user has credentials or not will vary by test
case.

I see 3 overall scenarios to test:

-  Enrollment by admin user
-  Enrollment by delegated user
-  Enrollment with one-time password (OTP)

Within each of these you should test:

-  The host exists and is unenrolled
-  The host exists and is enrolled
-  The host doesn't exist yet

You need to add the -k option if you are not doing the tests are root.
It will fail with a file permissions error otherwise.

For delegation, the rolegroup to be a member of is hostadmin

::

   ipa rolegroup-add-member --users=someuser hostadmin

Unjoin
------

For lack of a better word, when you want to leave the IPA realm and
clean up the client.

This will involve:

-  Restore the client to its previous state
-  Removing any IPA principals from /etc/krb5.keytab
-  Deleting the host from the IPA server

``ipa-rmkeytab`` will be used on the client to remove any principals. It
is a generic tool that can delete an individual principal or all
principals for a given realm. We will use the latter in our uninstaller.
Rather than try to find all possible keytabs we'll just do
/etc/krb5.keytab for now.
