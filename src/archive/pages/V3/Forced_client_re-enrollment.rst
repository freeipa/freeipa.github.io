Forced_client_re-enrollment
===========================

\__NOTOC_\_

Overview
========

Add support for forced client re-enrollment

A host that has been recreated and does not have its host entry disabled
or removed, can be re-enrolled (or rather client configuration can be
reconstructed) using an either previously backed up keytab file or
admin's credentials.

This feature has been added mainly to provide means of automatization of
client re-enrollment when the client machine has been deprovisioned.
This is especially common case with virtual machines.

| https://fedorahosted.org/freeipa/ticket/3374
| https://fedorahosted.org/freeipa/ticket/3482



Use Case
========

With keytab:

#. Machine has been enrolled using OTP or admin's credentials.
#. Keytab file has been backed up.
#. Machine has been effectively destroyed and recreated with the same
   name. This includes restoring from snapshots before enrollment,
   recreating the same host using kickstart, etc.
#. Machine is be re-enrolled using ipa-client-install
   --keytab=path_to_backed_up_keytab

Using admin's credentials:

#. Machine has been enrolled using OTP or admin's credentials.
#. Machine has been effectively destroyed and recreated with the same
   name. This includes restoring from snapshots before enrollment,
   recreating the same host using kickstart, etc.
#. Machine is be re-enrolled using ipa-client-install --force-join

Design
======

Re-enrollment requirements:

Keytab case:

-  Keytab file has been backed up from previous enrollment
-  No changes have been done to the host entry (host has not been
   un-enrolled using ipa-client-install --uninstall) and host has not
   been disabled using host-disable command.

Force-join case:

-  No changes have been done to the host entry (host has not been
   un-enrolled using ipa-client-install --uninstall) and host has not
   been disabled using host-disable command.

A new certificate, ssh keys are generated, ipaUniqueID stays the same.
Additionally, old certificate should be revoked without need for manual
intervention. For manually revoking the certificate, see the ipa
cert-revoke command.



Security Considerations
=======================

To re-enroll the client using --keytab option, one needs to backed up
keytab file from previous enrollment. A root access is required to read
the keytab file.

Even if an malicious user managed to acquire the keytab, only thing he
could achieve is to re-enroll a new machine with the same hostname as
the client. However, since re-enrollment generates new Kerberos keys,
the original client would be rendered invalid.

If the admin knows that there is a possibility that a host has been
compromised, he can run host-disable. This will prevent any
re-enrollment using keytab, because when host is un-enrolled, we disable
the host entry in LDAP and therefore effectively disable the Kerberos
key, SSL certificate and all services of a host.

Using --force-join, there are no newly introduced security threats as
the method of authentication has not changed.

Implementation
==============

Added --keytab option to ipa-client-install. This can be used to specify
path to the keytab and be used to authorize client instead of -p or -w
options. Effectively, only one of -p /-w / -k is required.

Added --force-join option to ipa-client-install. This enforces the host
enrollment, so that it will succeed even if the host entry already
exists. There's no need to specify both --force-join and --keytab,
--keytab is sufficient on its own.

A new option -f has been added to ipa-join. It forces client to join
even if the host entry already exits.

Host entry comparison:

https://www.redhat.com/archives/freeipa-devel/2013-March/msg00033.html
(see the bottom of the message)



Feature Managment
=================

UI

N/A

CLI

N/A



Major configuration options and enablement
==========================================

N/A

Replication
===========

N/A



Updates and Upgrades
====================

N/A

Dependencies
============

N/A



External Impact
===============

N/A



Test Plan
=========

Assumptions:

-  IPA server fqdn: ``server.example.com``
-  IPA replica fqdn:``replica.example.com``
-  IPA client fqdn: ``client.example.com``
-  IPA admin password: ``Secret123``

| 
| Install a FreeIPA server with ``ipa-server-install`` and create one
  replica using the combination of ``ipa-replica-prepare`` and
  ``ipa-replica-install``.



Simulate backup and restore
---------------------------

As machine-level backup and restore is difficult to automate for testing
purposes, restoring the client from backup can be simulated by
performing the following steps:

::

   # iptables -A INPUT -j REJECT -p all --source $MASTER_IP
   # ipa-client-install --uninstall -U
   # iptables -F

| 
| The steps described above will sever communication between server and
  client, and then uninstall the client. As a consequence, the client's
  host entry will remain on the server, ensuring that the forced
  re-enrollment feature works.