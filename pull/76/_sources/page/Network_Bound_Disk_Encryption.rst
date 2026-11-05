Network_Bound_Disk_Encryption
=============================

Overview
========

The purpose of this page is to outline a plan for binding an encrypted
disk to a network.



Use Case
--------

A server's storage contains sensitive data. Suddenly, the disk fails.
The disk needs to be sent in for warranty service. However, corporate
policy does not permit the disclosure of this data. Since the drive has
failed, it cannot be wiped.

While simply encrypting this disk with a master secret will suffice, the
management for this is cumbersome. Either manual intervention is
required for encryption and booting or the master secret needs to be
shared in automation tooling risking the security of the system.

What is required is an automated system for provisioning and acquisition
of per-host decryption keys.

Architecture
============

The basic workflow is two steps:

#. Provisioning: enabling the encrypted host for automatic decryption
#. Acquisition: fetching the decryption key and using it to unlock the
   disk

This process is implemented using
`Petera <https://github.com/npmccallum/petera>`__. For an example of
this workflow, see the `Petera Disk Encryption
instructions <https://github.com/npmccallum/petera/wiki>`__.

Provisioning
------------

Provisioning is the process whereby the encrypted host is enabled for
automatic decryption at boot. This process presumes that the disk is
already encrypted at install time using a recovery key. Alternatively,
the provisioning process can be cooked into Anaconda directly.

#. Generate a random key
#. Add the random key to LUKS
#. Encrypt the random key to the Petera server's encryption certificate
   (online or offline operation)
#. Store the encrypted random key on the local (encrypted) disk
#. Copy the encrypted random key into the initramfs

No data is ever stored on the Petera server. This entire procedure can
be performed offline if the Petera server's encryption certificate is
known.

Acquisition
-----------

These are the steps that occur in the initramfs in order to decrypt the
root volume.

#. Bring up the network
#. Connect to the Petera server and validate the TLS certificate
   (including stapled OCSP)
#. Send the encrypted random key to the Petera server
#. Receive the (decrypted) random key from the Petera server
#. Use the random key to decrypt the disk

This process is entirely automated and the random key can only ever be
decrypted by the Petera server. This requires network connectivity, but
no authentication is needed.



Other Considerations
====================

Escrow
------

Recovery key escrow is not part of this plan. It is likely that we will
also want to generate a random recovery key and store it somewhere
secure. This should probably be the FreeIPA Vault.

The escrow feature only requires changes to provisioning and/or
Anaconda. The acquisition phase should not be affected.