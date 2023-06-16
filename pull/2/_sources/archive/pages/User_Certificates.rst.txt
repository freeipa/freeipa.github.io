`Dogtag <PKI>`__ has support for Asymmetric (PKI) User Keys, but there
currently exists no way to access them from FreeIPA. Here is a staged
approach that attempts to show real value at each step without
committing to a huge development effort.

Prerequisite
============

Current Certificate calls from IPA to Dogtag are doing via the WebAPI,
which is not really meant to be used programatically. The following work
should likely be done talking to the proposed REST API, which means that
there is some Java/Dogtag work to be done first.

.. _just_fetch_new_certs:

Just Fetch new Certs
====================

Since Dogtag records the certificates locally, there is, at first, no
need to modify the FreeIPA datastore to report and display user
certificates. In fact, the most essential feature is the ability to
request and issue user certificates by way of FreeIPA for consumption
outside. As such, the first step is to modify the command used to
request certificates so that user certificates are avaialable.

ipa cert-request --uid will create an X509 certificate request for a
multi-use user certificate and submit it to Dogtag. All values for the
form will be populated with values from the users record. The CSR will
be automatically approved and returned to the client.

.. _fetch_existing_certs:

Fetch existing Certs
====================

The second step is to extend ipa cert-show –uid to fetch a user
certificate from the Dogtag back end.

.. _recording_user_certs:

Recording User Certs
====================

The third step is to record the user certificate in the users record in
IPA. There is already a field for this in the standard LDIFs used by
IPA. Ideally, the user would be able to have multiple certificates, so
the certificate would, ideally, be identified by not just the users
name, but an extension to the Subject specifying, for example, what
machine that certificate is registered on. However, a first
approximation can just treat the certificates as a multi-valued
attribute.

Once the user certificates are recorded in IPA, it should be possible to
use them to do a bind. This will allow IPA to be an authentication and
authorization source for servers that only have access to PKI, not
Kerberos.

Workflow
========

One design decision that has come up multiple times is whether the
Presence of Kerberos credentials is sufficient to grant a user a PKI
certificate. This is a policy decision that should be granted to the IPA
installation.

ipa cert-status will be extended to show User CSRs that have not yet
been approved.

A new API, ipa cert-approve, will allow a user with sufficient
privileges to approve an existing CSR

one ipa cert-approve has been implemented, User CSRs will no longer be
automatically signed by default, but instead will go into a queue.
Automatic signing will be activated by:

ipa cert-policy GROUP --autoapprove=[yes|no]

.. _agent_identification:

Agent Identification
~~~~~~~~~~~~~~~~~~~~

All IPA work is currently performed by the Same User agent inside
Dogtag. This bypasses the PKI auditing framework. Host certificates will
continue to be signed by thei s agent. However, a new API will create
PKI agents and associatate them with users in IPA.

ipa cert-agent create|find|show will manage the Agents on Dogtag.

The IPA user object will contain a dogtag-user field which will default
to blank. If this value is set, the first value will be assigned as the
user agent for working with PKI.

If autoapprove is set, the default IPA agent will be used to approve
User certificates.

If autoapprove is not set, Dogtag approval processing rules will work as
they do currently, based on the ACIs on the Dogtag Dir Server.
