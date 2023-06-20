Promotion_to_CRL_generation_master
==================================

Overview
========

In a topology with multiple CA replicas, only one of them acts as CA
renewal master and CRL generation master. Usually, the first server
where the CA was installed owns these 2 roles, but the system
administrator may want to move these roles to a different replica
(because the current instance will be decommissioned for instance).

FreeIPA already offers tools to change the CA renewal master, but the
CRL generation master configuration involves manual steps (to disable
CRL generation on the current CRL generation master, then enable CRL
generation on the new CRL generation master). This manual procedure is
error-prone and can leave the topology in a bad state.

This feature proposes to provide a tool in order to automate these
manual steps. See issue `5803 <https://pagure.io/freeipa/issue/5803>`__.

Design
======

While FreeIPA framework offers an API to define a server role, the
framework itself counts with all the necessary information to be
available in the backend database. Assigning an IPA server as a CRL
master requires access to the filesystem (`Promote CA to Renewal and CRL
Master <https://www.freeipa.org/page/Howto/Promote_CA_to_Renewal_and_CRL_Master>`__)
and therefore the server role framework should not be used, at least not
until PKI allows us to change this configuration aspect of the system
based on the values stored in the database.

The proposed solution does not store the name of the CRL generation
master in LDAP (otherwise inconsistencies between the actual Dogtag/HTTP
config and LDAP could occur and we did not want to implement a complex
solution based on a daemon reading the config and ensuring that it is
applied). As a consequence, the CRL generation master isn't integrated
into the server role model that is used for other components (for
instance the GUI shows a table with "Enabled server roles" for each
master).

The feature will be provided as a set of 3 commands that interact with
the local master

-  one command that basically answers the question "am I configured as a
   CRL generation master"
-  one command that performs all the steps required to unconfigure CRL
   generation on the local master
-  one command that performs all the steps required to configure CRL
   generation on the local master

Based on these 3 local commands, it will be possible to create an
Ansible playbook allowing to perform a modification of the CRL
generation master. The playbook would be a wrapper using the above
commands, to first find which server(s) is(are) configured for CRL
generation, then remove the role from this(these) server(s) and finally
configure the CRL generation on the target master.

The commands should be written in a way that allows Health Check tool to
internally use them, to avoid code duplication.

The project will modify the existing ipa-server-install --uninstall so
that it prevents removal of the local master if it is the CRL generation
master.

Implementation
==============

A new **ipa-crlgen-manage** command is created. This CLI will support 3
subcommands: enable, disable, status.

The script will query/modify the local configuration and restart the
required services. It will use interfaces provided in cainstance.py:

| ``class CAInstance(DogtagInstance):``
| ``    def is_crlgen_enabled(self):``
| ``        """Check if the local CA instance is generating CRL``
| ``        Three conditions must be met to consider that the local CA is CRL``
| ``        generation master:``
| ``        - in CS.cfg ca.crl.MasterCRL.enableCRLCache=true``
| ``        - in CS.cfg ca.crl.MasterCRL.enableCRLUpdates=true``
| ``        - in /etc/httpd/conf.d/ipa-pki-proxy.conf the RewriteRule``
| ``        ^/ipa/crl/MasterCRL.bin is disabled (commented or removed)``
| ``        If the values are inconsistent, an exception is raised``
| ``        :returns: True/False``
| ``        :raises: InconsistentCRLGenConfigException if the config is``
| ``                 inconsistent``
| ``        """``
| ``    def setup_crlgen(self, setup_crlgen):``
| ``        """Configure the local host for CRL generation``
| ``        :param setup_crlgen: if True enable CRL generation, if False, disable``
| ``        """``



Feature management - CLI
========================

+--------------------+-------------+---------------------------------+
| Command            | Arguments   | Functionality                   |
+====================+=============+=================================+
| ipa-crlgen-manage  | status      | Display whether the local host  |
|                    |             | is configured for CRL           |
|                    |             | generation, and if enabled,     |
|                    |             | print the date and number of    |
|                    |             | last update                     |
+--------------------+-------------+---------------------------------+
| ipa-crlgen-manage  | enable      | Enable the CRL generation role  |
|                    |             | on the local host               |
+--------------------+-------------+---------------------------------+
| ipa-crlgen-manage  | disable     | Disable the CRL generation role |
|                    |             | on the local host               |
+--------------------+-------------+---------------------------------+
| ipa-server-install | --uninstall | If the local server is CRL      |
|                    |             | generation master, refuse to    |
|                    |             | uninstall unless                |
|                    |             | --ignore-last-of-role is        |
|                    |             | provided or (interactive mode)  |
|                    |             | the user confirmed he is OK     |
|                    |             | with uninstallation             |
+--------------------+-------------+---------------------------------+

Usage:

Find whether the local host is configured for CRL generation:

| ``# kinit admin``
| ``# ipa-crlgen-manage status``
| ``CRL generation: enabled``
| ``Last CRL update: 2019-03-06 16:10:16``
| ``Last CRL Number: 2``
| ``The ipa-crlgen-manage command was successful``

Disable CRL generation on the local host

| ``# kinit admin``
| ``# ipa-crlgen-manage disable``

Enable CRL generation on the local host

| ``# kinit admin``
| ``# ipa-crlgen-manage enable``



Requirements for the ipa-crlgen-manage script
---------------------------------------------

-  The script must be run as a root user
-  The script requires a Kerberos ticket as admin
-  The script requires ipa services to be up and running as it checks if
   the local host is a CA master
-  If the script is executed on a host that is not an IPA server, it
   must exit with 2
-  If the script fails to execute, it must exit with 1
-  If the script succeeds, it must exit with 0

With "status" subcommand:

-  If the script detects an inconsistent configuration, it must print an
   error message and exit with 1
-  If the scripts detects that the server is not configured for CRL
   generation/not a CA master, it must output "CRL generation: disabled"
   and exit with 0
-  If the script detects that the server is properly configured for CRL
   generation, it must output "CRL generation: enabled" and exit with 0.
   If a CRL is available, the script must also print the date and number
   of the last CRL update.

With "enable" subcommand:

-  The script must validate that the local host is a FreeIPA master and
   provides a CA instance, and refuse to enable CRL generation if it's
   not the case
-  If the local host is already configured as CRL generation master, the
   script must print that no modification was done and exit with 0
-  If the local host needs to be configured, the script must perform all
   the config steps detailed in `Promote CA to Renewal and CRL
   Master <https://www.freeipa.org/page/Howto/Promote_CA_to_Renewal_and_CRL_Master>`__
   and trigger the generation of a new CRL before exiting with 0

With "disable" subcommand:

-  If the local host is not a CA instance, the script must exit with 0
-  If the local host is not configured for CRL generation, the script
   must print that no modification was done and exit with 0
-  If the local is configured for CRL generation, the script must
   perform all the unconfiguration steps detailed in `Promote CA to
   Renewal and CRL
   Master <https://www.freeipa.org/page/Howto/Promote_CA_to_Renewal_and_CRL_Master>`__
   and exit with 0.



Requirements for the ipa-server-install --uninstall script
----------------------------------------------------------

In interactive mode:

-  When run on a host that is CRL generation master, the script must
   warn the user that the uninstall operation will remove CRL generation
   role and prompt for confirmation.

In non-interactive mode:

-  When run on a host that is CRL generation master, the script must
   refuse to uninstall the server, unless the option
   --ignore-last-of-role was provided. In any case, the uninstall script
   must print a warning about removing a master with CRL generation
   role.



Test Plan
=========

¯\_(ツ)_/¯



Future considerations
=====================

Hopefully, Dogtag will once implement CRL master configuration in LDAP
(https://pagure.io/dogtagpki/issue/1262). Should that ever happen, we
may consider the following:

-  FreeIPA should provide a framework-based command allowing to find
   which replica is currently handling the CRL generation
-  FreeIPA should provide a framework-based command allowing to move the
   CRL generation to a different replica. It would be nice to have a
   single command to disable CRL generation on the current CRL
   generation master and enable CRL generation on the new CRL generation
   master.