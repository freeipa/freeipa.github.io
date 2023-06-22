\__NOTOC_\_

.. _main_highlights_of_the_beta_1:

Main Highlights of the Beta 1
-----------------------------

-  This beta is the first attempt to show all planned capabilities of
   the upcoming release.
-  For the first time the new UI is mostly operational and can be used
   to perform management of the system.
-  Some areas are still very rough and we will appreciate your help with
   those.

.. _focus_of_the_beta_testing:

Focus of the Beta Testing
-------------------------

-  Please take a moment and look at the new Web UI. Any feedback about
   the general approaches, work flows, and usability is appreciated. It
   is still very rough but one can hopefully get a good understanding of
   how we plan the final UI to function and look like.
-  Replication management was significantly improved. Testing of multi
   replica configurations should be easier.
-  We are looking for a feedback about the DNS integration and
   networking issues you find in your environment configuring and using
   IPA with the embedded DNS enabled.

.. _significant_changes_since_alpha_5:

Significant Changes Since Alpha 5
---------------------------------

-  FreeIPA has changed its license to GPLv3+
-  Having IPA manage the reverse zone is optional.
-  The access control subsystem was re-written to be more
   understandable. For details see `Permissions <V2/Permissions>`__ page
-  Support for SUDO rules
-  There is now a distinction between replicas and their replication
   agreements in the ipa-replica-manage command. It is now much easier
   to manage the replication topology.
-  Renaming entries is easier with the --rename option of the mod
   commands.
-  Fix special character handling in passwords, ensure that passwords
   are not logged.
-  Certificates can be saved as PEM files in service-show and host-show
   commands.
-  All IPA services are now started/stopped using the ipactl command.
   This gives us better control over the start/stop order during
   reboot/shutdown.
-  Set up ntpd first so the time is sane.
-  Better multi-valued value handle with --setattr and --addattr.
-  Add support for both RFC2307 and RFC2307bis to migration.
-  UID ranges were reduced by default from 1M to 200k.
-  Add ability to add/remove DNS records when adding/removing a host
   entry.
-  A number of i18n issues have been addressed.
-  Updated a lot of man pages.

.. _what_is_not_complete:

What is not Complete
--------------------

-  We are still using older version of the Dogtag. New version of the
   Dogtag Certificate System will be based on tomcat6 and is
   forthcoming.
-  We plan to take advantage of Kerberos 1.9 that was released today but
   we have not finished the integration effort yet.

.. _known_issues:

Known Issues
------------

-  IPV6 works in the installer but not the server itself
-  Make sure you machine can properly resolve its name before installing
   the server. Edit /etc/hosts to remove host name from the localhost
   and localhost6 lines if needed.
-  The UI is still rough in places
   Use the following query: `Open UI
   tickets <https://fedorahosted.org/freeipa/report/12>`__ to see the
   tickets currently open against UI.
-  Dogtag does not work out-of-the-box on Fedora 14. To fix it for for
   the time being run:

``      # ln -s /usr/share/java/xalan-j2-serializer.jar /usr/share/tomcat5/common/lib/xalan-j2-serializer.jar``

-  Instead of dogtag on F14 you can also try the self-signed CA which is
   similar to the CA that was provided in IPA v1. This was designed for
   testing and development and not recommended for deployment.
-  Make sure you enable updates-testing repository on your fedora
   machine.

.. _package_requirements:

Package Requirements
--------------------

The dogtag CA is an rpm dependency in. It will be configured by default
when installing IPA. To not configure dogtag pass the ``--selfsign``
flag to ``ipa-server-install``.

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
``/var/lib/ipa/ca_serialno``, will be upgraded to the v2 format.

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

If you find any problems then please file a ticket in the fre iPA track
https://fedorahosted.org/freeipa/ or open a bug against the freeIPA
product at the https://bugzilla.redhat.com/

.. _work_continues:

Work Continues
--------------

Our primary focus in the upcoming months will be bringing project to a
production quality.
