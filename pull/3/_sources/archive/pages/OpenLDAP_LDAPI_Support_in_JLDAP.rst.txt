Overview
========

A synchronization tool will communicate with the DS instances used by
IPA and Samba and synchronize their contents. At least initially the
sync tool will be implemented in Java. In the final implementation it
could be replaced by native code if necessary.

Communicating with IPA's DS instance can be via regular/secure LDAP
protocol. Communicating with Samba's DS instance requires the use of
LDAPI. The DS instances are identified by their URLs.

A regular LDAP URL will look like the following:

::

   ldap://localhost:389

A secure LDAP URL will look like the following:

::

   ldaps://localhost:636

An LDAPI URL will look like the following:

::

   ldapi://%2Fusr%2Flocal%2Fsamba%2Fprivate%2Fldap%2Fldapi

The LDAPI URL contains a path to a Unix domain socket file
/usr/local/samba/private/ldap/ldapi.

The `OpenLDAP JLDAP <http://www.openldap.org/jldap/>`__ library provides
a Java API to communicate with LDAP servers. However, the library
currently only supports regular LDAP and secure LDAP protocol schemes.
It currently does not support LDAPI.

The `junixsocket <http://code.google.com/p/junixsocket/>`__ library
provides a Java API to communicate via Unix domain socket. This library
could be used in JLDAP to provide LDAPI support. There are also other
libraries such as `JUDS <http://code.google.com/p/juds/>`__ but
junixsocket's API is based on Java Socket API which is used by JLDAP.

.. _current_code:

Current Code
============

.. _ietf_api:

IETF API
--------

Using the current IETF API, an LDAP connection can be created by
providing an LDAP or LDAPS URL containing the host name and port number
of the LDAP server:

::

   LDAPUrl url = new LDAPUrl("ldap://localhost:389");
   LDAPConnection connection = LDAPConnection();
   connection.connect(url.getHost(), url.getPort());

.. _ldap_url:

LDAP URL
--------

The com.novell.ldap.LDAPUrl class currently only recognizes ldap and
ldaps protocol schemes.

::

   private void parseURL(String url) {

       if (url.substring(scanStart, scanStart + 7).equalsIgnoreCase("ldap://")) {
           scanStart += 7;
           port = LDAPConnection.DEFAULT_PORT;

       } else if (url.substring(scanStart, scanStart + 8).equalsIgnoreCase("ldaps://")) {
           scheme = "ldaps";
           scanStart += 8;
           port = LDAPConnection.DEFAULT_SSL_PORT;

       } else {
           throw new MalformedURLException("LDAPUrl: URL scheme is not ldap");
       }
   }

.. _ldap_connection:

LDAP Connection
---------------

The com.novell.ldap.Connection class creates the socket using Java
Socket API by providing the host name and port number.

::

   private void connect(String host, int port, int semaphoreId) {

       if(mySocketFactory != null) {
           socket = mySocketFactory.createSocket(host, port);

       } else {
           socket = new Socket(host, port);
       }
   }

.. _proposed_solution:

Proposed Solution
=================

.. _ietf_api_1:

IETF API
--------

The IETF API should be modified to allow creating a connection by
providing an LDAPI URL which contains a path to the socket file:

::

   LDAPUrl url = new LDAPUrl("ldapi://%2Fusr%2Flocal%2Fsamba%2Fprivate%2Fldap%2Fldapi");
   LDAPConnection connection = LDAPConnection();
   connection.connect(url);

.. _ldap_url_1:

LDAP URL
--------

The com.novell.ldap.LDAPUrl should be modified to recognize the ldapi
protocol scheme.

::

   private void parseURL(String url) {

       if (url.substring(scanStart, scanStart + 8).equalsIgnoreCase("ldapi://")) {
           scheme = "ldapi";
           scanStart += 8;

           path = url.substring(scanStart, scanEnd);
           return;
       }
   }

.. _ldap_connection_1:

LDAP Connection
---------------

A new connect() method should to be added into
com.novell.ldap.Connection to create a socket by providing a path to the
socket file.

::

   private void connect(File path) {

       AFUNIXSocket unixSocket = AFUNIXSocket.newInstance();
       unixSocket.connect(new AFUNIXSocketAddress(path));

       socket = unixSocket;
   }

References
==========

-  `Using LDAP Over IPC
   Mechanisms <http://tools.ietf.org/html/draft-chu-ldap-ldapi-00>`__
-  `Running
   slapd <http://www.openldap.org/doc/admin24/runningslapd.html>`__
-  `Enabling
   LDAPI <http://www.redhat.com/docs/manuals/dir-server/8.1/admin/ldapi-enabling.html>`__
-  `OpenLDAP Java LDAP <http://www.openldap.org/jldap/>`__
-  `junixsocket <http://code.google.com/p/junixsocket/>`__

`Category:Obsolete <Category:Obsolete>`__
