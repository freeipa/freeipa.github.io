Overview
--------

**authselect** is a new tool to configure authentication sources on a
system. It is going to replace **authconfig** in the future (see the
page for
`Authselect <https://fedoraproject.org/wiki/Changes/Authselect>`__ on
Fedora wiki). Since IPA is dependent on authconfig, it is required to
extend its functionality so authselect can be used when it is available.
Authselect will be a default tool in Fedora 28, authconfig command will
still be available (provided by the authselect-compat package) but its
functionality will be limited.

The major difference between authconfig and authselect is the use of a
configuration **profile**: authselect provides **pre-defined profiles**
that can be tuned by enabling specific **features**. The available
profiles can be found using ``authselect list`` and their features with
``authselect show PROFILE-ID``. For instance, the sssd profile accepts
the features with-mkhomedir or with-smartcard, and the winbind profile
accepts the feature with-mkhomedir but not with-smartcard.

.. _current_implementation:

Current implementation
----------------------

This section lists the components of FreeIPA that are currently relying
on authconfig.

.. _package_dependency:

Package dependency
~~~~~~~~~~~~~~~~~~

FreeIPA currently has a dependency on authconfig package (freeipa-client
package).

.. _client_installation:

Client installation
~~~~~~~~~~~~~~~~~~~

Currently authconfig is triggered during client install to:

-  configure NIS. the commands below is used

`` authconfig --nisdomain=``

-  with sssd (default).

To enable SSSD for user information and authentication:

`` authconfig --enablesssd --enablesssdauth``

To enable kerberos authentication:

`` authconfig --enablekrb5 --nostart``

-  no_sssd (deprecated).

To enable LDAP for user information:

`` authconfig --enableldap --enableforcelegacy``

The following ipa-client-install options have an impact on
authentication configuration:

-  --nisdomain=NISDOMAIN: Set the NIS domain name as specified. By
   default, this is set to the IPA domain name.
-  --no-nisdomain: Do not configure NIS domain name.
-  --no-sssd: Do not configure the client to use SSSD for
   authentication, use nss_ldap instead.
-  --noac: Do not use Authconfig to modify the nsswitch.conf and PAM
   configuration.

Note that the ipa client code is using the **tasks** package to call
distribution-specific code: only the fedora-like distributions use
authconfig, other distributions must implement their own routines to
configure the authentication mechanisms.

Backup/restore
~~~~~~~~~~~~~~

The commands ipa-backup and ipa-restore both need to save/restore the
current configuration. ipa-backup is calling:

`` authconfig --savebackup ``

and saves the content of the backup directory in the backup file.
ipa-restore is calling

`` authconfig --restorebackup ``

to restore the saved configuration.

.. _ipa_advise_plugins:

ipa-advise plugins
~~~~~~~~~~~~~~~~~~

ipa-advise command provides configuration advices for various use cases.
For instance, ipa-advise config-server-for-smart-card-auth produces a
shell script that can be executed in order to configure the server to
allow Smart Card authentication. Some of the plugins produce shell
scripts that are calling authconfig:

-  ipa-advise config-client-for-smart-card-auth: Instructions for
   enabling Smart Card authentication on a single FreeIPA client.
-  ipa-advise config-fedora-authconfig: Authconfig instructions for
   configuring Fedora 18/19 client with IPA server without use of SSSD.
-  ipa-advise config-redhat-sssd-before-1-9: Instructions for
   configuring a system with an old version of SSSD (1.5-1.8) as a
   FreeIPA client.
-  ipa-advise config-redhat-nss-pam-ldapd: Instructions for configuring
   a system with nss-pam-ldapd as a FreeIPA client.
-  ipa-advise config-redhat-nss-ldap: Instructions for configuring a
   system with nss-ldap as a FreeIPA client.

Design
------

.. _general_decisions:

General decisions
~~~~~~~~~~~~~~~~~

-  As authselect may not be available on all distributions, we need to
   keep the same strategy as implemented by the current code: rely on
   the tasks package to have distribution-specific code. FreeIPA team
   will provide code for fedora-like distributions, based on authselect
   and the other distributions should not be disturbed by the changes.
-  Authselect-compat package provides an equivalent to authconfig, with
   a compatibility layer for some options and deprecating other options.
   Because of these deprecated options, it does not seem safe enough to
   keep on using authconfig tool and the choice is made to completely
   switch to using authselect. This implies that the dependency on
   authconfig will be completely removed and replaced with a dependency
   on authselect. Note that we will not have any dependency on
   authselect-compat, meaning that FreeIPA code should not rely any more
   on authconfig command.
-  It seems obvious to state that FreeIPA will use the sssd profile but
   this has major consequences. Previously, it was possible to install a
   client with --no-sssd or --noac options, but it will not be possible
   any more for new client installations on fedora-based distributions.

.. _new_client_installation:

New client installation
~~~~~~~~~~~~~~~~~~~~~~~

Currently, the algorithm for PAM stack configuration is the following:

| ````
| ``authconfig --nisdomain=<domain>``
| ``if (sssd) then ``
| ``   authconfig --enablesssd --enablesssdauth``
| ``else ``
| ``   authconfig --enableldap --enableforcelegacy --enablekrb5 --nostart``
| ``done``
| ``if (mkhomedir) then``
| ``   authconfig --enablemkhomedir``
| ``done``

.. _no_sssd_and___noac_options:

--no-sssd and --noac options
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

With the migration to authselect and the choice of using sssd profile,
we will now refuse the --no-sssd and -noac options for fedora-based
distributions. This can be achieved by adding a tasks method (i.e. with
a distribution-specific implementation) is_nosssd_supported(), and a
check in the client installer that refuses the option in case
is_nosssd_supported returns False.

.. _pam_stack_configuration:

PAM stack configuration
^^^^^^^^^^^^^^^^^^^^^^^

Calls to the authconfig tool are completely replaced by calls to
authselect, picking the sssd profile. PAM configuration steps are moved
into a separate class following bridge oop pattern. All related code is
under ipaplatform/redhat/authconfig.py, ensuring that only fedora-based
distributions are impacted by the modifications.

.. _mkhomedir_option:

--mkhomedir option
^^^^^^^^^^^^^^^^^^

The homedir creation can also be enabled with authselect with:
``authselect select sssd with-mkhomedir``.

.. _nis_domain_configuration:

NIS domain configuration
^^^^^^^^^^^^^^^^^^^^^^^^

Authconfig is currently used in the client installer to configure the
NIS domain. It is also possible to configure the NIS domain without a
call to authconfig tool, by `direct modification of a config
file <https://access.redhat.com/articles/2278>`__. This is the chosen
approach: append (or replace) the ``NISDOMAIN=value`` line in the file
/etc/sysconfig/network.

.. _client_uninstallation:

Client uninstallation
~~~~~~~~~~~~~~~~~~~~~

The client uninstallation needs to revert the system to the same state
as before client install. In order to do this, the client installation
will store the profile used pre-installation in the system store
(/var/lib/ipa-client/sysrestore/sysrestore.state) with the following
format:

| `` [authselect]``
| `` profile=``
| `` features_list=``

Profile and features_list will be used to revert to the previous state
during uninstallation.

Note: When the client was installed with the authconfig tool, the system
store does not contain this information. In this case, the uninstaller
will simply warn that it is not able to revert to the exact state before
installation and will apply the default authselect profile, namely the
sssd profile without any feature.

.. _new_server_installation:

New server installation
~~~~~~~~~~~~~~~~~~~~~~~

The server-specific install code is not impacted by this migration (only
the client-part of the installation is).



Backup and restore
~~~~~~~~~~~~~~~~~~

Backup
^^^^^^

The authselect tool offers the "current" command to retrieve the current
configuration (profile and enabled features). For instance:

| `` $ authselect current --raw``
| `` sssd with-mkhomedir``

The ipa-backup command needs to use this command to save the current
configuration inside a new file in the backup directory.

Restore
^^^^^^^

Note: only full restore is impacted by this feature. Data-only restore
does not touch the authentication configuration.

The ipa-restore command needs to read the saved configuration from the
backup directory and re-apply the same configuration using

`` $ authselect select ``\ `` ``\ `` --force``

Note: if the backup was done on a server \*before\* the migration to
authselect, the ipa-restore will detect that restore is trying to
restore data from a different release and prompt for user confirmation
with a warning. Unattended restore will fail.

.. _ipa_advise_plugins_1:

ipa-advise plugins
~~~~~~~~~~~~~~~~~~

.. _config_client_for_smart_card_auth_plugin:

config-client-for-smart-card-auth plugin
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This plugin configures a FreeIPA client for smart card authentication.
Instead of calling

`` authconfig --enablesssd --enablesssdauth --enablesmartcard ' '--smartcardmodule=sssd --smartcardaction=1 --updateall``

the plugin must use

`` authselect enable-feature with-smartcard``

.. _config_fedora_authconfig_plugin:

config-fedora-authconfig plugin
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This plugin configures Fedora 18/19 client without the use of sssd.
These versions are not suppported any more and the plugin can be
dropped.

.. _other_plugins:

other plugins
^^^^^^^^^^^^^

The other plugins (config-redhat-sssd-before-1-9,
config-redhat-nss-pam-ldapd and config-redhat-nss-ldap) are related to
RHEL 5, where authselect will not be available. The scripts produced by
ipa-advise can be generated on a recent FreeIPA server and run on a
RHEL5 system, meaning that we can keep them.

Upgrade
~~~~~~~

.. _migration_for_older_clients:

Migration for older clients
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Client upgrade will not modify the configuration since the PAM stack
configuration is already in place.

.. _migration_for_older_servers:

Migration for older servers
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The server has to be migrated to authselect to make sure that backup and
restore code works properly for new servers and also for older servers.
If no migration was implemented, this would imply that backup/restore
code must be able to handle 2 different types of configurations (with
authconfig or with authselect), leading to a maintenance nightmare.
Because of this, the choice is made to migrate the configuration to an
authselect profile during the upgrade.

The ipa-server-upgrade tool will perform the migration to an authselect
profile. It needs to take care of the following points:

-  check if the server was initially installed with the flag --mkhomedir
   (by reading the content of the system store). In this case, the sssd
   profile with enable-mkhomedir option must be selected. Otherwise use
   the sssd profile without the option.
-  update the configuration backed up in the system store
   (/var/lib/ipa-client/sysrestore/sysrestore.state). The system store
   may contain

| `` [authconfig]``
| `` mkhomedir=...``
| `` ldap=...``
| `` krb5=...``
| `` sssd=...``
| `` sssdauth=...``

and this would have to be replaced with

| `` [authselect]``
| `` profile=sssd``
| `` mkhomedir=...``



Use cases
---------

As this migration is mainly internal, it will not modify the interfaces
as seen by a user or system administrator, except for the use of
--no-sssd or --noac options in ipa-client-install (which will now be
refused on fedora-based distributions).

A careful user may notice the presence of a new directory
/etc/authselect created during the authselect package installation,
containing /etc/authselect/authselect.conf file storing the current
profile and features:

| `` $ cat /etc/authselect/authselect.conf ``
| `` sssd``

Testing
-------

The tests need to focus on 2 main parts, new installations and upgrades.
They can be run on fedora-based distributions.

.. _upgrade_1:

Upgrade
~~~~~~~

-  upgrade must keep the mkhomedir flag (if the server was installed
   with --mkhomedir, then the authselect config obtained after upgrade
   must also have this option)
-  backup and restore must still be working after the upgrade
-  uninstall must still be working after the upgrade (potentially with a
   warning if the client was installed with authconfig).

.. _new_installations:

New installations
~~~~~~~~~~~~~~~~~

-  new client installation must install the sssd profile, with or
   without the with-mkhomedir feature (depending on the presence of
   --mkhomedir flag)
-  ipa-client-install must refuse the --no-sssd and --noac options with
   a meaningful error message
-  client install / uninstall must revert to the previous authselect
   profile

-  new server installation must install the sssd profile, with or
   without the with-mkhomedir feature (depending on the presence of
   --mkhomedir flag)
-  backup/restore must work with a new server installation
