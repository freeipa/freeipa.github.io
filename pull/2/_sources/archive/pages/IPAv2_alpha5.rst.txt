\__NOTOC_\_

.. _significant_changes_since_alpha_4:

Significant Changes since alpha 4
---------------------------------

-  Dropped our PKCS#10 parser to use the one provided by python-nss
-  Started enforcing that hosts must be resolvable before adding them
   (use --force if you really want to add them).
-  Provide a reason when adding members to a group fails.
-  Allow de-coupling of user private groups (group-detach).
-  Support for ipa tool failover.
-  Hosts are allowed to retrieve keytabs for their services.
-  More configurable logging, see
   http://freeipa.org/page/IPAv2_config_files
-  Add support for ldap:///self aci rules
-  Use global time and size limit values when searching.
-  Don't include passwords in log files.
-  Work on F-14
-  Make ipactl a lot smarter and add a man page for it.
-  Have certmonger track the IPA service certificates.
-  Initial support for SUDO. You can create the objects but the
   client-side is not done yet.
-  The delete commands now take multiple arguments: ipa user-del user1
   user2 user3 ... usern
-  Remove reliance on 'admin' as a special user. All access control now
   granted via groups.
-  Groups are now created as POSIX by default.
-  Add options to control NTLM hashes. By default LM hash is disabled.
-  Remove the correct password from the history. We were mistakenly
   removing the latest password from the history instead of the oldest.
-  Rename user-lock and user-unlock to user-enable user-disable.
-  The ipa command should return non-zero when something fails.
-  Add gettext support for the C utilities.
-  Add capability to import automount files.
-  Add basic support for user and group renames (more work is needed).
   For now use ipa user-mod --setattr uid=newuser olduser
-  Add flag to group-find to only search on private groups.
-  Set default python encoding to utf-8. This should resolve a number of
   i18n problems.
-  Show indirect members (of groups, hostgroups, netgroups, etc).
-  Remove group nesting from the HBAC service groups.
-  Implement nested netgroups.
-  Add basic support for kerberos lockout policy. You can control how
   many failed attempts are allowed before lockout. What is missing is a
   way to unlock a user. This depends on fixes from MIT Kerberos 1.9.
-  Correct handling of userCategory and hostCategory in netgroups.
-  Updated a lot of man pages.

.. _package_requirements:

Package Requirements
--------------------

The dogtag CA is now an rpm dependency in this alpha. It will be
configured by default when installing IPA. To not configure dogtag pass
the ``--selfsign`` flag to ``ipa-server-install``.

.. _command_line:

Command Line
------------

One of the most visible changes is in the IPA command-line programs.
Rather than having a series of discrete commands for every function as
we had in v1 there is now a single command that takes as the first
argument the action you want to take.

For example, adding a user in v1:

``% ipa-adduser --first=Jim --last=Example jexample``

In v2:

``% ipa user-add --first=Jim --last=Example jexample``

For a list of topics: ``ipa help topics``

For a full list of available commands: ``ipa help commands``

We have tried to retain at least parity with the v1 commands, let us
know how we've done (and where we can improve).

In alpha 3 we added a man page for the ipa command.

Clients
-------

The client installer (``ipa-client-install``) by default configures the
client machine to use SSSD. If you prefer using nss_ldap/pam instead
pass the client installer the --no-sssd option.

Replication
-----------

As in v1 all replicas should be created from the initial master. In
theory it should be possible to create a clone from a clone when using
dogtag as the CA but some problems have been reported when doing this.
So it is recommended that all replicas be created from the initial IPA
server using ipa-replica-prepare as was required in IPA v1.

If you re-install IPA on a replica machine be sure to delete the host
entry from the primary master (ipa host-del replica.example.com) before
performing the re-install. You will need to remove the old LDAP
replication agreement as well:
``ipa-replica-manage del --force replica.example.com``. If you don't
then the re-installation will fail with errors about certificate already
being present in some entries.

CA
--

It is currently not possible to install the CA post IPA server
installation.

If installing v2 on a machine that was previously installed with IPA v1
and you are not configuring a dogtag CA your serial number file,
``/var/lib/ipa/ca_serialno``, will be upgrade to the v2 format.

Migration
---------

It is not possible to upgrade an IPA v1 server to an IPA v2 server, too
much has changed internally. We have provided a migration mechanism to
migrate users and groups from an LDAP server (so this will aid in
migration from other identity products as well) to IPA v2.

To perform a migration, install IPA v2 on a clean machine. Once it is
installed and working you can migrate users and groups using the
migrate-ds command (ipa migrate-ds --help to see options).

Sample usage from a default 389-ds installation:

``% ipa migrate-ds``\ ```ldap://ldap.example.com`` <ldap://ldap.example.com>`__

Migration from an IPA v1 server:

``% ipa migrate-ds --user-container=cn=users,cn=accounts --group-container=cn=groups,cn=accounts``\ ```ldap://ipa.example.com`` <ldap://ipa.example.com>`__

The output will be the list of users and groups that migrated and those
that did not.

It is not currently possible to migrate ONLY users or ONLY groups. It
must migrate both. If either no users or no groups is present on the
server being migrated from an Entry Not Found will be displayed and the
migration will stop before migrating anything.

Documentation
-------------

We are still in the process of writing documentation for the IPA v2. The
current progress can be followed at
http://freeipa.org/page/IPAv2_development_status#Documentation

Feedback
--------

The UI and command-line commands use the same underlying plug-ins for
functionality. The UI pages are generated on-the-fly using some
additional meta-data. This should improve our code maintainability but
we also want to create a system that works for you. Any feedback on how
this helps/hurts getting your job done would be greatly appreciated.

Also keep in mind that the UI isn't quite done yet, so be gentle :-)

Bugs
----

If you find any problems then please file a bug against the freeIPA
product at https://bugzilla.redhat.com/

.. _work_continues:

Work Continues
--------------

A high-level view of things to be completed before the general release
of IPA v2 includes:

-  Future version smooth migration
-  UI
-  Documentation
