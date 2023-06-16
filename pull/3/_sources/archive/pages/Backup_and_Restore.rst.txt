.. _what_is_backup_and_restore:

What is Backup and Restore?
===========================

In many cases there is a lot of confusion about what backup and restore
procedures are destined to solve. On the surface it sounds simple. Back
up data and save it aside; then when something goes wrong take the saved
data and copy it back. What could be simpler? However when
multi-instance deployment, different versions or configurations are
factored in, the backup and restoration becomes a real challenge.

If we step back and ask a question why would one need a backup and
restore or in other words a 'disaster recovery' procedure? Answer is -
to recover from a disaster! Let us see what kind of disasters we want to
recover from and how capability that already exists in the FreeIPA can
be used to overcome these scenarios.

There are two classes of the disaster scenarios:

-  **Server Loss**: The FreeIPA deployment loses one, several, or all
   servers due to a disaster (fire, earthquake, hardware malfunction,
   etc.) and needs to get back online as soon as possible.
-  **Data Loss**: FreeIPA data was accidentally deleted, either by a
   user or by a software bug, and the deletion was propagated to all
   servers, given that FreeIPA is a multi-master solution.

.. _server_loss_cases:

Server Loss Cases
=================

Our usual recommendation for redundancy is to run several (2-3) FreeIPA
Servers in each data center in customer deployment and let them
replicate with each other. This way, when one server is lost, it can be
recovered by simply creating a new FreeIPA Server (replica) and get back
to fully functional state really quickly.

However, FreeIPA Servers are not born equal and a lot depends on the
configuration. The first server that was ever installed has special
properties that make it slightly different from others. It can be called
as the *first master*. The first master is no different from other
masters in the deployment except for the certificate management aspects.
If the first master was installed with the full root or chained
`CA <PKI>`__ it is the server that:

-  **Tracks and renews internal certificates**: all other servers just
   get a copy of the renewed certs
-  **Publishes CRLs**: all other masters do not do that by default but
   can be configured

Other masters can be deployed with full `CA <PKI>`__ like the first
master or without `CA <PKI>`__ like a more lightweight replica. Since
`FreeIPA 3.2 <Releases/3.2.0>`__ there is also a way to install the
whole deployment without `CA <PKI>`__ (`CA-less
installation <V3/CA-less_install>`__). Then all FreeIPA Servers are
equal and there is no distinction between them.

.. _one_server_loss:

One Server Loss
---------------

The *one server loss* scenarios are related to the cases when, for
example, a hardware failure is experienced and a server in a deployment
needs to be removed and potentially replaced. It is worth mentioning
here that the replace scenario in the following procedures is always
about following a remove/cleanup procedure and then installing a new
server on the same or different hardware, or provisioning a new VM with
the same or different identity.

.. _one_server_loss___first_master:

One Server Loss - First Master
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If the first master is lost, other master with a full `CA <PKI>`__ (if
it is not a CA-less deployment) needs to be nominated as the new first
master following a manual procedure:

#. Clean deployment from the lost server by `removing all replication
   agreements <https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Identity_Management_Guide/removing-replica.html>`__
   with it.
#. Choose another FreeIPA Server with `CA <PKI>`__ installed to become
   the first master
#. Nominate this master to be the one in charge or renewing certs and
   publishing CRLS. This is a manual procedure at the moment.
#. Follow `standard installation
   procedure <https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Identity_Management_Guide/creating-the-replica.html>`__
   to deploy a new master on a hardware/VM of your choice

Changing the topology of a deployment has an impact on the clients. To
mitigate this impact we recommend to rely on the `DNS <DNS>`__ discovery
and let clients automatically adapt to the topology changes. If FreeIPA
`DNS <DNS>`__ is used the topology changes are reflected automatically.
If `DNS <DNS>`__ is managed manually, `DNS <DNS>`__ SRV records need to
be updated to reflect the new FreeIPA topology. If FreeIPA clients were
configured to explicitly connect to specific servers, their
configuration also needs to reflect new FreeIPA Server hostnames.

.. _one_server_loss___any_other_server:

One Server Loss - Any other server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When the first master is still running, procedure to recover a lost
FreeIPA Server is more straightforward:

#. Clean deployment from the lost server by `removing all replication
   agreements <https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Identity_Management_Guide/removing-replica.html>`__
   with it.
#. Follow `standard installation
   procedure <https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Identity_Management_Guide/creating-the-replica.html>`__
   to deploy a new master on a hardware/VM of your choice

.. _several_server_loss:

Several Server Loss
-------------------

If several servers are lost at the same time, one needs to determine
whether the deployment is rebuild-able from what is left or not.

.. _first_master_is_still_alive:

First Master is still alive
~~~~~~~~~~~~~~~~~~~~~~~~~~~

If first master is still alive `One Server Loss - Any other
server <#One_Server_Loss_-_Any_other_server>`__ procedure can be
followed to rebuild every lost server and restore the environment.

.. _first_master_is_lost:

First Master is lost
~~~~~~~~~~~~~~~~~~~~

If there was an installation with a `CA <PKI>`__ and at least one master
with `CA <PKI>`__ is available then first follow the procedure `One
Server Loss - First Master <#One_Server_Loss_-_First_Master>`__ to
establish the new first master and then follow the procedure `One Server
Loss - Any other server <#One_Server_Loss_-_Any_other_server>`__
procedure for every lost server to rebuild the environment.

If there is no master with a `CA <PKI>`__ left, the deployment
effectively lost an ability to rebuild itself from what is left.
Therefore, this case needs to be treated as a **total loss scenario**.

.. _this_is_a_ca_less_deployment:

This is a CA-less deployment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In a `CA-less deployment <V3/CA-less_install>`__ all masters are equal
and `One Server Loss - Any other
server <#One_Server_Loss_-_Any_other_server>`__ procedure can be
followed to rebuild the environment

.. _total_infrastructure_loss:

Total Infrastructure Loss
=========================

This is the case when all the servers in a deployment are lost or what
is left is not good enough to rebuild the deployment. For that specific
case we suggest that you run one of your replicas (with full
`CA <PKI>`__ if it is not a `CA-less install <V3/CA-less_install>`__) in
a VM then periodically stop this VM and have a full snapshot of it and
then bring it back again.

.. _why_snapshot_and_not_backup_and_restore_scripts:

Why snapshot and not backup and restore scripts?
------------------------------------------------

Let us step back and reflect a bit on the choice between snapshot and
backup scripts. Snapshot saves everything in a consistent state. Backup
is supposed to do it too. However backup has several major differences:

-  Backup stores only data and not the software itself while snapshot
   saves both data and the software
-  Backup selects what to save one by one based on the code written by
   developer, snapshot takes everything

As one can see with a backup script there is much more room for a human
mistake. The data when restored might not match the software expectation
because software was updated between the moments when backup and restore
events happened. Of course, special checks can be added but this means
more code, more complexity and more risk to make a mistake.

The selectiveness of the backup is yet another concern. The software
evolves and new features may affect the data so that backup might not
pick everything or restore would overwrite something. Since there is no
way to predict the state of the data when the restore will be run there
is a higher risk that problems would be introduced. Which is especially
troublesome in situations when coming back online as soon as possible is
crucial. We feel that using less risky procedures would enable FreeIPA
users to recover faster. This is the main reason why FreeIPA team was
reluctant to build custom backup and restore scripts.

.. _backup_and_restore_scripts:

Backup and restore scripts
~~~~~~~~~~~~~~~~~~~~~~~~~~

`FreeIPA 3.2 <Releases/3.2.0>`__ introduced experimental `backup and
restore scripts <V3/Backup_and_Restore>`__. See man pages for
``ipa-backup`` and ``ipa-restore`` scripts for the instructions how to
backup and restore FreeIPA software and/or the database. A feedback from
real user deployments is essential for decision if the scripts should be
further developed by the FreeIPA team.

.. _recovering_from_a_snapshot:

Recovering from a snapshot
--------------------------

.. _nothing_left_other_than_the_snapshot:

Nothing left other than the snapshot
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Boot the snapshot VM and follow the procedure
`CA <#Several_Server_Loss_-_There_is_a_full_%5B%5BPKI>`__ master|Several
Server Loss - There is a full `CA <PKI>`__ master]]. When the procedure
is completed, there is a restored and functional deployment but the VM
is now the first master. We recommend that other server is nominated and
updated to be the first master.

.. _something_is_left_other_than_the_snapshot:

Something is left other than the snapshot
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is the situation when there are remnants of the old infrastructure
that do not allow to fully rebuild (they for example miss an FreeIPA
Server with `CA <PKI>`__ configured), but they are still functional and
have the database intact so that `clients <Client>`__ can authenticate
while the environment is being rebuilt.

In such situation we recommend following procedure:

#. `Clean remaining FreeIPA Servers from replication
   agreements <https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Identity_Management_Guide/removing-replica.html>`__
   with the lost servers. The goal is to have have a set of synchronized
   remaining FreeIPA Servers with functional replication agreements
   between each other. Replication agreement with the snapshot VM can be
   left intact as it will be used to synchronized the snapshot back up
   with the remaining FreeIPA infrastructure later.
#. Boot the selected snapshot and start the restored FreeIPA Server
#. See if the FreeIPA Server running from snapshot `has a replication
   agreement <https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Identity_Management_Guide/ipa-replica-manage.html#viewing-repl-agmt>`__
   with one of the other FreeIPA Servers that survived. If not, `connect
   the FreeIPA Server running from the
   snapshot <https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Identity_Management_Guide/managing-sync-agmt.html#Creating_Synchronization_Agreements>`__
   to one of the servers that survived to replica data.
#. Check ``/var/log/dirsrv/slapd-YOUR-INSTANCE/errors`` and see if the
   FreeIPA Server running from the snapshot correctly synchronizes with
   the remaining FreeIPA Servers and if it received the fresh data. If
   the replication fails for the database being too old, it `can be
   reinitialized <https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Identity_Management_Guide/ipa-replica-manage.html#initialize>`__
   from a running FreeIPA Server.
#. If database is correctly synchronized, `install any required
   additional FreeIPA
   Servers <https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Identity_Management_Guide/creating-the-replica.html>`__
   to fully restore the FreeIPA infrastructure

If the backed up snapshot is too old and it's state is not consistent
with a state of the remaining FreeIPA Servers so that it's database can
be neither synchronized nor reinitialized, a different procedure needs
to be applied:

#. `Remove any replication
   agreement <https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Identity_Management_Guide/removing-replica.html>`__
   of the remaining FreeIPA Servers with the IPA Server that will be
   restored from a snapshot. This step will prevent replication of
   inconsistent data to the restored FreeIPA Server
#. Boot the selected snapshot and start the restored FreeIPA Server.
   Install a sufficient amount of FreeIPA replicas from the FreeIPA
   Server running from the snapshot to be able to handle the load of the
   deployment. When step 2 is finished, there will be 2 disconnected
   FreeIPA deployments
#. Switch `clients <Client>`__ to use the restored FreeIPA Servers
#. `Stop and
   uninstall <https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Identity_Management_Guide/Uninstalling_IPA_Servers.html>`__
   FreeIPA Servers of the old infrastructure
#. `Install any required additional FreeIPA
   Servers <https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Identity_Management_Guide/creating-the-replica.html>`__
   to fully restore the FreeIPA infrastructure

In this case, old FreeIPA Servers and the new FreeIPA Servers should run
in parallel only for a limited amount of time needed to create a
sufficient restored FreeIPA infrastructure to limit data inconsistencies
between these two disconnected FreeIPA realms.

.. _recovering_freeipa_clients:

Recovering FreeIPA clients
--------------------------

While FreeIPA Servers are restored, FreeIPA `clients <Client>`__ may
need changes as well:

-  **FreeIPA Server hostnames**: if `client <Client>`__ is configured
   with a hardcoded FreeIPA Server hostname and the hostname was changed
   (i.e. during restoration process), it's configuration needs to be
   updated to reflect the new hostnames. At minimum,
   ``/etc/sssd/sssd.conf`` and ``/etc/krb5.conf`` should be updated.
   Situation is easier if the FreeIPA `clients <Client>`__ are
   configured to use FreeIPA Server autodiscovery via `DNS <DNS>`__ SRV
   records. Then only the `DNS <DNS>`__ SRV needs to be updated to let
   the FreeIPA `clients <Client>`__ properly resolve the servers.
-  **Stale data**: especially in `Total Infrastructure
   Loss <#Total_Infrastructure_Loss>`__ cases, it may make sense to
   remove any stale data on `client <Client>`__ and `purge the SSSD
   cache <https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/sssd-cache.html>`__.
   This step is optional, but it should be considered if users
   experience inconsistent login behavior on FreeIPA
   `clients <Client>`__.

.. _data_loss_cases:

Data Loss Cases
===============

Up to this point, we have discussed only server losses and not about the
data losses. Data loss cases are more difficult to recover from and
there is no generic procedure that can be reused among all FreeIPA
deployments. The procedure depends on the data that was lost; Is it a
group? A set of users? Host data? Let us provide some guidelines on how
to recover from this situation in the best way.

As soon as it is determined that there is a data loss situation, actions
should be taken to try to stop the data loss proliferation in the
infrastructure. Affected replicas should be isolated and brought down as
soon as possible to prevent spread of the data loss.

If the data loss is stopped and there is still a part of the
infrastructure that is not affected, the situation may be treated as all
the servers that are corrupted are lost and left with the ones that are
not. The recovery procedures described in the `Server Loss
Cases <#Server_Loss_Cases>`__ can be used to rebuild the infrastructure.
Affected replicas should not be brought up until re-deployed to avoid
corrupted data spreading again.

If the data loss affected all servers there are 2 options:

-  **Start over** from a snapshot as described in the `Something is left
   other than the
   snapshot <#Something_is_left_other_than_the_snapshot>`__ and make
   sure there is no replication between old and new replicas. This is a
   pretty drastic approach and should be used when most of the data is
   lost and deployment is completely dysfunctional anyway.

-  **Re-add lost data**. Use this method if the scope of the data loss
   can be identified. This procedure expects that either there exists a
   VM snapshot an FreeIPA Server before the data loss that can be used
   to retrieve s snapshot of the database (LDIF) with the database or
   that the FreeIPA Server database was backed up, either by using
   standard `Directory Server <Directory_Server>`__ tools to back up the
   data (``db2ldif``) or by using FreeIPA backup command
   (``ipa-backup --data --online``) if it is available in the deployed
   version of the FreeIPA. When the database snapshot (LDIF) is
   retrieved, lost LDAP entries can be retrieved from it and added to
   the database via standard ``ldapadd`` command. The restored entries
   will be then automatically propagated to other masters by standard
   replication.
