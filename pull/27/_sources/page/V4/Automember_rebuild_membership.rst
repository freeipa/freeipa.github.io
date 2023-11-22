Automember_rebuild_membership
=============================

Overview
--------

Integrate the `rebuild membership
feature <https://fedorahosted.org/389/ticket/20>`__ of the automember DS
plugin into IPA CLI and Web UI.



Use Cases
---------

Make sure that user or host membership can be easily rebuilt, based on
new or updated automember rules.

Design
------

Add a new CLI command ``ipa automember-rebuild``, to rebuild auto
membership for specified entries. Hook the command into web UI
appropriately.

Note that automember rebuild command only **adds** new membership
relationship, it does not remove those that do not match automember
rules.

Implementation
--------------

The newly added command will invoke the
``automember rebuild membership`` task, by creating an LDAP entry under
``cn=automember rebuild membership,cn=tasks,cn=config``. The details of
automember plugin tasks implementation and usage are described
`here <https://fedorahosted.org/389/ticket/20#comment:10>`__.



Feature Management
------------------

UI

Hook the new command into the web UI: to the user and host pages. Add a
new action 'Rebuild auto membership', and place it appropriately on
these pages.

-  On the user search facet, add the new action to the action list. This
   makes it possible to rebuild auto membership for multiple users.
   Executing the action without selecting any users will run the task
   for all the users (using --type=group).
-  On the user details facet, add the new action to the action list.
   This makes it possible to rebuild automembership for a single user.
-  On the host search facet, add the new action to the action list. This
   makes it possible to rebuild auto membership for multiple hosts.

Executing the action without selecting any hosts will run the task for
all the hosts (using --type=hostgroup).

-  On the host details facet, add the new action to the action list.
   This makes it possible to rebuild automembership for a single host.

CLI

``ipa automember-rebuild`` can be used to rebuild membership for all
objects of certain type:

| ``   $ ipa automember-rebuild --type=group``
| ``   $ ipa automember-rebuild --type=hostgroup``

It can also be used to rebuild membership for the specified entries:

| ``   $ ipa automember-rebuild --hosts=HOSTNAME1 --hosts=HOSTNAME2``
| ``   $ ipa automember-rebuild --users=LOGIN1 --users=LOGIN2``



Updates and Upgrades
--------------------

A new ACI, a permission and a privilege will be created in order to
support creation of automember tasks:

-  privilege: ``Automember Task Administrator``, which will contain two
   permissions listed below
-  permission: ``Add Automember Rebuild Membership Task``
-  underlying ACI for the permission listed above



How to Test
-----------

Add a hostgroup:

``   $ ipa hostgroup-add --desc="Web Servers" webservers``

Add a host:

``   $ ipa host-add web1.example.com --force``

Add an automember rule:

| ``   $ ipa automember-add --type=hostgroup webservers``
| ``   $ ipa automember-add-condition --key=fqdn --type=hostgroup --inclusive-regex=^web[1-9]+\.example\.com webservers``

The automember feature is now working for newly added entries. If we add
a new host, it will be automatically placed in the appropriate
hostgroup:

| ``   $ ipa host-add web2.example.com --force``
| ``   $ ipa hostgroup-show webservers``
| ``     Host-group: webservers``
| ``     Description: Web Servers``
| ``     Member hosts: web2.example.com``

However, the old host entry for ``web1.example.com`` is still not a
member or the ``webservers`` hostgroup. By introducting the new
``automember-rebuild`` commands, we make it possible:

``   $ ipa automember-rebuild --type=hostgroup``

or

``   $ ipa automember-rebuild --hosts=web1.example.com``

will run the ``automember rebuild membership`` task and consequently
place the host in the appropriate hostgroup:

| ``   $ ipa hostgroup-show webservers``
| ``     Host-group: webservers``
| ``     Description: Web Servers``
| ``     Member hosts: web2.example.com, web1.example.com``

The same mechanism applies for users and groups.



Test Plan
---------

See
`test_automember_plugin.py <https://git.fedorahosted.org/cgit/freeipa.git/tree/ipatests/test_xmlrpc/test_automember_plugin.py?h=ipa-4-1>`__
for the list of test cases.



RFE Author
----------

`akrivoka <User:Akrivoka>`__ (`talk <User_talk:Akrivoka>`__)

`Category:FreeIPA V4 Test Plan <Category:FreeIPA_V4_Test_Plan>`__
`Category:FreeIPA Test Plan <Category:FreeIPA_Test_Plan>`__