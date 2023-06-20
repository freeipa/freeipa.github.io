IPAv2_alpha3
============

\__NOTOC_\_



Significant Changes since alpha 2
---------------------------------

-  Fix memory crash-bug in ipa-join
-  Add pwpolicy2 plugin, future replacement for pwpolicy
-  CSRs that don't include NEW in the header/footer blocks should work
   now
-  Lots of clean-ups in ipa-client-install
-  ipa-server-install and ipa-client-install now use backed-up files and
   state in /var/lib/ipa and /var/lib/ipa-client to determine whether
   they are already configured or not
-  Fixed bug in some DNS entries that were missing a trailing dot (.)
-  Fix bug in password plugin that prevented ldappasswd from working on
   non-kerberized users
-  In the client installer we will have certmonger issue certificate
   requests using the subject base that IPA is configured with. This
   will make certmonger play nicer with the selfsign CA.
-  IPA works when using external CA option again
-  Stop using LDAPv2-style escaped DNs where possible
-  Updated MITM integration with dogtag
-  Anonymous VLV is enabled when the compat plugin is enabled making
   Solaris 10 clients happy
-  Add a CRL URI to certificates that are issued by dogtag
-  Added an ipa man page
-  XML-RPC signature change. This will affect older alphas command-line
   utilities trying to talk to a new server
-  Fixed bug in host plugin where deleting a non-qualified hostname
   would delete just the host, not the service entries associated with
   that host.
-  ipa-replica-manage now uses kerberos to delete and list servers. Add
   still requires the DM password
-  Provide feedback if a -mod command is executed and no changes are
   performed
-  Don't log passwords into files during installation
-  Add option to enable pam_mkhomedirs in the IPA client installer
-  Fixed a number of bugs in the pwpolicy plugin
-  More detailed error messages when entries are not found
-  Viewing binary in the UI shouldn't cause it to fail
-  dogtag is a required component and now configured by default
-  Run the XML-RPC server under mod_wsgi instead of mod_python
-  Fix the --all and --raw options
-  8 translations:

   -  Bengali India
   -  Indonesian
   -  Ukrainian
   -  Kannada
   -  Polish
   -  Russian
   -  Spanish
   -  Chinese Simplified

-  Other minor polish and bug fixes



Package Requirements
--------------------

The dogtag CA is now an rpm dependency in this alpha. It will be
configured by default when installing IPA. To not configure dogtag pass
the ``--selfsign`` flag to ``ipa-server-install``.



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

In alpha 3 we have added a man page for the ipa command.

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



Work Continues
--------------

A high-level view of things to be completed before the general release
of IPA v2 includes:

-  Private groups
-  Future version smooth migration
-  UI
-  Documentation