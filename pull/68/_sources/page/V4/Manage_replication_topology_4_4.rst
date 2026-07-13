Manage_replication_topology_4_4
===============================

Overview
--------

Centralized topology management was one of the main features of `4.3.0
release <Releases/4.3.0>`__. Management of topology is working but user
experience is not at a desired level. This document covers missing parts
of topology management to be implemented in 4.4 release.



Use Cases
---------

Same as in `original design <V4/Manage_replication_topology>`__.

Design
------

Extends `original design <V4/Manage_replication_topology>`__. Main
topics are:

-  improvement of Web UI topology graph to be usable with bigger number
   of IPA servers
-  continue with deprecating of ``ipa-replica-manage`` and
   ``ipa-csreplica-manage`` for day to day topology management. The
   direction was discussed in freeipa-devel thread `Fate of
   ipa-replica-manage and ipa-csreplica-manage
   tools <https://www.redhat.com/archives/freeipa-devel/2015-October/msg00454.html>`__.



ipa(cs)replica manange changes
----------------------------------------------------------------------------------------------

The long term plan is to completely deprecate ``ipa-csreplica-manage``
tool. ``ipa-replica-manage`` will be used only for management of winsync
agreements, assuming domain level 1.



ipa-csreplica-manage
----------------------------------------------------------------------------------------------

The only remaining goal before depracation is to transform
``set-renewal-master`` into an API command. It will be done according to
`server roles design page <V4/Server_Roles>`__. `ticket
#5689 <https://fedorahosted.org/freeipa/ticket/5689>`__



ipa-replica-manage
----------------------------------------------------------------------------------------------

``del``: FreeIPA 4.3 changes allows to transform ``del`` command into an
API method: ``server_del``.
`#5588 <https://fedorahosted.org/freeipa/ticket/5588>`__.
``ipa-replica-manage`` should call this API method on domain level 1 for
backwards compatibility. This new API command should be then used in
``ipa-server-install --uninstall``, more details below.

``clean-dangling-ruvs``: description at `ticket
#5411 <https://fedorahosted.org/freeipa/ticket/5411#comment:7>`__

``list-ruv/clean-ruv``: extend the commands' default behavior so that
they search both ca and domain trees to list/clean from them. `ticket
#4987 <https://fedorahosted.org/freeipa/ticket/4987>`__. It is to
support the new ``clean-dangling-ruvs`` subcommand.

``abort-clean-ruv``: Should work with both suffixes. It should
find/check existence of the task first and then abort it. It should make
use of the proposed \`replica-force-cleaning: yes\` `ticket
#5396 <https://fedorahosted.org/freeipa/ticket/5396>`__ so that it won't
hang if a replica is shut down.

Non goals in FreeIPA 4.4 release:

-  move DNA ranges management to API
-  modify re-initialize and force-sync commands of both tools.

``server_del``
----------------------------------------------------------------------------------------------

New API method, will do the same as current ``ipa-replica-manage del``
on domain level 1. It will work only with domain level 1 because domain
level 0 requires also to connect to other servers.

Arguments and options:

-  ``server`` - argument, server to be deleted
-  ``--force`` - should be allowed to call the method even if it was
   already removed. Should perform cleaning tasks. If server was not
   deleted, ``--force`` doesn't have any additional meaning. This will
   be a simplification of ``ipa-replica-manage del --force --clean`` The
   difference is that ``--force`` won't skip topology disconnection
   check and last services check.
-  ``--ignore-topology-disconnect`` Will skip topology disconnection
   check.
-  ``--ignore-last-services`` Will skip last services check.

uninstallation
----------------------------------------------------------------------------------------------

In the past uninstallation was 2-3 step process:

#. ``ipa-csreplica-manage del`` (applicable only if the deleted master
   has CA)
#. ``ipa-replica-manage del``
#. ``ipa-server-install --uninstall``

Centralized topology management and changes done in
``ipa-replica-manage del`` in FreeIPA 4.3 allows to skip step 1. From
personal experience and post on FreeIPA users list step 1 and 2 are
often forgotten. Fortunately centralized topology management allows to
move ``ipa-replica-manage del`` into an API command. It allows to use it
from ``ipa-server-install --uninstall``. I.e. the uninstallation process
can be simplified into a single command:
``ipa-server-install --uninstall``

Proposal: ``ipa-server-install --uninstall`` should connect to API of
different master with host credential and call ``server-del`` command.
It should be done after initial validation. `ticket
#5588 <https://fedorahosted.org/freeipa/ticket/5588>`__. The reason is
that replica cleanup method removes DNS records and server keytabs.
Doing it on the to-be uninstalled server would be racy - some of the
changes wouldn't be replicated.

Implementation
--------------

Move of ``ipa-replica-manage del`` to API command requires little
refactoring - ipa-replica-manage prints to output directly but IPA
method doesn't do it. Additional change is to adjust ACIs and cleanup
method so that ``ipa server-del`` can be called with host credentials.



Feature Management
------------------

UI



Topology graph
^^^^^^^^^^^^^^

`4.3 release <V4/Manage_replication_topology>`__ came with topology
graph component. It is hard to use with higher number of replicas.
Canvas has a static size. It doesn't work well on small screens and
doesn't take benefit of bigger screens. Will be fixed by resizing canvas
size to a size of its container
`#5647 <https://fedorahosted.org/freeipa/ticket/5647>`__. It will create
a little drawback - higher number of replicas might not fit into a
smaller canvas and also while the canvas is shrank, some of the nodes
will be hidden. Therefore panning and zooming of canvas is required
`#5502 <https://fedorahosted.org/freeipa/ticket/5502>`__.

D3 force layout used in the canvas positions the nodes using force
simulation. The layout is different on each refresh. It makes impossible
to create a mental picture of the topology because position of a node is
always different. With following graph features: node can be dragged by
mouse. Graph supports switching nodes between static and float position.
Node's position when layout is static is stored into ``localStorage``.
User can then create his own design which is persisted across browser
session(but still limited to the browser instance). It is actually a
desired state but a path to get there is cumbersome because user has to
double click on each individual node. The initial layout can't start
with static mode because UI doesn't know position of nodes so starting
with float mode is required. Issue will be solved by switching all nodes
to static position after initial layout is done. E.g. it can be about
5s(better value should be found during implementation, e.g. it can be
computed). UI should provide a way to re-init the layout: a
``reset layout`` button.

Topology segments(links between nodes) are created using adder dialog.
It is initiated by ``add`` button. This is solution is standard in Web
UI but in this case it doesn't work very well. User sees which nodes he
wants to connect but then he needs to remember the nodes names, find it
in the dialog, choose suffix and add. It is slow. UI should provide an
interactive way to create the segment. Proposal is to drag-drop mouse
from one node to another
`#5648 <https://fedorahosted.org/freeipa/ticket/5648>`__. This move will
identify the two nodes and open the dialog. Right now it conflicts with
dragging node on a canvas. There should be a switch between dragging
node and creation connection.

Nice to have: segment name field in segment adder dialog should have a
placeholder set to "autogenerated".

Summary:

-  resize canvas on window resize to fill its container
   `#5647 <https://fedorahosted.org/freeipa/ticket/5647>`__
-  implement pan&zoom of canvas
   `#5502 <https://fedorahosted.org/freeipa/ticket/5502>`__
-  switch position of nodes to "static" after initial layout of nodes
   `#5649 <https://fedorahosted.org/freeipa/ticket/5649>`__
-  implement creation of topology segment by dragging from left node to
   right node `#5548 <https://fedorahosted.org/freeipa/ticket/5548>`__
-  set placeholder in segment adder dialog
   `#5867 <https://fedorahosted.org/freeipa/ticket/5867>`__

Server roles won't be displayed in the topology graph in 4.4 release.



Server management
^^^^^^^^^^^^^^^^^

-  Leverage server roles design. Server details page should contain
   ``set-renewal-master`` action in action list.
   `#5689 <https://fedorahosted.org/freeipa/ticket/5689>`__
-  Add ``delete`` action to server details page. It will call
   ``server_del`` API method. There has to be a confirmation window with
   a red(i.e. destroy action) confirm button with label *Delete IPA
   server*. `#5588 <https://fedorahosted.org/freeipa/ticket/5588>`__

CLI

Overview of new or modified CLI commands.

+----------------------------------+----------------------------------+
| Command                          | Options                          |
+==================================+==================================+
| ipa server-del                   | --force --ignore-last-services   |
|                                  | --ignore-topology-disconnect     |
+----------------------------------+----------------------------------+
| ipa config-mod                   | --ca-re                          |
|                                  | newal-master=server1.example.com |
+----------------------------------+----------------------------------+
| ipa replica-manage               |                                  |
| clean-dangling-ruvs              |                                  |
+----------------------------------+----------------------------------+

Configuration
----------------------------------------------------------------------------------------------

Nothing new.



How to Test
-----------

Easy to follow instructions how to test the new feature. FreeIPA user
needs to be able to follow the steps and demonstrate the new features.

The chapter may be divided in sub-sections per `Use
Case <#Use_Cases>`__.



Test Plan
---------

Test scenarios that will be transformed to test cases for FreeIPA
`Continuous Integration <V3/Integration_testing>`__ during
implementation or review phase. This can be also link to `source in
cgit <https://git.fedorahosted.org/cgit/freeipa.git/>`__ with the test,
if appropriate.

`Manage replication topology V4.4 test
plan <V4/Manage_replication_topology_4_4/Test_Plan>`__