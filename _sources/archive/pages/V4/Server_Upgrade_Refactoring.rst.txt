Server_Upgrade_Refactoring
==========================

Overview
--------

FreeIPA server becomes full of new features and enhancements over the
time, which makes upgrade process hard to keep in shape.

Following improvements, listed in this design, should make server update
process more deterministic, robust and working well in various update
environments (%posttrans update, containers, etc.).



Use Cases
---------

-  Make update process more deterministic
-  Merge server update commands into the one command
   (``ipa-server-upgrade``)
-  Solve upgrades in containers
-  Prevent to run IPA service, if code version and configuration version
   does not match

   -  ipactl should execute ipa-server-upgrade if needed

-  Prevent user to use ``ipa-upgradeconfig`` and ``ipa-ldap-updater``

   -  remove ipa-upgradeconfig
   -  refactor ipa-ldap-updater to allow execute updates for specified
      files with HUGE warning (disallow to execute overall update)

Design
------



Make upgrade process more deterministic
----------------------------------------------------------------------------------------------

`Freeipa-devel list
discussion <http://www.redhat.com/archives/freeipa-devel/2014-December/msg00183.html>`__

Currently update process do several optimizations which may prevent some
errors during update (do update in proper relation parent to children,
respectively children to parent during removing). In other hand, the
parent to child relation, is not the only one restriction. We use
several plugins in 389 directory server, which require proper order of
updates in different sub trees.

It causes unexpected errors because the order of updates in files is not
the same as real update. Developers should be responsible for proper
order of updates, as is not possible to create effective updates
sorting, which will respect all 389 DS plugins requirements.

This effort consist of following steps:

-  Do not sort order of updates based on length of DN.
-  Do not use dictionary to store updates. Dictionary mixes the order of
   particular updates per file. Using lists achieve, the order of
   updates will be same as order in files.
-  Apply updates per file, in specified order from top to bottom.



Ordering LDAP updates
^^^^^^^^^^^^^^^^^^^^^

All LDAP update files are ordered/executed in alphabetical order. Update
files should keep the number prefix, but this is not required anymore by
updater.

| ``05-pre-update-plugins.update``
| ``10-config.update``
| ``20-syncrepl.update``
| ``...``
| ``90-last.update``
| ``ZZ.update  # this will work, but we should avoid of using number less update files``



*plugin* - the new update file directive
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Plugins are called from update files, using new directive **plugin:**

Plugins which edits ldap data directly, without using modlists, should
be in separate files.



Update plugins modifications
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Execution order of plugins is specified in .update files. There is no
need to keep PRE_UPDATE and POST_UPDATE plugin classes and FIRST,
MIDDLE, LAST order specifications.

Class **Updater** is used for all update plugins instead of *PreUpdate*
and *Postupdate* classes.

The **execute** method of the class, returns only 2 values
(*restart_ds*, *modlist*)

-  *restart_ds* - boolean value, DS server will be restarted after
   *modlist* application
-  *modlist* - list of required changes



Merge server update commands into the one command
----------------------------------------------------------------------------------------------

Related ticket:
`#4834 <https://fedorahosted.org/freeipa/ticket/4834>`__,
`#3351 <https://fedorahosted.org/freeipa/ticket/3351>`__

Currently two commands exist, which have to be exucuted in proper order
to upgrade IPA:

| ``ipa-ldap-updater --upgrade``
| ``ipa-upgradeconfig``

Using these commands in wrong order, or execute ipa-upgradecofig if
ipa-ldap-updater failed, can break the system.

To resolve issues mentioned above only one command should do upgrade:
``ipa-server-upgrade``.



ipa-server-upgrade characteristics
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  LDAPI only is used for upgrade (no binding using DM password)
-  check the version of configuration and code version, run only if
   installed version is newer than configuration version
   (*--skip-version-check* overrides this check, see steps).
-  upgrade steps:

#. stop IPA services
#. check build and installed platform, if platform mismatch abort
   upgrade
#. check build version and installed version

   #. newer build than data: continue
   #. same version of data and build: skip data upgrade
   #. older version of build: abort upgrade

#. prepare DS to upgrade (stop DS, save configuration, disabling
   listeners, start DS)
#. SCHEMA update
#. LDAP data update + update plugins (update of user data in LDAP, and
   cn=config LDAP configurations)
#. upgrade configuration (upgrade of services: httpd, named, etc...)
#. restore DS (stop DS, restoring configuration)
#. (re)start IPA services

-  update configuration version, if update was successful
-  only former **--upgrade**/offline method is supported



Prevent to run IPA if version mismatch
----------------------------------------------------------------------------------------------

Related ticket: `#3849 <https://fedorahosted.org/freeipa/ticket/3849>`__

``ipactl {start|restart}``

#. compare build platform and platform from the last
   upgrade/installation (based on *ipaplatform* file)

   #. if platform mismatch, raise error and prevent to start IPA
      services

#. compare version of LDAP data(+schema included) and build version
   (*VENDOR_VERSION* will be used)

   #. if LDAP data version **>** build version: raise error and prevent
      services to start (newer data than IPA build)
   #. if LDAP data version **<** build version: upgrade required (data
      are older than IPA build)
   #. if LDAP data version **==** build version: continue (data up to
      date)

#. check if any of services requires upgrade\ **\*\***

   #. if any service requires upgrade, upgrade is required
   #. if any service raises an error about wrong configuration (which
      requires manual fix by user), raise error and prevent to start
      services

#. if any upgrade is required, prevent to start services and prompt user
   to run *ipa-server-upgrade* (ipactl will not execute upgrade itself)
#. (otherwise) start services

**\*\*** will be available after installers refactoring

This behavior is required in container environments (or fedup), where
the ipa-server-upgrade can not be executed as RPM %postrans operation,
but data and configuration must be updated before first start of newer
IPA.

``ipactl start|restart`` option ``--skip-version-check`` overrides this
check.



Refactor ipa-upgradeconfig into modules/plugins used by ipa-server-update
----------------------------------------------------------------------------------------------

This will be done during the installer refactoring.



Requirements for using updates in containers
----------------------------------------------------------------------------------------------

-  Upgrade must run before first start of IPA services (if required)
-  Switching between images based on different OS distribution is not
   supported, upgrade can't handle differences in distribution patches.

   -  ``ipactl start | restart`` refuse to start IPA services if
      ipaplatform doesn't match the platform in configuration
   -  ``ipa-server-upgrade`` refuse to start upgrading if ipaplatform
      doesn't match the platform in configuration
   -  ``--skip-version-check`` option allows to override this check, but
      there is no guarantee the IPA will work as expected

Implementation
--------------



Storing configuration version and platform (place, format)
----------------------------------------------------------------------------------------------

The ipapython.version.IPA_VENDOR_VERSION variable is used to determine
IPA version. The format is 4.1.2-0.fc21.

The platform value consist of ipaplatform file name which is used for a
build. The platform name is detected during first run of
ipa-server-upgrade on existing systems, respectively during installation
IPA 4.2+ servers.

Values are stored into **sysupgrade.state** file as **ipa_version** and
**ipa_platform**



Feature Management
------------------

UI

N/A

CLI

+-----------------------------------+-----------------------------------+
| Command                           | Options                           |
+===================================+===================================+
| ipa-server-upgrade                | +--------------+--------------+   |
|                                   | | --skip-v     | do not check |   |
|                                   | | ersion-check | IPA version  |   |
|                                   | +--------------+--------------+   |
|                                   | | --version    | show         |   |
|                                   | |              | program's    |   |
|                                   | |              | version      |   |
|                                   | |              | number and   |   |
|                                   | |              | exit         |   |
|                                   | +--------------+--------------+   |
|                                   | | -h, --help   | show this    |   |
|                                   | |              | help message |   |
|                                   | |              | and exit     |   |
|                                   | +--------------+--------------+   |
|                                   | | -d, --debug  | print        |   |
|                                   | |              | debugging    |   |
|                                   | |              | information  |   |
|                                   | +--------------+--------------+   |
|                                   | | -q, --quiet  | output only  |   |
|                                   | |              | errors       |   |
|                                   | +--------------+--------------+   |
|                                   | | --l          | log to the   |   |
|                                   | | og-file=FILE | given file   |   |
|                                   | +--------------+--------------+   |
+-----------------------------------+-----------------------------------+

Configuration
----------------------------------------------------------------------------------------------

N/A



How to Test
-----------

Run ``ipa-server-update`` on various old versions of IPA.



Test Plan
---------

`Test
Plan <http://www.freeipa.org/page/V4/Server_Upgrade_Refactoring/Test_Plan>`__

Author
------

`Martin Basti <User:Mbasti>`__