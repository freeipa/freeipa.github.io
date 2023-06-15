Overview
--------

We're proposing adding a KDC proxy function, interoperable with
Microsoft's
`MS-KKDCP <http://msdn.microsoft.com/en-us/library/hh553774.aspx>`__
implementation, to the IPA web server, to allow clients to access the
KDC and kpasswd services using HTTP and HTTPS.



Use Cases
---------

There are deployments in the wild where the only ports which can be made
reachable are those which are used for carrying HTTP (port 80) and HTTPS
(port 443) traffic. While clients can access the IPA web-based UI by
going through a login form, administrative command-line tools will not
function.

Being able to obtain Kerberos credentials by using the IPA HTTP service
as a proxy will allow those tools to be used, and provides for the
possibility that other Kerberos-authenticated services will also be made
available via HTTP, including — but not limited to — IPA services
accessed through its own HTTP server.

Design
======

While other designs are possible, for IPA, the consensus appears to be
that a python-based implementation which can be run by mod_wsgi is
preferred. Keeping clear of the details of protocol parsing, the handler
needs to:

-  Accept a request wrapped in a contained in an HTTP POST request.
-  Determine whether that request should be sent to the KDC or to the
   password-changing service.
-  Locate the server for the realm, the name of which is also contained
   in the client request.
-  Send the request to the right server.
-  Read the server's response.
-  Send the server's response as the body of the response to the HTTP
   request.

Implementation
--------------

By default, the IPA framework is run as a mod_wsgi process group. The
simplest implementation is single-threaded and blocking. The
implementation either needs to multiplex multiple in-flight requests, or
a second process group will need to be added to avoid starving clients
which attempt to use the framework's services.

Service location needs refinement to properly sort servers located using
DNS SRV records. Server affinity is highly preferred in order to avoid
breaking multiple-round-trip preauthentication if/when it appears. Only
TCP-based connections to the KDC are supported at first. Supporting
servers which can only be reached by UDP complicates things due to a
need to catch and handle KRB_ERR_RESPONSE_TOO_BIG errors. Additionally,
connecting over TCP only allows the proxy to avoid needing to manage
timeouts and retransmits.

The `client
support <http://k5wiki.kerberos.org/wiki/Projects/HTTP_Transport>`__ was
merged in time for the upstream 1.13 release (Fedora 22 and later), and
backported to 1.12 for Fedora 21 and RHEL 7.1.



Feature Management
------------------

Ideally, no management will be involved. If we need to be able to enable
or disable the feature, a command-line tool along the lines of
ipa-compat-manage and ipa-nis-manage is going to be the simplest option.



Major Configuration Options and Enablement
------------------------------------------

There are only a two things which might need to be configured:

-  Whether the proxy service is enabled or not. If we just declare that
   it's always enabled, this goes away.
-  The locations of KDCs for trusted realms. Currently we can dig
   information out of krb5.conf or DNS. If we need to consult other
   sources for this information (directory servers, SSSD .kdcinfo
   files), we'll need to add logic for that. If it comes from places we
   already consult, then it should already work.

Replication
-----------

No new data, nothing to replicate.



Updates and Upgrades
--------------------

Upgrades should need to drop the new module in place, and a new
configuration file to be picked up by httpd that instructs it to call
the module for requests to the proxy namespace (/KdcProxy seems
popular). If we can keep the explicit configuration at zero, that's all
there'll be to it.

Dependencies
------------

No new dependencies, except possibly for those required to handle server
location.



External Impact
---------------

No expected requirements on external teams; using the new feature is
optional.



Backup and Restore
------------------

There should be no additional configuration or data to back up or
restore.



Test Plan
---------

Given that there are two types of KDC requests and one supported type of
password-change request, we need to test with

-  kinit (KDC AS requests)
-  kvno (KDC TGS requests)
-  kpasswd (KDC AS requests and kpasswd requests)

Running these commands with KRB5_TRACE set may be necessary to ensure
that requests are going to the right port.

The `client
support <http://k5wiki.kerberos.org/wiki/Projects/HTTP_Transport>`__ in
libkrb5 expects configuration in krb5.conf's [realms] section,
specifying the location of the KDC or kpasswd service as a full HTTPS
URL (e.g. "kdc = https://kdc.example.com/KdcProxy" and either
"admin_server = https://kdc.example.com/KdcProxy" or "kpasswd_server =
https://kdc.example.com/KdcProxy", along with "http_anchors =
FILE:/etc/ipa/ca.crt" to allow the certificate to be verified). See the
`upstream
docs <http://web.mit.edu/kerberos/krb5-current/doc/admin/https.html>`__
for more information.

In particular, to ensure we're not making the sorts of mistakes that
lead to multiple security advisories, we also need to check that certain
things fail:

-  if the hostname in the server location URL doesn't match what's in
   the certificate (for example, because we added a mapping from an
   arbitrary name to the server's IP address to /etc/hosts, and used
   that name in the URL)
-  if the client isn't configured to trust the server's CA's certificate

The patch will need to be extended to include a modified version of the
service as we're implementing it, using the python-paste server to run
it as part of additions to the libkrb5 self-tests.

`Category:FreeIPA V4 Test Plan <Category:FreeIPA_V4_Test_Plan>`__
`Category:FreeIPA Test Plan <Category:FreeIPA_Test_Plan>`__
