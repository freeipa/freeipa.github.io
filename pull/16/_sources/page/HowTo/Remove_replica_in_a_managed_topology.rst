Remove_replica_in_a_managed_topology
====================================

Introduction
------------

In IPA 4.3 the replication topology is controlled by the topology
plugin. Connections between replicas can only be created or removed by
the

ipa topologysegment-[add|del] 

commands.

When a replica is removed by running

ipa-replica-manage del 

the removal of replication segments and corresponding replication
agreements is automatically handled by the topology plugin.

But to be able to propagate these changes throughout the topology all
the remaining servers need to be connected. You should be careful in the
setup of a replication topology and in the actions to remove a replica.
the following sections will make recommendations for designig a
replication topology and how to procede when a replica has to be removed



Designing a replication topology
--------------------------------

The replication topology deployed should follow these rules:

Avoid single points of failure, the unavailability of one server or one
connection between servers should not disconnect the remaining servers.

Avoid high replication traffic in one node.

This will be further explained for the following replication topology
types:

.. figure:: Topo-types.png
   :alt: Topo-types.png

   Topo-types.png

In Type 1 the replicas are connected in a string, any failing server
will disconnect the others.

In Type 2 the replicas are connected in a star like pattern, if replica
F is down all other replicas are disconnected, also replica F has to
manage all replicaion distribution from other servers, replica A,B ,C,D,
E will compete to update replica F, only one supplier can update a
replica at one time

In type 3 there is a good balance, all replicas are connected to others
vial different paths, also no replica has more than three replication
connections to handle



Safely removing a replica
-------------------------



Removing a "leaf" replica
----------------------------------------------------------------------------------------------

Assuming the following replication topology:

``A <--> B <--> C <--> D``

then the replicas at the endpoints of the replication chain, A or D, can
safely be removed by:

``ipa-replica-manage del A``

The command removes the master entry for A and this will trigger that
the topoloy plugin removes the segment connecting A and B an will remove
the replication agreements A-->B and B-->A



Removing an "internal" replica
----------------------------------------------------------------------------------------------

Assume that in the above topology the internal node C should be removed.

If the command

ipa-replica-manage del C 

is executed, the command will detect that the removal of C disconnects
the topology and rejects the command.

It will also list the replicas which will be no longer connected to each
other, use this information to add missing segments before finally
deleting the replica

| ``checking connectivity in topology suffix 'ipaca'``
| ``WARNING: Removal of 'vm-110.abc.idm.lab.eng.brq.redhat.com' will lead to disconnected topology in suffix 'ipaca'``
| ``Changes will not be replicated to all servers and data will become inconsistent.``
| ``You need to add segments to prevent disconnection of the topology.``
| ``Errors in topology after removal:``
| ``Topology does not allow server vm-072.abc.idm.lab.eng.brq.redhat.com to replicate with servers:``
| ``   vm-192.abc.idm.lab.eng.brq.redhat.com``
| ``Topology does not allow server vm-192.abc.idm.lab.eng.brq.redhat.com to replicate with servers:``
| ``   vm-072.abc.idm.lab.eng.brq.redhat.com``

``NOTE: the removal of an internal node can be enforced by adding --force to the command, but it should never be done``

There are two cases

a) node C is working and properly handling replication traffic

Even if the node C is still operable and replicating its removal should
not be enforced. The removal of C will trigger the removal of the
connections to B and D, there can be race conditions where not all
information on topology change is replicated to D before the agreement
to D is removed.

And node D will be isolated after removal of C, so new segments will
have to be added. Therefore the adding of the new segment should be done
before the removal.

b) node C is inoperable and the topology is already broken.

In this scenario there is no chance that the removal of C will be
propagated to D (if the removal is triggered on A or B) or to A,B (if
the removal is triggered from D). But the removal information is kept in
the changelog and will be sent after the topology is connected again by
adding new segments. Depending what other toplogy changes have been done
in between there could be race conditions where the late replication of
the removal could give unexpeted results.

As a conclusion, in both cases the topology should be extended first to
keep the topology connected, by running

``ipa topologysegment-add --leftnode=A --rightnode=D``

This ensures that the toplogy is fully connected again and keeps
connected after C is removed