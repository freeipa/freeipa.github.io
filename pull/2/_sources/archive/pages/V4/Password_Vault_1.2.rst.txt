Overview
========

Password Vault 1.2 provides several enhancements over `Password Vault
1.1 <V4/Password_Vault_1.1>`__.

New vault management commands:

-  Copying vault secrets.

.. _vault_management:

Vault Management
================

.. _copying_vault_secret:

Copying vault secret
--------------------

The vault-add and the vault-archive commands provide a mechanism to copy
a secret from one vault to another. On these commands if the source
vault is specified the secret will be retrieved from the source vault
then archived into the target vault. The retrieval parameters should be
specified as in the vault-retrieve command using the --source-\*
parameters. The vault-add will copy the secret into a new target vault.
The vault-archive will copy the secret into an existing target vault. In
either case the target vault may have a different vault type than the
source vault.

To copy a secret from a source vault to a new target vault:

::

   $ ipa vault-add <target vault> --source-vault <source vault> [--source-service <service>|--source-shared|--source-user <user>] [--source-password-file <password file>] [--source-private-key-file <private key file>]

To copy a secret from a source vault to an existing target vault:

::

   $ ipa vault-archive <target vault> --source-vault <source vault> [--source-service <service>|--source-shared|--source-user <user>] [--source-password-file <password> file] [--source-private-key-file <private key file>]

.. _service_operations:

Service Operations
==================

Vault 1.2 provides a mechanism to copy a secret from one vault to
another. This can be used to simplify provisioning service passwords.

.. _creating_service_password:

Creating service password
-------------------------

Initially a service administrator can store the service password in
admin's private vault:

::

   $ ipa vault-add http_password
   $ ipa vault-archive http_password --in password.txt

Then service password can be copied into a new service vault:

::

   $ ipa vault-add password --service HTTP/server.example.com --type asymmetric --public-key-file service-public.pem --source-vault http_password
   -------------------------------
   Archived data into vault "password"
   -------------------------------
     Vault name: password
     Type: asymmetric

.. _changing_service_vault_password:

Changing service vault password
-------------------------------

The service administrator can change the service vault password by
archiving a new secret:

::

   $ ipa vault-archive http_password --in new_password.txt
   ----------------------------------------
   Archived data into vault "http_password"
   ----------------------------------------
     Vault name: http_password
     Type: standard

The service administrator will need to re-provision the new service
vault password to each service instance using the following command:

::

   $ ipa vault-archive password --service HTTP/server.example.com --source-vault http_password
   -------------------------------
   Archived data into vault "password"
   -------------------------------
     Vault name: password
     Type: asymmetric

.. _escrow_operations:

Escrow Operations
=================

Vault encryption key can be escrowed such that if needed the escrow
officer can recover the secrets.

.. _creating_a_vault_with_escrow:

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
     Vault type: symmetric

An escrowed asymmetric vault can be created with the following command:

::

   $ ipa vault-add EscrowedAsymmetricVault --type asymmetric --public-key-file public.pem --escrow-public-key-file escrow-public.pem
   -------------------------------------
   Added vault "EscrowedAsymmetricVault"
   -------------------------------------
     Vault name: EscrowedAsymmetricVault
     Vault type: asymmetric

.. _escrowing_an_existing_vault:

Escrowing an existing vault
---------------------------

A vault owner can escrow an existing symmetric vault by providing the
escrow public key:

::

   $ ipa vault-mod SymmetricVault --escrow-public-key-file escrow-public.pem
   Password: ********
   -------------------------------
   Modified vault "SymmetricVault"
   -------------------------------

A vault owner can escrow an existing asymmetric vault by providing the
vault private key and the escrow public key

::

   $ ipa vault-mod AsymmetricVault --private-key-file private.pem --escrow-public-key-file escrow-public.pem
   --------------------------------
   Modified vault "AsymmetricVault"
   --------------------------------

A vault owner can unescrow a vault box as follows:

::

   $ ipa vault-mod Vault --escrow-public-key ''
   ----------------------
   Modified vault "Vault"
   ----------------------

.. _recovering_an_escrowed_secret:

Recovering an escrowed secret
-----------------------------

An escrow officer can recover the secret by specifying the escrow
private key to decrypt the secret key:

::

   $ ipa vault-retrieve EscrowedVault --escrow-private-key-file escrow-private.pem --out secret.txt
   -----------------------------------------
   Retrieved data from vault "EscrowedVault"
   -----------------------------------------

.. _changing_escrowed_vault_password:

Changing escrowed vault password
--------------------------------

If the current symmetric vault password is known, the owner can change
it by providing the old password and the new password. The new secret
key will automatically be escrowed.

::

   $ ipa vault-mod EscrowedSymmetricVault --change-password
   Password: *********
   New password: *********
   Verify password: ********
   -------------------------
   Password change completed
   -------------------------

If the current password is unknown, the owner can request password
reset:

::

   $ ipa vault-mod EscrowedSymmetricVault --reset-password
   New password: *********
   Verify password: ********
   -----------------------
   Password change pending
   -----------------------

The escrow officer can approve the request as follows:

::

   $ ipa vault-mod EscrowedSymmetricVault --approve-reset --escrow-private-key-file escrow-private.pem
   -------------------------
   Password change completed
   -------------------------

If necessary, the escrow officer can reject the request as follows:

::

   $ ipa vault-mod EscrowedSymmetricVault --reject-reset
   ------------------------
   Password change canceled
   ------------------------

.. _ldap_directory:

LDAP Directory
==============

Schema
------

::

   dn: cn=<vault name>, <container DN>
   ...
   ipaVaultPublicKey:: ...
   ipaVaultEscrowPublicKey:: ...
   ipaVaultPendingSalt:: ...
   ipaVaultPendingPublicKey:: ...
   ipaVaultPendingEscrowedSecretKey:: ...
   ipaVaultPendingEscrowedPrivateKey:: ...

.. _access_control_list:

Access Control List
-------------------

::

   dn: cn=kra,$SUFFIX
   ...

   dn: cn=vaults,cn=kra,$SUFFIX
   ...

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

Status
======

Planned changes:

-  Add mechanism for copying vault secrets (`pending
   review <https://fedorahosted.org/freeipa/ticket/5223>`__).
-  Add vault archival from standard input.
-  Add vault escrow (needs rebase).
-  Other improvements.

Note that the list of patches may change depending on the review
feedback.

References
==========

-  `Password Vault <V4/Password_Vault>`__
