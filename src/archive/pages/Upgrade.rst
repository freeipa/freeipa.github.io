Guidance
--------

When there is a new updated FreeIPA server version you want to upgrade
to, in most cases it is possible to simply update the underlying
operating system and FreeIPA software. FreeIPA upgrade procedure is
designed to upgrade the FreeIPA server `Directory
Server <Directory_Server>`__ instance and also other configured services
when needed. Always make sure, that you read upgrade section of the new
FreeIPA server release as it may contain important or useful information
related to upgrade process.

.. _words_of_caution:

Words of caution
~~~~~~~~~~~~~~~~

-  Note that the server is in a **maintenance mode** during upgrade and
   does not respond to requests!
-  Schema or `Directory Server <Directory_Server>`__ database object
   changes done during the upgrade are replicated to **all FreeIPA
   masters**

.. _versioned_upgrade_notes:

Versioned upgrade notes
-----------------------

.. _freeipa_3.3.0_or_newer:

FreeIPA 3.3.0 or newer
~~~~~~~~~~~~~~~~~~~~~~

A FreeIPA server can be upgraded simply by installing updated rpms. The
server does not need to be shut down in advance.

To upgrade an existing FreeIPA installation:

``# yum update freeipa-server``

Conditions
^^^^^^^^^^

-  Upgrading from FreeIPA older than 3.3.0 is not supported and has not
   been tested.
-  Downgrading a server once upgraded is not supported.
-  Enrolled client machines do not need to be updated unless you want to
   take advantage of new features.
-  If you have multiple servers you may upgrade them one at a time. It
   is expected that all servers will be upgraded in a relatively short
   period (days or weeks, not months). They should be able to co-exist
   peacefully but new features will not be available on old servers.

.. _freeipa_3.1.0___freeipa_3.3.0:

FreeIPA 3.1.0 - FreeIPA 3.3.0
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A FreeIPA server can be upgraded simply by installing updated rpms. The
server does not need to be shut down in advance. Note that the server is
in a **maintenance mode** during upgrade and does not respond to
requests!

To upgrade an existing FreeIPA installation:

``# yum update freeipa-server``

.. _conditions_1:

Conditions
^^^^^^^^^^

-  Upgrades of FreeIPA servers with CA installed prior to 3.1 requires
   `manual migration procedure <Howto/Dogtag9ToDogtag10Migration>`__.
-  Performance improvements introduced in v3.3 require an extended set
   of indexes to be configured. RPM update for an IPA server with a
   excessive number of users may require several minutes to finish.

.. _upgrade_in_special_environment_e.g._fedup:

Upgrade in special environment (e.g. FedUp)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Please note that if you are doing the upgrade in special environment
(e.g. FedUp) which does not allow running the LDAP server during upgrade
process, upgrade scripts need to be run manually after the first boot:

.. _freeipa_4.2.0_or_newer:

FreeIPA 4.2.0 or newer
^^^^^^^^^^^^^^^^^^^^^^

``# ipa-server-upgrade``

.. _freeipa_4.1.x_or_older:

FreeIPA 4.1.x or older
^^^^^^^^^^^^^^^^^^^^^^

| ``# ipa-ldap-updater --upgrade``
| ``# ipa-upgradeconfig``
