Overview
========

Password Vault 1.0 provides the initial implementation of `Password
Vault <V4/Password_Vault/Design>`__.

Secret
------

Secret is an information that should only be known by a limited number
of people. IPA provides a mechanism to archive, retrieve, share, and
recover secrets, and manage the access control. The secret itself will
actually be stored securely in the KRA backend.

A secret is a binary data, so anything can be stored as a secret (e.g.
password, PIN, private key), but there will be a limit imposed by the
server. A secret ID is a unique name to identify a secret within a
vault.

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

Vault is a secure place to store a secret. A vault may be privately
owned by a user, a service, or shared. A user or service may own
multiple vaults.

A vault is uniquely identified by its name within its container. See
`Container <#Container>`__.

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

Container is a collection of vaults. There are several built-in
containers:

-  service containers
-  shared container
-  user containers

Each user has a user container to store the user's private vaults. The
container is created automatically when the first private vault is
created. When the user is removed, the container and its contents will
be removed automatically.

The shared container is used to store vaults that can be shared by
different users. Only the admins can create shared vaults.

Each service has a service container to store the service's private
vaults. The container is created automatically when the first private
vault is created. When the service is removed, the container and its
contents will be removed automatically.



Access Control
--------------

The vault access control list allows the following operations:

-  Any user can create a private user container.
-  A container owner can manage vaults in the container.
-  A vault member can list, archive, and retrieve the secrets in a
   standard vault. With symmetric vault the member will need a vault
   password in order to archive and retrieve secrets. With asymmetric
   vault the member can only archive the secrets but not retrieve them
   since it only has the public key and not the private key.
-  A vault owner can list, archive, and retrieve the secrets like vault
   members, but it has the private key so it can retrieve secrets from
   asymmetric vault. The owner can also manage the list of members and
   owners of the vault, and change the vault password/keys.
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
     Description: Private vault
     Type: standard
   ----------------------------
   Number of entries returned 1
   ----------------------------

To find service vaults, specify --service :

::

   $ ipa vault-find --service HTTP/server.example.com
   ---------------
   1 entries found
   ---------------
     Vault name: test
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
     Type: standard
   ----------------------------
   Number of entries returned 1
   ----------------------------



Displaying vault info
---------------------

A user can view a particular vault info using the following command:

::

   $ ipa vault-show <name> [OPTIONS]

To display the basic vault info:

::

   $ ipa vault-show PrivateVault
     Vault name: PrivateVault
     Description: Private vault
     Type: standard
     Salt: ....
     Owner users: admin

To display the complete vault info:

::

   $ ipa vault-show PrivateVault --all
     dn: cn=PrivateVault,cn=admin,cn=users,cn=vaults,...
     Vault name: PrivateVault
     Description: Private vault
     Type: standard
     Salt: ....
     Owner users: admin
     objectclass: top, ipaVault



Creating a new vault
--------------------

A container member can create a vault using the following command:

::

   $ ipa vault-add <name> [OPTIONS]

Private vaults can be created by specifying a relative vault ID:

::

   $ ipa vault-add PrivateVault --desc "Private vault"
   --------------------------
   Added vault "PrivateVault"
   --------------------------
     Vault name: PrivateVault
     Description: Private vault
     Type: standard

Shared vaults can be created by specifying --shared:

::

   $ ipa vault-add SharedVault --desc "Shared vault" --shared
   ---------------------------------
   Added vault "SharedVault"
   ---------------------------------
     Vault name: SharedVault
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
     Description: Symmetric vault
     Type: symmetric

   $ ipa vault-add SymmetricVault --desc "Symmetric vault" --type symmetric --password mypassword
   ----------------------------
   Added vault "SymmetricVault"
   ----------------------------
     Vault name: SymmetricVault
     Description: Symmetric vault
     Type: symmetric

   $ ipa vault-add SymmetricVault --desc "Symmetric vault" --type symmetric -password-file password.txt
   ----------------------------
   Added vault "SymmetricVault"
   ----------------------------
     Vault name: SymmetricVault
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
     Description: Asymmetric vault
     Type: asymmetric



Archiving data
--------------

A vault member/owner can archive data using the following command:

::

   $ ipa vault-archive <name> [--in <input file> | --data <base-64 encoded data>] [OPTIONS]

With a standard vault the operation can be done directly.

::

   $ ipa vault-archive StandardVault --data c2VjcmV0
   ----------------------------------------
   Archived data into vault "StandardVault"
   ----------------------------------------
   $ ipa vault-archive StandardVault --in secret.txt
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

   $ ipa vault-retrieve <name> [--out <output file>] [OPTIONS]

With a standard vault the operation can be done directly.

::

   $ ipa vault-retrieve StandardVault --out secret.txt
   -----------------------------------------
   Retrieved data from vault "StandardVault"
   -----------------------------------------
   $ ipa vault-retrieve StandardVault
   -----------------------------------------
   Retrieved data from vault "StandardVault"
   -----------------------------------------
       Data: c2VjcmV0

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



Modifying a vault
-----------------

The vault owner can modify a vault using the following command:

::

   $ ipa vault-mod <name> [OPTIONS]

For example, to change vault description:

::

   $ ipa vault-show PrivateVault
     Vault name: PrivateVault
     Description: Private vault
     Type: standard

   $ ipa vault-mod PrivateVault --desc "This is a private vault"
   -----------------------------
   Modified vault "PrivateVault"
   -----------------------------
     Vault name: PrivateVault
     Description: This is a private vault
     Type: standard



Changing vault password/keys
----------------------------

To change the password of a symmetric vault execute the following
commands (Note: In `Password Vault 1.1 <V4/Password_Vault_1.1>`__ this
process will be merged into a single command):

::

   $ ipa vault-retrieve SymmetricVault --out secret.txt
   Password: ********
   ------------------------------------------
   Retrieved data from vault "SymmetricVault"
   ------------------------------------------

   $ ipa vault-del SymmetricVault
   ------------------------------
   Deleted vault "SymmetricVault"
   ------------------------------

   $ ipa vault-add SymmetricVault --type symmetric
   New Password: ********
   Verify Password: ********
   ----------------------------
   Added vault "SymmetricVault"
   ----------------------------
     Vault name: SymmetricVault
     Type: symmetric

   $ ipa vault-archive SymmetricVault --in secret.txt
   Password: ********
   -----------------------------------------
   Archived data into vault "SymmetricVault"
   -----------------------------------------

To change the public/private keys of an asymmetric vault execute the
following commands (Note: In `Password Vault
1.1 <V4/Password_Vault_1.1>`__ this process will be merged into a single
command):

::

   $ ipa vault-retrieve AsymmetricVault --out secret.txt --private-key-file private.pem
   -------------------------------------------
   Retrieved data from vault "AsymmetricVault"
   -------------------------------------------

   $ ipa vault-del AsymmetricVault
   -------------------------------
   Deleted vault "AsymmetricVault"
   -------------------------------

   $ ipa vault-add AsymmetricVault --type asymmetric --public-key-file public.pem
   -----------------------------
   Added vault "AsymmetricVault"
   -----------------------------
     Vault name: AsymmetricVault
     Type: asymmetric

   $ ipa vault-archive AsymmetricVault --in secret.txt
   ------------------------------------------
   Archived data into vault "AsymmetricVault"
   ------------------------------------------



Removing a vault
----------------

To remove a vault the owner can execute the following command:

::

   $ ipa vault-del <name> [OPTIONS]

For example:

::

   $ ipa vault-del PrivateVault
   ----------------------------
   Deleted vault "PrivateVault"
   ----------------------------



Access Control
==============



Adding vault member
-------------------

A vault owner can add members to the vault with the following command:

::

   $ ipa vault-add-member <name> [--users <list of users>] [--groups <list of groups>]

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

   $ ipa vault-remove-member <name> [--users <list of users>] [--groups <list of groups>]

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

   $ ipa vault-add-owner <vault ID> [--users <list of users>] [--groups <list of groups>]

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

   $ ipa vault-remove-owner <name> [--users <list of users>] [--groups <list of groups>]

For example:

::

   $ ipa vault-remove-owner MyVault --users testowner
   ------------------------------------
   Removed owners from "MyVault " vault
   ------------------------------------



Service Operations
==================



Creating service vault password
-------------------------------

A service administrator can create a service vault password by archiving
a new secret into a private vault:

::

   $ ipa vault-add http_password
   ---------------------------
   Added vault "http_password"
   ---------------------------
     Vault name: http_password
     Type: standard
     Owner users: admin

   $ ipa vault-archive http_password --in secret.txt
   ----------------------------------------
   Archived data into vault "http_password"
   ----------------------------------------



Provisioning service vault password for service instance
--------------------------------------------------------

A service administrator can provision the service vault password to a
specific service instance using a service vault. To create a service
vault, obtain the service public key, then execute the following command
(**Note:** In the future the service public key will be retrieved
automatically):

::

   $ ipa vault-add <service vault name> --service <service name> --type asymmetric --public-key <service public key>

To copy the service vault password from the service administrator's
private vault into the service vault execute the following commands
(**Note:** In `Password Vault 1.1 <Password_Vault_1.1>`__ this will be
merged into a single command):

::

   $ ipa vault-retrieve <private vault name> --out secret.txt
   $ ipa vault-archive <service vault name> --service <service name> --in secret.txt

The commands will retrieve the service vault password already archived
earlier, then encrypt it with the service instance's public key. The
public key will be obtained from the service certificate that's already
generated previously on the server.

For example:

::

   $ ipa vault-add password --service HTTP/server.example.com --type asymmetric --public-key-file service-public.pem
   ---------------------
   Added vault "password"
   ---------------------
     Vault name: password
     Type: asymmetric
     Public key: ...
     Owner users: admin

   $ ipa vault-retrieve http_password --out secret.txt
   -----------------------------------------
   Retrieved data from vault "http_password"
   -----------------------------------------

   $ ipa vault-archive password --service HTTP/server.example.com --in secret.txt
   -----------------------------------
   Archived data into vault "password"
   -----------------------------------



Retrieving service vault password for service instance
------------------------------------------------------

A service instance can retrieve the service vault password using the
service private key stored locally:

::

   $ kinit HTTP/server.example.com -k -t /etc/httpd/conf/ipa.keytab

   $ ipa vault-retrieve password --service HTTP/server.example.com --private-key-file service-private.pem --out secret.txt
   ------------------------------------
   Retrieved data from vault "password"
   ------------------------------------



Changing service vault password
-------------------------------

The service administrator can change the service vault password by
archiving a new secret:

::

   $ ipa vault-archive http_password --in new_secret.txt
   ----------------------------------------
   Archived data into vault "http_password"
   ----------------------------------------

The service administrator will need to re-provision the new service
vault password to each service instance using the following command:

::

   $ ipa vault-retrieve http_password --out secret.txt
   -----------------------------------------
   Retrieved data from vault "http_password"
   -----------------------------------------

   $ ipa vault-archive password --service HTTP/server.example.com --in secret.txt
   -----------------------------------
   Archived data into vault "password"
   -----------------------------------

This way if there's a compromised instance the service administrator can
isolate it by changing the service vault password and re-provisioning it
to non-compromised instances only.

Configuration
=============

The following command can be used to display vault configuration:

::

   $ ipa vaultconfig-show
     Transport Certificate: -----BEGIN CERTIFICATE-----
   ...
   -----END CERTIFICATE-----



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
   + cn=kra
      + cn=vaults
         + cn=users
            + cn=<username>
               + cn=<user vault name>
               + ...
         + cn=shared
            + cn=<shared vault name>
            + ...
         + cn=services
            + cn=<service name>
                + cn=<service vault name>
                + ...

Schema
------

Vault schema is defined in install/share/60basev3.ldif.

Attribute types:

::

   attributeTypes: (2.16.840.1.113730.3.8.18.2.1 NAME 'ipaVaultType' DESC 'IPA vault type' EQUALITY caseExactMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.15 X-ORIGIN 'IPA v4.2')
   attributeTypes: (2.16.840.1.113730.3.8.18.2.2 NAME 'ipaVaultSalt' DESC 'IPA vault salt' EQUALITY octetStringMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.40 X-ORIGIN 'IPA v4.2' )
   attributeTypes: (2.16.840.1.113730.3.8.18.2.3 NAME 'ipaVaultPublicKey' DESC 'IPA vault public key' SUP ipaPublicKey X-ORIGIN 'IPA v4.2' )

Object classes:

::

   objectClasses: (2.16.840.1.113730.3.8.18.1.1 NAME 'ipaVault' DESC 'IPA vault' SUP top STRUCTURAL MUST ( cn ) MAY ( description $ ipaVaultType $ ipaVaultSalt $ ipaVaultPublicKey $ owner $ member ) X-ORIGIN 'IPA v4.2' )
   objectClasses: (2.16.840.1.113730.3.8.18.1.2 NAME 'ipaVaultContainer' DESC 'IPA vault container' SUP top STRUCTURAL MUST ( cn ) MAY ( description $ owner ) X-ORIGIN 'IPA v4.2' )

See also `LDAP schema for PKCS#11
data <http://www.freeipa.org/page/V4/PKCS11_in_LDAP/Schema>`__.



Access Control List
-------------------

The LDAP ACI attributes are used to control the access to the LDAP
entries representing the vaults and the containers. The secrets
themselves are stored in KRA and accessed by IPA as KRA agent on behalf
of IPA users. The IPA user's access to the secrets will be determined by
IPA framework based on the user's membership or ownership of the vaults
and containers, not by LDAP ACI.

The ACI attributes are defined in the root entry of the vault subtree in
install/share/vault.update:

::

   dn: cn=kra,$SUFFIX
   ...

   dn: cn=vaults,cn=kra,$SUFFIX
   ...

   ################################################################################
   # Vault Container ACLs
   ################################################################################
   aci: (target="ldap:///cn=*,cn=users,cn=vaults,<suffix>")
     (version 3.0; acl "Allow users to create private container";
      allow (add) userdn = "ldap:///uid=($attr.cn),cn=users,cn=accounts,$SUFFIX";)

   ################################################################################
   # Vault ACLs
   ################################################################################
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

Demo
====

See `Demo <http://pki.fedoraproject.org/wiki/IPA_Password_Vault_1.0>`__.

Status
======

-  Fixed KRA backend
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=0b08043c37210d0f86cb0c66d659acafda0fb529>`__).
-  Modified NSSConnection not to shutdown existing database
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=80a8df3f193aa800740f1627a269e6973f57aa0a>`__).
-  Added vault plugin
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=fde21adcbd62b9a300740d9ba237ca9e89a905e4>`__).
   This implements section 3.1, 3.2, 3.3, 3.6, 3.7.
-  Added vault archive and vault retrieve commands
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=df1bd39a43f30138cf55e0e7720fa3dec1d912e0>`__).
   This implements section 3.4, 3.5, and 5.
-  Moved vaults to cn=vaults,cn=kra
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=81729e22d35c5313e85081b6b3e8658b3d542af1>`__).
-  Fixed ipa-kra-install
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=e7ac57e1390c76c3d7fdb2710808def107d21d6d>`__).
-  Added symmetric and asymmetric vaults
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=fc5c614950dd39c7d002377f810f37ef36b0e8a4>`__).
   This implements section 1.2.2 and 1.2.3 and adds python-cryptography
   dependency.
-  Added ipaVaultPublicKey attribute.
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=475ade4becd4cdb59a9bcf0da7de1d2739e293c8>`__).
-  Added vault access control.
   (`pushed <https://git.fedorahosted.org/cgit/freeipa.git/commit/?id=bf6df3df9b388753a52a0040d9c15b1eabce41ca>`__).
   This implements section 4.



Test Plan
=========

-  `Test
   Plan <http://www.freeipa.org/page/V4/Password_Vault/Test_Plan>`__

References
==========

-  `Password Vault <V4/Password_Vault>`__
-  `IPA KRA Agent
   Setup <http://pki.fedoraproject.org/wiki/IPA_KRA_Agent_Setup>`__
