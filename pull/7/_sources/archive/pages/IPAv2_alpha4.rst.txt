\__NOTOC_\_

.. _significant_changes_since_alpha_3:

Significant Changes since alpha 3
---------------------------------

-  Moved our dogtag SELinux to be installed with the rpm instead of
   during configuration.
-  Fedora 13 moved to gpg2 and dropped gpg. Fix our invocation so we
   work with either (this was preventing replica installations).
-  Query remote server during replica installation to see if the replica
   already exists. This prevents lots of really strange errors during
   replica installation.
-  Fixed SSL error in client enrollment.
-  Changed the way services are handled in HBAC. There is now a separate
   service and servicegroup object that you associate with HBAC rules.
   sssd is already using this new mechanism.
-  First pass at per-command documentation. It still needs a lot of
   work.
-  Fix aci-mod command. It wasn't really working well in almost all
   cases.
-  Add replication version checking. This is one step in better control
   during updates.
-  Don't try to convert a host's password into a keytab with bulk
   enrollment (this was causing krbPasswordExpiration to be set).
-  Add support for User-Private Groups.
-  Worked on error handling in mod_wsgi. Now hopefully a shorter and
   less scary backtrace will be thrown when things go bump in the night.
-  Add new api to disable service and host principals.
-  Significant cleanup of crypto code. Using python-nss for a lot more
   (and more to come).
-  Fixed some errirs in and made ipa-compat-manage and ipa-nis-manage
   more bullet-proof.
-  Fixed netgroups plugin, it was generating the wrong attributes.
-  Other minor polish and bug fixes.

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
and you are not configuring a dogtag CA you will need to remove the file
``/var/lib/ipa/ca_serialno`` This file contains the last serial number
that the IPA self-signed CA has issued. The file format has changed and
is not backward compatible. You'll see a message like this if you forget
to do this:

| ``Unexpected error - see ipaserver-install.log for details:``
| `` File contains no section headers.``
| ``file: /var/lib/ipa/ca_serialno, line: 1``
| ``'1003'``

To correct this remove the file, run ``ipa-server-install --uninstall``
then run ``ipa-server-install`` again to configure the services.

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
