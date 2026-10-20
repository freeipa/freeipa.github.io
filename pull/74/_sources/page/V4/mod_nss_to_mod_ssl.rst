mod_nss_to_mod_ssl
==================

Overview
--------

Use mod_ssl instead of mod_nss in Apache. This is part of a larger move
away from using NSS to using OpenSSL for crypto.

The target Fedora release for this is Fedora 28.



Use Cases
---------

-  mod_ssl is ubiquitous
-  mod_ssl uses flat PEM files instead of a set of complex tools to
   manipulate a database of certificates.
-  The database backend has changed in NSS from dbm to sqlite. Since
   changes need to be done in the source anyway to support that it is a
   more long-term solution to just drop mod_nss.

Design
------

Since inception the mod_nss certificate database is deeply integrated
into IPA and is or was the source of the RA agent certificate and the CA
chain. The RA agent cert has already been extracted but the CA chain
will need to be ripped out as well.

The installer has a lot of certificate handling code for NSS databases
but little for managing discrete PEM files. A lot of support code is
going to be needed and/or other calls abstracted to handle operations
like CA chain extraction, certmonger integration (request, start/stop
tracking, etc), validation and likely more.

The certificate and private key will be stored in
/etc/pki/tls/{certs|private} unencrypted and named httpd.{crt|key}.



New install
----------------------------------------------------------------------------------------------

Installing a new server using mod_ssl is likely to be conceptually
simpler than the upgrade case but the vast majority of the support code
will be written at this point in order to handle flat files vs databases
and remove the assumptions that the Apache mod_nss database is the
center of the universe.

The tasks roughly break down into:

-  generate cert/key as files via certmonger
-  track and untrack the certs
-  identify and install CA certificates where required (probably just
   use /etc/ipa/ca.crt)
-  validate the server certificate and its chain
-  change ssl.conf directives
-  handle installing via PKCS#12
-  identify where the NSS db is the center of the universe and change



Upgrading an existing installation
----------------------------------------------------------------------------------------------

Upgrading is going to be fragile in cases where users have modified
nss.conf for their own purposes. There is no plan currently to try to
identify or handle these cases.

The tasks roughly break down into:

-  extract cert and key into files
-  stop tracking NSS, start tracking files
-  change ssl.conf directives
-  disable mod_nss engine (leave package installed) and/or change the
   port to something other than 844*.
-  remove IPA includes from nss.conf (cleanliness)
-  add IPA includes to ssl.conf
-  deal with uninstall, ensure restored state is sane
-  potentially remove Server-Cert and chain from database

This is likely to be implemented as a generic upgrade task to use in
backup/restore as well.



ipa-server-certinstall
----------------------------------------------------------------------------------------------

Will need to change to handle file-based certificates. This will likely
end up as copying files to the proper location(s) in the filesystem.



Backup and Restore
----------------------------------------------------------------------------------------------

IPA typically doesn't allow one to backup and restore different versions
but code will be needed to catch the case where a user forces the issue.
Some mechanism would be needed to detect this case and call the upgrade
script in this case.

certmonger
----------------------------------------------------------------------------------------------

Parallel changes are going to be required in certmonger because it uses
the Apache mod_nss database as its source of CA trust. It may be that
/etc/pki/nssdb can be used instead, or the client can be switched to
using OpenSSL. This is TBD. Great care will be needed to ensure that
updated certmonger will continue to work with older IPA.

FIPS
----------------------------------------------------------------------------------------------

NSS can detect when FIPS is enabled and automatically enable the NSSFIPS
option in mod_nss. It remains to be seen what if anything will need to
change in mod_ssl. If FIPS is needed then there is an SSLFIPS option
that can be set.



Open questions
----------------------------------------------------------------------------------------------

After an upgrade, should we delete the Server-Cert from /etc/httpd/alias
to avoid confusion? It might be wise to leave it available for the
upgrade case, or generate a PKCS#12 file of it before removing it.

I suspect we are going to have to zap nss.conf and replace it with a
0-length file because the default port for mod_nss is 8443 which is used
by dogtag. We could pick some other random port to set it to but that
would raise SELinux issues. Zeroing out the config file seems more
efficient.

Implementation
--------------

-  **Dependencies**:

   -  updated certmonger

-  **Contingencies**:

   -  In the case of upgrade failure manual instructions will be needed
      to walk a user through the changes needed to convert to a mod_ssl
      database.



Feature Management
------------------

UI

N/A

CLI

N/A

Configuration
----------------------------------------------------------------------------------------------

Replace:

-  NSSProtocol -> SSLProtocol. IMHO we should still proactively set the
   protocols we want.
-  NSSCertificateDatabase -> SSLCertificateFile, SSLCertificateKeyFile
   and SSLCertificateChainFile.

Drop:

-  NSSNickname
-  NSSRenegotiation
-  NSSRequireSafeNegotiation
-  NSSCipherSuite (use global config cipher settings)
-  NSSPassPhraseDialog
-  NSSOCSP\*

Upgrade
-------

See the design section.



How to Use
----------

This should be generally invisible to the end-user with the following
exceptions:

-  The case where custom changes have been made to nss.conf. Those
   should be easily portable to ssl.conf in most cases. For those that
   rely on specific mod_nss behavior they will need to re-implement.
-  Users will need to get used to using openssl tools instead of NSS
   tools when managing the Apache certificates. This is going to require
   documentation changes as well. And this is why I want to remove the
   certificates from the database, so users don't change things here
   expecting it to affect the running server.



Test Plan
---------

The following scenarios will need to be tested.

For each case one can confirm basic functionality with:

-  ipactl restart (does Apache start?)
-  ipa user-show admin (do the cert and chain work?)
-  ipa cert-find
-  getcert list -f /etc/pki/tls/certs/httpd.crt (verify that tracking is
   correct)

TBD: Add sample of a properly tracked mod_ssl cert



New installations
----------------------------------------------------------------------------------------------

No user-provided options should impact functionality.

-  Install CA-ful
-  Install CA-less

Upgrades
----------------------------------------------------------------------------------------------

-  In-place upgrade from F-26
-  In-place upgrade from F-27



Creating replicas from mod_nss to mod_ssl masters
----------------------------------------------------------------------------------------------

-  Create replica from DL0 master

   -  3.0
   -  4.4
   -  4.5

-  Create replica from DL1 master

   -  4.5



Replacing certificates
----------------------------------------------------------------------------------------------

-  Use ipa-server-certinstall to replace the Apache cert

Renewal
----------------------------------------------------------------------------------------------

Move time to near expiration of the PEM-based certificates and force a
renewal and ensure that:

-  The PEM certificate is updated
-  LDAP is updated with the new certificate
-  Apache is restarted

This will need to be done for the new install and upgrade cases.

KRA
----------------------------------------------------------------------------------------------

-  Test basic vault operations (TBD)
-  KRA REST operations

Uninstall
----------------------------------------------------------------------------------------------

-  ssl.conf is restored
-  nss.conf is restored if previously installed
-  The Apache cert and key are removed