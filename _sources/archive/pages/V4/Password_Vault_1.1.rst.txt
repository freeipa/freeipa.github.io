Overview
========

Password Vault 1.1 provides several enhancements over `Password Vault
1.0 <V4/Password_Vault_1.0>`__.

New vault management commands:

-  Listing all accessible service and user vaults.
-  Changing vault type.
-  Changing vault password.
-  Changing vault keys.

New access control list:

-  A container owner can create and remove sub-containers and vaults in
   the container, and manage the members and owners of the container,
   but it cannot remove the container itself.
-  A container member can list sub-containers and vaults in the
   container.
-  An escrow officer can recover secrets and reset the vault password.



Vault Management
================



Listing accessible vaults
-------------------------

A user can search the vaults that it owns or it's a member of using the
following command:

::

   $ ipa vault-find [OPTIONS]

By default the command will list the vaults in the private container:

::

   $ ipa vault-find
   ---------------
   1 entries found
   ---------------
     Vault name: PrivateVault
     User name: testuser
     Type: standard
   ----------------------------
   Number of entries returned 1
   ----------------------------

To find all service vaults, specify --services:

::

   $ ipa vault-find --services
   ---------------
   1 entries found
   ---------------
     Vault name: test
     Service name: HTTP/server.example.com
     Type: standard
   ----------------------------
   Number of entries returned 1
   ----------------------------

To find service vaults belonging to a specific service, specify
--service :

::

   $ ipa vault-find --service HTTP/server.example.com
   ---------------
   1 entries found
   ---------------
     Vault name: test
     Service name: HTTP/server.example.com
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
     Vault name: test
     Shared: True
     Type: standard
   ----------------------------
   Number of entries returned 1
   ----------------------------

To find all user vaults, specify --users:

::

   $ ipa vault-find --users
   ---------------
   1 entries found
   ---------------
     Vault name: test
     User name: testuser
     Type: standard
   ----------------------------
   Number of entries returned 1
   ----------------------------

To find user vaults, specify --user :

::

   $ ipa vault-find --user testuser
   ---------------
   1 entries found
   ---------------
     Vault name: test
     User name: testuser
     Type: standard
   ----------------------------
   Number of entries returned 1
   ----------------------------



Changing vault type
-------------------

An owner can change the vault type using the following command.

::

   $ ipa vault-mod <name> --type <new type> [OPTIONS]

To change vault type, the old encryption parameter need to be specified:

-  standard: nothing
-  symmetric: password (--old-password or --old-password-file)
-  asymmetric: private key (--private-key-file)

and the new encryption parameter need to be specified:

-  standard: nothing
-  symmetric: password (--new-password or --new-password-file)
-  asymmetric: public key (--public-key-file)

If the passwords is not specified, they will be asked interactively.

To change a standard vault into an symmetric vault the new password must
be specified:

::

   $ ipa vault-show test
     Vault name: test
     Type: standard

   $ ipa vault-mod test --type symmetric
   New password: ********
   Verify password: ********
   ---------------------
   Modified vault "test"
   ---------------------
     Vault name: test
     Type: symmetric

To change a symmetric vault into an asymmetric vault the old password
and the new public key must be specified:

::

   $ ipa vault-mod test --type asymmetric --public-key-file public.pem
   Password: ********
   ---------------------
   Modified vault "test"
   ---------------------
     Vault name: test
     Type: asymmetric

To convert an asymmetric vault into a standard vault the old private key
must be specified:

::

   $ ipa vault-mod test --type standard --private-key-file private.pem
   ---------------------
   Modified vault "test"
   ---------------------
     Vault name: test
     Type: standard



Changing vault password
-----------------------

An owner can change the password of a symmetric vault using the
following command.

::

   $ ipa vault-mod <name> [OPTIONS]

To change the password interactively:

::

   $ ipa vault-mod test --change-password
   Password: ********
   New password: ********
   Verify new password: ********
   ---------------------
   Modified vault "test"
   ---------------------
     Vault name: test
     Type: symmetric

To change the password silently:

::

   $ ipa vault-mod test --old-password-file <old password file> --new-password-file <new password file>
   ---------------------
   Modified vault "test"
   ---------------------
     Vault name: test
     Type: symmetric



Changing vault keys
-------------------

An owner can change the keys of an asymmetric vault using the following
command.

::

   $ ipa vault-mod <name> [OPTIONS]

For example:

::

   $ ipa vault-mod test --private-key-file private.pem --public-key-file new-public.pem
   ---------------------
   Modified vault "test"
   ---------------------



Access Control
==============

In Vault 1.1 a service can be added as a vault owner or members.



Adding vault member
-------------------

A vault owner can add members to the vault with the following command:

::

   $ ipa vault-add-member <name> [--users <list of users>] [--groups <list of groups>] [--services <list of services>]

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

   $ ipa vault-remove-member <name> [--users <list of users>] [--groups <list of groups>] [--services <list of services>]

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

   $ ipa vault-add-owner <vault ID> [--users <list of users>] [--groups <list of groups>] [--services <list of services>]

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

   $ ipa vault-remove-owner <name> [--users <list of users>] [--groups <list of groups>] [--services <list of services>]

For example:

::

   $ ipa vault-remove-owner MyVault --users testowner
   ------------------------------------
   Removed owners from "MyVault " vault
   ------------------------------------



Managing vault containers
-------------------------

Works in the same way as ``vault-show``, ``vault-del``,
``vault-add-owner`` and ``vault-remove-owner`` commands. Vault container
contains vault. There are three types: shared, per-user, per-service.
Per-user and per-service container is created with a first user/service
vault.

::

    vaultcontainer-show [--service <service>|--user <user>|--shared ]
    vaultcontainer-del [--service <service>|--user <user>|--shared ]
    vaultcontainer-add-owner
            [--service <service>|--user <user>|--shared ]
            [--users <users>]  [--groups <groups>] [--services <services>]
    vaultcontainer-remove-owner
            [--service <service>|--user <user>|--shared ]
            [--users <users>]  [--groups <groups>] [--services <services>]



Reworked permissions
--------------------

-  Added new "Vault administrators" privilege. Vault administrators have
   unrestricted access to vaults and vault containers, including the
   power to add/remove owners of vaults and vault containers.

-  Remove the ability of vault owners to add/remove other vault owners.
   If vault owner needs to be changed, vault administrator has to do it.
   Note that vault owners will still have the ability to add/remove
   vault members.

-  When adding new vault container, set owner to the current user. If
   vault container owner needs to be changed, vault administrator has to
   do it.

-  Allowed adding of vaults and vault containers only if the owner is
   set to the current user.

Status
======

Completed changes:

-  Skip tests if KRA not available
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=8eb26e9230e43eb2683778b8d667c6c7e632ec36>`__).
-  Validate vault's file parameters
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=8e28ddd8fab40e985756729f23e8f352d2dab071>`__).
-  Fixed missing KRA agent cert on replica
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=c8882f7d1c98a795195e7bd2e48323ce95edc858>`__).
-  Validate mutually exclusive options in vault-add
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=7d7ffb62526595433412633c05af5af7909124c8>`__).
-  Validate public key in client
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=e4dff25838f7a2342779851bd68460080d77683b>`__).
-  Add CLI param and ACL for vault service operations
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=0dd95a19ee87a04836f12ad4c1194ad31ac22b93>`__).
-  Allow overriding member param label in LDAPModMember
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=d2da0d89d194f198728b858800dfec447c5d9595>`__).
-  Fix param labels in output of vault owner commands
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=d9e9e5088fe3e093e3291a5e8877e8651645fc61>`__).
-  Fixed vault container ownership
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=419754b1c11139435ae5b5082a51026da0d5e730>`__).
-  Normalize service principal in service vault operations
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=76ab7d9bae1a1381af9e7ed51297b00823cce857>`__).
-  Validate vault type
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=6941f4eec70456c542fb565405eed02cceb54e10>`__).
-  Fix vault-find with criteria
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=29cee7a4bc5a6d2506e7937c982339274fa0edb4>`__).
-  Add container information to vault command results
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=01dd951ddc0181b559eb3dd5ff0336c81e245628>`__).
-  Add flag to list all service and user vaults
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=0abaf195dc3b0920d2439dd4ec6df61e0aadc4f9>`__).
-  Add support for changing vault encryption
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=e46d9236d19f714b67fdf2865f19146c3016f46d>`__).
-  Change default vault type to symmetric
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=19dd2ed758210e859a5b0085de558cf13ba09104>`__).
-  Fix vault tests after default type change
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=9b0a01930bcefda1f37d7de147fed0856c28296f>`__).
-  Limit size of data stored in vault
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=02ab34c60b5e624ef0653a473316633a5618b07c>`__).
-  Using LDAPI to setup CA and KRA agents
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=72cfcfa0bd1e867537fcc788512e5fca20708b83>`__).



Test Plan
=========

http://www.freeipa.org/page/V4/Password_Vault/Test_Plan

References
==========

-  `Password Vault <V4/Password_Vault>`__
