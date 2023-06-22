\__NOTOC_\_ March 25, 2011

The FreeIPA project team is pleased to announce the availability of the
`freeIPA 2.0 server <http://www.freeipa.org/page/Downloads>`__.

It is available in Fedora 15 and Fedora rawhide.

.. _known_issues:

Known Issues
------------

-  Installing IPA on Fedora-15 works but can take more time than Fedora
   14 due to systemd. It is not recognizing some restarts as being
   successful so only continues after a 3-minute timeout. We are working
   on a solution.
-  The latest tomcat6 package has not been pushed to updates-testing.
   You need tomcat6-6-0.30-5 or higher. The packages can be retrieved
   from koji at
   http://koji.fedoraproject.org/koji/buildinfo?buildID=231410. The
   installation will fail restarting the CA with the current tomcat6
   package in Fedora 15.
-  If the domain and realm do not match you may need to use the --force
   flag with ipa-client-install.
-  Dogtag replication is done separately from IPA replication. The
   ipa-replica-manage tool does not currently operate on dogtag
   replication agreements.
-  The OCSP URL encoded in dogtag certificates is by default the CA
   machine that issued the certificate.

Changlog since FreeIPA v2.0.0 rc3

Adam Young (1):

-  pwpolicy priority Priority is now a required field in order to add a
   new password policy. Thus, not having the field present means we
   cannot create one.

Endi S. Dewata (1):

-  Removed nested role from UI.

Martin Kosek (2):

-  Wait for Directory Server ports to open
-  Prevent stacktrace when DNS AAAA record is added

Pavel Zuna (1):

-  Update translation file (ipa.pot).

Rob Crittenden (4):

-  Always consider domain and server when doing DNS discovery in client.
-  Fix SELinux errors caused by enabling TLS on dogtag 389-ds instance.
-  Ensure that the system hostname is lower-case.
-  Automatically update IPA LDAP on rpm upgrades

Simo Sorce (1):

-  Domain to Realm Explicitly use the realm specified on the command
   line. Many places were assuming that the domain and realm were the
   same.

-  Fix uninitialized variable.
