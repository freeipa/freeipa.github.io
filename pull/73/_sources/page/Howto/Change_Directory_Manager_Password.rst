Change_Directory_Manager_Password
=================================

*cn=Directory Manager* password is used by FreeIPA installation tools
when bootstrapping the `PKI <PKI>`__ installation and for the *admin*
user in the PKI. While the FreeIPA web service itself does not use the
password after the PKI is installed as it authenticates itself using a
certificate, the password is still used to encrypt the CA certificate
stored in ``/root/cacert.p12`` on FreeIPA servers with PKI or to
authenticate a new replica with a CA to communicate to the other master
and obtaining an installation token for the PKI.

Thus, if the Directory Manager's password `is
changed <http://directory.fedoraproject.org/docs/389ds/howto/howto-resetdirmgrpassword.html>`__,
additional updates are required to propagate the change to PKI.

Procedure
=========

Following procedure needs to be performed on **all FreeIPA replicas
with**\ `PKI <PKI>`__.

In the procedure below:

-  ``$DM_PASSWORD`` is the new Directory Manager password
-  ``$KEYDB_PIN`` is the PIN for PKI certificate storage. It can be
   retrieved from ``internal`` configuration option in
   ``/etc/pki-ca/password.conf`` for Dogtag 9 or from
   ``/etc/pki/pki-tomcat/password.conf`` in Dogtag 10
-  ``$ALIAS_PATH`` is a path to PKI certificate storage. Use
   ``/var/lib/pki-ca/alias/`` for Dogtag 9 and
   ``/var/lib/pki/pki-tomcat/alias/`` for Dogtag 10
-  ``$CA_PORT`` is a port of `Directory Server <Directory_Server>`__
   instance where CA database is running. Use 7389 for Dogtag and 389
   for Dogtag 10



1. Update LDAP bind password
----------------------------

Configure all replicas to use the new password by editing
``/etc/pki-ca/password.conf`` for Dogtag 9 or
``/etc/pki/pki-tomcat/password.conf`` for Dogtag 10:

::

   ...
   internaldb=<password>
   ...



2. Update password of cacert.p12
--------------------------------

On all replicas create the password files using the following commands:

::

   # echo -n $DM_PASSWORD > /root/dm_password
   # echo -n $KEYDB_PIN > /root/keydb_pin

**Important:** If you use a text editor instead of echo -n, the files
might contain EOL characters. Trim the EOL characters with the following
commands:

::

   # tr -d '\n' < /root/dm_password > /root/dm_password.new
   # mv /root/dm_password.new /root/dm_password
   # tr -d '\n' < /root/keydb_pin > /root/keydb_pin.new
   # mv /root/keydb_pin.new /root/keydb_pin

Then re-generate the ``cacert.p12`` file with the following command:

::

   # /usr/bin/PKCS12Export -d $ALIAS_PATH -p /root/keydb_pin -w /root/dm_password -o /root/cacert.p12



3. Update PKI admin password
----------------------------

From one of the replicas change PKI admin password with the following
command:

::

   # ldappasswd -h localhost -ZZ -p $CA_PORT -x -D "cn=Directory Manager" -W -T /root/dm_password "uid=admin,ou=people,o=ipaca"

**Important:** Verify the password with the following command:

::

   # ldapsearch -h localhost -ZZ -p $CA_PORT -x -D "uid=admin,ou=people,o=ipaca" -W -b "" -s base

4. Cleanup
----------

Remove password files from all replicas:

::

   # rm /root/dm_password
   # rm /root/keydb_pin