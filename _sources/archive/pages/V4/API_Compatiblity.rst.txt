API_Compatiblity
================

Overview
--------

The purpose of this feature is to make the IPA CLI tool (IPA client) as
simple as possible and develop a way to maintain compatibility among
various versions of servers and clients.



Use Cases
---------

-  be able to use a new IPA client to run commands on an old IPA server
-  be able to use an old IPA client to run commands on a new IPA server
   after incompatible changes were made to the commands

Design
------



Backward compatibility with old servers
----------------------------------------------------------------------------------------------

Current server code check client version and rejects requests from
clients newer than the server. In order to allow the client to talk to
old servers, interface definitions for supported old API versions will
be bundled with the client.

The client will use interface definition appropriate for the API version
on the server - if it has interface definition for the exact API
version, it will use that, otherwise it will use the closest lower API
version interface definition available.

Interface definitions for the following API versions will be included:

-  2.49 (RHEL and CentOS 6.5+)
-  2.114 (Fedora 22)
-  2.156 (RHEL and CentOS 7.2, Fedora 23)
-  2.164 (Fedora 24)

When an API version becomes unsupported, its interface definition can be
removed from the client.

Additionaly, the client will no longer send argument default values with
each request, in order not to trigger an invocation error because of
unknown arguments.



Forward compatibility
----------------------------------------------------------------------------------------------

Current server is backward compatible with old clients, except when an
incompatible change is made to the API. To allow backward incompatible
changes in commands, it will be made possible to provide additional
versions of a command.

In order to avoid having to deal with API compatibility on a per-version
basis in the future, a thin client approach will be used for the client.
The server will provide API schema describing commands, their arguments
and output fields available over its API. The client will fetch the
schema from the server and will dynamically generate and provide the
commands to user.



Versioned commands
^^^^^^^^^^^^^^^^^^

The client will use the highest version of command known to it at build
time by default, or version 1 for unknown commands. Users will be able
to explicitly specify the version of command in each request. The server
will use version 1 by default, in order not to break old and 3rd party
clients.



Thin client
^^^^^^^^^^^

Server and client side plugins needs to be split to allow separation.
Module ``ipalib.plugins`` will be split into ``ipaserver.plugins`` and
``ipaclient.plugins``.

On the server, API schema will be generated from modules and commands in
``ipaserver.plugins``. This schema will be provided to the client.
Documentation for plugins and commands will be part of the schema.

On client, proxy objects of server side plugins will be created using
the schema fetched from server. Client will be then able to display
command signature and help, perform basic validation of provided
arguments (data type check), forward the command call to the server and
display returned result. Also client side plugins will be available and
usually will preprocess user input, call one or more of server side
plugins, and possibly postprocess the result.

Due to security considerations there can't be executable code in
metadata. To allow commands to have arguments with dynamic default
values separate server command to retrieve them will be provided.

For the same reason validation of arguments that require normalization
or rely on server-side information will be done on the server and
clients must send such arguments as plain strings.

When type of an argument is unknown to the client it must be sent as
plain string. Server will convert and validate the value.

Caching
^^^^^^^

IPA client can cache downloaded API schema to reduce traffic and reduce
start up time. IPA client can send fingerprints of known schemata on API
initialization. When new API schema is available server will return the
full schema. When no new API schema is available server will add warning
to the response containing the current API version and schema
fingerprint.

API schema fingerprint is calculated on the server and must not change
for given schema. Client can't interpret or calculate the fingerprint.

Implementation
--------------



Backward compatibility with old servers
----------------------------------------------------------------------------------------------

The interface definitions for old API versions are bundled in
``ipaclient.remote_plugins.compat.``\ *``major``*\ ``_``\ *``minor``*
Python packages. They look like normal command and object plugin
definitions, except they contain no code, only parameter definitions.



Forward compatibility
----------------------------------------------------------------------------------------------



Versioned commands
^^^^^^^^^^^^^^^^^^

Command version is specified as part of the command name throughout the
framework, e.g. ``user-show/1``. If not specified, the default version
is used.

In order to add new versions of a command, use the following pattern:

| ``class MyCommandBase(Command):``
| ``    # put param definitions common to both versions here``
| ``    ...``
| ``@register()``
| ``class my_command(MyCommandBase):``
| ``    # this is the current and main version of the command``
| ``    version = '2'``
| ``    ...``
| ``@register()``
| ``class my_command_1(MyCommandBase):``
| ``    # this is the old compatibility version 1``
| ``    name = 'my_command'``
| ``    ...``



Thin client
^^^^^^^^^^^

TBD



Caching
^^^^^^^

TBD



Feature Management
------------------

UI

Not applicable - UI currently uses ``json_metadata`` API call to
retrieve information about objects, commands and parameters from server.
It's reflecting current version and changing this is not in a scope of
this design.

CLI

TBD

Configuration
----------------------------------------------------------------------------------------------

Client
^^^^^^

TBD

Server
^^^^^^

No new configuration.

Upgrade
-------

Not applicable - There is no change to the LDAP schema nor the stored
data.



How to Use
----------



Backward compatibility with old servers
----------------------------------------------------------------------------------------------

The ``ipa`` command line tool will now work on new clients enrolled
against old server:

| ``client$ rpm -q freeipa-client``
| ``freeipa-client-``\ **``4.4.1``**\ ``-1.fc25.x86_64``
| ``client$ ipa ping``
| ``------------------------------------------``
| ``IPA server version ``\ **``3.0.0``**\ ``. API version 2.49``
| ``------------------------------------------``

On clients without this feature, this would fail:

| ``client$ rpm -q freeipa-client``
| ``freeipa-client-``\ **``4.3.2``**\ ``-2.fc24.x86_64``
| ``client$ ipa ping``
| ``ipa: ERROR: 2.164 client incompatible with 2.49 server at 'https://ipa.example.com/ipa/xml'``



Forward compatibility
----------------------------------------------------------------------------------------------



Versioned commands
^^^^^^^^^^^^^^^^^^

New client will request the highest available version of a command by
default:

::

   | ``client$ ipa -v ``\ **``ping``**
   | ``ipa: INFO: trying https://ipa.example.com/ipa/session/json``
   | ``ipa: INFO: Forwarding '``\ **``ping/1``**\ ``' to server 'https://ipa.example.com/ipa/session/json'``
   | ``------------------------------------------``
   | ``IPA server version 4.4.1. API version 2.212``
   | ``------------------------------------------``

It is possible to explicitly request a specific command version instead:

::

   | ``client$ ipa -v ``\ **``ping/1``**
   | ``ipa: INFO: trying https://ipa.example.com/ipa/session/json``
   | ``ipa: INFO: Forwarding '``\ **``ping/1``**\ ``' to server 'https://ipa.example.com/ipa/session/json'``
   | ``------------------------------------------``
   | ``IPA server version 4.4.1. API version 2.212``
   | ``------------------------------------------``

Requesting an unknown version of a command will result in an error:

::

   | ``client$ ipa -v ``\ **``ping/2``**
   | ``ipa: INFO: trying https://ipa.example.com/ipa/session/json``
   | ``ipa: INFO: Forwarding '``\ **``ping/2``**\ ``' to server 'https://ipa.example.com/ipa/session/json'``
   | ``ipa: ERROR: unknown command '``\ **``ping/2``**\ ``'``



Thin client
^^^^^^^^^^^

Thin client is transparent to the user, i.e. everything will work the
same as on clients without this feature.

It is possible to inspect the API schema using the new API introspection
commands:

::

   | ``client$ ipa command-show hostgroup-add``
   | ``  Name: hostgroup_add``
   | ``  Version: 1``
   | ``  Full name: hostgroup_add/1``
   | ``  Documentation: Add a new hostgroup.``
   | ``  Help topic: hostgroup/1``
   | ``  Method of: hostgroup/1``
   | ``  Method name: add``
   | ``client$ ipa param-find hostgroup-add``
   | ``  Name: cn``
   | ``  Documentation: Name of host-group``
   | ``  Type: str``
   | ``  CLI name: hostgroup_name``
   | ``  Label: Host-group``
   | ``  Convert on server: True``
   | ``  Name: description``
   | ``  Documentation: A description of this host-group``
   | ``  Type: str``
   | ``  Required: False``
   | ``  CLI name: desc``
   | ``  Label: Description``
   | ``  Name: setattr``
   | ``  Documentation: Set an attribute to a name/value pair. Format is attr=value.``
   | ``For multi-valued attributes, the command replaces the values already present.``
   | ``  Exclude from: webui``
   | ``  Type: str``
   | ``  Required: False``
   | ``  Multi-value: True``
   | ``  CLI name: setattr``
   | ``  Name: addattr``
   | ``  Documentation: Add an attribute/value pair. Format is attr=value. The attribute``
   | ``must be part of the schema.``
   | ``  Exclude from: webui``
   | ``  Type: str``
   | ``  Required: False``
   | ``  Multi-value: True``
   | ``  CLI name: addattr``
   | ``  Name: all``
   | ``  Documentation: Retrieve and print all attributes from the server. Affects command output.``
   | ``  Exclude from: webui``
   | ``  Type: bool``
   | ``  CLI name: all``
   | ``  Default: False``
   | ``  Positional argument: False``
   | ``  Name: raw``
   | ``  Documentation: Print entries as stored on the server. Only affects output format.``
   | ``  Exclude from: webui``
   | ``  Type: bool``
   | ``  CLI name: raw``
   | ``  Default: False``
   | ``  Positional argument: False``
   | ``  Name: no_members``
   | ``  Documentation: Suppress processing of membership attributes.``
   | ``  Exclude from: webui``
   | ``  Type: bool``
   | ``  Default: False``
   | ``  Positional argument: False``
   | ``----------------------------``
   | ``Number of entries returned 7``
   | ``----------------------------``
   | ``client$ ipa class-show hostgroup``
   | ``  Name: hostgroup``
   | ``  Version: 1``
   | ``  Full name: hostgroup/1``
   | ``client$ ipa param-find hostgroup``
   | ``  Name: cn``
   | ``  Documentation: Name of host-group``
   | ``  Type: str``
   | ``  Label: Host-group``
   | ``  Name: description``
   | ``  Documentation: A description of this host-group``
   | ``  Type: str``
   | ``  Required: False``
   | ``  Label: Description``
   | ``  Name: member_host``
   | ``  Type: str``
   | ``  Required: False``
   | ``  Label: Member hosts``
   | ``  Name: member_hostgroup``
   | ``  Type: str``
   | ``  Required: False``
   | ``  Label: Member host-groups``
   | ``  Name: memberof_hostgroup``
   | ``  Type: str``
   | ``  Required: False``
   | ``  Label: Member of host-groups``
   | ``  Name: memberof_netgroup``
   | ``  Type: str``
   | ``  Required: False``
   | ``  Label: Member of netgroups``
   | ``  Name: memberof_sudorule``
   | ``  Type: str``
   | ``  Required: False``
   | ``  Label: Member of Sudo rule``
   | ``  Name: memberof_hbacrule``
   | ``  Type: str``
   | ``  Required: False``
   | ``  Label: Member of HBAC rule``
   | ``  Name: memberindirect_host``
   | ``  Type: str``
   | ``  Required: False``
   | ``  Label: Indirect Member hosts``
   | ``  Name: memberindirect_hostgroup``
   | ``  Type: str``
   | ``  Required: False``
   | ``  Label: Indirect Member host-groups``
   | ``  Name: memberofindirect_hostgroup``
   | ``  Type: str``
   | ``  Required: False``
   | ``  Label: Indirect Member of host-group``
   | ``  Name: memberofindirect_sudorule``
   | ``  Type: str``
   | ``  Required: False``
   | ``  Label: Indirect Member of Sudo rule``
   | ``  Name: memberofindirect_hbacrule``
   | ``  Type: str``
   | ``  Required: False``
   | ``  Label: Indirect Member of HBAC rule``
   | ``-----------------------------``
   | ``Number of entries returned 13``
   | ``-----------------------------``
   | ``client$ ipa output-find hostgroup-add``
   | ``  Name: summary``
   | ``  Documentation: User-friendly description of action performed``
   | ``  Type: str``
   | ``  Required: False``
   | ``  Name: result``
   | ``  Type: dict``
   | ``  Name: value``
   | ``  Documentation: The primary_key value of the entry, e.g. 'jdoe' for a user``
   | ``  Type: str``
   | ``----------------------------``
   | ``Number of entries returned 3``
   | ``----------------------------``
   | ``client$ ipa topic-show hostgroup``
   | ``  Name: hostgroup``
   | ``  Version: 1``
   | ``  Full name: hostgroup/1``
   | ``  Documentation: Groups of hosts.``
   | ``Manage groups of hosts. This is useful for applying access control to a``
   | ``number of hosts by using Host-based Access Control.``
   | ``EXAMPLES:``
   | `` Add a new host group:``
   | ``   ipa hostgroup-add --desc="Baltimore hosts" baltimore``
   | `` Add another new host group:``
   | ``   ipa hostgroup-add --desc="Maryland hosts" maryland``
   | `` Add members to the hostgroup (using Bash brace expansion):``
   | ``   ipa hostgroup-add-member --hosts={box1,box2,box3} baltimore``
   | `` Add a hostgroup as a member of another hostgroup:``
   | ``   ipa hostgroup-add-member --hostgroups=baltimore maryland``
   | `` Remove a host from the hostgroup:``
   | ``   ipa hostgroup-remove-member --hosts=box2 baltimore``
   | `` Display a host group:``
   | ``   ipa hostgroup-show baltimore``
   | `` Delete a hostgroup:``
   | ``   ipa hostgroup-del baltimore``



Caching
^^^^^^^

API schema is cached on the client for an hour. During this interval,
the client will not try to contact the server about the schema:

| ``$ ipa -v ping``
| ``ipa: INFO: trying https://ipa.example.com/ipa/session/json``
| ``ipa: INFO: Forwarding 'ping/1' to json server 'https://ipa.example.com/ipa/session/json'``
| ``-------------------------------------------------------------------``
| ``IPA server version 4.4.1. API version 2.212``
| ``-------------------------------------------------------------------``

To refresh the cache (e.g. if you want the client to immediately use an
up-to-date API schema after server upgrade), use the
``force_schema_check`` option:

::

   | ``$ ipa -v ``\ **``-e``\ ````\ ``force_schema_check=1``**\ `` ping``
   | ``ipa: INFO: trying https://ipa.example.com/ipa/session/json``
   | **``ipa:``\ ````\ ``INFO:``\ ````\ ``Forwarding``\ ````\ ``'schema'``\ ````\ ``to``\ ````\ ``json``\ ````\ ``server``\ ````\ ``'https://ipa.example.com/ipa/session/json'``**
   | ``ipa: INFO: trying https://ipa.example.com/ipa/session/json``
   | ``ipa: INFO: Forwarding 'ping/1' to json server 'https://ipa.example.com/ipa/session/json'``
   | ``-------------------------------------------------------------------``
   | ``IPA server version 4.4.1. API version 2.212``
   | ``-------------------------------------------------------------------``



Test Plan
---------



Regression testing
----------------------------------------------------------------------------------------------

New IPA client (resp. server) MUST behave exactly the same as the old
IPA client (resp. server) when communicating with the old IPA server
(resp. client).



Feature testing
----------------------------------------------------------------------------------------------

TBD



Test Plan
----------------------------------------------------------------------------------------------

`Thin Client V4.4 test plan <V4/Thin_Client/Test_Plan>`__