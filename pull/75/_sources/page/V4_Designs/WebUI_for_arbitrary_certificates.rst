WebUI_for_arbitrary_certificates
================================

Overview
--------

FreeIPA CLI contains commands
(``{user|host|service}_{add|remove}_cert``) for adding arbitrary
certificates. Nowadays WebUI displays certificates as list of base64
blobs. It's not much user-friendly, because it is not possible to read
any information about the certificate. This document designs extended
version of WebUI for CLI described in this design: `User
Certificates <http://www.freeipa.org/page/V4/User_Certificates>`__ and
also the CLI part which is necessary for WebUI.



Use Cases
---------

`Use
Cases <http://www.freeipa.org/page/V4/User_Certificates#Use_Cases>`__

Design
------

The new WebUI component should be able to display multiple certificate
and provide basic information about each certificate. I.e. cn of
user|host|service, serial number, issuer and validity. There should be a
way how to view data about certificate, how to get certificate in PEM
format, how to download, revoke and remove certificate hold (the last
two will be available only for certificates which are issued by our CA).
Also a possibility to add a new certificate should be present.

It would be good to perform certificate actions immediately without
using the save button. Therefore it will need to somehow mark and cover
the row when some action is running (i.e. revoking the certificate). The
immediate performation of action is more user friendly because for
example marking two certificates as "will be revoked", one as "will be
removed", and one as "certificate hold will be removed" is not so clear
as confirmation a immediate action.

WebUI needs to get information from base64 blob of arbitrary
certificate. We already have these information for certificates which
are issued by our CA, but not for other certificates, therefore the
ASN.1 parser is needed. That parser should support basic parts of ASN.1
format, no extensions needed. The WebUI needs information about
certificate and because there may be people who wants to get data from
certificate on command line, so it would be better to implement the
parser on the backend side.

Implementation
--------------

TODO: How certificates are stored in LDAP? More details about ASN.1
parser, about finding certificates?



Feature Management
------------------

UI

It requires new WebUI part which will be able to show certificates on
user|host|service's details pages. It could be done by multivalued
widget, where one row would represent one certificate. Each row should
have some basic information about certificate. These could be shown as
'label: value' list. Each row should also have dropdown menu with
actions for open view and get dialog, for downloading certificate as a
file, for revoking and removing certificate hold and for deleting it.
Add button for adding another certificate should be somewhere under the
last row of certificates. User add certificate by clicking on add button
and insert the certificate encoded in PEM format.

Every action which will be performed on certificates could be performed
immediately without clicking save button. It is more intuitive than, for
example, mark two certificates as revoked, one as deleted and then hit
the save button and wait whether everything went correctly.

CLI

There will be new options in ``cert-find``. Options which can specify
who owns the certificates which we want to find. For instance to be
possible to find all certificates for users 'cert_user', and the same
for hosts and services, ``--users``, ``--hosts``, ``--services``. Then
``--certificate`` option for searching certificates by inserting base64
blob. The last change in this command is that it will not return any
issuer in case that the certificate is not issued by our CA.

+-----------+---------------------------------------------------------+
| Command   | Options                                                 |
+===========+=========================================================+
| cert-find | --users=[ multivalued ] --hosts=[ multivalued ]         |
|           | --services=[ multivalued ] --certificate=STR            |
+-----------+---------------------------------------------------------+



How to Test
-----------

TBD



Test Plan
---------

TBD