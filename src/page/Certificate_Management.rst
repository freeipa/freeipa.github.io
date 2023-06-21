Certificate_Management
======================

Introduction
============

One of the major requirements for the IPA v2 release is to integrate the
Certificate Authority into the IPA server. The main use case for this is
provisioning certificates for use by applications, for example web
server or IMAP server. There is no requirement to provision user
certificates. This is deferred to later version of IPA. Also there is no
plan to deploy certificates to all the IPA clients. Instead the clients
will be using kerberos keytab to authenticate to IPA server.

Overview
========



Certificate Management Interfaces
---------------------------------

Certificate life cycle management includes the following basic
operations:

-  Requesting certificates. If a new certificate needs to be issued it
   should be possible to request a new certificate from the system. The
   assumption is that the keypair is generated on the client system, and
   the client is submitting a public key to the CA. In IPA v2 we are not
   planning to support key escrow use case and server side key
   generation. This will be added later when add support for user
   certificate authentication.
-  Checking request status. In different situations the certificate
   issuance can be delayed pending administrative approval. There is a
   need to be able to track and check the status of outstanding
   requests.
-  Getting certificate. When the request is approved there should be a
   way to retrieve the certificate and store it in the appropriate
   location.
-  Revoking certificates. If the certificate is lost or compromised it
   should be possible to revoke it so that it can not be used any more.
   There is also an option to not completely revoke certificate by put
   it on hold. The example of the situation would be when the user left
   his smart card at home. His certificates can be put on hold till next
   day.
-  Take certificate off "hold". When the certificate is on hold take it
   off hold.

All these operations are performed by the Certificate Authority (CA). To
prevent direct access to the CA by IPA clients, the CA is front-ended
with the Registration Authority (RA). It will be impossible to request
anything from the CA bypassing RA. The RA will be embedded into the
XML-RPC back end as an XML-RPC back end plugin. This plugin will be
responsible for validating the permission of the client and proxying its
request to the CA through RA, and then collecting the response and
sending it back.

The back end management plugin can be used for management of the
certificates in the system through UI and CLI. For v2 we are planning to
allow certificates to be associated with a host. If there are multiple
services running on the same host there might be a need to issue
multiple certificates. There are several approaches how this can be
done:

-  The simplest approach would be to allow just multiple certificates to
   be issued for one host. In this case the CA will publish the issued
   certificates into the multi valued attribute in the host entry. There
   is a concern about this solution that is currently being
   investigated. It is related to unpublishing the certificate when it
   is revoked. I might be that the code responsible for this feature is
   not capable of removing the certificate from the multi value
   attribute. If this is the case then we would have to solve the
   problem by publishing the certificates into separate entries.
-  The second approach is to have child entries under host entries where
   the certificates will be stored. This way there will be no problems
   with publishing and unpublishing the certificates but we would a bit
   violate the flat approach to our tree.
-  The third option would be to put the certificate into the service
   entry. Conceptually service entries are now used to store a kerberos
   principal's keys. Adding auxiliary class to store certificate
   attribute to the service entry might also be a viable option.
-  The last option is not to publish the certificates at all and for any
   administrative need query the CA using RA. This would be a bit of
   performance burden and would require RA to expose a more robust
   querying interface than currently planned.

| Currently we are leaning towards the combination of the first and
  third options. If the first option is not available we might decide to
  allow one certificate for a host entry and if there are more needed
  the certificates can be issued to the services running on the host in
  this case we would have to modify the service to alternatively be a
  container for the kerberos data and for the certificate.
| It should be possible to perform the following tasks via either
  management interface:

-  List the certificates issued to a host. If the certificates are
   issued to the host itself then the command or UI would be based on
   the host identity. In CLI the command can be something like:

::

   >ipa-certs list --host=my.host.somewhere.com

In the UI it can be a section in the edit/view host screen. Since the UI
will use AJAX we will be able to have this section collapsed by default
and retrieve cert information only explicitly on demand if we see
performance issues (especially if end up not publishing the certs at all
and would have to retrieve information form CA itself). If we put the
certificates into the host entry, the command will just retrieve the
entry and list the certificates that are stored in the multi valued
attribute. If we have to put the certificates in special entries under
host, the host management plugin would have to enumerate these entries
and build the certificate list this way. If we decide to store the
certificates in services, we would have to list the services themselves
in the UI. The services will have the fully qualified host name as a
part of the principal and this is how they can be matched and associated
to the host they are running on. If we decide not to publish certs at
all we would have to have the RA request that would ask the CA about all
the certificates issued to a given host.

-  Get status of a certificate. Display the properties of the
   certificate including host name, serial number, expiration etc. In UI
   there should be a button that would allow selecting certificate from
   the list and clicking “details” button. On command line this should
   be a command like:

::

   >ipa-certs list --serial=<serial number>

-  Revoke a certificate. In the UI there should be a button that would
   allow selecting certificate from the list and clicking “revoke”
   button. On command line this should be a command like:

::

   >ipa-certs revoke --serial=<serial number>

-  Renew certificate. If the certificate is going to expire there should
   be a way to issue a replacement certificate. In the UI there should
   be a button that would allow selecting a certificate from the list
   and clicking “renew” button. On command line this should be a command
   like:

::

   >ipa-certs renew --serial=<serial number>

-  Get a new certificate. There should be a way to request a certificate
   for host (or service) from the UI and command line.



Access Control
--------------

The certificate operations can be performed only via RA. The management
plugin that interacts with RA would have to check if the user is allowed
to perform certificate operations. In IPA v2 we decided that we will not
have a granular access control over individual operations. Instead we
will check that a user who is trying to execute a certificate command is
a member of the specific group. If he is a member of the group then he
would be able to perform any of the certificate related operations. The
name of the group will default to "administrators" but would be
changeable in a specific configuration entry (see below).



Actual Operations
-----------------

Since in v2 we are not planning to support certificate operations with
user certificates not all operations that will be implemented would
actually be used and exposed via UI and CLI. For example pulling on hold
and talking off hold will not possible in v2 despite the fact that RA
and management plugin will be aware of this functionality. Also there
will be no approval process. This means that any request for a
certificate will be automatically approved. In this situation a request
for new or renewed certificate should be satisfied by CA within seconds.
It would be a responsibility of the management plugin to issue request
and then immediately retrieve the certificate. If might require some
polling logic to make sure that we do not time out a bit prematurely if
it took a bit longer for CA to issue a certificate.



Automatic Cert Provisioning
---------------------------

| So far we have been talking about the administrative tasks performed
  by the administrator via UI and CLI. In those scenarios the
  administrator is the actor and his access control properties are
  checked before a certificate operation is granted. Potentially an
  administrator (depending upon his privileges of cause) can perform all
  the certificate operations including issuance, renewal and revocation.
  Any of these operations is manual and requires user authentication
  before it can be performed.
| There is one more use case when the certificate shall be issued or
  renewed but the request is not authorized by user (administrator) but
  rather by a host. The use case would be as following:

-  IPA enabled host runs a web application that needs certificate for
   its functioning. The certificate provisioning can be scripted as a
   part of the installation.
-  IPA enabled host (if told to) would renew certificate on behalf of
   the application if it detects that the certificate is about to
   expire.

| We will talk about how it would be actually accomplished below.
| After some evaluation we agreed that there will be limitations of what
  a host can request. We decided that it can request only certificate
  for itself (or service running on the same host) and not for any other
  host. It can also can't automatically revoke certificates since in
  case of a bug or some misconfiguration it can have unrepairable impact
  on the customer environment.



IPA Client Design
----------------------------------------------------------------------------------------------

To be able to request a certificate from the IPA client automatically
there should be a utility tightly integrated with the IPA client that
would aid requesting the certificate for a service running on the host.
The following diagram shows all the IPA client components involved in
the operation.

`image:CertProvisioningSmall.jpg <image:CertProvisioningSmall.jpg>`__

#. The cert utility, named ipa-get-cert on the diagram, will accept
   command line parameters and store them in the LDB. This is the only
   thing it will do.
#. The XML-RPC Client (formerly known as policy downloader) will be a
   daemon. It will be event based as any other service on the client
   designed so far. It will wake up periodically and see if there is
   anything it needs to do. Originally we planned that the XML-RPC
   Client will be responsible for only dealing with policies, but now
   its responsibilities will broaden (thus the name change). The XML-RPC
   Client being a component capable of establishing XML-RPC connection
   will now be in charge of requesting certificates too, since that can
   be done only through XML-RPC interface. So when XML-RPC Client wakes
   up it will check if it is time to get policies or do something with
   certificates. Here we really have several options on how we will
   implement the flow of the operations. One scenario is shown on the
   diagram but there are others that we will discuss below.
#. So as shown on the diagram the XML-RPC Client will wake up and if it
   is time for check for cert tasks ask the Data Provider is there is
   anything to do with the certs. The data provider will get data from
   LDB.
#. Before sending it back it might check if the cert is already
   provisioned into the LDAP entry and pull it from there or do some
   other checks if needed. This is really an optional step and
   implementation would decide if it is needed.
#. The IPA Data Provider will respond to the XML-RPC Client. It can
   respond that there is nothing to do (which might be caused by the
   fact that there is actually nothing to do or by the fact that the
   client is offline), it can hand the XML-RPC Client back the
   parameters of the request it should submit over XML-RPC or it can
   return the certificate if we decide that step 4 makes sense.
#. If there is work to do the XML-RPC Client will formulate a request to
   the IPA server over the XML-RPC connection.
#. The results of the request need to be recorded in the LDB so the
   XML-RPC Client would call the Data Provider to request update to the
   LDB.

Alternatively if we think that the XML-RPC Client would not need the
Data Provider to do any lookups the XML-RPC Client will directly
interact with LDB but it still needs to ask data provider about the
offline status. The Data Provider is the only service that has authority
to determine the offline status. So the XML-RPC Client logic might be:

-  Wake up and determine what operation it is supposed to do: certs or
   policies (let us say certs)
-  Ask data provider is it is offline. If yes – sleep. If no continue.
-  Check if there is the cert related data to process. If no – sleep. If
   yes continue.
-  If the data shows that XML-RPC request needs to be issued, then issue
   request and process the results potentially updating LDB. If the data
   shows that there needs to be an LDAP lookup, then request the LDAP
   lookup from IPA Data Provider over DBUS interface. Process results.
   Update LDB.

The exact logic will be figured out during the implementation but the
following main points should be assumed and not changed:

-  The XML-RPC Client will be a daemon (fork-exec by the service
   controller).
-  The XML-RPC Client will have a main loop that will be event based and
   events will be triggered by time elapsed or emitted at the start of
   the daemon.
-  The XML-RPC Client will be a client of the IPA data provider as all
   other services (PAM responder, NSS responder etc.) are.
-  The XML-RPC Client is in charge of the XML-RPC connection no other
   process should be aware how to do XML-RPC.
-  The XML-RPC Client is in charge of all operations that can be done
   only via XML-RPC. This now includes not only getting policies but
   also requesting operations from CA via RA.
-  The XML-RPC Client may or may not directly connect to LDB. It
   definitely would directly connect to the configuration instance of
   LDB. If we would save the certificate related data there we would not
   need to use data provider to connect to it. If we decide to store
   this data in the LDB instance that stores other information we might
   decide that it would be beneficial to access this data through data
   provider.



Server Policy about Clients
----------------------------------------------------------------------------------------------

Though the clients would have the capability to request new certificates
or track and renew old ones it does not mean that the server would
blindly respect and execute these requests. The management plugin that
interfaces with RA would have to determine that the current XML-RPC
request is executed under host principal and not by an administrative
user (see above). In the same configuration entry that will hold the
name of the group of user that would have access to certificate
operations, there will be an attribute that would hold the policy that
will control how server would react to the client requests to issue or
renew a certificate. In IPA v2 there will be 3 supported values.

-  Always - always respect host's requests
-  Never (default) - automatically ignore all requests coming from hosts
-  Renew - respect only renew requests. The RA uses one and the same
   call for issuance and renewal so it is hard to differentiate the two
   scenarios. To react differently in case of certificate renewal the
   plugin will check the host (or service entry) for a certificate. If
   the certificate already exists in the entry, then the request will be
   treated as a renewal. If there is no certificate in the entry then it
   would be treated as a new request.

The list of the values can be later extended if needed.



Command Line Utility
----------------------------------------------------------------------------------------------

Now it is time to talk about the data that utility would collect. At its
core, the tool's function is very similar to that of ipa-getkeytab, so
we're aiming for a command-line interface which feels similar.

There will be only four commands that the utility would be able to
issue:

-  Request certificate and track (by default or not track if told not
   to) its expiration.
-  Start tracking expiration of an already provisioned certificate
-  Stop tracking expiration of an already provisioned certificate
-  Status – list current pending requests and/or currently tracked certs

In the case of requesting a certificate the command might look like this
(the specific details will be determined at the implementation phase):

::

   ipa-getcert request [options]
   * If the client can conceive of more than one CA:
     -c        location of CA (format TBD)
   * If we're generating a key:
     -g        generate a new key (default: use an already-generated key)
     -G size   size of new key (default TBD)
   * Whether we're generating a key or not:
     -d DIR    database containing / for storing private key and cert (NSS)
     -n NAME   nickname to give issued certificate (only valid with -d)
     -k PATH   file containing / for storing PEM private key
     -f PATH   file for storing issued PEM certificate (only valid with -k)
   * Whether or not to track expiration:
     --no-track-expiration
   * Optional bits:
     -S NAME   requested subject name (default: CN=<fqdn>)
     -u USAGE  requested usage/eku (default: tls-server)
     -s NAME   requested service name part (default: host)

In case of starting to track a certificate's expiration, one shall
provide the following command line:

::

   ipa-getcert start-tracking [options]
   * General options:
     -d DIR    database containing private key and cert (NSS)
     -n NAME   nickname of issued certificate (only valid with -d)
     -k PATH   file containing PEM private key
     -f PATH   file containing / for storing issued PEM certificate (only valid with -k)
   * If the client can conceive of more than one CA:
     -c ARG    location of CA (format TBD)

The utility will make sure that a certificate in the given format and
place already exists and then save the information about it in the LDB.

In case of stopping the tracking of a certificate's expiration, it would
be:

::

   ipa-getcert stop-tracking [options]
   * General options:
     -s NUM    serial number of certificate
     -S KEYID  subject key identifier for certificate
   * In case the serial number corresponds to more than one certificate, and the key identifier is not known:
     -d DIR    database containing private key and cert (NSS)
     -n NAME   nickname of issued certificate (only valid with -d)
     -k PATH   file containing PEM private key
     -f PATH   file containing / for storing issued PEM certificate (only valid with -k)

To stop tracking of the cert expiration the utility will just remove an
entry that corresponds to the given cert from the LDB.

The status commend will list the contents of the cert data in the LDB.

::

   ipa-getcert list [options]
   * General options:
     --requests  List only information about outstanding requests
     --tracking  List only information about tracked certificates



Data Stored in LDB
----------------------------------------------------------------------------------------------

The data stored in the LDB will look like this:

-  Entry that will contain request for a new certificate

   -  Location of the CA (if more than one can be known to the client)
   -  Server-supplied identifier for tracking the state of the request
   -  Date when the request was submitted
   -  Format that the certificate shall be saved in
   -  Path where the certificate shall be saved
   -  Should its expiration be tracked or not once it's issued
   -  Format that the private key is stored in (for later)
   -  Path where the private key can be found (for later)

-  Entry that will contain expiration tracking information

   -  Serial number – retrieved from the issued certificate
   -  Subject name – retrieved from the issued certificate
   -  Subject key identifier – retrieved / calculated from the issued
      certificate
   -  Certificate expiration date – retrieved from the issued
      certificate
   -  Format that the private key is stored in (for generating a new
      request)
   -  Path where the private key can be found (for generating a new
      request)
   -  Format that the certificate is saved in
   -  Path where the certificate was saved

The policy downloader will look at the data taken from these LDB entries
and take appropriate action.



Implementation Details
======================



Proposed Administrative Interfaces
----------------------------------

The following administrative utilities are proposed. These commands are
lacking details. For example the certificate request should contain
information whether we are requesting the certificate for the service or
for the host itself.

::

   | ``   ./ipa request-certificate [--ca=``\ ``] [--request_type=``\ ``] ``
   | ``   where``
   | ``       ``\ ``    'ipa-ca' is default backend plugin accessing IPA's internal CA``
   | ``       ``\ ``      'pkcs10' is a default request type supported by default CA plugin``
   | ``       ``\ ``           certificate request``
   | ``   returning               error_code, error_message, issued_certificate``
   | ``  ``
   | ``   ./ipa revoke-certificate [--ca=``\ ``] [--reason=``\ ``] ``
   | ``   where``
   | ``       ``\ ``    'ipa-ca' is default backend plugin accessing IPA's internal CA``
   | ``       ``\ ``     certificate serial number of the certificate to be revoked``
   | ``       ``\ `` certificate revocation reason``
   | ``   returning               error_code, error_message``
   | ``  ``
   | ``   ./ipa take-certificate-off-hold [--ca=``\ ``]  ``
   | ``   where``
   | ``       ``\ ``    'ipa-ca' is default backend plugin accessing IPA's internal CA``
   | ``       ``\ ``     certificate serial number of the certificate to be taken off hold``
   | ``   returning               error_code, error_message``
   | ``  ``
   | ``   ./ipa check_request_status [--ca=``\ ``] ``
   | ``   where``
   | ``       ``\ ``    'ipa-ca' is default backend plugin accessing IPA's internal CA``
   | ``       ``\ ``        is request id of the request to be verified``
   | ``   returning               error_code, error_message, certificate_serial_number``
   | ``  ``
   | ``   ./ipa get-certificate [--ca=``\ ``] ``
   | ``   where``
   | ``       ``\ ``    'ipa-ca' is default backend plugin accessing IPA's internal CA``
   | ``       ``\ ``     certificate serial number (of previously generated certificate) to be retrieved``
   | ``   returning               error_code, error_message, issued_certificate``



Multiple CA Support
-------------------

You can customize this list to provide required certificate management
capabilities to IPA.

The IPA server is very extensible and pluggable. In IPA v2 we plan to
embed the CA but that does not mean that other CA's can't be used. To
support other CAs the only change required is a different management
plugin that will implement interfaces described below. The --ca option
allows selecting the plugin that will serve the request. This means that
one would be able to add support to any number of CAs without changes to
UI or CLI.

These are preliminary interfaces. They might need to be extended to
reflect other parameters. For example we would need to add the name of
the service a certificate is requested for.

| ``       def request_certificate(self, certificate_request=None, request_type="pkcs10"):``
| ``           # . . .``
| ``           return (error_code, error_message, issued_certificate)``
| ``  ``
| ``       def revoke_certificate(self, serial_number=None, revocation_reason=0):``
| ``           # . . .``
| ``           return (error_code, error_message)``
| ``  ``
| ``       def take_certificate_off_hold(self, serial_number=None):``
| ``           # . . .``
| ``           return (error_code, error_message)``
| ``  ``
| ``       def check_request_status(self, request_id=None):``
| ``           # . . .``
| ``           return (error_code, error_message, certificate_serial_number)``
| ``  ``
| ``       def get_certificate(self, serial_number=None):``
| ``           # . . .``
| ``           return (error_code, error_message, issued_certificate)``



Configuration Entry
-------------------

The configuration entry most likely will be created under cn=etc. The
configuration entry will contain two attributes:

Standard "manager", "member" or "owner" attribute. This attribute has a
DN syntax. It will point to DN of the group of users that are allowed to
issue certificate commands. By default it will point to DN of the
"admins" group that is created by default at the installation. If not
present reference to "admins" group should be assumed. If we choose
"manager" and want the referential integrity plugin to be able to track
changes we would have to add "manager" attribute to the list of the
attributes tracked by the referential integrity plugin. The "member" and
"owner" attributes are already listed in the referential integrity
attribute.

| ``attribute ( 2.16.840.1.113730.3.8.3.TBD``
| ``   NAME 'hostCApolicy' ``
| ``   DESC 'Policy on how to treat host requests for cert operations.' ``
| ``   EQUALITY caseIgnoreMatch ``
| ``   ORDERING caseIgnoreMatch ``
| ``   SUBSTR caseIgnoreSubstringsMatch ``
| ``   SYNTAX 1.3.6.1.4.1.1466.115.121.1.15``
| ``   SINGLE-VALUE``
| ``   X-ORIGIN 'IPA v2')``

If not present "Never" should be assumed.

Object class will look like this:

| `` objectclass ( 2.16.840.1.113730.3.8.4.TBD``
| ``   NAME 'ipaCAaccess' ``
| ``   STRUCTURAL ``
| ``   MAY (member $ hostCApolicy) ``
| ``   X-ORIGIN 'IPA v2' )``



Additional Research
===================

CA provides three authentication methods to access publishing directory:

| 1. basic authentication - using specified DN with password
| 2. basic authentication over SSL - using specified DN with password
  over SSL
| 3. client authentication - using client certificate over SSL

We have decided to automatically create a special CA administrative user
in pretty much the same way we create the "kdc" account during
installation.

This account will look like this:

| `` # CAadmin, sysaccounts, etc, example.com``
| `` dn: uid=CAadmin,cn=sysaccounts,cn=etc,dc=example,dc=com``
| `` objectClass: account``
| `` objectClass: simplesecurityobject``
| `` objectClass: top``
| `` uid: CAadmin``
| `` userPassword: ...``

The account like this is currently used by KDC to connect to the DS.
Similar approach should be taken by the CA. CA will use this account to
bind to DS.

`Category:FreeIPA v2 <Category:FreeIPA_v2>`__ `Category:What
is <Category:What_is>`__