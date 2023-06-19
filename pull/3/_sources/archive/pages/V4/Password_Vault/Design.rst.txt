Overview
--------

Users often want a way to securely store passwords so they don't have to
remember them. This is often handled by a password keyring type
functionality on a client system. This works fine from a single client
system, but it doesn't work well if a user wants to access their
passwords from multiple systems. A network-based approach allows an
authenticated user to retrieve passwords from multiple client systems.

It is also common for services to require knowledge of passwords or
keys, which is typically handled by storing those secrets locally on the
system where the service is installed. Storing these secrets locally is
inconvenient and has security risk. This inconvenience and risk is also
compounded when multiple instances of the same service require knowledge
of the same secrets. Storing these secrets centrally makes
authorization, distribution and rollover of secrets for services much
easier to maintain.



Use Cases
---------

#. A service wants to securely store passwords in a centralized location
   and have the ability to easily retrieve them programatically when the
   password needs to be used. Nobody other than the service and the
   service administrator should be able to access the password.

#. A user wants to securely store passwords in a centralized location
   and have the ability to easily retrieve them manually when the
   password is forgotten. An escrow key will optionally be available to
   authorized administrators to prevent loss of the passwords.

Design
------

A user or service who archives secrets in the password vault will
encrypt their secrets on the client side prior to sending them to IPA.
This encryption will use a symmetric key that is generated from a vault
password that is only known by the user or service. This approach
ensures that IPA never has direct knowledge of one of the clear-text
secrets that a user or service is storing.

If escrow functionality is desired, an escrow officer will need to be
created. This entails generating an escrow officer keypair that will be
used to encrypt/decrypt the vault encryption key associated with a user
or service. The ability to have multiple escrow officers is possible.
This would allow different groups of users to be assigned to different
escrow officers, which could be used to have different levels of
security with regards to escrowed key access. The escrow officer public
keys must be available to IPA clients. The escrow officer private keys
can be stored on an IPA server, or offline (preferred for security
paranoid situations).

Definitions
~~~~~~~~~~~

-  **Secret** - A *Secret* is a password or key that a user or service
   wants to securely archive in the Password Vault.
-  **Vault Password** - A password that is used by a user or a service
   when archiving or retrieving secrets from the Password Vault. A
   *Vault Password* is used to derive a *Vault Encryption Key* for a
   user or service. The *Vault Password* is only known by the user or
   service who owns it.
-  **Vault Encryption Key** - A symmetric key that is used to encrypt
   secrets that a user or a service stores in the Password Vault. Each
   user or service usually has their own encryption key, though an
   encryption key might be shared among multiple instances of a service
   if desired.
-  **Data Recovery Manager (DRM)** - A Dogtag subsystem that is designed
   for secure archival and retrieval of keys. A *DRM* subsystem will be
   created when the Password Vault feature is enabled.
-  **Secret ID** - An identifier that the user or service uses to
   identify a *Secret*. This identifier is a simple name that is unique
   per user or service. The user or service will use this identifier
   directly when performing archival and retrieval operations.
-  **Client ID** - An identifier that the *DRM* uses as a convenience
   label for an archived key. This identifier is a simple name. The
   *DRM* does not enforce the uniqueness of this identifier, though IPA
   should. This identifier is not exposed to the user or service.
-  **Key ID** - An identifier that the *DRM* uses to identify an
   archived key. This is a unique hexadecimal identifer that the *DRM*
   generates when a key is archived. This identifier is later used
   during key retrieval. This identifier is not exposed to the user or
   service.

.. _authentication_and_authorization:

Authentication and Authorization
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For simplicity, the workflow diagrams in this document do not show
details about authentication or authorization. This section describes
how authentication and authorization are handed.

A user or service authenticates to the IPA Server using Kerberos/GSSAPI
just as they do when running any of the IPA client commands. The IPA
Server is responsible for authorizing a user or service to retrieve or
store a secret.

The IPA Server authenticates to the DRM as a special *Agent* user using
an x.509 certificate. This *Agent* is allowed to archive or retrieve any
secret that is stored in the DRM, though it is important to note that
the secrets are encrypted such that the IPA Server is unable to decrypt
them.

.. _common_workflows:

Common Workflows
~~~~~~~~~~~~~~~~

The following workflows apply to the common use-cases of archiving and
retrieving secrets from the Password Vault.

Archival
^^^^^^^^

The following diagram illustrates the workflow that is used when a user
or service wants to archive a secret in the Password Vault. From the
user or service's point of view, they simply supply their *Vault
Password*, the *Secret* that they want to store, and the *Secret ID*
that they want to use to identify the *Secret* for future retrieval.

.. figure:: Password_Vault_Archival.png
   :alt: Password_Vault_Archival.png

   Password_Vault_Archival.png

Retrieval
^^^^^^^^^

The following diagram illustrates the workflow that is used when a user
or service wants to retrieve a secret from the Password Vault. From the
user or service's point of view, they simply supply their *Vault
Password* and the *Secret ID* that identifies the *Secret* that they
want to retrieve. They will be given the clear-text *Secret* in
response.

.. figure:: Password_Vault_Retrieval.png
   :alt: Password_Vault_Retrieval.png

   Password_Vault_Retrieval.png

.. _vault_password_reset:

Vault Password Reset
^^^^^^^^^^^^^^^^^^^^

-  User uses IPA CLI reset their vault password.
-  User supplies old vault password, which derives the current symmetric
   encryption key on the client side.
-  User supplies a new vault password, which derives a new symmetric
   encryption key on the client side.
-  Client CLI contacts IPA to retrieve all stored secrets.
-  Retrieved secrets are decrypted using the current encryption key,
   then re-encrypted using the new encryption key.
-  Newly encrypted secrets are sent to IPA as archival requests.

.. _service_workflows:

Service Workflows
~~~~~~~~~~~~~~~~~

The following workflows apply to the use case where a service is storing
secrets in the Password Vault. These workflows assume that the process
of a service retrieving and decrypting secrets is handled in an
automated fashion. In particular, this changes things with respect to
the use of a vault password, as there is no way for the service to
interactively provide this password to encrypt/decrypt secrets.

Instead of simply storing the service's vault password (that might be
shared by multiple instances of a service) in a file on the system where
the service is installed, public key cryptography is used to allow the
service to securely retrieve it's vault password from the password
vault. This requires each instance of a service to have it's own
public/private key pair that a service administrator sets up on the
system where the service runs. The private key could be protected by
file system permissions to allow it to be used in a completely automated
fashion, or it could be password protected (requiring it to be manually
unlocked each time the service is started).

A vault password for a service (or a group of services that need to
access the same archived secrets) is stored in the password vault using
a service administrator vault password. This is done using the user use
case workflow described above. This allows the service administrator to
retrieve the service vault password when they set up a new instance of
the service. The service administrator can then archive a copy of the
service vault password that is encrypted using the service instance's
public key so it can only be decrypted by that single instance of the
installed service. This approach allows an administrator to add or
remove instances of a service without affecting other instances of the
same service. It also allows an admin to change the service vault
password for all instances of a service by archiving encrypted copies
the new vault password with the public keys of the service instances.

.. _archival_of_a_service_vault_password_at_service_set_up_time:

Archival of a service vault password (at service set up time)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following diagram illustrates the workflow that is used when a
service administrator is setting up a new instance of a service. This
workflow assumes that a few prerequisite tasks have been completed:

-  The service administrator has the desired *Service Vault Password*
   archived as one of their personal secrets. This can be completed
   using the normal archival workflow.
-  The new service instance has been defined in IPA.
-  IPA has issued a certificate for the new service instance.

From the service administrator's point of view, they simply supply their
*Vault Password*, the *Secret ID* that identifies the *Service Vault
Password* that is shared by all instances of the service, and the
*Service Principal* that identifies the instance of the service that
they are setting up. The result is that a copy of the *Service Vault
Password* is archived that can only be retrieved and decrypted by the
new service instance.

.. figure:: Password_Vault_Service_Password_Archival.png
   :alt: Password_Vault_Service_Password_Archival.png

   Password_Vault_Service_Password_Archival.png

.. _retrieval_of_service_vault_password_automated:

Retrieval of service vault password (automated)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  The service retrieves it's copy of the encrypted service vault
   password from the password vault.
-  The service decrypts the service vault password using it's private
   key. This decrypted vault password can be used to decrypt retrieved
   secrets that the service is allowed to access.

.. _escrow_workflows:

Escrow Workflows
~~~~~~~~~~~~~~~~

.. _archival_1:

Archival
^^^^^^^^

-  If escrow functionality is desired, the IPA client framework
   retrieves escrow officer public key from LDAP and uses it to encrypt
   the new encryption key. This encrypted encryption key is sent to IPA
   over a GSSAPI protected connection and is stored in the DRM.

.. _retrieval_escrow_officer_initiated:

Retrieval (escrow officer initiated)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  Authenticated escrow officer requests administrative retrieval of a
   user's secret from IPA CLI.
-  IPA receives the administrative retrieval request and requests
   retrieval from the DRM using the agent certificate to authenticate.
-  IPA receives the encrypted secret from the DRM and sends it back to
   the escrow officer.
-  IPA retrieves the user's encrypted escrowed encryption key and sends
   it back to the escrow officer.
-  IPA client framework decrypts the encrypted encryption key using the
   escrow officer's private key on the escrow officer's workstation.
-  IPA client framework uses the encryption key to decrypt the secret
   and presents it to the escrow officer.

.. _vault_password_reset_with_escrowed_encryption_key:

Vault Password Reset (with escrowed encryption key)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  User forgets their vault password.
-  User requests to reset their vault password from CLI.
-  User supplies a new vault password, which derives a new symmetric
   encryption key on the client side.
-  IPA client framework retrieves escrow officer public key from LDAP
   and uses it to encrypt the new encryption key.
-  Encrypted new encryption key is sent to IPA over a GSSAPI protected
   connection.
-  IPA accesses the online private escrow officer key that is associated
   with this user.

   -  For the offline private key case, admin intervention is required
      here to perform the stored secret rewrapping. This will require a
      two-step process or an admin initiated password reset.

-  Escrowed user encryption key is fetched and decrypted using the
   private escrow officer key on the IPA server.

   -  This would happen on the escrow officer workstation for the
      offline private key case.

-  New user encryption key is decrypted using the private escrow officer
   key on the IPA server.

   -  This would happen on the escrow officer workstation for the
      offline private key case.

-  User's stored secrets are extracted from the DRM, decrypted using the
   escrowed user encryption key, and re-encrypted using the new user
   encryption key. The re-encrypted secrets are stored in the DRM,
   replacing the old secrets. This all takes place on the IPA server.

   -  This decryption/encryption would happen on the escrow officer
      workstation for the offline private key case. The escrow officer
      would need to be allowed to retrieve and store secrets for users
      they are assigned to.

-  New encryption key (encrypted with the escrow officer's public key)
   is stored, replacing the old escrowed encryption key.

Implementation
--------------

The `implementation details <V4/Password_Vault_Implementation>`__ still
need to be defined. Consider this section as a work in progress.

.. _drm_namespace:

DRM Namespace
~~~~~~~~~~~~~

The namespace used to store secrets in the DRM needs to be defined. We
likely want a naming scheme that gives each user a namespace. One
possible naming scheme is:

``-``

.. _ldap_implementation_details:

LDAP Implementation Details
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _escrow_officer_assignment:

Escrow Officer Assignment
^^^^^^^^^^^^^^^^^^^^^^^^^

The LDAP schema used to assign an escrow officer to users and groups
needs to be defined.

.. _stored_secret_tracking:

Stored Secret Tracking
^^^^^^^^^^^^^^^^^^^^^^

We will need to track the secrets that are stored for users in LDAP. The
main reason for this is to keep track of the names of secrets that a
user has stored in the DRM. This will allow IPA to perform some basic
access control checks, such as rejecting attempts to retrieve a secret
that IPA is not aware of.

Having entries that track the stored secrets will also allow us to have
some more advanced access control, such as host-based restrictions. An
LDAP entry that represents a stored secret could hold attributes that
list hosts or hostgroups that are allowed to retrieve a stored secret.

The LDAP schema and entry location for this needs to be defined.



Feature Management
------------------

UI
~~

TBD

CLI
~~~

TBD



Major configuration options and enablement
------------------------------------------

The password vault should be an optional feature that is enabled at
install time, or afterwards. This will require changes to
ipa-server-install as well as a new command such as ipa-enable-vault.
When a vault is desired, a new DRM instance will need to be created and
configured using pkispawn.

Replication
-----------

The password vault should be replicated for redundancy. This would
leverage the existing Dogtag cloning functionality for the DRM
subsystem. This would require separate replication agreements since the
DRM internal database would be stored in a separate suffix in 389 DS.
This would allow some FreeIPA replicas to have a password vault while
other do not.



Updates and Upgrades
--------------------

The password vault requires that a DRM instance is created on each
FreeIPA replica. If an existing IPA replica does not have a DRM created,
the new functionality will not be available. We should not create a DRM
at upgrade time. Instead, one would have to manually enable the password
vault feature on existing FreeIPA replicas, which would trigger DRM
creation.

Currently, the Dogtag DRM subsystem requires a CA subsystem. This means
we will not be able to offer password vault functionality in a CA-less
installation. There are plans for Dogtag to support a standalone DRM,
which will allow FreeIPA to have a password vault without a CA.

Dependencies
------------

The password vault would use the Dogtag DRM subsystem. FreeIPA already
relies on Dogtag for it's CA subsystem, but the DRM subsystem is
contained in an additional package. This dependency should be optional
since not everyone would have the password vault functionality enabled.



External Impact
---------------

Development of this functionality will require working closely with the
Dogtag development team. There might be new DRM interfaces that are
required by this feature that will require RFEs for Dogtag. One such RFE
that is known is support for a standalone DRM subsystem to allow it to
work in a CA-less FreeIPA installation.



Backup and Restore
------------------

The password vault will need to be backed up. This will require backing
up the DRM data from LDAP, DRM certificate databases, and DRM
configuration files.



Test Plan
---------

-  `Test
   Plan <http://www.freeipa.org/page/V4/Password_Vault/Test_Plan>`__

References
----------

-  `JavaScript
   crypto <https://developer.mozilla.org/en-US/docs/JavaScript_crypto>`__
-  `keygen <https://developer.mozilla.org/en-US/docs/Web/HTML/Element/keygen>`__
-  `crypto-js <http://code.google.com/p/crypto-js/>`__
-  `Forge <https://github.com/digitalbazaar/forge>`__
-  `Salted Password Hashing - Doing it
   Right <https://crackstation.net/hashing-security.htm>`__
