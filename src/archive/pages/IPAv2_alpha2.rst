IPAv2_alpha2
============

\__NOTOC_\_



Significant Changes since last alpha
------------------------------------

-  There is now a basic UI available. It is still rather rough around
   the edges. A menu is available by clicking on the FreeIPA in the
   upper-left of the screen.
-  DNS server support can be added post-installation using the
   ipa-install-dns script.
-  The dogtag packages are available in the Fedora repos making it very
   easy to install IPA backed with a real CA.
-  SSSD integration. sssd is a client-side replacement for nss_ldap and
   brings a ton of features. See https://fedorahosted.org/sssd/ for more
   details
-  Certmonger integration. certmonger is a client-side program to
   monitor SSL server certificates and automatically renew them. See
   https://fedorahosted.org/certmonger/ for more details
-  ipa-client-install now also joins the host to the IPA realm. This
   does a number of things:

   -  Retrieves a host service principal and installs it into
      /etc/krb5.keytab. These credentials are used by sssd.
   -  Retrieves a SSL server certificate for the machine and stores it
      in /etc/pki/nssdb. This also uses the host principal for
      authentication/authorization.
   -  Joining requires either a kerberos principal with the right
      privileges or a password:

      -  You can pre-create the host and set a one-time password to be
         used for bulk enrollment, say in cobbler with: ipa host-add
         --password=secret123 client.example.com
      -  You can pre-create the host which will be joined by an admin
         user on the client machine: ipa host-add client.example.com
      -  If you join the host using an admin user it will be
         automatically added if not already present



Package Requirements
--------------------

The dogtag CA and bind DNS servers are optional components in this
alpha. In future versions the CA will be added as an rpm dependency and
be enabled by default in the installer. To install dogtag run
``yum install pki-ca``. It should pull in all dependencies. Pass the
--ca flag to the IPA installer to enable CA creation at install time.



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
installation. It must be done when the server is initially installed
using the --ca option.

If for some reason the IPA server installation fails while installing
the CA, pass the --ca option to the uninstaller to ensure everything is
cleaned up: ``ipa-server-install --uninstall --ca.``

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