This is current as of freeiPA 4.6.1, October 2017.

History
-------

First, some history.

With IPAv1 we didn't have a cohesive idea on how to structure the code
so it was grown organically. There was a WSGI server piece and some
discrete client commands (ipa-adduser, ipa-deluser, etc). A webUI was
provided by TurboGears. A client and server installer were provided to
aid in configuration.

This generally worked but there were some significant deficiencies: -
Every command was separate with little to no code-sharing - We lacked
sessions so every request was very expensive with a full Kerberos
handshake plus TLS handshake - Input validation was done in both the
client and server and also in the webUI so lots of code was duplicated
and it was difficult to be in sync

It was not all bad, some users deployed production servers using v1.
Despite the warts it was still better than many hand-rolled identity
management systems. With v2 we planned to address the major problems
with v1. For details on this see `V2_Designs <V2_Designs>`__ and
`V2_Proposals <V2_Proposals>`__

From an architecture perspective the whole client/server model was
rewritten from scratch. In order to ensure good client interaction it
was decided that the client and server would run the same input
validation, including the webUI as a client. It was important that the
server re-check input in case someone used the API directly. It would be
some time before any discussion of "support" for the XML-RPC API would
happen but this ensured that there would be fewer surprises.

This implementation centered around a framework that defined the basic
rules for a plugable architecture:

- A base set of argument and option types including validators - A
defintion of inputs and outputs - A plugin execution structure that
consisted of pre-operation, execution, post-operation and exception
handling - Centralized error handling with control over public and
private exceptions to prevent information disclosure - Plugin
extensibility to provide a way to extend an existing plugin without
having to override it - A set of CRUD rules

The long-term plan was always to break ipalib out as a separate package
to live alongside IPA. We gave up on this eventually in order to speed
development.

The basic idea of ipalib is that both the client and server have exactly
the same set of options and arguments which means the same validations
could be run. The difference being that the client would call the
forward() method when the validations were complete and the server would
call execute().

Data returned was defined in a more hand-wavy manner by using Output
parameters. These generally consisted of returning the dn of the object
created/modified/deleted, an optional exception, an optional dict of the
entry a summary message and in the case of find methods, a count.

The pre and post operation routines would allow the user to modify the
data before or after execution. The execution method would commit any
changes to LDAP. The exception handlers would let one customize what
gets raised.

This basic framework lasted through v3. freeIPA v4 brought many changes
including a thin client which eliminated the client-side validation and
moved the plugins out of ipalib into ipaserver.

Overview
--------

At a high level the tree can be broken into four pieces: - server -
client - common - tests

Server
------

The server-side specific portions consist of:

daemons - 380-ds plugins, the OTP daemon, etc. util - common functions
used by daemons ipaserver - the RPC handler and plugins init - systemd
and tmpfiles configuration files install - common install items used by
both client and server though I include it here because it is heavily
server-based ipaplatform - used primarily by server now but abstracts
distro-level options asn1 - auto-generated ASN1. for IPA keytab
generation

Client
------

client - client installers ipaclient - client installer library

Common
------

ipalib - IPA framework ipapython - common functions beween server and
client that fall outside ipalib

Tests
-----

ipatests - unit and integration tests

Other
-----

po - translations. We use zanata doc - some oldish documentation on
using ipalib and writing plugins contrib - handy tools that are not
shipped pypi - used for generating PyPi packages

An important file that lives on the top-level is freeipa.spec.in. This
is the base spec file that is used to generated rpms in-tree and should
match any downstream bulids (Fedora and RHEL). Keeping downstream builds
as close as possible makes identifying differences on rebased
substantially easier.
