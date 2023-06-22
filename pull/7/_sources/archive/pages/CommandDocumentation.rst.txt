.. _command_line_documentation_guidelnes:

Command-line Documentation Guidelnes
------------------------------------

IPA provides a set of command-line utilities that perform actions as
varied as installing the product, managing replicas and administering
the IPA data. The clearest differentiator is the command which executes
these.

The ``ipa`` command executes data management commands (user, group,
etc.) and the ``ipa-`` (dash) commands are generally used in a more
limited way (e.g. ``ipa-server-install``, ``ipa-replica-manage``). I
refer to these as the standalone commands.

.. _command_types:

Command types
-------------

The standalone commands each have their own man page. This includes a
description of what the command does and a list of the available options
(and perhaps even an example or two).

The ``ipa`` command executes commands from the XML-RPC framework and
manages data in the IPA database. There is a single man page for the ipa
command with an overview of its capabilites, but not a complete list of
all plugins and what their various options do.

.. _getting_help:

Getting help
------------

.. _ipa__commands:

ipa- commands
~~~~~~~~~~~~~

To get help from a standalone command use either the --help option or
see the man page:

| ``% ipa-replica-manage --help``
| ``Usage: ipa-replica-manage [options]``

| ``Options:``
| ``  --version             show program's version number and exit``
| ``  -h, --help            show this help message and exit``
| ``  -H HOST, --host=HOST  starting host``
| ``  -p DIRMAN_PASSWD, --password=DIRMAN_PASSWD``
| ``                        Directory Manager password``
| ``  -v, --verbose         provide additional information``
| ``  -f, --force           ignore some types of errors``
| ``  --port=PORT           port number of other server``
| ``  --binddn=BINDDN       Bind DN to use with remote server``
| ``  --bindpw=BINDPW       Password for Bind DN to use with remote server``
| ``  --winsync             This is a Windows Sync Agreement``
| ``  --cacert=CACERT       Full path and filename of CA certificate to use with``
| ``                        TLS/SSL to the remote server``
| ``  --win-subtree=WIN_SUBTREE``
| ``                        DN of Windows subtree containing the users you want to``
| ``                        sync (default cn=Users,<domain suffix)``
| ``  --passsync=PASSSYNC   Password for the Windows PassSync user``

``% man ipa-replica-manage``

.. _framework_commands_ipa:

Framework commands (ipa)
~~~~~~~~~~~~~~~~~~~~~~~~

There are a number of different ways to get help from the ``ipa``
command.

The overall man page for the command:

``% man ipa``

Note that ipa --help and ipa help are two different, if confusing,
things.

``ipa --help`` provides assistance on using the ipa command itself.

``ipa help`` provides assistance on the data management commands within
ipa.

A list of available topics. A topic is a high-level view of the data
management. For example user, group and host are topics.

| ``% ipa help topics``
| ``Usage: ipa [global-options] COMMAND ...``

| ``Built-in commands:``
| ``  console     Start the IPA interactive Python console.``
| ``  help        Display help for a command or topic.``
| ``Help topics:``
| ``  aci         Directory Server Access Control Instructions (ACIs)``
| ``  automount   Automount``
| ``  cert        Command plugins for IPA-RA certificate operations.``
| ``  config      IPA configuration``
| ``  dns         Domain Name System (DNS) plugin``
| ``  group       Groups of users``
| ``  hbac        Host based access control``
| ``  host        Hosts/Machines (Identity)``
| ``  hostgroup   Groups of hosts.``
| ``  krbtpolicy  Kerberos ticket policy``
| ``  migration   Migration to IPA``
| ``  misc        Misc plugins``
| ``  netgroup    Netgroups``
| ``  passwd      Password changes``
| ``  pwpolicy    Password policy``
| ``  rolegroup   Rolegroups``
| ``  service     Services (Identity)``
| ``  taskgroup   Taskgroups``
| ``  user        Users (Identity)``
| :literal:`Try `ipa --help` for a list of global options.`

To dive into a particular topic:

| ``% ipa help user``
| ``Users (Identity)``
| ``Related commands:``
| ``  user-add     Create new user.``
| ``  user-del     Delete user.``
| ``  user-find    Search for users.``
| ``  user-lock    Lock user account.``
| ``  user-mod     Modify user.``
| ``  user-show    Display user.``
| ``  user-unlock  Unlock user account.``

To get help on a particular framework command:

| ``% ipa help user-add``
| ``Purpose: Create new user.``
| ``Usage: ipa [global-options] user-add LOGIN``
| ``Options:``
| ``  -h, --help       show this help message and exit``
| ``  --first=STR      First name``
| ``  --last=STR       Last name``
| ``  --homedir=STR    Home directory``
| ``  --gecos=STR      GECOS field``
| ``  --shell=STR      Login shell``
| ``  --principal=STR  Kerberos principal``
| ``  --email=STR      Email address``
| ``  --password       Set the user password``
| ``  --uid=INT        UID (use this option to set it manually)``
| ``  --street=STR     Street address``
| ``  --addattr=STR    Add an attribute/value pair. Format is attr=value``
| ``  --setattr=STR    Set an attribute to an name/value pair. Format is``
| ``                   attr=value``
| ``  --all            retrieve all attributes``
| ``  --raw            print entries as stored on the server``

The framework commands are supposed to be self-documenting, with the ipa
man page there to describe the basic layout of how things should work.
Not all plugins currently have extra documentation but the goal is to
have help like the dns plugin:

| ``% ipa help dns``
| ``Domain Name System (DNS) plugin``
| ``Implements a set of commands useful for manipulating DNS records used by``
| ``the BIND LDAP plugin.``
| ``EXAMPLES:``
| `` Add new zone;``
| ``   ipa dns-add example.com nameserver.example.com admin@example.com``
| `` Add second nameserver for example.com:``
| ``   ipa dns-add-rr example.com @ NS nameserver2.example.com``
| `` Delete previously added nameserver from example.com:``
| ``   ipa dns-del-rr example.com @ NS nameserver2.example.com``
| `` Add new A record for www.example.com: (random IP)``
| ``   ipa dns-add-rr example.com www A 80.142.15.2``
| ``...``
| ``...``

.. _rules_of_the_road:

Rules of the Road
-----------------

.. _standalone_commands:

Standalone commands
~~~~~~~~~~~~~~~~~~~

Every standalone command must have:

-  A man page
-  Usage output

.. _framework_commands:

Framework commands
~~~~~~~~~~~~~~~~~~

Framework commands must have:

-  A single man page, ipa
-  Basic usage output for options, this is automatic
   (``ipa user-add --help``)
-  An overview of the command via ``ipa help <topic>``

The overview comes from the initial docstring in the plugin itself. It
should include:

-  User-understandable plugin name
-  Basic description of what the plugin does
-  Usage examples
