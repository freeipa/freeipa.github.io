Overview
--------

IPA currently uses XML-RPC to communicate with the server. The Web UI
uses JSON-RPC.

Using JSON-RPC also in the IPA client will allow us to include
additional information in errors, such as instructions or log messages.

Also, switching the protocol will allow us to assume the latest client
version when the client doesn't send a version option (see `discussion
on
freeipa-devel <http://www.redhat.com/archives/freeipa-devel/2012-December/msg00164.html>`__).

This RFE is only for the ``ipa`` client. Other (especially non-Python)
tools such as ipa-join will continue to use XML-RPC. The features that
JSON-RPC will allow aren't essential for these tools. The features will
simply not be available over XML-RPC.



Use Cases
---------

N/A

Design
------

Two options will be added to default.conf:

-  rpc_protocol
-  jsonrpc_uri

If jsonrpc_uri is not given, it will be derived from xmlrpc_uri by
replacing "/xml" with "/json".

If rpc_protocol is set to "jsonrpc" (the default), the client will use
JSON-RPC to talk to the server. If it's set to "xmlrpc", the client will
use XML-RPC.

Implementation
--------------

No additional requirements or changes discovered during the
implementation phase.

.. _feature_managment_cli:

Feature Managment: CLI
----------------------

Developers can use the ``-e rpc_protocol=xmlrpc`` option to ``ipa`` to
use the old protocol.



Major configuration options and enablement
------------------------------------------

Two new env variables, see Design.

Replication
-----------

N/A



Updates and Upgrades
--------------------

N/A

Dependencies
------------

N/A



External Impact
---------------

N/A
