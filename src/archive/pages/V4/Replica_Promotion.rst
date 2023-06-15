Overview
--------

The current method to install a replica requires a 2 phases approach
where an admin physically logs into a master and generates an
installation package, then transfers it to a new server and performs the
replica install procedure. This method is cumbersome, require
interaction (the ipa-replica-prepare command wants the Directory Manager
password) and is generally hard to deal with in an automated fashion. A
new method to promote a regular client to a replica and in general
simplify replica installs will make a number of maintenance operations
easier. A new replica promotion sequence will allow easier provisioning
via an external infrastructure management system, while retaining a
reasonable level of security (or increasing the security of the solution
in some areas).



Use Cases
---------

-  Automatic provisioning and installation of replicas (automatic load
   balancing for example) will be simpler to drive, without direct
   access to the *Directory Manager* credentials by the provisioning
   system.
-  Admins will be able to stand up and then immediately promote servers
   to replicas without having to interactively log in as root into an
   existing master, a single command will be able to install a full
   replica.

Design
------

.. _bootstrapping_problem:

Bootstrapping problem
~~~~~~~~~~~~~~~~~~~~~

The only real issue in allowing a streamlined installation for a replica
is that we need “some” trusted credentials that allow to create the
chain of trust necessary to allow a “self-join” to happen. So no matter
what is the technical "transport” mechanism, eventually a secret need to
be landed in the replica before it can be joined to a domain. There are
2 possible approaches with different properties:

-  A privileged account (and related credentials) that is provided to
   the replica
-  A pre-created set of credentials for a specific replica (OTP or
   keytab)

The following is a pro's and con's tab of using a special privileged
“Replica enrollment” account versus creating short term temporary
credentials

+----------------------+----------------------+----------------------+
|                      | Privileged Account   | Preset credentials   |
+======================+======================+======================+
| Pre-install step     | Not required         | Required             |
+----------------------+----------------------+----------------------+
| Orchestration access | Not required         | Required             |
| to master            |                      |                      |
+----------------------+----------------------+----------------------+
| Re-installation on   | Always possible      | Requires interaction |
| failure              |                      | with master to reset |
|                      |                      | OTP / Keytab         |
+----------------------+----------------------+----------------------+
| Credentials handling | Long term secure     | Short term secure    |
|                      | storage required     | storage required     |
+----------------------+----------------------+----------------------+

The downside of using a privileged account is that it is critical to
keep these credentials locked up and disclosed exclusively to the
replica image at install time. However this also means external
provisioning software needs no access to the credentials (it merely
needs a way to make them available which could be accomplished by a
separate privileged process dedicated to this kind of operations). The
use of a OTP/Keytab prepared ad hoc for the new replica may look safer.
But it is not so, an OTP or a keytab that has been given the power to
escalate all the way to become a replica is extremely sensitive as those
credentials, if stolen, can be used to get access to all keys in the
domain. The situation is completely different from a OTP client install
where the consequence of stealing such credentials is a limited access
to the directory and no access to any important key material. On top of
this preparing an OTP requires the orchestration service to have
privileged access to an IPA server.

Both methods are useful, a manual installation performed by an
administrator can do all operations in one sweep, while a provisioning
system might prefer to provide only credentials that are temporarily
valid to an unattended system, so that provisioning failures do not risk
exposing long term, privileged credentials in logs or on disks.

.. _bootstrapping_workflows:

Bootstrapping Workflows
^^^^^^^^^^^^^^^^^^^^^^^

.. figure:: Replica_bootstrap_1.svg
   :alt: Replica_bootstrap_1.svg

   Replica_bootstrap_1.svg

.. _secrets_sharing_problem:

Secrets sharing problem
~~~~~~~~~~~~~~~~~~~~~~~

Regardless which bootstrapping method is chosen there are a number of
secrets/keys known by an IPA server that need to be shared between all
masters.

Secrets that need to be shared in a safe way:

-  Currently the *Directory Manager* password is shared among all
   servers, this proposal will try to eliminate any reason to obtain the
   domain clear text *Directory Manager* password for setting up a
   replica, but it is possible that some operation will require it.
-  The RA agent and other certificates, currently are copied from master
   to replica and again each time they are renewed they need to be
   shared. This design document aims at eliminating private keys sharing
   as much as possible, but it will still be needed in some cases.
-  `Kerberos <Kerberos>`__ Master key (though this can be deferred, it
   would be really nice if we could move it out of LDAP and store it in
   the file system so that only the KDC/Kadmin process has access to
   it).
-  `CA <PKI>`__ private key (for CA replicas only)
-  DNSSEC keys (already implemented ?)

A mechanism needs to be provide such that only legitimate replicas can
get access to private keys and other highly sensitive secrets.

.. _new_replica_install_workflow:

New replica install workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This feature may depend on a new features, like a plugin to `manage the
replication topology <V4/Manage_replication_topology>`__. It requires
changes in the order in which tasks are performed as part of the
ipa-replica-install tool, but some changes will be conditional to the
`Domain Level <V4/Domain_Levels>`__ (a Level 0 domain will still need to
use prepare files for example).

Installation tools in FreeIPA are built as group of steps performed to
install a subsystem instance. Each subsystem installation is usually
self-contained but ordered such that each macro-step has all necessary
dependencies installed in previous ones.

The new workflow aims at simplifying replica bootstrap and reducing the
amount of data to be gathered upfront in a "prepare" step to a single
set of credentials, either a privileged admin account or a keytab.

Once we have credentials the ipa-replica-install tool will be employed
to install all parts as usual, but the installation order will be
substantially changed from the current one in order to harmonize
installation regardless of which type of initial credentials are
provided.

A key change in the workflow will be that the `CA <PKI>`__ and
`DNS <DNS>`__ (and in future Vault and other) components will be
considered optional and pushed to the end of the installation phase.
They will also not be directly integrate within the ipa-replica-install
tool per-se but rather the DNS, CA, etc.. own installer will be invoked
at the end of a fully successful and functional basic replica install.
This will allow a failure in an optional component to be remediated
separately and will not force a full reinstall of the replica.

.. _ipa_replica_conncheck:

ipa-replica-conncheck
^^^^^^^^^^^^^^^^^^^^^

Currently, ipa-replica-conncheck is being run at the start of the
replica installation process. The goal is to verify that both
replica-server and server-replica communication works for all required
protocols. Without this check, ipa-replica-install may fail in
unexpected ways.

ipa-replica-conncheck tests the replica-server part by simply opening
ports to the remote server. To check the server-replica direction, it
SSH's as *admin* to the remote server and then run the checker from the
server to replica. However, this does not run well with streamlining the
replica installation and avoiding passing additional secrets to the
installer.

The connection check should be instead transformed in an API call that
will check server-replica part. Note that SELinux policy will need to be
updated to allow ``httpd`` connecting to the remote FreeIPA ports [or a
new service that can be instantiated via the system message bus be
created]. As this is FreeIPA specific, the additional policy should be
based on ``httpd_manage_ipa`` conditional.

.. _ipa_client_install:

ipa-client-install
^^^^^^^^^^^^^^^^^^

The very first step will involve calling ipa-client-install (unless we
are promoting an already installed client). The final step of the client
install procedure will be to rotate the host keytab if the install
credentials were keytab based.

.. _directory_server_initialization:

Directory Server initialization
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The first step of the new replica install procedure will involve
installing the Directory Server, however given the
`Kerberos <Kerberos>`__ infrastructure is already available, and a host
keytab is available, the `Directory Server <Directory_Server>`__ install
does not depend on having LDAPS available. Instead the new topology work
will be leveraged to join the 389 server directly using GSSAPI for
setting up replication agreements. This will avoid replication
agreements conversions later on. The new `Directory
Server <Directory_Server>`__ instance installation procedure will
perform the following steps:

-  Retrieve ldap/fqdn keytab and drop it in /etc/dirsrv/ds.keytab

      Will be used to setup replication agreements

-  Generate random *Directory Manager* password

      Will be used to perform local installation steps that may still
      require a *Directory Manager* password. The random *Directory
      Manager* password will be discarded when the installation is over.

-  Stand up the `Directory Server <Directory_Server>`__ Instance using
   the new replication topology facilities
-  Make sure that the replication agreements are set up by the Topology
   plugin and initialize the tree from remote server

   -  When assigning a *replica ID* to the replica, make sure that the
      change is done as *add&delete* LDAP update and not LDAP *replace*
      to make sure that there is not a race when multiple replicas are
      being installed
      (`#4378 <https://fedorahosted.org/freeipa/ticket/4378>`__). When
      the update fails, script may retry for defined number of times
      (e.g. 10). This does not cover the case when replicas are being
      installed against different masters, this situation does not need
      to be solved in this RFE.

-  Finally use `Kerberos <Kerberos>`__ credentials to request a X509
   cert and configure `Directory Server <Directory_Server>`__ to also
   provide TLS support

.. _kerberos_kdc_initialization:

Kerberos KDC initialization
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The current `KDC <Kerberos>`__ instance setup will be simplified, mostly
removing code that retrieves the LDAP and host keys, which we already
have at this point.

.. _other_core_components_initialization:

Other core components initialization
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Most other core components like, the HTTP framework, memcache service,
NTP server, will need no or minor changes. For example:

-  The HTTP instance will copy the preference.html file from another
   master by simply fetching it via HTTPS, the configure.jar file will
   also be copied or regenerated the same way. Note: configure.jar and
   preferences.html are used for configuration of Kerberos of ancient
   Firefox versions (<10). It could be safely removed. Manual
   configuration steps are provided.
-  The default.conf installed by ``ipa-client-install`` will need to be
   updated to also contain the server settings.
-  The necessary DNS records (SRV, TXT, NS) will have to be added during
   this phase too, as there is no "prepare phase" in which to add them
   anymore. Note that forward (A, AAAA) DNS records are expected to be
   created during ipa-client-install phase.
-  The CA server and ports will need to be detected via checking in LDAP
   and probing and stored in this phase as well.

.. _key_sharing_component_initialization:

Key sharing component initialization
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The last step of the main replica installation phase will involve
installing a new internal service that handles the management and
transfer of core key material. Fundamental installation steps for this
component:

-  generate public/private key-pair for the replica (to be stored in
   HSM/SoftHSM)
-  store the public key in LDAP so that all master have access to it.
-  request any necessary secrets using own key

For example we may want to request a hash of the *Directory Manager*
password so that all servers have the same password for admins
convenience.

.. _dns_installation:

DNS Installation
^^^^^^^^^^^^^^^^

If required the normal ipa-dns-install script is executed

-  changes to this script are TBD

.. _ca_installation:

CA installation
^^^^^^^^^^^^^^^

The CA installation procedure will be changed to require less secrets be
shared between clones and also avoid the need for obtaining the
*Directory Manager* password.

.. _no_clear_text_dm_password_for_the_install:

No clear text DM password for the install
'''''''''''''''''''''''''''''''''''''''''

In order to set up the necessary schema and options in the replica
cn=config database the CA install scripts need access as the Directory
Manager. Most of this process is not done as the root user but rather as
the pkiuser after the tomcat VM has been bootstrapped. Options to avoid
obtaining the real *Directory Manager* password for the `Directory
Server <Directory_Server>`__ instillation step:

-  Enhance `Directory Server <Directory_Server>`__ to be able to map
   no-root users to the *Directory Manager*, in order to use LDAPI from
   the pkiuser
-  Allow multiple *Directory Manager* password and generate a new random
   one
-  Save the real password hash and temporarily replace with a new random
   one
-  Externalize configuration servlet into a standalone tool that can be
   run as root (LDAPI)

Temporarily replacing the *Directory Manager* password with a random one
at CA install time is a bit hackish, but can be done todaywithout any
changes to `Directory Server <Directory_Server>`__ or `CS <PKI>`__.

.. _multiple_admin_users:

Multiple Admin users
''''''''''''''''''''

The ipa install scripts will create a new `CS <PKI>`__ Admin (Security
Domain User) user for each CA replica, will assign this user a random
password and add the user to the `CS <PKI>`__ Admins group. This will
avoid the current practice of assigning the *Directory Manager* password
as the Admin user password and having to share it with the clone in
order for the clone to obtain an installation token. In future we may
need to store the admin user password into the key service process, but
for now we can simply destroy the password at the end of the install
procedure as the admin user is not used for now. (It may be needed in
future to allow the creation of subCAs, but we can generate a new secret
on updates if needed).

.. _certificates_and_public_key_wrapping:

Certificates and public key wrapping
''''''''''''''''''''''''''''''''''''

Given secrets will be transferred via the privileged key service, there
will be no need to use the *Directory Manager* password to wrap the p12
file containing the CA and other certs.

.. _ca_certificates:

CA Certificates
'''''''''''''''

Currently most of the internal certificates (and their keys) used by the
CA are copied to replicas. This is not strictly necessary and from an
auditing/security point of view it may even be desirable to avoid.
Keeping keys private to replicas also makes it much easier to renew
expiring certificates and avoid the need to transfer certificates around
every time one is renewed, each replica is responsible for its own on
its own schedule.

The following certificates will be generated on each replica:

-  **Subsystem certificate**: this certificate is used for internal CA
   communications and there can be one per replica
-  **Audit certificate**: the audit certificate is used to sign the
   audit log, having a certificate per replica will insure the auditors
   can verify which replica generated each auditable event, improving
   the auditability of the CA.
-  **Server certificate**: this is already per server today, no changes
-  **RA Agent Certificate**: A new agent user will be created on each
   replica, added to the RAs group, and a new certificate per replica
   will be created
-  **OCSP certificate**: some more research needs to be done, but it
   appears that multiple OCSP certificates can coexist. This certificate
   is optional (necessary only when the CRL/OCSP role is transferred) so
   this cert will be generated only when needed and not copied over from
   the initial CA

Certificates/keys that still need to be transferred to replicas:

-  **CA signing key**: in order to be the same CA all replicas need a
   copy of the CA key
-  **KRA certificates**: The storage certificate must be shared between
   KRA/Vault servers so that all servers can encrypt/decrypt the
   storage. The transport key certificate could be generate per replica
   but that would make clients more complex as they would need to know
   which one to use for each replica making load-balancing and fallback
   clients much harder.

.. _sharing_secrets_securely:

Sharing Secrets Securely
~~~~~~~~~~~~~~~~~~~~~~~~

Requirements:

-  All secrets must be encrypted so that only the target replica can get
   access to them. The most straightforward way to achieve that is to
   use public key crypto and a replica's public key to wrap a package
   containing secrets to be shared.
-  Replicas must be able to get a secret on demand.

Examples:

-  A replica is promoted to be a `DNS <DNS>`__ server (needs access to
   the master DNSKEY for the first time)
-  A replica is promoted to be a `CA <PKI>`__ server (needs access to CA
   private key)
-  A replica is promoted to be a Vault server (needs KRA storage and
   transport keys)

.. _secrets_sharing_service_custodia:

Secrets Sharing Service (Custodia)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A Custodia service is needed to handle encryption/decryption and
delivery on both the sending side and the receiving side; this service
is the only component that have access to the replica's own private
keys/secrets. (this might also be based on softHSM work already done).

An authenticated communication mechanism between a remote replica and
the Custodia service is required. There are two options:

-  A request using a specific principal using a GSSAPI channel
-  A request package, signed by the remote replica private key [this may
   be preferable given everything else in the mechanism also uses
   public-key crypto]

A good solution is to actually use both:

-  one to secure/authenticate the transport layer between servers
-  the other to secure and add an additional authorization level to the
   exchange

.. _transport_mechanisms:

Transport mechanisms
^^^^^^^^^^^^^^^^^^^^

Requirements:

-  authentication and access control
-  do not expose the Custodia service directly to the network

External
''''''''

-  HTTP API.

The API is exposed via mod_proxy which takes care of Negotiate
authentication for additional access control and forwards requests to a
local Custodia process listening on a Unix Socket.

Internal
''''''''

The Custodia daemon will listen for HTTP requests on a local Unix
Socket. Requests will be partially authenticated by the frontend
mod_proxy process, and all communication will depend on additional
signature verification on each request. The verification will be done
via public key published in IPA's LDAP server and retrieved based on the
Kerberos Principal used to authenticate to the apache server.

.. _exchange_flow:

Exchange Flow
^^^^^^^^^^^^^

#. Requesting replica prepares a request package and signs it with the
   private key.
   The package is a JOSE object, signed by the client and encrypted to
   the server's public key. The payload is a JWT Claims Set with the
   following claims

   -  A timestamp to prevent replay attacks ("exp" claim)
   -  The name of the key being requested ("sub" claim)

#. Authenticate to other master using host keytab and send package. The
   key being sought determines the request path (see `#Handlers (reading
   and writing keys) <#Handlers_(reading_and_writing_keys)>`__ for
   details).
#. The receiver checks credentials and passes the package to the
   privileged key service

   -  The credentials must be of a principal in the master's group

#. The privileged key service gathers the requested data and creates a
   reply package
   The package is a JOSE object that includes:

   -  A timestamp
   -  The requested key material

#. The package is encrypted with the replica public key and sent back
#. The replica decrypts and verifies the package and store the keys in
   the local (soft)HSM or where appropriate, based on the secret type.

.. figure:: Replica_KISS_1.svg
   :alt: Replica_KISS_1.svg

   Replica_KISS_1.svg

.. _ldap_dit_layout:

LDAP DIT Layout
^^^^^^^^^^^^^^^

Two objects for each for each IPA Server: one signing key, and one
encryption key.

::

   dn: cn={sig,enc}/{fqdn},cn=custodia,cn=ipa,cn=etc,dc=ipa,dc=local
   objectClass: nsContainer
   objectClass: ipaKeyPolicy
   objectClass: ipaPublicKeyObject
   objectClass: groupOfPrincipals
   objectClass: top
   cn: {sig,enc}/rhel76-1.ipa.local
   ipaKeyUsage: {digitalSignature,dataEncipherment}
   memberPrincipal: host/{fqdn}@{realm}
   ipaPublicKey:: <DER encoded SubjectPublicKeyInfo>

.. _handlers_reading_and_writing_keys:

Handlers (reading and writing keys)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the original implementation the ``ipa-custodia`` server and Custodia
client code ran as root and was not properly confined by SELinux (see
`Ticket 6888 <https://pagure.io/freeipa/issue/6888>`__). As of FreeIPA
4.8.0, server and client key database handlers are run as separate
processes, as a non-root user where possible, and the ``DAC_OVERRIDE``
capability is no longer required for either the main process or the
handler processes.

+-----------------+-----------------+-----------------+-------------+
| Request path    | Handler         | Purpose         | Run as      |
| (/ipa/keys/...) | (/usr/lib/ip    |                 |             |
|                 | a/custodia/...) |                 |             |
+=================+=================+=================+=============+
| dm/DMHash       | ipa-            | Directory       | root        |
|                 | custodia-dmldap | Manager         |             |
|                 |                 | password        |             |
+-----------------+-----------------+-----------------+-------------+
| ca/{nickname}   | ipa-cust        | CA, OCSP,       | **pkiuser** |
|                 | odia-pki-tomcat | subsystem,      |             |
|                 |                 | audit and KRA   |             |
|                 |                 | keys            |             |
+-----------------+-----------------+-----------------+-------------+
| ca_wrappe       | i               | Lightweight CA  | **pkiuser** |
| d/{nickname}\ * | pa-custodia-pki | (sub-CA) key    |             |
| *[/{alg-oid}]** | -tomcat-wrapped | replication     |             |
+-----------------+-----------------+-----------------+-------------+
| ra/ipaCert      | ipa-cu          | IPA RA agent    | root        |
|                 | stodia-ra-agent | key             |             |
+-----------------+-----------------+-----------------+-------------+

The optional **{alg-oid}** parameter for Lightweight CA signing keys
requests the specified encryption algorithm be used. This was
implemented as part of `Ticket 8020 - Support AES in Lightweight CA key
replication <https://pagure.io/freeipa/issue/8020>`__. This "parameters
as additional path components" facility is available to all handlers,
but only ``ca_wrapped`` uses it (as of September 2019).

Upgrades
~~~~~~~~

To make sure that upgraded replicas can be used as the source servers
for spawning replicas, all necessary infrastructure will need to be
prepared during the upgrade phase, including replica private/public key
generation.



Backward Compatibility
~~~~~~~~~~~~~~~~~~~~~~

Backport candidates for better compatibility with older FreeIPA or
`Directory Server <Directory_Server>`__ versions:

-  `#47667: Allow nsDS5ReplicaBindDN to be a group
   DN <https://fedorahosted.org/389/ticket/47667>`__



Feature Management
------------------

There are no new UI or CLI features associated with replica promotion or
management of Custodia keys or secrets. The replica installation user
experience is simplified compared to the old procedure (i.e.
``ipa-replica-prepare`` is not needed, nor is the replica-file option).

.. _how_to_test32:

How to Test
-----------

Prerequisites
~~~~~~~~~~~~~

Two machines that will become IPA masters.

Testing
~~~~~~~

1. Install a first IPA server - or - upgrade an existing one to the bits
including this feature and raise the domain level to 1.

For better coverage install the DNS server and KRA servers too.

2. Join the future replica as an ipa client with the normal
ipa-client-install command

2.b(optional) kinit as admin and check everything works fine

3. Run ipa-replica-install for better coverage feel free to pass in
--setup-dns --setup-ca --setup-kra or any combination of these flags.

3.b(optional) kinit as a user, check the logs to verify it was
authenticated by the KDC running on the replica.

3.c(optional) turn off the initial master and verify every major
subsystem (KDC, DNS, CA, KRA) keeps working as expected.



Test Plan
---------

`Test plan is
here <http://www.freeipa.org/page/V4/Replica_Promotion/Test_plan>`__
