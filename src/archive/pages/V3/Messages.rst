.. _overview_2732:

Overview (`#2732 <https://fedorahosted.org/freeipa/ticket/2732>`__)
-------------------------------------------------------------------

Currently the only way to display a warning on client is to raise
NonFatalError. This is not particularly good, as it mutes normal command
output and only one warning can be displayed at a time.

Provide a mechanism for displaying arbitrary number of warnings and
other messages on clients along with the normal command output.

.. _additional_considersations:

Additional considersations
~~~~~~~~~~~~~~~~~~~~~~~~~~

The client validates the response it receives from the server. If it
gets any extra items, the validation fails. Relaxing our validation is
not an option. To support older clients, we need a mechanism for
backwards-incompatible extensions to the API.

Design
------

Backend
~~~~~~~

Introduce a "capability" mechanism for backwards-incompatible API
changes. The first capability will be "messages": a client with this
capability can get an extra key in the response dictionary, "messages",
which contains a list of warnings and other messages. Each message will
be a dict with the following keys:

| `` {``
| ``   'type': 'warning',``
| ``   'name': 'BadIdea',``
| ``   'message': "What you're doing is a bad idea",``
| ``   'code': 12345,``
| `` }``

The "type" will be "debug", "info", "warning", or "error". Numeric codes
for messages will be in the range 10000-10999, so that they don't clash
with error codes.

Capabilities are determined by API version. The version is linear; it is
not possible for a client to support a custom subset of capabilities.

If a JSON-RPC client does not send an API version number, we will assume
this is a testing client (such as a curl from the command line). Such a
client is assumed to have all capabilities, but it will always receive a
warning saying that forward compatibility is not guaranteed if the
version number is not sent. The warning name and number will be
VersionMissing, 13001.

If a XML-RPC client does not send an API version number, we will assume
this is an old client (the version just before capabilities are
introduced). This is needed to preserve compatibility with old IPA
clients affected by issue
`#3294 <https://fedorahosted.org/freeipa/ticket/3294>`__.

Frontend
~~~~~~~~

In the CLI, messages will be logged to stderr with the appropriate
severity for their type. Debug messages will not be shown unless
--verbose is given. Example (from
`#2563 <https://fedorahosted.org/freeipa/ticket/2563>`__):

| `` # ipa dnsrecord-add --ttl=555 localnet r1 --txt-rec=TEST``
| `` ipa: WARNING: It's not possible to have two records with same name and different TTL values.``
| ``   Record name: r1``
| ``   Time to live: 555``
| ``   A record: 1.2.3.4``
| ``   TXT record: TEST``

The Web UI will display warnings and above in a modal message box. In
the future, another notification mechanism may be added.

.. _use_cases10f:

Use Cases
---------

None; there are no user-visible changes.

Implementation
--------------

Capabilities will be recorded in API.txt. When a new one is added, the
API version must be incremented.

All Commands will be updated to explicitly list 'version?' as one of
their options in API.txt. (All commands already take it.)

A missing version option will be set as part of validation/filling in
defaults, so execute() will always get it.

In IPA code, messages will be objects. The classes will live in a new
module, ipalib.messages, modeled after ipalib.errors, and will subclass
warnings.Warning from the standard library.

Helper methods will be available for checking a capability and for
adding a message to the result:

``   add_message(version, result, BadIdeaWarning())``

will be equivalent to:

| ``   if client_has_capability(version, 'messages'):``
| ``       result.setdefault('messages', []).append(BadIdeaWarning())``

.. _rfe_authors:

RFE authors
-----------

Proposed by `jcholast <User:jcholast>`__, document by
`Pviktorin <User:Pviktorin>`__
