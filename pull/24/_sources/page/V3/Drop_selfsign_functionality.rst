Drop_selfsign_functionality
===========================

\__NOTOC_\_

Not to be confused with `V3/Drop_selfsign <V3/Drop_selfsign>`__, a RFE
to only drop the --selfsign option.

Overview
========

Ticket `3494 <https://fedorahosted.org/freeipa/ticket/3494>`__ Drop
--selfsign server functionality:

IPA supports 2 flavors of certificate management:

-  IPA with pki-ca (dogtag) with either a self-signed certificate or
   with a certificate signed by external CA (--external-ca option)
-  IPA with no pki-ca installed (i.e CA-less), with certificates signed
   and provided by an external CA.

Previously, IPA had a "self-signed" mode, where certificate management
was done without pki-ca. This mode will be replaced by CA-less mode on
upgrade.



Use Cases
=========

#. User upgrades a server that uses the self-signed CA
#. The CA functionality is removed.
#. User uses commands below to manage certificates manually.

Design
======

On upgrade, selfsign masters will be converted to CA-less. The existing
certificate database and files the selfsign CA used will be left on disk
and may be used to issue new certificates manually.

IPA's ``cert-*`` commands will be no longer available. The following
commands will no longer issue certificates automatically:

-  host-del
-  host-mod
-  host-disable
-  service-del
-  service-mod
-  service-disable

Certificates may be issued manually (see instructions below) and loaded
with ``host-mod`` or ``service-mod``.

Server certificates tracked by certmonger will be untracked during the
upgrade.

The self-sign CAs were incapable of replication. With this change,
replicas can be created given appropriate (possibly wildcard) server
certificates.



Manual certificate management
=============================

This section shows commands that the removed selfsign backend ran behind
the scenes. This serves as a baseline or tutorial -- the reason why IPA
no longer runs the commands manually is to provide flexibility for users
that need it. If you want a simple solution, please use IPA's default
Dogtag backend.



Selfsign CA files
-----------------



NSS database
----------------------------------------------------------------------------------------------

The NSS database containing certs and keys is in ``/etc/httpd/alias``.



Noise file
----------------------------------------------------------------------------------------------

A noise file is generally put at ``/etc/httpd/alias/noise.txt``. Fill it
with random data whenever you need it:

``   head -c12 /dev/random | sha1sum | cut -d' ' -f1 > /etc/httpd/alias/noise.txt``

Be sure to remove the file after it's used.



NSS database password
----------------------------------------------------------------------------------------------

The NSS database password is stored in ``/etc/httpd/alias/pwdfile.txt``.



Serial number
----------------------------------------------------------------------------------------------

The file ``/var/lib/ipa/ca_serialno`` contains the CA's serial numbers
in INI format:

| ``   [selfsign]``
| ``   nextreplica = 500000``
| ``   replicainterval = 500000``
| ``   lastvalue = 1005``

Of these values, only ``lastvalue`` is used (replication of selfsign CAs
was never implemented). It is recommended to note the number, store it
in a more convenient format, and delete the ``ca_serialno`` file.

Each certificate issued by a particular CA must have a unique serial
number. To ensure this, increment the ``lastvalue`` before using it.

Installation
------------

Note that installation is *not needed* after an upgrade from selfsign;
these files are not removed by the upgrade.

Store a password in ``/etc/httpd/alias/pwdfile.txt``.

Then run:

``   /usr/bin/certutil -d /etc/httpd/alias -N -f /etc/httpd/alias/pwdfile.txt``

Create a noise file (see above), and create a CA cert by:

``   /usr/bin/certutil -d /etc/httpd/alias -S -n "$REALM IPA CA" -s "CN=$REALM Certificate Authority" -x -t CT,,C -1 -2 -5 -m $NEXT_SERIAL -v 120 -z $NOISE_FILE -f /etc/httpd/alias/pwdfile.txt``

Give the following answers:

| ``   Create key usage extension:``
| ``       0 - Digital Signature``
| ``       1 - Non-repudiation``
| ``       5 - Cert signing key``
| ``       Is this a critical extension [y/N]? y``
| ``   Create basic constraint extension``
| ``       Is this a CA certificate [y/N]?  y``
| ``   Enter the path length constraint, enter to skip [<0 for unlimited path]``
| ``       0``
| ``       Is this a critical extension [y/N]? y``
| ``   Extensions:``
| ``       5 6 7 9 n (SSL, S/MIME, Object signing CA)``

Export the CA cert:

``   /usr/bin/pk12util -d /etc/httpd/alias -o /etc/httpd/alias/cacert.p12 -n "$REALM IPA CA" -w /etc/httpd/alias/pwdfile.txt -k /etc/httpd/alias/pwdfile.txt``



Generating a certificate request
--------------------------------

Create a noise file (see above).

``   /usr/bin/certutil -d /etc/httpd/alias -R -s CN=$HOSTNAME,O=IPA -o $CERTREQ_FILENAME -k rsa -g 2048 -z /etc/httpd/alias/noise.txt -f /etc/httpd/alias/pwdfile.txt -a``

Example values:

-  HOSTNAME=ipaserver.ipadomain.example.com
-  CERTREQ_FILENAME=/tmp/service.csr



Issuing a certificate
---------------------

First generate a certificate request (see above). Then run:

| ``   NEXT_SERIAL=$(($NEXT_SERIAL + 1))  # (be sure to also store the number on disk!)``
| ``   /usr/bin/certutil -d /etc/httpd/alias -C -c "CN=$REALM Certificate Authority" -i $CERTREQ_FILENAME -o $CERT_FILENAME -m $NEXT_SERIAL -v 120 -f /etc/httpd/alias/pwdfile.txt -1 -5 -a``

Example values:

-  REALM=IPADOMAIN.EXAMPLE.COM
-  CERTREQ_FILENAME=/tmp/service.csr
-  CERT_FILENAME=/tmp/service.cert
-  NEXT_SERIAL - unique serial number, see above

For a server certificate (e.g. for a new replica), give the following
answers:

| ``   Create key usage extension:``
| ``       2 - Key encipherment``
| ``       9 - done``
| ``       n - not critical``
| ``   Create netscape cert type extension:``
| ``       1 - SSL Server``
| ``       9 - done``
| ``       n - not critical``

For an object signing certificate, give the following answers:

| ``   Create key usage extension:``
| ``       0 - Digital Signature``
| ``       5 - Cert signing key``
| ``       9 - done``
| ``       n - not critical``
| ``   Create netscape cert type extension:``
| ``       3 - Object Signing``
| ``       9 - done``
| ``       n - not critical``

For a service certificate (ipa service-add, ipa cert-request, ipa
host-add), add the ``-6`` option. The IPA commands also validate the
certificate, and with Dogtag, the old host/service certis revoked. These
steps are left entirely to the user. Answer:

| ``   Create key usage extension:``
| ``       0 - Digital Signature``
| ``       1 - Cert signing key``
| ``       2 - Key encipherment``
| ``       3 - Data encipherment``
| ``       9 - done``
| ``       n - not critical``
| ``   Create netscape cert type extension:``
| ``       0 - Server Auth``
| ``       9 - done``
| ``       n - not critical``
| ``   Create extended key usage extension:``
| ``       1 - SSL Server``
| ``       9 - done``
| ``       n - not critical``

This will put a PEM-encoded certificate in $CERT_FILENAME.

You may want to import the certificate into the DB, and track it; see
below.



Importing issued certificate into the database
----------------------------------------------

If you have a PEM certificate, open it in an editor, remove the start
and end markers, and save it in a new file. This will be a

``   /usr/bin/certutil -d /etc/httpd/alias -A -i $CERT_DER_FILENAME -n $CERT_NICKNAME -a -t ,,``

Example values:

-  CERT_DER_FILENAME=/tmp/service.der
-  CERT_NICKNAME=Server-Cert



Exporting server cert into PKCS#12
----------------------------------

Run:

``   /usr/bin/pk12util -o $CERT_PKCS_FILENAME -n $CERT_NICKNAME -d /etc/httpd/alias -k /etc/httpd/alias/pwdfile.txt -w /etc/httpd/alias/pwdfile.txt``

Example values:

-  CERT_PKCS_FILENAME=/tmp/service.p12
-  CERT_NICKNAME=Server-Cert

The resulting file can be given to ``ipa-replica-prepare``, with
contents of /etc/httpd/alias/pwdfile.txt as the password.



Tracking a certificate with certmonger
--------------------------------------

| ``   systemctl enable certmonger.service``
| ``   systemctl start certmonger.service``

``   /usr/bin/ipa-getcert start-tracking -d /etc/httpd/alias -n $CERT_NICKNAME -p /etc/httpd/alias/pwdfile.txt``

Implementation
==============

No additional requirements or changes discovered during the
implementation phase.



Feature Managment
=================

N/A



Major configuration options and enablement
==========================================

Upgrading from selfsign sets the following env settings
(/etc/ipa/default.conf):

-  enable_ra=False
-  ra_plugin=none

Replication
===========

Self-signed CAs were incapable of replication. With this change,
replicas can be created given appropriate (possibly wildcard) server
certificates.



Updates and Upgrades
====================

Selfsign certificates will be converted to CA-less on upgrade.

Dependencies
============

N/A



External Impact
===============

Documentation may need updating.



RFE Author
==========

`pviktori <User:pviktorin>`__