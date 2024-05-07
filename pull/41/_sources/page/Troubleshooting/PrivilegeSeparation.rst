PrivilegeSeparation
===================



How to debug FreeIPA privilege separation issues
================================================

Introduction
------------

FreeIPA 4.5 implements privilege separation between different components
of its management framework. To understand what changed, first let's
look at how the FreeIPA management framework is built.

The FreeIPA management framework is a web application served via
Apache's mod_wsgi module. Any user accessing the FreeIPA management
framework is supposed to have a Kerberos principal associated with the
user's account. The framework itself talks to the FreeIPA LDAP server
and needs to authenticate to it. When a user connects to the framework
over HTTPS, it either needs to authenticate with Kerberos or present a
cookie from a previous authentication. For clients that don't support
Kerberos, a special end point is available for password-based
authentication. It produces a cookie that can be presented in subsequent
requests.

For each service, the FreeIPA framework, the LDAP server Kerberos
authentication is required. In Kerberos users obtain an initial ticket
granting ticket (TGT) from a Kerberos KDC. This is used to obtain
per-service authentication tickets and is never transferred over the
network. A new, service ticket is obtained from the KDC and then
presented to that service when authentication is required. The service
ticket cannot be re-used by the target service to impersonate a user to
another service unless the KDC allows it.

A Kerberos protocol extension called S4U2Proxy (or
constrained-delegation) was developed by Microsoft to allow trusted
services to impersonate Kerberos principals when performing operations
on their behalf. The key word here is **trusted**, the ultimate decision
point belongs to the KDC itself.

The FreeIPA management framework uses S4U2Proxy to authenticate to the
LDAP server, therefore it is a **trusted** service and sits in a very
valuable (for an attacker) position; it can impersonate any user that
authenticated to the service against the LDAP server. Any security
breach in the framework code could allow an attacker to sift through the
authentication tokens received by the framework, find administrator
credentials, steal them, and then use them against the LDAP server. This
is especially bad when password-based authentication is performed,
because in that case the framework does have access to a locally stored
TGT.

The only way to prevent this exposure is to deny the FreeIPA framework
direct access to the tickets and service credentials. But this
effectively means inability to perform any Kerberos authentication on
behalf of the user by the service itself.

To solve this problem GSS-Proxy was developed. GSS-Proxy is a service
comprised of a special "interposer" plugin for the GSSAPI library and a
separate daemon. The GSS-Proxy plugin intercepts all GSSAPI operations
and diverts their execution to the GSS-Proxy daemon. The daemon performs
these operations and returns encrypted structures which are them stored
in a credential cache used by the client application. The client
application cannot decode these structures by itself. This approach
allows GSS-Proxy to implement privilege separation on a per ticket
level.

With the help of GSS-Proxy the FreeIPA management framework is denied
direct access to its own Kerberos keys. Any intruder running under
Apache privileges will not be able to steal the service keys.
Technically an intruder would be able to request a ticket but would not
be able to retrieve actual credentials.

Session management was also changed. Instead of managing sessions in the
framework and storing tickets in a cache, session are now manage by the
mod_auth_gssapi module and tickets are stored on the filesystem, their
contents encrypted by GSS-Proxy. Additionally the framework has been
changed to run as a separate user named "ipaapi", it does not have the
same privilegese the Apache server has. This, in combination with
GSS-Proxy allows to prevent attacks that would result in stealing any
keys and perform an attack from a remote system. The attack surface is
seriously reduced.

FreeIPA 4.5 introduces also client certificate authentication support.
In fact, it was possible to set up client certificate authentication
since FreeIPA 4.4 but with version 4.5 the whole setup was streamlined
and integrated with the rest of the framework.

Client certificate authentication is handled by ``mod_nss``, an Apache
module, but authentication is managed via the ``mod_auth_gssapi`` module
in all cases to establish a session and to obtain Kerberos credentials.
In fact ``mod_auth_gssapi`` has been enhanced to use another kerberos
protocol extension called S4U2Self (or protocol-transition) to obtain
tickets for users that were successfully authenticated by ``mod_nss``.
With this extension a **trusted** service (application) can request to
the KDC a service ticket to itself on behalf of the user.

If the KDC trusts a service for both S4U2Self and S4U2Proxy operations,
a service can fully impersonate any user to the target services (i this
case the LDAP server). To prevent a breach in the framework to allow it
to impersonate arbitrary users, the GSS-Proxy allow the
protocol-transition operation for the HTTP service only if the request
comes from the Apache user. This privilege separation mechanism prevents
a breach of the framework code from allowing direct privilege escalation
via impersonation, because the framework code runs as a different user
in the system. Likewise constrained-delegation to obtain a ticket to the
LDAP server is permitted only to the ``ipaapi`` user.

The consequence of privilege separation enhancements is that the FreeIPA
management framework does not handle sessions directly and handles user
authentication only for forms-based authentication, but does not retain
access to user's TGTs in that case. Due to this change separate URLs for
session and non-session based access are not necessary anymore,
applications that present a cookie to the Apache server have immediate
access regardless of the endpoint and lack of a cookie results in
automatic GSSAPI negotiation. The session cookie is effectively just an
optimization.



Logging in FreeIPA management framework
---------------------------------------

With multiple components involved in a user request processing, it is
important to enable logging properly in all relevant components. There
are several layers of logging available:



FreeIPA framework's Python code logging
----------------------------------------------------------------------------------------------

FreeIPA has a global configuration file in ``/etc/ipa/default.conf``.
Settings in this file apply to all instances of the FreeIPA python code,
be it client or server. However, a specific context, that has its own
configuration file in ``/etc/ipa/server.conf``, is used to specify
options to the server side of the framework. This allows to avoid
excessive debug information on the command line when debugging problems
on IPA servers.

Add a ``debug = True`` option to the ``[global]`` section to enable
debug logs. For the server side of the framework a restart of the
``httpd.service`` is required. Logs will appear in
``/var/log/httpd/error_log`` and will be intermixed with other log
entries from other Apache modules.



Logging the activity of Apache modules
----------------------------------------------------------------------------------------------

FreeIPA does only uses the logging functionality available in Apache.
Its documentation is available at the `Apache
website <https://httpd.apache.org/docs/current/logs.html>`__.



Logging of Samba client operations
----------------------------------------------------------------------------------------------

FreeIPA utilizes Python bindings to Samba libraries to set-up trusts to
Active Directory. These libraries have their own logging facility. The
FreeIPA code initializes Samba bindings with a special configuration
file named ``/usr/share/ipa/smb.conf.empty``. It is an empty
configuration file that forces Samba bindings to ignore system-wide
settings. In order to debug what the FreeIPA management framework does
during trust to Active Directory operations, add the
``log level = <LEVEL>`` option to ``/usr/share/ipa/smb.conf.empty``. Log
levels aren't really standardized and different parts of Samba use
different levels but for trust operations it is useful to set the log
level to ``10``:

::

   [global]
     log level = 10

This will allow Samba Python bindings to print all low level information
and dump details of network packets in a human readable form. For DCE
RPC operations both input and output structures will be decoded and
printed. This is very useful to see what's wrong with a request or a
response.

The configuration file ``smb.conf.empty`` is read every time a request
that requires using Samba Python libraries is processed. Therefore a
change of the log level does not require a restart of the
``httpd.service``.

The same configuration file is used by an oddjobd helper that performs
retrieval of the trusted forest topology. Since the helper is executed
by the FreeIPA management framework via a D-BUS request, its output is
returned back to the framework and is logged into Apache'a ``error_log``
along the other log entries of the framework.



Logging of GSS-Proxy
--------------------

GSS-Proxy's configuration is defined in the ``/etc/gssproxy`` directory.
It has series of files for each GSSAPI service and also a general
``/etc/gssproxy/gssproxy.conf``. To enable a reasonable level of debug
information use the following options:

::

   [gssproxy]
     debug = true
     debug_level = 2

To apply configuration changes, reload or restart the
``gssproxy.service``. Logs will be stored in the system journal and can
be viewed with the help of ``journalctl -u gssproxy`` command.

GSS-Proxy logs every incoming request and the result of its processing.
This information includes dates in UTC, so there are two dates displayed
by ``journalctl``: a date in local time zone and UTC, the latter is part
of the string that GSS-Proxy logged to ``journald``.

Below is a typical session that starts from an ``httpd`` process
connecting to GSS-Proxy and asking to acquire credentials for the HTTP
service. A log is redacted to remove a common date and hostname prefix
displayed by ``journalctl`` utility:

::

   -- Logs begin at Fri 2017-01-27 16:08:43 CET, end at Fri 2017-04-21 18:15:10 CEST. --
   gssproxy[7186]: [2017/04/21 11:46:20]: Client connected (fd = 18)
                   [2017/04/21 11:46:20]:  (pid = 25925) (uid = 48) (gid = 48)
                   [2017/04/21 11:46:20]:  (context = system_u:system_r:httpd_t:s0)
                   [2017/04/21 11:46:20]:
   gssproxy[7186]: [2017/04/21 11:46:20]: gp_rpc_execute: executing 6 (GSSX_ACQUIRE_CRED) for service &quot;ipa-httpd&quot;, euid: 48,socket: (null)
   gssproxy[7186]:     GSSX_ARG_ACQUIRE_CRED( call_ctx: { &quot;&quot; [  ] } input_cred_handle: { &quot;HTTP/nyx.xs.ipa.cool@XS.IPA.COOL&quot; [ { &quot;HTTP/nyx.xs.ipa.cool@XS.IPA.COOL&quot; { 1 2 840 113554 1 2 2 } BOTH 86400 86400 } ] [ ...........L..N.... ] 0 } add_cred: 0 desired_name: &lt;Null&gt; time_req: 4294967295 desired_mechs: { { 1 2 840 113554 1 2 2 } } cred_usage: BOTH initiator_time_req: 0 acceptor_time_req: 0 )
   gssproxy[7186]:     GSSX_RES_ACQUIRE_CRED( status: { 0 { 1 2 840 113554 1 2 2 } 0 &quot;&quot; &quot;&quot; [  ] } output_cred_handle: { &quot;HTTP/nyx.xs.ipa.cool@XS.IPA.COOL&quot; [ { &quot;HTTP/nyx.xs.ipa.cool@XS.IPA.COOL&quot; { 1 2 840 113554 1 2 2 } BOTH 86400 86400 } ] [ .h..LWMJ...k....... ] 0 } )

Lines with ``GSSX_ARG_`` show arguments passed for this operation. Lines
with ``GSSX_RES_`` show the results of the operation. In the example
above the application asks to acquire credential for the Kerberos
principal ``HTTP/nyx.xs.ipa.cool@XS.IPA.COOL``. The sequence
``{ 1 2 840 113554 1 2 2 }`` is the OID of Kerberos V5:
1.2.840.113554.1.2.2.

Kerberos service accepts incoming connections by using the
``gss_accept_sec_context()`` API call. This can be seen in the following
example:

::

   gssproxy[7186]: [2017/04/21 11:46:20]: gp_rpc_execute: executing 9 (GSSX_ACCEPT_SEC_CONTEXT) for service &quot;ipa-httpd&quot;, euid: 48,socket: (null)
   gssproxy[7186]:     GSSX_ARG_ACCEPT_SEC_CONTEXT( call_ctx: { &quot;&quot; [  ] } context_handle: &lt;Null&gt; cred_handle: { &quot;HTTP/nyx.xs.ipa.cool@XS.IPA.COOL&quot; [ { &quot;HTTP/nyx.xs.ipa.cool@XS.IPA.COOL&quot; { 1 2 840 113554 1 2 2 } BOTH 86400 86400 } ] [ .h..LWMJ...k....... ] 0 } input_token: [ ...i....H.......... ] input_cb: &lt;Null&gt; ret_deleg_cred: 1 )
   gssproxy[7186]:     GSSX_RES_ACCEPT_SEC_CONTEXT( status: { 0 { 1 2 840 113554 1 2 2 } 0 &quot;&quot; &quot;&quot; [  ] } context_handle: { [ ......H............ ] [  ] 0 { 1 2 840 113554 1 2 2 } &quot;admin@XS.IPA.COOL&quot; &quot;HTTP/nyx.xs.ipa.cool@XS.IPA.COOL&quot; 86692 443 0 1 } output_token: [ .......H........... ] delegated_cred_handle: { &quot;admin@XS.IPA.COOL&quot; [ { &quot;admin@XS.IPA.COOL&quot; { 1 2 840 113554 1 2 2 } INITIATE 86392 0 } ] [ l.P...8H......T.... ] 0 } )

The ``httpd`` service asks to perform a ``gss_accept_sec_context()``
call and requires that delegated credential are returned
(``ret_deleg_cred: 1``). The result of running
``gss_accept_sec_context()`` is an output token. Note that the delegated
credential describes the Kerberos principal of the user that performed
the access, ``admin@XS.IPA.COOL`` in this case.

After the user is authenticated, its cookie is parsed and the Kerberos
credential is located. Note that the operation is performed by a
different process, actual FreeIPA framework instance running under the
``ipaapi`` user identity:

::

   gssproxy[7186]: [2017/04/21 11:46:20]: gp_rpc_execute: executing 6 (GSSX_ACQUIRE_CRED) for service &quot;ipa-api&quot;, euid: 384,socket: (null)
   gssproxy[7186]:     GSSX_ARG_ACQUIRE_CRED( call_ctx: { &quot;&quot; [  ] } input_cred_handle: { &quot;admin@XS.IPA.COOL&quot; [ { &quot;admin@XS.IPA.COOL&quot; { 1 2 840 113554 1 2 2 } INITIATE 86392 0 } ] [ l.P...8H......T.... ] 0 } add_cred: 0 desired_name: &lt;Null&gt; time_req: 4294967295 desired_mechs: { { 1 2 840 113554 1 2 2 } } cred_usage: INITIATE initiator_time_req: 0 acceptor_time_req: 0 )
   gssproxy[7186]:     GSSX_RES_ACQUIRE_CRED( status: { 0 { 1 2 840 113554 1 2 2 } 0 &quot;&quot; &quot;&quot; [  ] } output_cred_handle: { &quot;admin@XS.IPA.COOL&quot; [ { &quot;admin@XS.IPA.COOL&quot; { 1 2 840 113554 1 2 2 } INITIATE 86392 0 } ] [ l.P...8H......T.... ] 0 } )

This process has no access to the keytab used by ``httpd`` but can ask
GSS-Proxy to return a ticket for a user that was authenticated. Once the
ticket is obtained, it can be used to talk to a different service, in
our case the IPA LDAP server:

::

   gssproxy[7186]: [2017/04/21 11:46:20]: gp_rpc_execute: executing 8 (GSSX_INIT_SEC_CONTEXT) for service &quot;ipa-api&quot;, euid: 384,socket: (null)
   gssproxy[7186]:     GSSX_ARG_INIT_SEC_CONTEXT( call_ctx: { &quot;&quot; [  ] } context_handle: &lt;Null&gt; cred_handle: { &quot;admin@XS.IPA.COOL&quot; [ { &quot;admin@XS.IPA.COOL&quot; { 1 2 840 113554 1 2 2 } INITIATE 86392 0 } ] [ l.P...8H......T.... ] 0 } target_name: &quot;ldap@nyx.xs.ipa.cool&quot; mech_type: { 1 2 840 113554 1 2 2 } req_flags: 58 time_req: 0 input_cb: &lt;Null&gt; input_token: &lt;Null&gt; [ { [ sync.modified.cr... ] [ 64656661756c740 ] } ] )
   gssproxy[7186]: [2017/04/21 11:46:20]: Credentials allowed by configuration
   gssproxy[7186]:     GSSX_RES_INIT_SEC_CONTEXT( status: { 1 { 1 2 840 113554 1 2 2 } 0 &quot;The routine must be called again to complete its function&quot; &quot;&quot; [  ] } context_handle: { [ ......H............ ] [  ] 0 { 1 2 840 113554 1 2 2 } &quot;&quot; &quot;&quot; 0 314 1 0 } output_token: [ ...A....H.......... ] [ { [ 73796e635f63726564730 ] [ ....admin.XS.IPA... ] } ] )

To receive a proper ticket a few more exchanges are required, this can
be seen by the response from GSS-Proxy telling that "The routine must be
called again to complete its function". This is a normal flow for GSSAPI
exchanges.



Logging of KDC operations
----------------------------------------------------------------------------------------------

Each Kerberos request to obtain an initial (TGT) or a service ticket
will be reflected in the KDC logs. The KDC writes its log in the
``/var/log/krb5kdc.log`` file. Below is an example of how the KDC log
looks like with common date and hostname information removed. In the
original log file timestamps are in local time zone format.

::

   krb5kdc[15953](info): TGS_REQ (8 etypes {18 17 20 19 16 23 25 26}) IP.AD.DR.ES: ISSUE: authtime 1492775177, etypes {rep=18 tkt=18 ses=18}, admin@XS.IPA.COOL for HTTP/nyx.xs.ipa.cool@XS.IPA.COOL
   krb5kdc[15953](info): closing down fd 11
   krb5kdc[15954](info): AS_REQ (8 etypes {18 17 16 23 25 26 20 19}) IP.AD.DR.ES: NEEDED_PREAUTH: HTTP/nyx.xs.ipa.cool@XS.IPA.COOL for krbtgt/XS.IPA.COOL@XS.IPA.COOL, Additional pre-authentication required
   krb5kdc[15954](info): closing down fd 11
   krb5kdc[15954](info): AS_REQ (8 etypes {18 17 16 23 25 26 20 19}) IP.AD.DR.ES: ISSUE: authtime 1492775180, etypes {rep=18 tkt=18 ses=18}, HTTP/nyx.xs.ipa.cool@XS.IPA.COOL for krbtgt/XS.IPA.COOL@XS.IPA.COOL
   krb5kdc[15954](info): closing down fd 11
   krb5kdc[15953](info): TGS_REQ (8 etypes {18 17 20 19 16 23 25 26}) IP.AD.DR.ES: ISSUE: authtime 1492775177, etypes {rep=18 tkt=18 ses=18}, HTTP/nyx.xs.ipa.cool@XS.IPA.COOL for ldap/nyx.xs.ipa.cool@XS.IPA.COOL
   krb5kdc[15953](info): ... CONSTRAINED-DELEGATION s4u-client=admin@XS.IPA.COOL
   krb5kdc[15953](info): closing down fd 11

A log entry includes the type of operation (AS_REQ or TGS_REQ), the list
of requested encryption types, the IP address of the client and the
result of the operation. We can see that an administrator requested a
service ticket to the ``HTTP/..`` service to perform Kerberos
authentication as requested by ``mod_auth_gssapi``. On the server side
the ``HTTP/..`` service needs to obtain its own initial ticket (a
request for service ticket to ``krbtgt/..``). After the FreeIPA
framework got to actually process the user's request, it needed to
obtain a service ticket to the ``LDAP/..`` service. The KDC recorded in
the log that this request was actually done as a S4U2Proxy operation,
performing constrained delegation for the client ``admin@..`` which is
the original user Kerberos principal.



ccache storage
--------------

The ccaches are stored by mod_auth_gssapi in /run/ipa/ccaches. This
directory is managed by ``/usr/lib/tmpfiles.d/ipa.conf``. If the
permissions or ownership are incorrect then one may get the error: <code
ipa: ERROR: cannot connect to
'https://ipa.example.test/ipa/session/json': Exceeded number of tries to
forward a request.

To re-apply the tmpfiles configuration run (in this case fixing bad
permissions on /run/ipa/ccaches):

::

   # SYSTEMD_LOG_LEVEL=7 systemd-tmpfiles --create  /usr/lib/tmpfiles.d/ipa.conf
   Looking for configuration files in (higher priority first):
           /etc/tmpfiles.d
           /run/tmpfiles.d
           /usr/local/lib/tmpfiles.d
           /usr/lib/tmpfiles.d
   Successfully loaded SELinux database in 1.514ms, size on heap is 329K.
   Reading config file "/usr/lib/tmpfiles.d/ipa.conf"…
   Running create action for entry d /run/ipa
   Found existing directory "/run/ipa".
   "/run/ipa" has correct mode 40711 already.
   Running create action for entry d /run/ipa/ccaches
   Found existing directory "/run/ipa/ccaches".
   Changing "/run/ipa/ccaches" to mode 6770.
   Running create action for entry a /run/ipa/ccaches
   Setting access ACL u::rwx,g::rwx,g:apache:rwx,m::rwx,o::--- on /run/ipa/ccaches.