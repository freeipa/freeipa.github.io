.. _ipa_command_line_tools_and_services:

IPA Command-Line Tools and Services
===================================

.. _ipa_command_line_tools:

IPA Command-Line Tools
----------------------

The following is a list of the available IPA command-line tools.

   |Note.png|\ **Note:**

      Some of these tools require root privileges. Refer to the man
      pages for full details of each command.

-  ipa-adddelegation

      **Description:** Adds a new delegation. A delegation is used to
      grant write access to certain attributes from one group to
      another.

-  ipa-addgroup

      **Description:** Adds a new group.

-  ipa-addservice

      **Description:** Adds a new service principal.

-  ipa-adduser

      **Description:** Adds a new user.

-  ipa-client-install

      **Description:** Runs the IPA client installation script. This is
      currently only available for a limited number of operating
      systems.

-  ipa-deldelegation

      **Description:** Deletes an existing delegation.

-  ipa-delgroup

      **Description:** Deletes an existing group.

-  ipa-delservice

      **Description:** Deletes an existing service principal.

-  ipa-deluser

      **Description:** Deletes an existing user. Users are automatically
      removed from groups when they are deleted.

-  ipa-findgroup

      **Description:** Searches for a group that contains a specified
      string. The search is a substring search in the **name** and
      **description** attributes.

-  ipa-findservice

      **Description:** Searches for a service principal that contains a
      specified string. The search is a substring search in the service
      principal.

-  ipa-finduser

      **Description:** Searches for a user that contains a specified
      string. The search is a substring search in the **username**,
      **given name**, **family name**, **telephone number**,
      **organization** and **title** attributes.

-  ipa-getkeytab

      **Description:** Retrieves a Kerberos keytab and optionally adds a
      service principal.

-  ipa-listdelegation

      **Description:** Lists all current delegations.

-  ipa-lockuser

      **Description:** Locks or unlock a user account.

-  ipa-moddelegation

      **Description:** Modifies an existing delegation.

-  ipa-modgroup

      **Description:** Modifies an existing group.

-  ipa-moduser

      **Description:** Modifies an existing user.

-  ipa-passwd

      **Description:** Changes a user’s password.

-  ipa-pwpolicy

      **Description:** View and update the password policy.

-  ipa-replica-install

      **Description:** Runs the IPA replica installation script.

-  ipa-replica-manage

      **Description:** Manages (lists, adds, deletes) IPA server
      replicas.

-  ipa-replica-prepare

      **Description:** Creates a replica information file for use by
      **ipa-replica-install**.

-  ipa-server-certinstall

      **Description:** Installs a CA certificate for use by IPA.

-  ipa-server-install

      **Description:** Runs the IPA server installation script.

.. _ipa_services:

IPA Services
------------

-  ipactl

      **Description:** A wrapper script to start and stop IPA-related
      services.

-  ipa_kpasswd

      **Description:** Forwards password change operations to Directory
      Server.

-  ipa_webgui

      **Description:** The IPA Web gui service.

.. |Note.png| image:: Note.png
