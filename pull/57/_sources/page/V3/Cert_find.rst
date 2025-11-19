Cert_find
=========

\__NOTOC_\_

Overview
========

The original CA integration didn't specify a mechanism to search for
existing certificates. It only specified a way to retrieve a single cert
based on serial number.

This RFE is being worked as https://fedorahosted.org/freeipa/ticket/2528



Use Cases
=========

The pki cert-find output can be used to compare the output against the
ipa cert-find output. They should be the same with only formatting
differences.

-  Bare find: return all certificates
-  Short subject: search certs by short hostname. Using the IPA server
   hostname should return > 1
-  Exact subject: search certs by short hostname and --exactly set.
   Should return nothing
-  Exact subject: search certs by FQDN and --exactly set. Should return
   correct list
-  Using the full list as a starting point, try to bisect the list by
   serial number to be sure that the limits are applied
-  Revoke a certificate then search by revocation reason.
-  Search for invalid revocation reasons (letters, > 10)
-  Search for unused revocation reasons, should return nothing
-  Search by notbefore/notafter dates
-  Search by issuedon dates

One thing explicitly not done is comparing begin and end dates when
searching. This is left as an exercise for the user.

Design
======



Dogtag capabilities
-------------------

This will be implemented in dogtag 10+ only, and using only the new
RESTful interface.

The interface provides 32 different search options:

-  certTypeSecureEmail <on|off> : Certifiate Type: Secure Email
-  certTypeSSLClient <on|off> : Certifiate Type: SSL Client
-  certTypeSSLServer <on|off> : Certifiate Type: SSL Server
-  certTypeSubEmailCA <on|off> : Certifiate type: Subject Email CA
-  certTypeSubSSLCA <on|off> : Certificate type: Subject SSL CA
-  country : Subject's country
-  email : Subject's email address
-  input : File containing the search constraints
-  issuedBy : Issued by
-  issuedOn : Date issued
-  locality : Subject's locality
-  matchExactly : Match exactly with the details provided
-  maxSerialNumber : Maximum serial number
-  minSerialNumber : Minimum serial number
-  name : Subject's common name
-  org : Subject's organization
-  orgUnit : Subject's organization unit
-  revocationReason : Reason for revocation
-  revokedBy : Certificate revoked by
-  revokedOnFrom : Revoked on or after this date
-  revokedOnTo : Revoked on or before this date
-  size : Page size
-  start : Page start
-  state : Subject's state
-  uid : Subject's userid
-  validityCount : Validity count
-  validityOperation : Validity operation: "<=" or ">="
-  validityUnit : Validity unit
-  validNotAfterFrom : Valid not after start date
-  validNotAfterTo : Valid not after end date
-  validNotBeforeFrom : Valid not before start date
-  validNotBeforeTo : Valid not before end date



What IPA will provide
---------------------

-  Serial number range (min/max/none)
-  Issued On
-  revoked certificates
-  subject
-  valid not after from
-  valid not after to
-  valid not before from
-  valid not before to

Some of these can be overlapping, for example you can search for a
revoked certificate for a given host within a serial number range.

Output
------

In the initial implementation and to save cycles we'll just display what
dogtag returns:

-  Serial Number (dec and hex)
-  Subject DN
-  Status

In order to display more details we'd need to look up each serial number
one at a time which would be agonizingly slow.

Like all IPA commands, a bare find command should return all
certificates. In this case we have no control over the search limit. We
may still want to impose a limit on the amount of data returned to the
client.



Access Control
--------------

Issued certificates are considered public so no additional
roles/permissions are planned. As with other commands one will need to
be an authenticated IPA user to execute the command.

The interface is available internally on port 8080 which is not a port
we advertise as needing to be open generally.



Dogtag details
--------------

There is a command-line tool, pki, which seems modeled on the ipa tool.
The cert-find command is what we're interested in.

The interface is not currently documented but it is easy to snoop on the
request using ssltap.

In order to make a request, POST to /ca/rest/certs/search

The content of the request is XML, as is the content of the response.

I snooped using two terminals.

terminal 1: ``ssltap localhost:8080``

terminal 2: ``pki cert-find -p 1924``

The requests are made and received in XML.

There are a series of boolean elements that control how the search is
done, in addition to optional extra elements depending on what you are
searching for (subject, revocation reason, etc).



Current bugs
------------

Date parsing is broken. https://fedorahosted.org/pki/ticket/416
Certificate dates are not returned in find output.
https://fedorahosted.org/pki/ticket/498

Implementation
==============

Any additional requirements or changes discovered during the
implementation phase.



Feature Management
==================

UI

The UI should be similar to the command line. It is unclear how date
formatting will be handled but bare text boxes would be usable for a
first attempt.

Unlike other object-find pages there are some additional ways of
narrowing down the terms, by adding dates, serial numbers, etc. The
searchs are all AND, there is no way to do an OR (limitation of CS).

CLI

Overview of the CLI commands



Major configuration options and enablement
==========================================

Nothing beyond cert-find. Command only works with dogtag CA backend.

Replication
===========

None



Updates and Upgrades
====================

None

Dependencies
============

Date parsing works in current dogtag 10 in F-18, but output currently
limited by ticket #498.



External Impact
===============

None

Future
------

As more certificate types are added this can easily be extended to
search for those types without impacting backwards compatibility. It
seems like overkill to do that now.