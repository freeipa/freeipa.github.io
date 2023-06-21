Trust_config_command
====================

\__NOTOC_\_

Overview
========

Related tickets `#3333 <https://fedorahosted.org/freeipa/ticket/3333>`__

Related designs: `AD Trusts <IPAv3_AD_trust>`__, `Multiple Trust
servers <V3/MultipleTrustServers>`__

We expose an interface to show configuration of a foreign trusted domain
(for example NetBIOS name or SID) with ``trust-show`` command. We should
also expose an interface to actually show global configuration of
FreeIPA domain which is already being stored in
``cn=$IPA_DOMAIN,cn=ad,cn=etc,$IPA_SUFFIX``.



Use Cases
=========

-  Trust administrator wants to get properties of FreeIPA domain global
   configuration, for example:

   -  auto-generated GUID or SID
   -  NetBIOS name configured during ``ipa-adtrust-install``
   -  Primary fallback group

-  Trust administrator wants to modify some of the FreeIPA domain
   configuration settings which are allowed to be modified (most of the
   configuration settings will be read-only as any change could break
   existing trusted domain relationships)

Design
======

Add 2 new commands ``trustconfig-show`` and ``trustconfig-mod`` to
manipulate the global configuration. Even though we currently (FreeIPA
3.1) support only Active Directory type of trusts, these commands should
be already prepared for more types as they will come in the future.



Feature Management
==================

UI

UI will need to implement new page in IPA Server - Trusts section,
similarly to "DNS Global Configuration", except this one would be named
"Trusts Global Configuration".

The page which would take advantage of the new commands to change these
settings. It of course need to respect which fields are read only and
which are not. Primary fallback group field should use a widget allowing
selecting an existing group in FreeIPA domain or use the default hidden
group "SMB Default Group".

CLI



trustconfig-show
----------------------------------------------------------------------------------------------

``trustconfig-show`` will display configuration of FreeIPA domain,
stored in ``cn=$DOMAIN,cn=$TYPE,cn=etc,$SUFFIX``. When the trust is not
configured (``ipa-adtrust-install`` was not run), an error will be
returned.

Required command options:

-  ``--type``: The exactly same meaning as with ``trust-add``.
   Currently, the only allowed value will be ``ad``.



trustconfig-mod
----------------------------------------------------------------------------------------------

``trustconfig-mod`` will allow Administrator to modify selected
attributes which would not cause breakage of current trust
relationships:

-  Primary fallback group (ipaNTFallbackPrimaryGroup): attribute will
   accept either a group name for group located in a standard group
   container (``cn=groups,cn=accounts,$SUFFIX``) or a DN of the group.
   The group must have a ``posixGroup`` objectclass so that its GID
   number can be retrieved by ``ipa-kdb`` (and other components).
   Default value for this attribute is a group named
   ``Default SMB Group`` which resides in standard IPA group container
   (but is hidden on purpose from ``group-find`` command).

Required command options:

-  ``--type``, see ``trustconfig-show`` command for details.



Major configuration options and enablement
==========================================

These commands will be only functional when the trusts are configured.

Replication
===========

N/A



Updates and Upgrades
====================

N/A

Dependencies
============

N/A



External Impact
===============

N/A



Design page authors
===================

`Mkosek <User:Mkosek>`__