LDAP_Connection_Management_Refactoring
======================================

Overview
--------

Improvements in managing the LDAP connections have been made as a part o
the 4.5 refactoring effort. This document describes how to properly use
LDAP connections now and what changes have been made.

Historically, LDAP connections to Directory Server (DS) have been
established adhoc and there was duplicated code scattered around the
code base. Some connections weren't closed properly. There were multiple
ways to connect to LDAP.



Use Cases
---------



In framework
----------------------------------------------------------------------------------------------

When working with a framework, you can use api.Backend.ldap2.

You do not need to worry about connects and disconnects. These are
handled in an upper abstraction layer along with setting proper size and
time limits.



Local server
----------------------------------------------------------------------------------------------

When you're writing a script that needs to connect to a local server,
you need to explicitly connect and disconnect from the LDAP server. The
following code will connect with LDAPI and use external bind:

.. code:: python

   from ipalib import api

   api.Backend.ldap2.connect()
   try:
       # Use api.Backend.ldap2
       # ...
   finally:
       api.Backend.ldap2.disconnect()



Remove server
----------------------------------------------------------------------------------------------

When connecting to a remote server, use LDAPClient. An example with
LDAPClient and simple_bind():

.. code:: python

   from ipapython import ipaldap

   ldap_uri = ipaldap.get_ldap_uri(host, port, ...)
   with ipaldap.LDAPClient(ldap_uri) as conn:
       conn.simple_bind(bind_dn, bind_pw)
       conn.unbind()



Proper usage
------------

api.Backend.ldap2
----------------------------------------------------------------------------------------------

-  Use whenever accessing local directory server
-  Connect only at start and end of script
-  Connection is automatically re-established if DS is restarted
-  Uses ldapi



Restarting Directory Server
----------------------------------------------------------------------------------------------

-  Use start(), stop() and restart() methods of DsInstance

   -  These already manage the api.Backend.ldap2 connection

-  Using knownservices to restart a DS instance is deprecated and
   requires a manual dis/re/connect of api.Backend.ldap2



Adhoc connections
----------------------------------------------------------------------------------------------

api.Backend.ldap2 should be always preferred if connecting to the local
server. In the cases, where you need to establish a connection to other
server, you can use an adhoc connection.

-  Use get_ldap_uri() to generate ldap_uri from protocol, host, port,
   ...
-  Initialize the connection with LDAPClient
-  Make sure to properly close the connection when it is no longer used

ldap2
----------------------------------------------------------------------------------------------

-  No time_limit and size_limit by default
-  Set time_limit=None and size_limit=None to respect values in ipa
   config file

Changes
-------

The following improvements have been made as a part of the 4.5
refactoring effort.



Merge IPAdmin to LDAPClient
----------------------------------------------------------------------------------------------

IPAdmin was merged into LDAPClient and removed completely. The following
changes were made in ipaldap:

-  Removed wait and timeout for establishing a connection (in IPAdmin).

   -  This code was obsolete, because there is a wait when the directory
      server is start / restarted.

-  Used simple_bind(), external_bind() and gssapi_bind() instead of
   their do\_\* alternatives, which were removed.
-  Removed user name from external_bind() and always set it to effective
   user name.
-  Removed obsolete and unused IPAdmin properties

   -  ldapi
   -  realm
   -  suffixes

-  Moved the following IPAdmin properties to LDAPClient:

   -  host (automatically parsed from ldap_uri)
   -  port (automatically parsed from ldap_uri)
   -  cacert (IPAdmin) -> \_cacert (LDAPClient)

-  Added the following options to LDAPClient constructor:

   -  cacert
   -  sasl_nocanon

-  Created get_ldap_uri() function to determine ldap_uri from former
   IPAdmin constructor arguments
-  Replaced all occurrences of IPAdmin with LDAPClient

   -  get_ldap_uri() is used to construct ldap_uri
   -  LDAPClient object is initialized with the ldap_uri



Use ldapi when connecting to localhost
----------------------------------------------------------------------------------------------

When a local connection is established, it should use ldapi whenever
possible. api.Backend.ldap2 was configured to use ldapi. Some adhoc
connections were replaced with api.Backend.ldap2.

ldap2 default time and size limit was set to unlimited. Limit for use in
rpc still respects the ipa config file.



Use a shared LDAP connection in installers and install tools
----------------------------------------------------------------------------------------------

In installers and install tools, an ldap connection (if needed) should
be established at the start of the script and properly closed at the end
of the script. When a directory server is started, stopped or restarted,
the connection should dis/re/connect accordingly.

-  api.Backend.ldap2 was used across installers and install tools for
   local LDAP connection
-  Directory Server installation required the following modifications:

   -  Enabled ldapi and configured autobind for root after instance
      creation
   -  Overriden start, stop and restart method of DsInstance to also
      dis/re/connect the api.Backend.ldap2.connection



Future effors
-------------

Not all the issues with LDAP connection management were removed due to
time and scope constraints of this effort.

Future changes may include the following:

-  Replaces instance creations of ldap2() with LDAPClient
-  Remove all unnecessary adhoc connections
-  Consider using a single adhoc connection to a replica during
   install/promote instead of repeatedly connecting.
-  Remove modify_s()