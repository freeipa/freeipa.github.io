Monitor_Replication_Topology
============================

Overview
--------

In a replication topology there exist many replication agreements and to
monitor the state of these agreements each of the servers in the
topology has to be queried. With the deployment of the topology plugin
it is possible to have the complete view of the topology on any server.
This paper describes how the topology plugin functions can be etxended
to provide replication monitoring.



Use Cases
---------

TODO

Design
------

The feature is implemented as an extension to the topology plugin. It
implements a query for replication state, which is propagated to all
servers in the topology, the result of the query is conatained in the
topology segments and can be listed or visualized by contacting any
server.



New attributes in topology objects
----------------------------------------------------------------------------------------------

The suffix and segment entries will be extended to manage monitoring
requests and responses

Extension of the ipaReplicationTopologyConfig entry:

::

   | ``cn=``\ ``,cn=topology,``
   | ``objectclass: ipaReplicationTopologyConfig``
   | ``ipaReplTopoConfigRoot: < dn >``
   | ``ipaReplTopoMonitorRequest: ``

The request ID is used to map the monitoring responses to a request, to
trigger a new request a ldap replace operation for this attribute is
applied. A simple time stamp is used as requestID.

Extension to the ipaReplicationTopologySegment:

| ``cn: ``\ ``, cn=``\ ``,cn=topology,``
| ``objectclass: ipaReplicationTopologySegment``
| ``ipaReplSegmenState;left : ``\ ``:``
| ``ipaReplSegmenState;right : ``\ ``:``

For each agreement a replication state attribute is added ";left"
";right" indicate to which of the two agreements that can be represented
by a segment the information refers



plugin operation
----------------------------------------------------------------------------------------------

The topology plugin monitors updates to the ipaReplTopoMonitorRequest
attribute, if this attribute is replaced wit a new monitoringID it
checks the status of the local agreements and updates the
ipaReplTopoState attributes in the corresponding segments. These updates
will be replicated to the other replicas



Monitoring procedure
----------------------------------------------------------------------------------------------

A client, CLI or GUI, initiates a monitoring cycle by modifying the
ipaReplTopoMonitorRequest attribute on one server. It then contacts
this, or any otehr, server and reads the topology segments for the
monitored suffix. It may take some time until the state information has
been updated on all servers and finally replicated, so this maybe has to
be repeated until a defined maximal waiting period passed.

Based on the received information, the state of the agreements can be
listed. The state of the agreement will be:

| ``green: the agrement is operational``
| ``red: agreement is not working (no more refinements yet)``
| ``grey: no response for the monitoring request with the given monitoringID was received``



Required actions
----------------------------------------------------------------------------------------------

If all agreements are "green", no actions required

For "red" agreements further investigation on the affected serves is
required.

If "grey" agreements exist, this means that part of the topology cannot
be reached by the server where the monitoring request was issued. Send
the monitoring request with the same monitoringID to a server in the
"grey" area, and merge the monitoring information for a more complete
picture of replication state.

Implementation
--------------

Any implementation details you would like to spell out. Describe any
technical details here. Make sure you cover

-  **Dependencies**: any new dependencies that FreeIPA project or it's
   part would gain and that needs to be packaged in distros?
-  **Backup and Restore**: any new file to back up or change required in
   `Backup and Restore <V3/Backup_and_Restore>`__?

If this section is not trivial, move it to ``/Implementation`` sub page
and only include link.



Feature Management
------------------

UI

TODO

CLI

TODO

Configuration
----------------------------------------------------------------------------------------------

TODO

Upgrade
-------

TODO



How to Test
-----------

TODO



Test Plan
---------

TODO