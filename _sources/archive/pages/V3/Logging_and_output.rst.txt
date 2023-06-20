Logging_and_output
==================

Overview
========

The output, logging, and command-line options of our install/management
tools are inconsistent. This RFE aims to clean up some of the
inconsistency.

Specifically it addresses:

-  Inconsistencies between console output and log files
-  Tools using either --verbose or --debug used for the same purpose

And some collateral issues:

-  The --version option missing in some tools
-  The problem of e.g. an accidental ipa-server-install overwriting
   ipaserver-install.log by the useless message, "The server is already
   installed"

This is part of `#2652 Framework for admin/install
tools <https://fedorahosted.org/freeipa/ticket/2652>`__

Design
======

Each command will have these command-line options (added automatically
by the framework):

| `` --version        show program's version number and exit``
| `` -h, --help       show the help message and exit``

| `` output and logging options:``
| ``   -q, --quiet    Output only errors``
| ``   -v, --verbose  Print debugging information``
| ``   --log-file     (alternate) log file name``

The precise meanings of the output options are:

| `` (nothing): print INFO-level messages and above to stderr``
| `` -q: only print ERROR-level messages and above to stderr``
| `` -v: print all messages to stderr, prefixed with severity level``

In all cases, if there is a log file, all messages go to it

In commands that currently have it, the \`-d, --debug\` option will
become a deprecated alias for --verbose.

Note that this RFE only adds new options, so backwards compatibility is
maintained.

The tools will use the following log message severities:

-  CRITICAL for fatal errors
-  ERROR for critical things that the admin must see even in --quiet
   mode
-  WARNING for things that need to stand out in the log
-  INFO to display normal messages
-  DEBUG to spam about everything the program does
-  a plain print statement for things that should not be log, for
   example interactive prompting should use the console and follow up
   with a DEBUG mentioning the final value

The commands check their arguments and most do some validation and
possible interactive prompting before doing any modification of the
system. Since the system is not changed, the user can always re-run with
--verbose if there are problems in this phase. Logging information from
this phase to a file is useless. The file logging will start after the
validation/prompting. The first logged message(s) will mention the
command and options used.



Use Cases
=========

#. User runs several /usr/sbin/ipa-\* commands with the --help option
#. User sees that the logging-related options are the same in all of
   those programs.

#. User runs an ipa-\* command with the -q option
#. User only sees errors.

#. User runs an ipa-\* command (possibly with --log-file, if the command
   doesn't log by default)
#. User later reviews the log file and finds all relevant information.

Implementation
==============

The RFE will be implemented gradually as new commands are ported to the
framework. The following table will be updated as work progresses.

+---------+---------+---------+---------+---------+---------+---------+
| Command | --      | --quiet | --      | --debug | --l     | Changed |
|         | version |         | verbose |         | og-file | in      |
|         |         |         |         |         |         | version |
+=========+=========+=========+=========+=========+=========+=========+
| ip      | present | added   | added   | present | added   | 3.1     |
| a-ldap- |         |         |         |         |         | (e4     |
| updater |         |         |         |         |         | 37491); |
|         |         |         |         |         |         | 3.2     |
|         |         |         |         |         |         | (5      |
|         |         |         |         |         |         | 5cfd06) |
+---------+---------+---------+---------+---------+---------+---------+
| ipa-r   | present | added   | added   | present | added   | 3.1     |
| eplica- |         |         |         |         |         | (f6     |
| prepare |         |         |         |         |         | a5647); |
|         |         |         |         |         |         | 3.2     |
|         |         |         |         |         |         | (2      |
|         |         |         |         |         |         | 6c4987) |
+---------+---------+---------+---------+---------+---------+---------+
| i       | added   | added   | added   | not     | added   | 3.3     |
| pa-serv |         |         |         | added   |         | (02     |
| er-cert |         |         |         | (-d     |         | 214c4); |
| install |         |         |         | means   |         | 3.4     |
|         |         |         |         | --      |         | (0      |
|         |         |         |         | dirsrv) |         | 2be7ac) |
+---------+---------+---------+---------+---------+---------+---------+
|         |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| ipa-a   | present | -       | -       | present | -       |         |
| dtrust- |         |         |         |         |         |         |
| install |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| ipa-ca- | present | -       | -       | present | -       |         |
| install |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| ipa-cl  | -       | -       | -       | present | -       |         |
| ient-au |         |         |         | (no -d) |         |         |
| tomount |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| ipa-    | present | -       | -       | present | -       |         |
| client- |         |         |         |         |         |         |
| install |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| ipa     | -       | -       | -       | present | -       |         |
| -compat |         |         |         |         |         |         |
| -manage |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| ipa-cs  | present | -       | present | -       | -       |         |
| replica |         |         |         |         |         |         |
| -manage |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| i       | present | -       | -       | present | -       |         |
| pa-dns- |         |         |         |         |         |         |
| install |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| ipa-ge  | -       | present | -       | -       | -       |         |
| tkeytab |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| i       | -       | present | -       | present | -       |         |
| pa-join |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| ipa-m   | -       | -       | -       | present | -       |         |
| anaged- |         |         |         |         |         |         |
| entries |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| ipa-nis | -       | -       | -       | present | -       |         |
| -manage |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| ipa-rep | present | present | -       | present | -       |         |
| lica-co |         |         |         |         |         |         |
| nncheck |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| ipa-r   | present | -       | -       | present | -       |         |
| eplica- |         |         |         |         |         |         |
| install |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| ipa-    | present | -       | present | -       | -       |         |
| replica |         |         |         |         |         |         |
| -manage |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| ipa-r   | -       | -       | -       | present | -       |         |
| mkeytab |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| ipa-    | present | -       | -       | present | -       |         |
| server- |         |         |         |         |         |         |
| install |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| ipa     | present | present | -       | present | -       |         |
| -upgrad |         |         |         |         |         |         |
| econfig |         |         |         |         |         |         |
+---------+---------+---------+---------+---------+---------+---------+
| ipactl  | -       | -       | -       | present | -       |         |
+---------+---------+---------+---------+---------+---------+---------+



RFE author
==========

`Pviktorin <User:Pviktorin>`__