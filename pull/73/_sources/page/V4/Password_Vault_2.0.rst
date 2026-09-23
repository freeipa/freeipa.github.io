

Notice - Design Document
========================

This document is a design document for a **future** version of Password
Vault.

For existing deployments, please see `the current
implementation's <https://www.freeipa.org/page/V4/Password_Vault_1.2>`__
page instead.

Overview
========

This page describes the full implementation of `Password
Vault <V4/Password_Vault/Design>`__. The content may still change due to
an active development.

Secret
------

Secret is an information that should only be known by a limited number
of people. IPA provides a mechanism to archive, retrieve, share, and
recover secrets, and manage the access control. The secret itself will
actually be stored securely in the KRA backend.

A secret is a binary data, so anything can be stored as a secret, but
there will be a limit imposed by the server. A secret ID is a unique
name to identify a secret within a vault (e.g. email, pin).

During archival, the client will generate a random session key and use
it to encrypt the secret, then send both the encrypted secret and the
session key wrapped with KRA's transport public key to KRA. KRA will
unwrap the session key with the transport private key, then use it to
decrypt the secret. Then KRA will encrypt the secret with the storage
key and store it in the LDAP backend.

During retrieval, the client will generate a random session key and send
it to the server wrapped with KRA's transport public key. KRA will
decrypt the secret stored in the LDAP backend using the storage key,
return the secret encrypted using the session key.

Vault
-----

Vault is a secure place to store data or a collection of secrets. A
vault may be privately owned by a user, or shared among users, groups,
or services. A user may have multiple vaults.

Vault name is an identifier for the vault that is unique within a vault
container (e.g. PrivateVault is unique within /users/testuser). Vault ID
is a globally unique identifier which consists of vault name and the
container ID (e.g. /users/testuser/PrivateVault).

There are three types of vault which may offer additional level of
security:



Standard Vault
----------------------------------------------------------------------------------------------

A standard vault will use the KRA's standard transport and storage
encryption as described above. Any authorized vault members, vault
owners, or escrow officers can read and write the secrets without having
to use a password/keys.



Symmetric Vault
----------------------------------------------------------------------------------------------

With symmetric vault the secrets are additionally protected with a
symmetric key generated from a vault password. The client will encrypt
secrets before sending them to the server and decrypt them after
receiving them from the server. Any authorized vault members, vault
owners, or escrow officers can read and write the secrets, but they will
have to use a shared vault password.



Asymmetric Vault
----------------------------------------------------------------------------------------------

With asymmetric vault (drop box) the secrets are additionally protected
with asymmetric keys. The client will archive the secrets using the
public key, and retrieve using the private key. The public key is
available to all vault members, but the private key is only available to
vault owners. This way vault members can archive secrets, but only vault
owners/escrow officers can retrieve them.

Container
---------

Container is a hierarchical collection of vaults and other containers.
Container name is an identifier for the container that is unique within
the parent container (e.g. testuser is unique within /users). Container
ID is a globally unique identifier that consists of the container name
and the parent container ID (e.g. /users/testuser).

There are several built-in vault containers:

-  root container: /
-  container for user's private vaults: /users
-  container for group's private vaults: /groups
-  container for shared vaults: /shared
-  container for service vaults: /services

The root container is owned by the admins group.

The /users container is owned by the admins group. When a user is added,
a private container (/users/) will be created automatically for this
user. The user will own the container and be able to create private
vaults (/users//) in it. When the user is removed, the private container
and its contents will be removed automatically.

The /shared container is owned by the admins group. The container has
ipausers as a member so all users can access shared vaults. Only the
admins can create shared vaults (/shared/).

The /services container is owned by the service admins group so any
service administrator can create sub-containers for the servers
(/services/) and the service vaults in it (/services//).



Access Control
--------------

User's role in determines the operations that it can execute:

-  A non-member cannot access the container or vault.
-  A container member can list sub-containers and vaults in the
   container.
-  A container owner can create and remove sub-containers and vaults in
   the container, and manage the members and owners of the container,
   but it cannot remove the container itself.
-  A vault member can list, archive, and retrieve the secrets in a
   standard vault. With symmetric vault the member will need a vault
   password in order to archive and retrieve secrets. With asymmetric
   vault the member can only archive the secrets but not retrieve them
   since it only has the public key and not the private key.
-  A vault owner can list, archive, and retrieve the secrets like vault
   members, but it has the private key so it can retrieve secrets from
   asymmetric vault. The owner can also manage the list of members and
   owners of the vault, and change the vault password/keys.
-  An escrow officer can recover secrets and reset the vault password.
-  An administrator can do everything except accessing the secrets in
   symmetric or asymmetric vaults and change/reset the password/keys.

Installation
============

First install IPA server:

::

   $ ipa-server-install
   ...

Then install KRA component:

::

   $ ipa-kra-install
   ...

Authenticate as an IPA user:

::

   $ kinit testuser
   Password for testuser@EXAMPLE.COM: ********

The vault is ready to use.



Container Managerment
=====================



Listing available containers
----------------------------

Any user can list the sub-containers within a specified container using
the following command:

::

   $ ipa vaultcontainer-find [parent ID] [OPTIONS]

If the parent ID is not specified, it will return the user's private
containers:

::

   $ ipa vaultcontainer-find
   --------------------------
   2 vault containers matched
   --------------------------
     Container name: personal
     Container ID: /users/testuser/personal/
     Description: Personal vaults

     Container name: work
     Container ID: /users/testuser/work/
     Description: Work vaults
   ----------------------------
   Number of entries returned 2
   ----------------------------

If the parent ID is specified, it will return the sub-containers within
that container:

::

   $ ipa vaultcontainer-find /services
   --------------------------
   2 vault containers matched
   --------------------------
     Container name: server1.example.com
     Container ID: /services/server1.example.com/
     Description: Vaults of services on server1.example.com

     Container name: server2.example.com
     Container ID: /services/server2.example.com/
     Description: Vaults of services on server2.example.com
   ----------------------------
   Number of entries returned 2
   ----------------------------

Top-level containers can be listed by searching from the root container:

::

   $ ipa vaultcontainer-find /
   --------------------------
   3 vault containers matched
   --------------------------
     Container name: users
     Container ID: /users/
     Description: Users vault container

     Container name: shared
     Container ID: /shared/
     Description: Shared vault container

     Container name: services
     Container ID: /services/
     Description: Services vault container
   ----------------------------
   Number of entries returned 3
   ----------------------------



Displaying container info
-------------------------

Any user can display the container info using the following command:

::

   $ ipa vaultcontainer-show <container ID> [OPTIONS]

To display user's private container's info:

::

   $ ipa vaultcontainer-show personal
     Container name: personal
     Container ID: /users/testuser/personal/
     Description: Personal vault container

To display a public container's info:

::

   $ ipa vaultcontainer-show /services/server.example.com
     Container name: server.example.com
     Container ID: /services/server.example.com/
     Description: Services vault container for server.example.com



Adding a container
------------------

::

   $ ipa vaultcontainer-add <container ID> [OPTIONS]

To add a private container:

::

   $ ipa vaultcontainer-add personal
   --------------------------------
   Added vault container "personal"
   --------------------------------
     Container name: personal
     Container ID: /users/testuser/personal/

To add a public container:

::

   $ ipa vaultcontainer-add /services/server.example.com
   ------------------------------------------
   Added vault container "server.example.com"
   ------------------------------------------
     Container name: server.example.com
     Container ID: /services/server.example.com/



Modifying a container
---------------------

The container owner can modify a container using the following command:

::

   $ ipa vaultcontainer-mod <container ID> [OPTIONS]

For example, to change container description:

::

   $ ipa vaultcontainer-show /services/server.example.com
     Container name: server.example.com
     Container ID: /services/server.example.com/

   $ ipa vaultcontainer-mod /services/server.example.com --desc "Services vault container for server.example.com"
   ---------------------------------------------
   Modified vault container "server.example.com"
   ---------------------------------------------
     Container name: server.example.com
     Container ID: /services/server.example.com/
     Description: Services vault container for server.example.com



Removing a container
--------------------

::

   $ ipa vaultcontainer-del <container ID> [OPTIONS]

For example:

::

   $ ipa vaultcontainer-del /services/server.example.com
   --------------------------------------------
   Deleted vault container "server.example.com"
   --------------------------------------------



Adding container member
-----------------------

A container owner can add a member to the container with the following
command:

::

   $ ipa vaultcontainer-add-member <container ID> --users <member ID> [OPTIONS]

For example:

::

   $ ipa vaultcontainer-add-member /services/server.example.com --users testmember
     Container name: server.example.com
     Container ID: /services/server.example.com/
     Member users: testmember
   -------------------------
   Number of members added 1
   -------------------------



Removing container member
-------------------------

A container owner can remove a member from the container with the
following command:

::

   $ ipa vaultcontainer-remove-member <container ID> --users <member ID> [OPTIONS]

For example:

::

   $ ipa vaultcontainer-remove-member /services/server.example.com --users testmember
     Container name: server.example.com
     Container ID: /services/server.example.com/
   ---------------------------
   Number of members removed 1
   ---------------------------



Adding container owner
----------------------

A container owner can add another owner to the container with the
following command:

::

   $ ipa vaultcontainer-add-owner <container ID> --users <owner ID> [OPTIONS]

For example:

::

   $ ipa vaultcontainer-add-owner /services/server.example.com --users testowner
     Container name: server.example.com
     Container ID: /services/server.example.com/
   -------------------------
   Number of members added 1
   -------------------------



Removing container owner
------------------------

A container owner can remove another owner from the container with the
following command:

::

   $ ipa vaultcontainer-remove-owner <container ID> --users <owner ID> [OPTIONS]

For example:

::

   $ ipa vaultcontainer-remove-owner /services/server.example.com --users testowner
     Container name: server.example.com
     Container ID: /services/server.example.com/
   ---------------------------
   Number of members removed 1
   ---------------------------



Vault Management
================



Listing available vaults
------------------------

A user can search the vaults that it owns or it's a member of using the
following command:

::

   $ ipa vault-find [container ID] [OPTIONS]

By default the command will list the vaults in the user's private
container:

::

   $ ipa vault-find
   ---------------
   1 entries found
   ---------------
     Vault name: PrivateVault
     Vault ID: /users/testuser/PrivateVault
     Description: Private vault
     Type: standard
   ----------------------------
   Number of entries returned 1
   ----------------------------

To find shared vaults, specify -shared:

::

   $ ipa vault-find --shared
   ---------------
   1 entries found
   ---------------
     Vault name: IPA
     Vault ID: /shared/projects/IPA
     Description: IPA project
     Type: standard
   ----------------------------
   Number of entries returned 1
   ----------------------------

To find service vaults, specify --services:

::

   $ ipa vault-find --services
   ---------------
   1 entries found
   ---------------
     Vault name: HTTP
     Vault ID: /services/server.example.com/HTTP
     Description: HTTP service on server.example.com
     Type: standard
   ----------------------------
   Number of entries returned 1
   ----------------------------



Displaying vault info
---------------------

A user can view a particular vault info using the following command:

::

   $ ipa vault-show <vault ID> [OPTIONS]

To display the basic vault info:

::

   $ ipa vault-show /shared/SymmetricVault
     Vault name: SymmetricVault
     Vault ID: /shared/SymmetricVault
     Description: Symmetric vault
     Type: standard

To display the complete vault info:

::

   $ ipa vault-show /shared/SymmetricVault --all
     Vault name: SymmetricVault
     Vault ID: /shared/SymmetricVault
     Description: Symmetric vault
     Type: symmetric
     Secret salt: ....



Creating a new vault
--------------------

A container member can create a vault using the following command:

::

   $ ipa vault-add <vault ID> [OPTIONS]

Private vaults can be created by specifying a relative vault ID:

::

   $ ipa vault-add PrivateVault --desc "Private vault"
   --------------------------
   Added vault "PrivateVault"
   --------------------------
     Vault name: PrivateVault
     Vault ID: /users/testuser/PrivateVault
     Description: Private vault
     Type: standard

Non-private vaults can be created by specifying an absolute vault ID:

::

   $ ipa vault-add /shared/SharedVault --desc "Shared vault"
   ---------------------------------
   Added vault "SharedVault"
   ---------------------------------
     Vault name: SharedVault
     Vault ID: /shared/SharedVault
     Description: Shared vault
     Type: standard

Symmetric vaults can be created by specifying the type and the password.
The password can be provided interactively, specified in the command
option, or specified in a file.

::

   $ ipa vault-add SymmetricVault --desc "Symmetric vault" --type symmetric
   New password: ********
   Verify password: ********
   ----------------------------
   Added vault "SymmetricVault"
   ----------------------------
     Vault name: SymmetricVault
     Vault ID: /users/testuser/SymmetricVault
     Description: Symmetric vault
     Type: symmetric

   $ ipa vault-add SymmetricVault --desc "Symmetric vault" --type symmetric --password mypassword
   ----------------------------
   Added vault "SymmetricVault"
   ----------------------------
     Vault name: SymmetricVault
     Vault ID: /users/testuser/SymmetricVault
     Description: Symmetric vault
     Type: symmetric

   $ ipa vault-add SymmetricVault --desc "Symmetric vault" --type symmetric -password-file password.txt
   ----------------------------
   Added vault "SymmetricVault"
   ----------------------------
     Vault name: SymmetricVault
     Vault ID: /users/testuser/SymmetricVault
     Description: Symmetric vault
     Type: symmetric

Asymmetric vaults can be created by specifying the type and the public
key:

::

   $ ipa vault-add AsymmetricVault --desc "Asymmetric vault" --type asymmetric --public-key-file public.pem
   -----------------------------
   Added vault "AsymmetricVault"
   -----------------------------
     Vault name: AsymmetricVault
     Vault ID: /users/testuser/AsymmetricVault
     Description: Asymmetric vault
     Type: asymmetric



Archiving data
--------------

A vault member/owner can archive data using the following command:

::

   $ ipa vault-archive <vault ID> [--in <input file> | --text <text> | --data <base-64 encoded data> | --stdin] [OPTIONS]

With a standard vault the operation can be done directly.

::

   $ ipa vault-archive StandardVault --in secret.txt
   ----------------------------------------
   Archived data into vault "StandardVault"
   ----------------------------------------

   $ ipa vault-archive StandardVault --text "secret"
   ----------------------------------------
   Archived data into vault "StandardVault"
   ----------------------------------------

   $ ipa vault-archive StandardVault --data c2VjcmV0Cg==
   ----------------------------------------
   Archived data into vault "StandardVault"
   ----------------------------------------

   $ echo secret | ipa vault-archive StandardVault --stdin
   ----------------------------------------
   Archived data into vault "StandardVault"
   ----------------------------------------

With a symmetric vault the operation requires a password:

::

   $ ipa vault-archive SymmetricVault --in secret.txt
   Password: ********
   -----------------------------------------
   Archived data into vault "SymmetricVault"
   -----------------------------------------

With an asymmetric vault the operation does not require anything since
the vault public key is stored in one of vault attributes.

::

   $ ipa vault-archive AsymmetricVault --in secret.txt
   ------------------------------------------
   Archived data into vault "AsymmetricVault"
   ------------------------------------------



Retrieving data
---------------

A vault member/owner can be retrieve data using the following command:

::

   $ ipa vault-retrieve <vault ID> [--out <output file> | --stdout] [OPTIONS]

With a standard vault the operation can be done directly.

::

   $ ipa vault-retrieve StandardVault --out secret.txt
   -----------------------------------------
   Retrieved data from vault "StandardVault"
   -----------------------------------------

   $ ipa vault-retrieve StandardVault --stdout
   secret

With a symmetric vault the operation requires a password:

::

   $ ipa vault-retrieve SymmetricVault --out secret.txt
   Password: ********
   ------------------------------------------
   Retrieved data from vault "SymmetricVault"
   ------------------------------------------

With an asymmetric vault the operation requires a private key:

::

   $ ipa vault-retrieve AsymmetricVault --out secret.txt --private-key-file private.pem
   -------------------------------------------
   Retrieved data from vault "AsymmetricVault"
   -------------------------------------------



Copying a vault
---------------

A container member can copy a vault that it has access to using the
following command:

::

   $ ipa vault-add <vault ID> --source-vault-id <source vault ID> [OPTIONS]

Password or private of the source vault is not required to copy, but it
is still required to access the secrets.

To copy a private vault into a new private vault:

::

   $ ipa vault-add NewPrivateVault --source-vault-id PrivateVault
   -----------------------------
   Added vault "NewPrivateVault"
   -----------------------------
     Vault name: NewPrivateVault
     Vault ID: /users/testuser/NewPrivateVault
     Type: standard

To copy a private vault into a new shared vault:

::

   $ ipa vault-add /shared/NewSharedVault --source-vault-id PrivateVault
   ----------------------------
   Added vault "NewSharedVault"
   ----------------------------
     Vault name: NewSharedVault
     Vault ID: /shared/NewSharedVault
     Type: standard



Modifying a vault
-----------------

The vault owner can modify a vault using the following command:

::

   $ ipa vault-mod <vault ID> [OPTIONS]

For example, to change vault description:

::

   $ ipa vault-show PrivateVault
     Vault name: PrivateVault
     Vault ID: /users/testuser/PrivateVault
     Description: Private vault
     Type: standard

   $ ipa vault-mod PrivateVault --desc "This is a private vault"
   -----------------------------
   Modified vault "PrivateVault"
   -----------------------------
     Vault name: PrivateVault
     Vault ID: /users/testuser/PrivateVault
     Description: This is a private vault
     Type: standard

To convert a symmetric vault into an asymmetric vault the old password
and the new public key must be specified:

::

   $ ipa vault-show PrivateVault
     Vault name: PrivateVault
     Vault ID: /users/testuser/PrivateVault
     Description: Private vault
     Type: symmetric

   $ ipa vault-mod PrivateVault --type asymmetric --public-key-file public.pem
   Password: ********
   -----------------------------
   Modified vault "PrivateVault"
   -----------------------------
     Vault name: PrivateVault
     Vault ID: /users/testuser/PrivateVault
     Description: Private vault
     Type: asymmetric

To convert an asymmetric vault into a symmetric vault the old private
key and the new password must be specified:

::

   $ ipa vault-show PrivateVault
     Vault name: PrivateVault
     Vault ID: /users/testuser/PrivateVault
     Description: Private vault
     Type: asymmetric

   $ ipa vault-mod PrivateVault --type symmetric --private-key-file private.pem
   Password: ********
   Verify password: ********
   -----------------------------
   Modified vault "PrivateVault"
   -----------------------------
     Vault name: PrivateVault
     Vault ID: /users/testuser/PrivateVault
     Description: Private vault
     Type: symmetric



Removing a vault
----------------

To remove a vault the owner can execute the following command:

::

   $ ipa vault-del <vault ID> [OPTIONS]

For example:

::

   $ ipa vault-del PrivateVault
   ----------------------------
   Deleted vault "PrivateVault"
   ----------------------------



Changing vault password
-----------------------

An owner can change the vault password or keys using the following
command.

::

   $ ipa vault-password <vault ID> [OPTIONS]

An owner can change the password of a symmetric vault by providing the
old password and the new password:

::

   $ ipa vault-password SymmetricVault
   Password: ********
   New password: ********
   Verify new password: ********
   ---------------------------------------
   Changed "SymmetricVault" vault password
   ---------------------------------------

An owner can change the keys of an asymmetric vault by providing the old
private key and the new public key:

::

   $ ipa vault-password AsymmetricVault --private-key-file private.pem --new-public-key-file new-public.pem
   ------------------------------------
   Changed "AsymmetricVault" vault keys
   ------------------------------------



Adding vault member
-------------------

A vault owner can add members to the vault with the following command:

::

   $ ipa vault-add-member <vault ID> [--users <list of user IDs>] [--groups <list of group IDs>]

For example:

::

   $ ipa vault-add-member MyVault --users testmember
   ---------------------------------
   Added members to "MyVault " vault
   ---------------------------------



Removing vault member
---------------------

A vault owner can remove a member from the vault with the following
command:

::

   $ ipa vault-remove-member <vault ID> [--users <list of user IDs>] [--groups <list of group IDs>]

For example:

::

   $ ipa vault-remove-member MyVault --users testmember
   -------------------------------------
   Removed members from "MyVault " vault
   -------------------------------------



Adding vault owner
------------------

An owner can add another owner to the vault with the following command:

::

   $ ipa vault-add-owner <vault ID> [--users <list of user IDs>] [--groups <list of group IDs>]

For example:

::

   $ ipa vault-add-owner MyVault --users testowner
   ----------------------------------
   Added owners from "MyVault " vault
   ----------------------------------



Removing vault owner
--------------------

An owner can remove another owner from the vault with the following
command:

::

   $ ipa vault-remove-owner <vault ID> [--users <list of user IDs>] [--groups <list of group IDs>]

For example:

::

   $ ipa vault-remove-owner MyVault --users testowner
   ------------------------------------
   Removed owners from "MyVault " vault
   ------------------------------------



Secret Management
=================



Listing secrets in a vault
--------------------------

A vault member/owner can list the secrets in a vault using the following
command:

::

   $ ipa vaultsecret-find <vault ID> [OPTIONS]

With a standard vault the secrets can be listed directly:

::

   $ ipa vaultsecret-find StandardVault
   ---------------
   2 entries found
   ---------------
     Secret ID: gmail
     Description: Gmail password

     Secret ID: yahoo
     Description: Yahoo! Mail password
   ----------------------------
   Number of entries returned 2
   ----------------------------

With a symmetric vault the operation requires a password:

::

   $ ipa vaultsecret-find SymmetricVault
   Password: ********
   ---------------
   2 entries found
   ---------------
     Secret ID: gmail
     Description: Gmail password

     Secret ID: yahoo
     Description: Yahoo! Mail password
   ----------------------------
   Number of entries returned 2
   ----------------------------

A vault owner can list the secrets in an asymmetric vault by providing
the vault private key:

::

   $ ipa vaultsecret-find AsymmetricVault --private-key-file private.pem
   ---------------
   2 entries found
   ---------------
     Secret ID: gmail
     Description: Gmail password

     Secret ID: yahoo
     Description: Yahoo! Mail password
   ----------------------------
   Number of entries returned 2
   ----------------------------



Adding a secret
---------------

A vault member/owner can add a secret using the following command:

::

   $ ipa vaultsecret-add <vault ID> <secret ID> [OPTIONS]

With a standard vault the operation can be done directly. The secret can
be provided interactively, via an input file, or via standard input.

::

   $ ipa vaultsecret-add StandardVault MySecret
   Secret: ********
   Verify secret: ********
   -----------------------------
   Added vault secret "MySecret"
   -----------------------------
     Secret name: MySecret
     Data: c2VjcmV0

   $ ipa vaultsecret-add StandardVault MySecret --in secret.txt
   -----------------------------
   Added vault secret "MySecret"
   -----------------------------
     Secret name: MySecret
     Data: c2VjcmV0

   $ echo secret | ipa vaultsecret-add StandardVault MySecret --stdin
   -----------------------------
   Added vault secret "MySecret"
   -----------------------------
     Secret name: MySecret
     Data: c2VjcmV0

With a symmetric vault the operation requires a password:

::

   $ ipa vaultsecret-add SymmetricVault MySecret --in secret.txt
   Password: ********
   -----------------------------
   Added vault secret "MySecret"
   -----------------------------
     Secret name: MySecret
     Data: c2VjcmV0

With an asymmetric vault the operation requires a private key.

::

   $ ipa vaultsecret-add AsymmetricVault MySecret --in secret.txt --private-key private-key.pem
   -----------------------------
   Added vault secret "MySecret"
   -----------------------------
     Secret name: MySecret
     Data: c2VjcmV0



Retrieving a secret
-------------------

A vault member/owner can be retrieve a secret using the following
command:

::

   $ ipa vaultsecret-show <vault ID> <secret ID> [OPTIONS]

With a standard vault the operation can be done directly. The secret can
be stored in an output file or directed to standard output.

::

   $ ipa vaultsecret-show StandardVault MySecret
     Secret name: MySecret
     Data: c2VjcmV0

   $ ipa vaultsecret-show StandardVault MySecret --out secret.txt

   $ ipa vaultsecret-show StandardVault MySecret --stdout
   secret

With a symmetric vault the operation requires a password:

::

   $ ipa vaultsecret-show SymmetricVault MySecret --out secret.txt
   Password: ********

With an asymmetric vault the operation requires a private key:

::

   $ ipa vaultsecret-show AsymmetricVault MySecret --out secret.txt --private-key-file private.pem



Copying a secret
----------------

Secret can be copied using the following command:

::

   $ ipa vaultsecret-add <vault ID> <secret ID> [--source-vault <source vault ID>] [--source-secret <source secret ID>] [OPTIONS]

The copy operation will be done using the retrieval and archival
operations, so depending on the vault types, it may require the
password/key of all vaults involved.

To copy a secret into another secret in the same vault:

::

   $ ipa vaultsecret-add StandardVault NewSecret --source-secret MySecret
   ------------------------------
   Added vault secret "NewSecret"
   ------------------------------

To copy a secret from a vault into another vault:

::

   $ ipa vaultsecret-add /shared/SharedVault MySecret --source-vault PrivateVault
   -----------------------------
   Added vault secret "MySecret"
   -----------------------------

To copy a secret into another vault with a different name:

::

   $ ipa vaultsecret-add /shared/SharedVault NewSecret --source-vault PrivateVault --source-secret MySecret
   ------------------------------
   Added vault secret "NewSecret"
   ------------------------------

To copy a secret from a symmetric vault into an asymmetric vault (this
will replace all secrets in the asymmetric vault):

::

   $ ipa vaultsecret-add AsymmetricVault MySecret --source-vault SymmetricVault
   Source Password: ********
   -----------------------------
   Added vault secret "MySecret"
   -----------------------------

To copy a secret from an asymmetric vault into a symmetric vault:

::

   $ ipa vaultsecret-add SymmetricVault MySecret --source-vault AsymmetricVault --source-private-key private-key.pem
   Password: ********
   -----------------------------
   Added vault secret "MySecret"
   -----------------------------



Modifying secret attributes
---------------------------

Secret attributes can be modified using the following command:

::

   $ ipa vaultsecret-mod <vault ID> <secret ID> [OPTIONS]

For example, to modify secret description:

::

   $ ipa vaultsecret-mod StandardVault MySecret --desc "My secret"
   --------------------------------
   Modified vault secret "MySecret"
   --------------------------------
     Secret name: MySecret
     Description: My secret
     Data: c2VjcmV0



Deleting a secret
-----------------

A secret can be removed using the following command:

::

   $ ipa vaultsecret-del <vault ID> <secret ID> [OPTIONS]

With a standard vault the operation can be done directly:

::

   $ ipa vaultsecret-del StandardVault MySecret
   -------------------------------
   Deleted vault secret "MySecret"
   -------------------------------

With a symmetric vault the operation requires a vault password:

::

   $ ipa vaultsecret-del SymmetricVault secret
   Password: ********
   -------------------------------
   Deleted vault secret "MySecret"
   -------------------------------

With an asymmetric vault the operation requires a vault private key:

::

   $ ipa vaultsecret-del AsymmetricVault secret --private-key private-key.pem
   -------------------------------
   Deleted vault secret "MySecret"
   -------------------------------



Escrow Operations
=================

Vault encryption key can be escrowed such that if needed the escrow
officer can recover the secrets.



Creating a vault with escrow
----------------------------

An escrowed symmetric vault can be created with the following command:

::

   $ ipa vault-add EscrowedSymmetricVault --type symmetric --escrow-public-key-file escrow-public.pem
   New password: ********
   Verify password: ********
   ------------------------------------
   Added vault "EscrowedSymmetricVault"
   ------------------------------------
     Vault name: EscrowedSymmetricVault
     Vault ID: /users/testuser/EscrowedSymmetricVault
     Vault type: symmetric

An escrowed asymmetric vault can be created with the following command:

::

   $ ipa vault-add EscrowedAsymmetricVault --type asymmetric --public-key-file public.pem --escrow-public-key-file escrow-public.pem
   -------------------------------------
   Added vault "EscrowedAsymmetricVault"
   -------------------------------------
     Vault name: EscrowedAsymmetricVault
     Vault ID: /users/testuser/EscrowedAsymmetricVault
     Vault type: asymmetric



Escrowing an existing vault
---------------------------

A vault owner can escrow an existing symmetric vault by providing the
escrow public key:

::

   $ ipa vault-mod SymmetricVault --escrow true --escrow-public-key-file escrow-public.pem
   Password: ********
   -------------------------------
   Modified vault "SymmetricVault"
   -------------------------------

A vault owner can escrow an existing asymmetric vault by providing the
vault private key and the escrow public key

::

   $ ipa vault-mod AsymmetricVault --escrow true --private-key-file private.pem --escrow-public-key-file escrow-public.pem
   --------------------------------
   Modified vault "AsymmetricVault"
   --------------------------------

A vault owner can unescrow a vault box as follows:

::

   $ ipa vault-mod Vault --escrow-public-key NONE
   ----------------------
   Modified vault "Vault"
   ----------------------



Recovering an escrowed secret
-----------------------------

An escrow officer can recover the secret by specifying the escrow
private key to decrypt the secret key:

::

   $ ipa vault-retrieve EscrowedVault --escrow-private-key-file escrow-private.pem --out secret.txt
   -----------------------------------------
   Retrieved data from vault "EscrowedVault"
   -----------------------------------------



Changing escrowed vault password
--------------------------------

If the current symmetric vault password is known, the owner can change
it by providing the old password and the new password. The new secret
key will automatically be escrowed.

::

   $ ipa vault-password EscrowedSymmetricVault
   Password: *********
   New password: *********
   Verify password: ********
   -------------------------
   Password change completed
   -------------------------

If the current password is unknown, the owner can request password
reset:

::

   $ ipa vault-password EscrowedSymmetricVault --reset
   New password: *********
   Verify password: ********
   -----------------------
   Password change pending
   -----------------------

The escrow officer can approve the request as follows:

::

   $ ipa vault-password /users/testuser/EscrowedSymmetricVault --approve --escrow-private-key-file escrow-private.pem
   -------------------------
   Password change completed
   -------------------------

If necessary, the escrow officer can reject the request as follows:

::

   $ ipa vault-password /users/testuser/EscrowedSymmetricVault --reject
   ------------------------
   Password change canceled
   ------------------------



Service Operations
==================



Creating service vault password
-------------------------------

A service administrator can create a service vault password by archiving
a new secret into a private vault:

::

   $ ipa vault-add ldap_password --in password.txt
   ---------------------------
   Added vault "ldap_password"
   ---------------------------
     Vault name: ldap_password
     Vault ID: /users/admin/ldap_password
     Type: standard



Provisioning service vault password for service instance
--------------------------------------------------------

A service administrator can provision the service vault password to a
specific service instance using a service vault. To create a service
vault:

::

   $ ipa vaultcontainer-add /services/<server name>
   $ ipa vault-add /services/<server name>/<service name> --type asymmetric --public-key <service public key>

To copy the service vault password into the service vault:

::

   $ ipa vault-archive /services/<server name>/<service name> --source-vault-id <vault ID>

The command will retrieve the service vault password already archived
earlier, then encrypt it with the service instance's public key. The
public key will be obtained from the service certificate that's already
generated previously on the server.

For example:

::

   $ ipa vaultcontainer-add /services/server.example.com
   ------------------------------------------
   Added vault container "server.example.com"
   ------------------------------------------
     Container name: server.example.com
     Container ID: /services/server.example.com/

   $ ipa vault-add /services/server.example.com/LDAP --type asymmetric --public-key-file service-public.pem
   ------------------
   Added vault "LDAP"
   ------------------
     Vault name: LDAP
     Vault ID: /services/server.example.com/LDAP
     Type: asymmetric

   $ ipa vault-archive /services/server.example.com/LDAP --source-vault-id ldap_password
   -------------------------------
   Archived data into vault "LDAP"
   -------------------------------
     Vault name: LDAP
     Vault ID: /services/server.example.com/LDAP
     Type: asymmetric



Retrieving service vault password for service instance
------------------------------------------------------

A service instance can retrieve the service vault password using the
service private key stored locally:

::

   $ ipa vault-retrieve /services/server.example.com/LDAP --private-key-file service-private.pem --out password.txt
   --------------------------------
   Retrieved data from vault "LDAP"
   --------------------------------
     Vault name: LDAP
     Vault ID: /services/server.example.com/LDAP
     Type: asymmetric



Changing service vault password
-------------------------------

The service administrator can change the service vault password by
archiving a new secret:

::

   $ ipa vault-archive ldap_password --in new_password.txt
   ----------------------------------------
   Archived data into vault "ldap_password"
   ----------------------------------------
     Vault name: ldap_password
     Vault ID: /users/admin/ldap_password
     Type: standard

The service administrator will need to re-provision the new service
vault password to each service instance using the following command:

::

   $ ipa vault-archive /services/server.example.com/LDAP --source-vault-id ldap_password
   -------------------------------
   Archived data into vault "LDAP"
   -------------------------------
     Vault name: LDAP
     Vault ID: /services/server.example.com/LDAP
     Type: asymmetric

This way if there's a compromised instance the service administrator can
isolate it by changing the service vault password and re-provisioning it
to non-compromised instances only.



Configuration Management
========================



Displaying global vault configuration
-------------------------------------

A user can view the global vault configuration using the following
command:

::

   $ ipa vault-config-show
     Maximum secret size: 1024 bytes
     Maximum vault size: 10 secrets



Modifying global vault configuration
------------------------------------

An administrator can modify the global vault configuration using the
following command:

::

   $ ipa vault-config-mod [OPTIONS]

Note that configuration changes will only affect operations executed
after the change.

For example, to change the maximum secret size:

::

   $ ipa vault-config-mod --max-secret-size 1024



Crypto Library
==============

A crypto libray is needed to generate encryption keys and salts, and
encrypt/decrypt the secrets in vault. On the server side IPA and Dogtag
use the NSS library. On the client side the IPA client will use Python
NSS for transport encryption and Python Cryptography for additional
client-side encryption.

The `Python
NSS <https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS/Python_binding_for_NSS>`__
is a Python interface for NSS. IPA already has a dependency on Python
NSS on both client and server.

The `Python
Cryptography <http://pki.fedoraproject.org/wiki/Python_Cryptography>`__
provides a generic interface for various crypto libraries such as
OpenSSL and CommonCrypto as backends. Currently it does not support NSS
backend, but it may be added in the future.

Currently Python Cryptography is only available in Fedora via
`pip <https://cryptography.io/en/latest/installation/>`__. `Ticket
#1114267 <https://bugzilla.redhat.com/show_bug.cgi?id=1114267>`__ will
add the package to Fedora.

The `X.509 certificate
support <https://github.com/pyca/cryptography/issues/1036>`__ is
supposed to be added in version 0.6.



Generating salt
---------------

Python NSS:

::

   salt = nss.generate_random(salt_length)

Python Cryptography:

::

   salt = os.urandom(salt_length)



Generating password-based symmetric encryption key
--------------------------------------------------

Symmetric encryption key can be generated using a key derivation
algorithm such as `PBKDF2 <http://en.wikipedia.org/wiki/PBKDF2>`__ which
takes a vault password and a randomly generated salt. In order to
generate the same key from the same password, the same salt must be used
again.

In NSS the PBKDF2 can be invoked using the following C code. However,
currently Python NSS does not provide an interface to call these NSS
functions.

::

   SECItem password = {
       siBuffer,
       <password>,
       <password length>
   };

   SECItem salt = {
       siBuffer,
       <salt>,
       <salt length>
   };

   void *nullptr = NULL;

   SECAlgorithmID *algID = PK11_CreatePBEV2AlgorithmID(
       SEC_OID_PKCS5_PBKDF2,
       <hash function>,
       <pseudo random function>,
       <key length>,
       <iteration>,
       &salt);

   PK11SlotInfo *slot = PK11_GetBestSlot(SEC_OID_PKCS5_PBKDF2, nullptr);

   PK11SymKey *symKey = PK11_PBEKeyGen(
       slot,
       algID,
       &password,
       PR_FALSE,
       nullptr);

   SECOID_DestroyAlgorithmID(algID, PR_TRUE); 

Python Cryptography
(`docs <https://cryptography.io/en/latest/hazmat/primitives/key-derivation-functions/>`__):

::

   kdf = PBKDF2HMAC(
       algorithm=hashes.SHA256(),
       length=256,
       salt=salt,
       iterations=100000,
       backend=default_backend()
   )

   symmetric_key = kdf.derive(vault_password)

If FIPS certification is not required, the
`scrypt <https://github.com/ricmoo/pyscrypt>`__ might be a better
option.

::

   symmetric_key = pyscrypt.hash(
       vault_password,
       salt=salt,
       N=1024,
       r=1,
       p=1,
       dkLen=256
   )

Web Crypto
(`spec <https://dvcs.w3.org/hg/webcrypto-api/raw-file/tip/spec/Overview.html#pbkdf2>`__,
`implementation <https://bugzilla.mozilla.org/show_bug.cgi?id=1021607>`__,
`example <https://www.w3.org/Bugs/Public/show_bug.cgi?id=25819>`__):

::

   var enc_salt = ...

   var deriveBitsOp = window.crypto.subtle.deriveBits(
       {
           name : "PBKDF2",
           salt: salt,
           iterations: 100000,
           hash: { name: "SHA-256" }
       },
       vault_password,
       256
   );

   deriveBitsOp.oncomplete = function(event) {
       symmetric_key = event.target.result;
   };



Generating asymmetric key pair
------------------------------

The asymmetric key pair can be generated using OpenSSL:

::

   $ openssl genrsa -out private.pem 2048
   $ openssl rsa -in private.pem -out public.pem -pubout

Then the above PEM files can be loaded into Python Cryptography:

::

   public_key = load_pem_public_key(
       data=public_pem,
       backend=default_backend()
   )

   private_key = load_pem_private_key(
       data=private_pem,
       password=None,
       backend=default_backend()
   )

PyCrypto:

::

   private_key = RSA.generate(2048)
   private_pem = private_key.exportKey('PEM')
   public_pem = private_key.publickey().exportKey('PEM')



Encryption with symmetric algorithm
-----------------------------------

In a symmetric vault the secrets will be encrypted/decrypted using
symmetric-key algorithm.

Python NSS:

::

   iv_si = nss.SecItem(salt)
   iv_param = nss.param_from_iv(mechanism, iv_si)

   encryptor = nss.create_context_by_sym_key(
       mechanism,
       nss.CKA_ENCRYPT,
       symmetric_key,
       iv_param)

   encrypted_data = encryptor.cipher_op(data) + encryptor.digest_final()

   decryptor = nss.create_context_by_sym_key(
       mechanism,
       nss.CKA_DECRYPT,
       symmetric_key,
       iv_param)

   data = decryptor.cipher_op(encrypted_data) + decryptor.digest_final()

Python Cryptography
(`docs <https://cryptography.io/en/latest/hazmat/primitives/symmetric-encryption/>`__):

::

   f = Fernet(symmetric_key)
   encrypted_data = f.encrypt(data)
   data = f.decrypt(encrypted_data)

   cipher = Cipher(algorithms.AES(symmetric_key), modes.CBC(iv), backend=default_backend())
   encryptor = cipher.encryptor()
   encrypted_data = encryptor.update(data) + encryptor.finalize()

   decryptor = cipher.decryptor()
   data = decryptor.update(encrypted_data) + decryptor.finalize()

Web Crypto
(`spec <https://dvcs.w3.org/hg/webcrypto-api/raw-file/tip/spec/Overview.html#aes-cbc>`__):

::

   var encryptOp = window.crypto.subtle.encrypt(
       {
           name : "AES-CBC",
           iv: ...
       },
       symmetric_key,
       data
   );

   encryptOp.oncomplete = function(event) {
       encrypted_data = event.target.result;
   };

   var decryptOp = window.crypto.subtle.decrypt(
       {
           name : "AES-CBC",
           iv: ...
       },
       symmetric_key,
       encrypted_data
   );

   decryptOp.oncomplete = function(event) {
       data = event.target.result;
   };



Encryption with asymmetric algorithm
------------------------------------

In an asymmetric vault the secrets will be encrypted/decrypted using
asymmetric algorithm:

Python NSS:

::

   encryrpted_data = nss.pub_wrap_sym_key(mechanism, public_key, data)

   data = ... encrypted_data  ...?

Python Cryptography
(`docs <https://cryptography.io/en/latest/hazmat/primitives/asymmetric/>`__):

::

   encrypted_data = public_key.encrypt(
       data,
       padding.OAEP(
           mgf=padding.MGF1(algorithm=hashes.SHA1()),
           algorithm=hashes.SHA1(),
           label=None
       )
   )

   data = private_key.decrypt(
       encrypted_data,
       padding.OAEP(
           mgf=padding.MGF1(algorithm=hashes.SHA1()),
           algorithm=hashes.SHA1(),
           label=None
       )
   )

Web Crypto
(`spec <https://dvcs.w3.org/hg/webcrypto-api/raw-file/tip/spec/Overview.html#rsa-oaep>`__):

::

   var encryptOp = window.crypto.subtle.encrypt(
       {
           name : "RSA-OAEP",
       },
       public_key,
       data
   );

   encryptOp.oncomplete = function(event) {
       encrypted_data = event.target.result;
   };

   var decryptOp = window.crypto.subtle.decrypt(
       {
           name : "RSA-OAEP",
       },
       private_key,
       encrypted_data
   );

   decryptOp.oncomplete = function(event) {
       data = event.target.result;
   };



Client API
==========

The Client API provides the client interface to access the vault
services. The client API will be primarily used to implement the CLI,
but it can also be used by custom client application.



IPAConnection class
-------------------

The IPAConnection class represents a connection to the IPA server. It
will be used internally by the client API to wrap the `Web
Services <#Web_Services>`__.



Vault class
-----------

The Vault class represents a vault. It contains vault attributes
accessible to the client.

::

   class Vault:
       d

           # basic attributes
           self.id = id
           self.description = None
           self.type = "standard"

           # access control attributes
           self.owners = []
           self.members = []

           # secret attributes
           self.secrets = {}
           self.secret_salt = None
           self.public_key = None

           # escrow attributes
           self.escrow = False
           self.escrow_public_key = None
           self.escrowed_secret_key = None
           self.escrowed_private_key = None

           # password reset attributes
           self.new_secret_salt = None
           self.new_public_key = None
           self.new_escrowed_secret_key = None
           self.new_escrowed_private_key = None



Secret class
------------

The Secret class represents a secret. It contains secret attributes
accessible to the client.

::

   class Secret:
       def __init__(self, id,
               description=None,
               data=None):
           self.id = id
           self.description = description
           self.data = data



VaultClient class
-----------------

VaultClient class provides a client interface to access vaults. It uses
an IPAConnection object to communicate with the server.

::

   class VaultClient:
       d
           self.connection = connection

For example:

::

   connection = ... connection to IPA server ...
   vault_client = VaultClient(connection)

vaultcontianer_find()
----------------------------------------------------------------------------------------------

This method returns a list of subcontainers within the provided
container. By default each list element will contain the basic
attributes of the subcontainer, but additional attributes can be
requested as well.

::

   def vaultcontianer_find(self, parent_id=None, attributes=None):
       return self.connection.vaultcontianer_find(parent_id, attributes)

vaultcontainer_get()
----------------------------------------------------------------------------------------------

This method returns the attributes of the container specified by the ID.
By default it will return the basic attributes, but additional
attributes can be requested as well.

::

   def vaultcontainer_get(self, container_id,
           attributes=None):

       return self.connection.vaultcontainer_get(container_id, attributes)

vaultcontainer_add()
----------------------------------------------------------------------------------------------

This method creates a new container.

::

   def vaultcontainer_add(self, container_id,
           description=None)

       self.connection.vaultcontainer_add(container_id, description)

vaultcontainer_del()
----------------------------------------------------------------------------------------------

This method removes an existing vault.

::

   def vaultcontainer_del(self, container_id):
       self.connection.vaultcontainer_del(container_id)

vault_find()
----------------------------------------------------------------------------------------------

This method returns a list of available vaults within the provided
container. By default each list element will contain the basic
attributes of the vault, but additional attributes can be requested as
well.

::

   def vault_find(self, container_id=None, attributes=None):
       return self.connection.vault_find(container_id, attributes)

For example:

::

   vaults = vault_find()
   for vault in vaults:
       print vault.id + ": " + vault.description

vault_get()
----------------------------------------------------------------------------------------------

This method returns the attributes of the vault specified by the ID. By
default it will return the basic attributes, but additional attributes
can be requested as well.

::

   def vault_get(self, vault_id,
           attributes=None):

       return self.connection.vault_get(vault_id, attributes)

For example:

::

   vault = vault_get("PrivateVault", attributes=["description", "members"])
   print "ID: " + vault.id
   print "Description: " + vault.description

   print "Members:"
   members = vault.members
   for member in members:
       print " - " + member

vault_add()
----------------------------------------------------------------------------------------------

This method creates a new vault on the server.

::

   def vault_add(self, vault_id,
           description=None,
           data=None,
           type="standard",
           vault_password=None,
           vault_public_key=None,
           vault_private_key=None,
           escrow=False,
           escrow_public_key=None):

       vault = Vault(vault_id)
       vault.description = description
       vault.type = type
       vault.public_key = vault_public_key
       vault.escrow = escrow

       if vault.type == "standard":
           pass

       elif vault.type == "symmetric":

           # generate secret key and salt
           vault.secret_salt = generate_salt()
           vault_secret_key = generate_key(vault_password, vault.secret_salt)

           # encrypt data with vault secret key
           vault.secrets = encrypt(data, vault_secret_key)

       elif vault.type == "asymmetric":
           # encrypt data with vault public key
           vault.secrets = encrypt(data, vault.public_key)

       if vault.escrow:

           vault.escrow_public_key = escrow_public_key

           if vault.type == "symmetric":
               # encrypt vault secret key with escrow public key
               vault.escrowed_secret_key = encrypt(vault_secret_key, vault.escrow_public_key)

           elif vault.type == "asymmetric":
               # encrypt vault private key with escrow public key
               vault.escrowed_private_key = encrypt(vault_private_key, vault.escrow_public_key)

       self.connection.vault_add(vault)

       return vault

A standard vault can be created without specifying a password/key:

::

   vault = vault_add("StandardVault",
       description="Standard vault")

A symmetric vault can be created by specifying the type and the vault
password:

::

   vault = vault_add("SymmetricVault",
       description="Symmetric vault",
       type="symmetric",
       vault_password=...)

An escrowed symmetric vault can be created by specifying the type, the
vault password, and the escrow public key:

::

   vault = vault_add("EscrowedSymmetricVault",
       description="Escrowed vault",
       type="symmetric",
       vault_password=...,
       escrow=True,
       escrow_public_key=...)

An asymmetric vault can be created by specifying the type and the vault
public key:

::

   vault = vault_add("AsymmetricVault",
       description="Asymmetric vault",
       type="asymmetric",
       vault_public_key=...)

An escrowed asymmetric vault can be created by specifying the type, the
vault public and private keys, and the escrow public key:

::

   vault = vault_add("EscrowedAsymmetricVault",
       description="Escrowed asymmetric vault",
       type="asymmetric",
       vault_public_key=...,
       vault_private_key=...,
       escrow=True,
       escrow_public_key=...)

vault_update()
----------------------------------------------------------------------------------------------

This method stores changes to the vault object to the server.

::

   def vault_update(self, vault):
       self.connection.vault_update(vault)

vault_change_type()
----------------------------------------------------------------------------------------------

This method modifies the type of an existing vault.

::

   def vault_change_type(self,
           vault_id=None,
           vault=None,
           type=None,
           vault_password=None,
           vault_public_key=None,
           vault_private_key=None):

       if not vault:
           vault = self.connection.get_vault(vault_id, attributes=[...])

       # retrieve existing data based on the old type
       data = vault_retrieve(
           vault=vault,
           vault_password=vault_password,
           vault_private_key=vault_private_key)

       # change the type
       vault.type = type

       # re-archive the data based on the new type
       vault_archive(
           vault=vault,
           data=data,
           vault_password=vault_password,
           vault_public_key=vault_public_key,
           vault_private_key=vault_private_key)

       if vault_id:
           vault_update(vault)

To convert a standard vault into a symmetric vault:

::

   vault_change_type("Vault",
       type="symmetric",
       vault_password=new_vault_password)

To convert a symmetric vault into an asymmetric vault:

::

   vault_change_type("Vault",
       type="asymmetric",
       vault_password=vault_password,
       vault_public_key=new_vault_public_key,
       vault_private_key=new_vault_private_key)

To convert an asymmetric vault into a standard vault:

::

   vault_change_type("Vault",
       type="standard",
       vault_private_key=vault_private_key)

vault_change_escrow()
----------------------------------------------------------------------------------------------

This method modifies the escrow info of an existing vault.

::

   def vault_change_escrow(self,
           vault_id=None,
           vault=None,
           vault_password=None,
           vault_private_key=None,
           escrow=False,
           escrow_public_key=None):

       if not vault:
           vault = vault_get(vault_id, attributes=[...])

       vault.escrow = escrow
       vault.escrow_public_key = escrow_public_key

       if vault.escrow:

           if vault.type == "standard":
               pass

           elif vault.type == "symmetric":

               # generate secret key
               vault_secret_key = generate_key(vault_password, vault.secret_salt)

               # encrypt vault secret key with escrow public key
               vault.escrowed_secret_key = encrypt(vault_secret_key, vault.escrow_public_key)

           elif vault.type == "asymmetric":
               # encrypt vault private key with escrow public key
               vault.escrowed_private_key = encrypt(vault_private_key, vault.escrow_public_key)

       else:
           vault.escrowed_secret_key = None
           vault.escrowed_private_key = None

       if vault_id:
           vault_update(vault)

To escrow a standard vault:

::

   vault_change_escrow("StandardVault",
       escrow=True)

To escrow a symmetric vault:

::

   vault_change_escrow("SymmetricVault",
       escrow=True,
       vault_password=...)

To escrow an asymmetric vault:

::

   vault_change_escrow("AsymmetricVault",
       escrow=True,
       vault_public_key=...,
       vault_private_key=...)

To unescrow a vault:

::

   vault_change_escrow("Vault", escrow=False)

vault_del()
----------------------------------------------------------------------------------------------

This method removes an existing vault on the server.

::

   def vault_del(self, vault_id):
       self.connection.vault_del(vault_id)

vault_archive()
----------------------------------------------------------------------------------------------

This method archives a blob of data into a vault replacing existing
data.

::

   def vault_archive(self,
           vault_id=None,
           vault=None,
           data=None,
           vault_password=None,
           vault_secret_key=None,
           escrow_private_key=None):

       if not vault:
           vault = self.connection.get_vault(vault_id, attributes=[...])

       if vault.type == "standard":
           pass

       elif vault.type == "symmetric":

           if not vault_secret_key:

               if vault_password:
                   # generate vault secret key from vault password and secret salt
                   vault_secret_key = generate_key(vault_password, vault.secret_salt)

               elif escrow_private_key:
                   # decrypt vault secret key with escrow private key
                   vault_secret_key = decrypt(vault.escrowed_secret_key, escrow_private_key)

           # encrypt secrets with vault secret key
           encrypted_data = encrypt(data, vault_secret_key)

       elif vault.type == "asymmetric":

           # encrypt secrets with vault public key
           encrypted_data = encrypt(data, vault.public_key)

       vault_send_data(vult_id, encrypted_data)

A member can archive data into a standard vault without any password or
key:

::

   vault_client.archive_secrets("StandardVault",
       data="mydata")

A member can archive data into a symmetric vault by providing a vault
password:

::

   vault_client.archive_secrets("SymmetricVault",
       data="mydata",
       vault_password=...)

A member can archive data into a symmetric vault by providing a
pre-generated vault secret key:

::

   vault_client.archive_secrets("SymmetricVault",
       data="mydata",
       vault_secret_key=...)

An escrow officer can archive data into a symmetric vault by providing
an escrow private key:

::

   vault_client.archive_secrets("SymmetricVault",
       data="mydata",
       escrow_private_key=...)

A member can archive data into an asymmetric vault without any password
or key:

::

   vault_client.archive_secrets("AsymmetricVault",
       data="mydata")

vault_retrieve()
----------------------------------------------------------------------------------------------

This method retrieves a blob of data stored in a vault and decrypt it
based on the vault type.

::

   def vault_retrieve(self,
           vault_id=None,
           vault=None,
           vault_password=None,
           vault_secret_key=None,
           vault_private_key=None,
           escrow_private_key=None):

       if not vault:
           vault = self.connection.get_vault(vault_id, attributes=[...])

       encrypted_data = vault_get_data(vault.id);

       if vault.type == "standard":
           data = encrypted_data

       elif vault.type == "symmetric":

           if not vault_secret_key:

               if vault_password:
                   # generate vault secret key from vault password and secret salt
                   vault_secret_key = generate_key(vault_password, vault.secret_salt)

               elif escrow_private_key:
                   # decrypt vault secret key with escrow private key
                   vault_secret_key = decrypt(vault.escrowed_secret_key, escrow_private_key)

           # decrypt secrets with vault secret key
           data = decrypt(encrypted_data, vault_secret_key)

       elif vault.type == "asymmetric":

           if escrow_private_key:
               # decrypt vault private key with escrow private key
               vault_private_key = decrypt(vault.escrowed_private_key, escrow_private_key)

           if vault_private_key:
               # decrypt secrets with vault private key
               data = decrypt(encrypted_data, vault_private_key)

           else:
               # return empty data
               data = ''

       return data

A member can retrieve data from a standard vault without any password or
key:

::

   data = vault_client.vault_retrieve("StandardVault")

A member can retrieve data from a symmetric vault by providing the vault
password:

::

   data = vault_client.vault_retrieve("SymmetricVault", vault_password=...)

A member can retrieve data from a symmetric vault by providing the vault
secret key:

::

   data = vault_client.vault_retrieve("SymmetricVault", vault_secret_key=...)

An escrow officer can recover data from an escrowed symmetric vault by
providing the escrow private key:

::

   data = vault_client.vault_retrieve("EscrowedSymmetricVault", escrow_private_key=...)

An owner can retrieve secrets from an asymmetric vault by providing the
vault private key:

::

   data = vault_client.vault_retrieve("AsymmetricVault", vault_private_key=...)

An escrow officer can recover secrets from an escrowed asymmetric vault
by providing the escrow private key:

::

   data = vault_client.vault_retrieve("EscrowedAsymmetricVault", escrow_private_key=...)

vaultsecret_archive()
----------------------------------------------------------------------------------------------

This method encrypt a secret based on the vault type and archive it into
a collection of secrets in the vault.

::

   def vaultsecret_archive(self,
           vault_id=None,
           vault=None,
           secret_id=None,
           description=None,
           data=None,
           vault_password=None,
           vault_secret_key=None,
           vault_private_key=None,
           escrow_private_key=None):

       # retrieve vault info
       if not vault:
           vault = vault_get(vault_id, attributes=[...])

       # retrieve existing secrets
       json_encoded_secrets = vault_retrieve(
           vault=vault,
           vault_password=vault_password,
           vault_secret_key=vault_secret_key,
           vault_private_key=vault_private_key)
           escrow_private_key=escrow_private_key)

       secrets = json.decode(json_encoded_secrets)

       # add the new secret
       secret = {
           id: secret_id,
           description: description
           data: data
       }
       secrets[secret_id] = secrets

       # re-archive secrets
       json_encoded_secrets = json.encode(secrets)

       vault_archive(
           vault=vault,
           data=json_encoded_secrets,
           vault_password=vault_password,
           vault_secret_key=vault_secret_key,
           escrow_private_key=escrow_private_key)

A member can archive a secret into a standard vault without any password
or key:

::

   vaultsecret_archive("StandardVault",
       secret_id="mysecret",
       description="My secret",
       data="secret data")

A member can archive a secret into a symmetric vault by providing a
vault password:

::

   vaultsecret_archive("SymmetricVault",
       secret_id="mysecret",
       description="My secret",
       data="secret data",
       vault_password=...)

A member can archive a secret into a symmetric vault by providing a
pre-generated vault secret key:

::

   vaultsecret_archive("SymmetricVault",
       secret_id="mysecret",
       description="My secret",
       data="secret data",
       vault_secret_key=...)

An escrow officer can archive a secret into a symmetric vault by
providing an escrow private key:

::

   vaultsecret_archive("SymmetricVault",
       secret_id="mysecret",
       description="My secret",
       data="secret data",
       escrow_private_key=...)

An owner can archive a secret into an asymmetric vault by providing a
vault private key:

::

   vaultsecret_archive("AsymmetricVault",
       secret_id="mysecret",
       description="My secret",
       data="secret data",
       vault_private_key=...)

vaultsecret_retrieve()
----------------------------------------------------------------------------------------------

This method retrieves a secret from a collection of secrets in a vault
and decrypt it based on the vault type.

::

   def vaultsecret_retrieve(self,
           vault_id=None,
           vault=None,
           secret_id=None,
           vault_password=None,
           vault_secret_key=None,
           vault_private_key=None,
           escrow_private_key=None):

       if not vault:
           vault = vault_get(vault_id, attributes=[...])

       # retrieve secrets
       json_encoded_secrets = vault_retrieve(
           vault=vault,
           vault_password=vault_password,
           vault_secret_key=vault_secret_key,
           vault_private_key=vault_private_key)
           escrow_private_key=escrow_private_key)

       secrets = json.decode(json_encoded_secrets)

       return secrets[secret_id]

A member can retrieve a secret from a standard vault without any
password or key:

::

   secret = vaultsecret_retrieve("StandardVault", secret_id="mysecret")

A member can retrieve a secret from a symmetric vault by providing the
vault password:

::

   secret = vaultsecret_retrieve("SymmetricVault", secret_id="mysecret", vault_password=...)

A member can retrieve a secret from a symmetric vault by providing the
vault secret key:

::

   secret = vaultsecret_retrieve("SymmetricVault", secret_id="mysecret", vault_secret_key=...)

An escrow officer can recover a secret from an escrowed symmetric vault
by providing the escrow private key:

::

   secret = vaultsecret_retrieve("EscrowedSymmetricVault", secret_id="mysecret", escrow_private_key=...)

An owner can retrieve a secret from an asymmetric vault by providing the
vault private key:

::

   secret = vault_retrieve("AsymmetricVault", secret_id="mysecret", vault_private_key=...)

An escrow officer can recover a secret from an escrowed asymmetric vault
by providing the escrow private key:

::

   secret = vault_retrieve("EscrowedAsymmetricVault", secret_id="mysecret", escrow_private_key=...)

vault_change_password()
----------------------------------------------------------------------------------------------

This method will change the vault password if the current password is
known, or change the vault private key if the current vault private key
is known.

::

   def vault_change_password(self,
           vault_id=None,
           vault=None,
           vault_password=None,
           new_vault_password=None,
           vault_private_key=None,
           new_vault_public_key=None,
           new_vault_private_key=None):

       if not vault:
           vault = vault_get(vault_id, attributes=[...])

       # retrieve secrets with old vault password or private key
       data = vault_retrieve(
           vault=vault,
           vault_password=vault_password,
           vault_private_key=vault_private_key)

       if vault.type == "symmetric":

           # generate vault secret salt
           vault.secret_salt = generate_salt()

           # generate vault secret key from vault password and secret salt
           new_vault_secret_key = generate_key(new_vault_password, vault.secret_salt)

       elif vault.type == "asymmetric":

           # store vault public key
           vault.public_key = new_vault_public_key

       if vault.escrow:

           if vault.type == "symmetric":
               # encrypt vault secret key with escrow public key
               vault.escrowed_secret_key = encrypt(new_vault_secret_key, vault.escrow_public_key)

           elif vault.type == "asymmetric":
               # encrypt vault private key with escrow public key
               vault.escrowed_private_key = encrypt(new_vault_private_key, vault.escrow_public_key)

       # archive data with new vault password or public key
       vault_archive(
           vault=vault,
           data=data,
           vault_secret_key=new_vault_secret_key)

       vault_update(vault)

An owner can change the vault password of a symmetric vault as follows:

::

   vault_client.change_vault_password("SymmetricVault",
       vault_password=...,
       new_vault_password=...)

An owner can change the keys of an asymmetric vault as follows:

::

   vault_client.change_vault_password("AsymmetricVault",
       vault_private_key=...,
       new_vault_public_key=...,
       new_vault_private_key=...)

vault_reset_password()
----------------------------------------------------------------------------------------------

This method will request a vault password reset in case the current
password or private key is lost. Password reset will only work if the
vault is escrowed. The request must be approved by the escrow officer
before it will become effective.

::

   def vault_reset_password(self,
           vault_id=None,
           vault=None,
           new_vault_password=None,
           new_vault_public_key=None,
           new_vault_private_key=None):

       if not vault:
           vault = vault_get(vault_id, attributes=[...])

       if vault.type == "symmetric":

           # generate vault secret salt
           vault.new_secret_salt = generate_salt()

           # generate vault secret key from vault password and secret salt
           new_vault_secret_key = generate_key(new_vault_password, vault.new_secret_salt)

       elif vault.type == "asymmetric":

           # store vault public key
           vault.new_public_key = new_vault_public_key

       if vault.escrow:

           if vault.type == "symmetric":
               # encrypt vault secret key with escrow public key
               vault.new_escrowed_secret_key = encrypt(new_vault_secret_key, vault.escrow_public_key)

           elif vault.type == "asymmetric":
               # encrypt vault private key with escrow public key
               vault.new_escrowed_private_key = encrypt(new_vault_private_key, vault.escrow_public_key)

       if vault_id:
           vault_update(vault)

An owner can request a password reset for a symmetric vault as follows:

::

   vault_reset_password("SymmetricVault",
       new_vault_password=...)

An owner can request a key reset for an asymmetric vault as follows:

::

   vault_reset_password("AsymmetricVault",
       new_vault_public_key=...,
       new_vault_private_key=...)

approve_vault_password_reset()
----------------------------------------------------------------------------------------------

This method can be used by an escrow officer to reset vault password.

::

   def approve_vault_password_reset(self,
           vault_id=None,
           vault=None,
           escrow_private_key=None):

       if not vault:
           vault = self.connection.get_vault(vault_id, attributes=[...])

       # recover secrets
       secrets = self.retrieve_secrets(vault=vault, escrow_private_key=escrow_private_key)

       vault.secret_salt = vault.new_secret_salt
       vault.escrowed_secret_key = vault.new_escrowed_secret_key
       vault.public_key = vault.new_public_key
       vault.escrowed_private_key = vault.new_escrowed_private_key

       vault.new_secret_salt = None
       vault.new_escrowed_secret_key = None
       vault.new_public_key = None
       vault.new_escrowed_private_key = None

       # re-archive secrets
       self.archive_secrets(vault=vault, secrets=secrets, escrow_private_key=escrow_private_key)

       if vault_id:
           self.connection.update_vault(vault)

reject_vault_password_reset()
----------------------------------------------------------------------------------------------

This method can be used by an escrow officer to reject password reset
request.

::

   def reject_vault_password_reject(self,
           vault_id=None,
           vault=None):

       if not vault:
           vault = self.connection.get_vault(vault_id, attributes=[...])

       # clear request
       vault.new_secret_salt = None
       vault.new_escrowed_secret_key = None
       vault.new_public_key = None
       vault.new_escrowed_private_key = None

       if vault_id:
           self.connection.update_vault(vault)



Web Services
============

The vault web services initially may be implemented using the existing
IPA's XML/JSON RPC framework. In the future it may be converted to use
REST API as follows.



Container resource
------------------



PUT /ipa/rest/vaults/
----------------------------------------------------------------------------------------------

An admin can use this operation to create a container. This operation
will be wrapped in IPAConnection.create_container().



GET /ipa/rest/vaults/
----------------------------------------------------------------------------------------------

A user can use this operation to return the container attributes and the
list of vaults in the container. This operation will be wrappped in
IPAConnection.find_vaults().

Response:

::

   {
       id: "/users/testuser",
       ...
       vaults: [
           {
               id: "/users/testuser/PrivateVault",
               description: "Private vault",
               type: "standard"
           },
           {
               id: "/shared/SharedVault",
               description: "Shared vault",
               type: "standard"
           }
       ],
       totalVaults: 2
   }



Vault resource
--------------



GET /ipa/rest/vaults//
----------------------------------------------------------------------------------------------

A member can use this operation to get the vault info. This operation
will be wrappped in IPAConnection.get_vault().

The client can specify the attributes to return in the query:

::

   attributes=<comma-separated attribute list>

Response:

::

   {
       id: "/users/testuser/PrivateVault",
       description: "Private vault",
       type: "standard",
       secret_salt: ...,
       owners: [ "testuser", "testowner" ],
       members: [ "testmember" ],
       public_key: null,
       escrow_public_key: null,
       escrowed_secret_key: null,
       escrowed_private_key: null
   }



POST /ipa/rest/vaults/
----------------------------------------------------------------------------------------------

A user can use this operation to add a vault into a container. This
operation will be wrappped in IPAConnection.create_vault().

Request:

::

   {
       id: "PrivateVault",
       description: "Private vault",
       type: "standard"
   }

The server will return the normalized values of the vault attributes:

::

   {
       id: "/users/testuser/PrivateVault",
       description: "Private vault",
       type: "standard"
   }



POST /ipa/rest/vaults//
----------------------------------------------------------------------------------------------

An owner can use this operation to modify a vault. This operation will
be wrapped in IPAConnection.update_vault().

The client will send the attributes to be modified:

::

   {
       id: "PrivateVault",
       description: "Private vault"
       ...
   }

The server will return the normalized vault attributes after
modification:

::

   {
       id: "/users/testuser/PrivateVault",
       description: "Private vault",
       type: "standard"
   }



DELETE /ipa/rest/vaults/
----------------------------------------------------------------------------------------------

An admin can use this operation to remove a container and all vaults in
it. This operation will be wrapped in IPAConnection.remove_container().



DELETE /ipa/rest/vaults//
----------------------------------------------------------------------------------------------

An owner can use this operation will remove a vault. This operation will
be wrapped in IPAConnection.remove_vault().



Secret resource
---------------

The secrets are accessible under the following URL:

::

   /ipa/rest/vaults/<container>/<vault name>/secrets

The secrets are stored as base-64 encoded of encrypted JSON collection.
For example:

::

   {
       "secret1": {
           "description": "First secret",
           "data": "Secret data"
       },
       "secret2": {
           "description": "Second secret",
           "data": "Secret data"
       }
   }



GET /ipa/rest/vaults///secrets
----------------------------------------------------------------------------------------------

A member can use this operation to return the encrypted secrets as
base-64 encoded data. This operation will be wrapped in
IPAConnection.get_secrets().

Response:

::

   ewogICAg...Qp9Cg==

To retrieve the secrets, the data needs to be base-64 decoded, then
decrypted using the vault secret key or vault private key, then
deserialized using JSON.



PUT /ipa/rest/vaults///secrets
----------------------------------------------------------------------------------------------

A member can use this operation to store base-64 encoded encrypted
secrets. This operation will be wrapped in
IPAConnection.update_vault_secrets().

Request:

::

   ewogICAg...Qp9Cg==

To store the secrets, the secret collection needs to be serialized into
JSON, then encrypted using the vault secret key or the vault public key,
then base-64 encoded.



LDAP Directory
==============



Directory Structure
-------------------

The containers and vaults are represented as LDAP entries in a subtree
in the IPA directory. The root container is represented by the root
entry of the subtree. Sub-containers are represented by entries directly
under the parent container. Vaults are represented by entries stored
under the container.

::

   <suffix>
   + cn=vaults
      + cn=users
         + cn=<user ID>
            + cn=<private vault name>
            + ...
      + cn=shared
         + cn=<shared vault name>
         + ...
      + cn=services
         + cn=<server name>
             + cn=<service vault name>
             + ...



LDAP Schema
-----------

See also `LDAP schema for PKCS#11
data <http://www.freeipa.org/page/V4/PKCS11_in_LDAP/Schema>`__.

Container entry:

::

   dn: cn=<container name>, <parent container DN>
   objectClass: top
   objectClass: ipaVaultContainer
   cn: ...
   description: ...
   owner: ...
   member: ...
   maxSecretSize: ...
   maxVaultSize: ...

Vault entry:

::

   dn: cn=<vault name>, <container DN>
   objectClass: top
   objectClass: ipaVault
   cn: ...
   description: ...
   owner: ...
   member: ...
   ipaVaultType: ...
   ipaVaultSalt:: ...
   ipaVaultPublicKey:: ...
   ipaVaultEscrowPublicKey:: ...
   ipaVaultPendingSalt:: ...
   ipaVaultPendingPublicKey:: ...
   ipaVaultPendingEscrowedSecretKey:: ...
   ipaVaultPendingEscrowedPrivateKey:: ...



Access Control Attributes
-------------------------

The LDAP ACI attributes are used to control the access to the LDAP
entries representing the vaults and the containers. The secrets
themselves are stored in KRA and accessed by IPA as KRA agent on behalf
of IPA users. The IPA user's access to the secrets will be determined by
IPA framework based on the user's membership or ownership of the vaults
and containers, not by LDAP ACI.

The ACI attributes are defined in the root entry of the vault subtree:

::

   dn: cn=vaults, <suffix>
   ...

   ################################################################################
   # Vault Container ACLs
   ################################################################################
   aci: (target="ldap:///cn=*,cn=users,cn=vaults,<suffix>")
     (targetattr="*")
     (version 3.0; acl "Allow users to create private container";
      allow (add) userdn = "ldap:///uid=($attr.cn),cn=users,cn=accounts,<suffix>";)

   aci: (targetfilter="(objectClass=ipaVaultContainer)")
     (targetattr="*")
     (version 3.0; acl "Container members can access the container";
      allow(read, search, compare) userattr="member#USERDN";)
   aci: (targetfilter="(objectClass=ipaVaultContainer)")
     (targetattr="*")
     (version 3.0; acl "Indirect container members can access the container";
      allow(read, search, compare) userattr="member#GROUPDN";)

   aci: (targetfilter="(objectClass=ipaVaultContainer)")
     (targetattr="*")
     (version 3.0; acl "Container members can access sub-containers";
      allow(read, search, compare) userattr="parent[1].member#USERDN";)
   aci: (targetfilter="(objectClass=ipaVaultContainer)")
     (targetattr="*")
     (version 3.0; acl "Indirect container members can access sub-containers";
      allow(read, search, compare) userattr="parent[1].member#GROUPDN";)

   aci: (targetfilter="(objectClass=ipaVaultContainer)")
     (targetattr="*")
     (version 3.0; acl "Container owners can modify the container";
      allow(read, search, compare, write) userattr="owner#USERDN";)
   aci: (targetfilter="(objectClass=ipaVaultContainer)")
     (targetattr="*")
     (version 3.0; acl "Indirect container owners can modify the container";
      allow(read, search, compare, write) userattr="owner#GROUPDN";)

   aci: (targetfilter="(objectClass=ipaVaultContainer)")
     (targetattr="*")
     (version 3.0; acl "Container owners can manage sub-containers";
      allow(read, search, compare, add, delete) userattr="parent[1].owner#USERDN";)
   aci: (targetfilter="(objectClass=ipaVaultContainer)")
     (targetattr="*")
     (version 3.0; acl "Indirect container owners can manage sub-containers";
      allow(read, search, compare, add, delete) userattr="parent[1].owner#GROUPDN";)

   ################################################################################
   # Vault ACLs
   ################################################################################
   aci: (targetfilter="(objectClass=ipaVault)")
     (targetattr="*")
     (version 3.0; acl "Container members can access vaults in the container";
      allow(read, search, compare) userattr="parent[1].member#USERDN";)
   aci: (targetfilter="(objectClass=ipaVault)")
     (targetattr="*")
     (version 3.0; acl "Indirect container members can access vaults in the container";
      allow(read, search, compare) userattr="parent[1].member#GROUPDN";)

   aci: (targetfilter="(objectClass=ipaVault)")
     (targetattr="*")
     (version 3.0; acl "Container owners can manage vaults in the container";
      allow(read, search, compare, add, delete) userattr="parent[1].owner#USERDN";)
   aci: (targetfilter="(objectClass=ipaVault)")
     (targetattr="*")
     (version 3.0; acl "Indirect container owners can manage vaults in the container";
      allow(read, search, compare, add, delete) userattr="parent[1].owner#GROUPDN";)

   aci: (targetfilter="(objectClass=ipaVault)")
     (targetattr="*")
     (version 3.0; acl "Vault members can access the vault";
      allow(read, search, compare) userattr="member#USERDN";)
   aci: (targetfilter="(objectClass=ipaVault)")
     (targetattr="*")
     (version 3.0; acl "Indirect vault members can access the vault";
      allow(read, search, compare) userattr="member#GROUPDN";)

   aci: (targetfilter="(objectClass=ipaVault)")
     (targetattr="*")
     (version 3.0; acl "Vault owners can manage the vault";
      allow(read, search, compare, write) userattr="owner#USERDN";)
   aci: (targetfilter="(objectClass=ipaVault)")
     (targetattr="*")
     (version 3.0; acl "Indirect vault owners can manage the vault";
      allow(read, search, compare, write) userattr="owner#GROUPDN";)

   ################################################################################
   # Security Officer ACLs
   ################################################################################
   aci: (targetfilter="(objectClass=ipaVaultContainer)")
     (targetattr="*")
     (version 3.0; acl "Security officers can access all container";
      allow(read, search, compare) groupdn="ldap:///cn=security officers,cn=groups,cn=accounts,<suffix>";)
   aci: (targetfilter="(objectClass=ipaVault)")
     (targetattr="*")
     (version 3.0; acl "Security officers can access all vaults";
      allow(read, search, compare) groupdn="ldap:///cn=security officers,cn=groups,cn=accounts,<suffix>";)
   aci: (targetfilter="(objectClass=ipaVault)")
     (targetattr="ipaVaultEncSalt || ipaVaultPublicKey || ipaVaultEscrowPublicKey || ipaVaultEscrowedEncKey
      || ipaVaultEscrowedPublicKey || ipaVaultNewSecretSalt || ipaVaultNewPublicKey || ipaVaultNewEscrowedSecretKey || ipaVaultNewEscrowedPublicKey")
     (version 3.0; acl "Security officer can reset/change vault password/keys";
      allow(write) groupdn="ldap:///cn=security officers,cn=groups,cn=accounts,<suffix>";)



KRA Service
===========



Installing KRA service
----------------------

The KRA service must be installed on each replica before the vault
services can be used properly using the following command:

::

   $ ipa-kra-install -p <Directory Manager password>



Uninstalling KRA service
------------------------

If the vault functionality is no longer needed, it can be removed using
the following command:

::

   $ ipa-kra-uninstall -p <Directory Manager password> --uninstall



KRA authentication
------------------

IPA is currently accessing CA services as a user that belongs into CA
agents group, so this user needs to be added into KRA agents group as
well during installation.

All operations against the KRA will be executed as KRA agent using KRA
Python client, so the client certificate needs to be stored on IPA
server file system with the appropriate file protection.

IPA user's access to the secrets stored in KRA will be determined by IPA
framework based on the user's membership or ownership of the vaults and
containers.



Mapping a vault into KRA
------------------------

The encrypted secrets in a vault will be stored as a blob in KRA as a
key under the following client key ID:

::

   <namespace>/<vault ID>

The namespace is used to distinguish secrets from different applications
stored in the same KRA. For IPA the namespace will be "ipa".



Archiving secrets in KRA
------------------------

Regardless of the vault type, the client will transmit the (possibly)
encrypted secrets as generic data to IPA server. The data will be
encrypted with a session key that is unique for each transmission. The
session key itself will be wrapped with KRA transport public key. The
encrypted data and wrapped session key will be sent to IPA server.

::

   class VaultClient():

       def archive_data(self, vault_id, data=None, encrypted_data=None, wrapped_session_key=None, nonce_iv=None):

           if data:
               transport_cert = self.connection.get_transport_cert()
               session_key = generate_session_key()
               nonce_iv = generate_nonce_iv()

               encrypted_data = encrypt(
                   data,
                   session_key,
                   nonce_iv)

               wrapped_session_key = wrap(
                   session_key,
                   transport_cert)

           self.vault_service.archive_data(
               vault_id,
               encrypted_data,
               wrapped_session_key,
               nonce_iv)

The IPA server will then forward the encrypted data to KRA:

::

   class VaultService():

       def archive_data(self, vault_id, encrypted_data, wrapped_session_key, nonce_iv):

           vault = self.get_vault(vault_id)

           # verify access rights
           user = ...
           roles = get_user_roles(user)

           is_member = user.id in vault.members
           is_owner = user.id in vault.owners
           is_escrow_officer = "Security Officers" in roles

           if not is_member and not is_owner and not is_escrow_officer:
               raise Exception("Unauthorized access")

           # archive data as KRA agent
           key_client = api.Backend.kra.get_key_client()
           client_key_id = "ipa/" + vault_id

           key_client.archive_encrypted_data(
               client_key_id,
               encrypted_data,
               wrapped_session_key,
               nonce_iv)



Retrieving secrets from KRA
---------------------------

Regardless of the vault type, the client will send the vault ID to IPA
server to retrieve the encrypted secrets:

::

   class VaultClient():

       def retrieve_data(self, vault_id, wrapped_session_key=None):

           if not wrapped_session_key:
               transport_cert = self.connection.get_transport_cert()
               session_key = generate_session_key()

               wrapped_session_key = wrap(
                   session_key,
                   transport_cert)

           data = self.vault_service.retrieve_data(vault_id, wrapped_session_key)

           return data

When IPA server receives this call, it will retrieve the encrypted
secrets from KRA and then return it to the client:

::

   class VaultService():

       def retrieve_data(self, vault_id, wrapped_session_key):

           vault = self.get_vault(vault_id)

           # verify access rights
           user = ...
           roles = get_user_roles(user)

           is_member = user.id in vault.members
           is_owner = user.id in vault.owners
           is_escrow_officer = "Security Officers" in roles

           if not is_member and not is_owner and not is_escrow_officer:
               raise Exception("Unauthorized access")

           # retrieve data as KRA agent
           key_client = api.Backend.kra.get_key_client()
           client_key_id = "ipa/" + vault_id

           key_info = key_client.get_active_key_info(client_key_id)
           data = key_client.retrieve_key(key_info.key_id)

           return data

Dependencies
============

-  `Ticket #47904 - RFE: new ACI to limit new entry
   RDN <https://fedorahosted.org/389/ticket/47904>`__
-  `NSSConnection shutting down existing
   database <https://fedorahosted.org/freeipa/ticket/4638>`__
-  `Missing PBKDF2
   support <https://bugzilla.redhat.com/show_bug.cgi?id=1159462>`__
-  `Missing public/private key encryption
   support <https://bugzilla.redhat.com/show_bug.cgi?id=1159463>`__
-  `python-cryptography <https://bugzilla.redhat.com/show_bug.cgi?id=1114267>`__

Testing
=======

::

   $ ./make-test ipatests.test_xmlrpc.test_vault_plugin



Frequently Asked Questions
==========================



Why use Python Cryptography instead of Python NSS?
----------------------------------------------------------------------------------------------

Although IPA and Dogtag have been using Python NSS, the Python
Cryptography is simpler to use and it can support various crypto
backends. Since there is no strict requirement to use NSS, it's
recommended to use Python Cryptography for new development.



Can different vault members use different vault passwords?
----------------------------------------------------------------------------------------------

No. There is only one vault password per vault, and it has to be shared
with all members.



Can the secrets in the same vault be archived with different passwords?
----------------------------------------------------------------------------------------------

No. There's only one vault password per vault and it will be used to
encrypt all secrets.



Will the secrets in the same vault encrypted with different encryption keys?
----------------------------------------------------------------------------------------------

No. All secrets in the same vault are encrypted with a single encryption
key which is generated from the vault password and the secret salt.



Can a vault have more than one escrow officer?
----------------------------------------------------------------------------------------------

Yes. A vault can be assigned a pair of escrow public and private keys.
The escrow public key will be stored as an attribute in the vault such
that the owners can use it to encrypt the vault secret/private key to be
escrowed. The escrow private key will be stpred by the escrow officers,
but it can be shared by multiple escrow officers, so any of the escrow
officers will be able to decrypt the vault secret/private key to recover
the secrets.



To Do
=====

-  Merge vault and vault container
-  Add support for renaming users.
-  Add support for symmetric/asymmetric vault without KRA transport
   encryption.
-  Add attribute to indicate single secret / multiple secrets in a
   vault.
-  Clarify how the services container is supposed to be used.
-  Online escrow: need a vault to store escrow private key.
-  Add option to generate public & private keys automatically and
   archive it in KRA.

References
==========

-  `Password Vault <V4/Password_Vault>`__
-  `Using NSS to perform miscellaneous cryptographic
   operations <http://www-archive.mozilla.org/projects/security/pki/nss/tech-notes/tn5.html>`__
-  `The State of Crypto in Python - PyCon
   2014 <https://www.youtube.com/watch?v=r_Pj__qjBvA>`__
-  `Web Cryptography
   API <https://dvcs.w3.org/hg/webcrypto-api/raw-file/tip/spec/Overview.html>`__
-  `Bug 865789 - (web-crypto) Implement W3C Web Crypto
   API <https://bugzilla.mozilla.org/show_bug.cgi?id=865789>`__
-  `WebCrypto Feature
   Matrix <https://docs.google.com/spreadsheet/ccc?key=0AiAcidBZRLxndE9LWEs2R1oxZ0xidUVoU3FQbFFobkE&usp=sharing>`__
-  `RHDS 9.0 Administration Guilde - Managing Access
   Control <https://access.redhat.com/documentation/en-US/Red_Hat_Directory_Server/9.0/html/Administration_Guide/Managing_Access_Control.html>`__
-  `Using the Python Key
   Client <http://pki.fedoraproject.org/wiki/Using-the-Python-Key-Client>`__
-  `PKI REST API -
   Introduction <http://pki.fedoraproject.org/wiki/PKI_ReST_API_-_Introduction>`__
-  `How to securely hash
   passwords? <http://security.stackexchange.com/questions/211/how-to-securely-hash-passwords/31846>`__
-  `PyCrypto <https://www.dlitz.net/software/pycrypto/>`__
