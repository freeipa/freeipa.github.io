Help
====

\__NOTOC_\_

Overview
========

Ticket `#3060 <https://fedorahosted.org/freeipa/ticket/3060>`__;
additional ticket
`#3247 <https://fedorahosted.org/freeipa/ticket/3247>`__

Fix several problems with the ipa tool online help. Individual
improvements are detailed below.



Use Cases
=========

Please see Design. Substitute ``user`` for ``TOPIC`` and ``user-add``
for ``COMMAND`` to get concrete use cases.

Design
======

-  Output of ``ipa help`` shows the usage string ("ipa [global-options]
   COMMAND [command-options]"), a purpose string ("Manage an IPA
   domain"), a list of global options (with consistent descriptions:
   start with a capital, no dot at the end), and the following
   references to more info:

| ``   See "ipa help topics" for available help topics.``
| ``   See "ipa help ``\ ``" for more information on a specific topic.``
| ``   See "ipa help commands" for the full list of commands.``
| ``   See "ipa ``\ `` --help" for more information on a specific command.``

-  ``ipa --help``, ``ipa -h`` and ``ipa help`` produce the same output.

-  Executing ``ipa`` without a command (a violation of the declared
   usage) outputs the help (as with ``ipa help``) to stderr, along with
   an error message:

``   ipa: ERROR: command not specified``

-  Executing ``ipa``\ *``TOPIC``* (again a usage violation) outputs the
   topic help (``ipa help``\ *``TOPIC``* output) to stderr, along with
   an "unknown command" error.

-  ``ipa help COMMAND`` is equivalent to ``ipa COMMAND --help``, except
   when there is a topic with the same name.

-  When a command has the same name as a topic (for example, ``ping``),
   ``ipa help``\ *``ping``* gives the topic help, and
   ``ipa``\ *``ping``*\ ``--help`` gives the command help.

-  The output of ``ipa help help`` (and ``ipa help --help``) makes it
   clear that ``ipa help`` accepts both topics and commands.

-  Output of ``ipa help commands`` and ``ipa help topics`` is formatted
   the same way: two columns, one with the command/topic name, the other
   with a short purpose string. No extra unrelated information is given.

-  Output of ``ipa COMMAND --help`` contains the purpose string that
   appears in ``ipa help commands``.

-  Output of ``ipa help TOPIC`` contains the purpose string that appears
   in ``ipa help topics``.

-  ``ipa COMMAND --help`` and ``ipa help`` give help even without
   Kerberos credentials.

Implementation
==============

No additional requirements or changes were discovered during the
implementation phase.



Feature Managment
=================

N/A



Major configuration options and enablement
==========================================

N/A

Replication
===========

N/A



Updates and Upgrades
====================

N/A; update will change help output

Dependencies
============

N/A



External Impact
===============

Example outputs in the documentation might need to be updated



RFE author
==========

`Pviktorin <User:Pviktorin>`__