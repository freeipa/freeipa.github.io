Healthcheck
===========

Overview
--------

IPA provides no way to do introspection to discover possible issues. A
framework is needed to assist with the identification, diagnosis and
potentially repair of problems. This has the benefit of increasing
confidence in an IPA installation and reducing costs associated with
addressing issues.

The purpose of the healthcheck tool is to find and report error
conditions that may impact the IPA environment. Automated repair would
be possible in some limited cases.

Source is available at https://github.com/freeipa/freeipa-healthcheck



Minimum Viable Product
----------------------

As the explicit goal of the healthcheck tool is to detect issues with
the environment, the Minimum Viable Product is a local, modular tool
that can be run to check IPA status.

The tool must include:

-  local server-specific checks
-  a few topology-wide checks like checking for replication conflicts,
   topology deficiencies, valid CA renewal master configuration, with
   the number of checks expected to be expanded over time. Analysis of
   topology-wide checks is left as an exercise for the user for now.

It is expected to be easily integrated with different monitoring tools.



MVP Acceptance Criteria
----------------------------------------------------------------------------------------------

-  a program that can be run, on-demand or via cron/systemd timer, which
   executes health checks.
-  the output is a machine-readable file (e.g. JSON).
-  the output location is controlled by the caller (e.g. via CLI option
   and/or configuration).
-  the output includes a timestamp.
-  each issue is recorded with a severity related to potential
   impact(s).
-  each executed test results in a success or failure status so that an
   administrator can be sure that the tests are running properly.
-  a human-readable version of the output is available (e.g. via a CLI
   result printer).



High-level Architecture
-----------------------

The tool itself which consists of plugins to analyze current
configuration to determine the state of the installation.



Use Cases
---------

The high level use case is:

As an administrator I want to be able to identify and correct issues in
my IPA installation.

This will be achieved through plugins to a framework which implement the
checks. Some specific use cases implemented by these plugins could
include:

As an administrator I want to be able to identify and correct
replication issues.

As an administrator I want to be able to identify and correct
replication conflicts.

As an administrator I want to be able to ensure that my certificates are
valid.

As an administrator I want to be able to diagnose issues with my CA
infrastructure

As an administrator I want to be able to identify and correct issues
with AD Trust.

As an administrator I want to know that all file permissions/ownership
are ok.

All tools run without issue during upstream integration testing.



Related Tickets
----------------------------------------------------------------------------------------------

-  `#1749 Need way to remove dangling managed
   groups <https://pagure.io/freeipa/issue/1749>`__

Important note: v1 healthcheck will only identify issues, not provide a
mechanism to remediate them.



How to Use
----------

Healthcheck executes a series of plugins to collect its information.
Each plugin, referred to later as a source, is organized around a
specific theme (certificate system, file system permissions and
ownership, replication, etc.). A source is a collection of tests,
refered to as checks, that should test one small piece of IPA. The
purpose is so that when developing and running the tests one can control
which are executed.

All checks must report a value unless a feature is not implemented. This
is to prevent the case where the tool reports everything is a-ok because
some unrelated issue causes the tool to not execute properly.

The report will consist of a message describing what was run and the
status. If the status is not successful it may include additional
information for the administrator to use to correct the issue (e.g. a
file has the wrong permissions, expected X and got Y).

By default the collector will execute once nightly on every master where
the package is installed.



Running it manually
----------------------------------------------------------------------------------------------

The ipa-healthcheck command will run nightly by default.

Normally ipa-healthcheck will exit with a returncode of 0, even if any
checks discovered issues with the IPA installation. A non-zero
returncode means that ipa-healthcheck failed in a non-recoverable way.

To run it manually simply execute:

``# ipa-healthcheck``

A specific check can be executed as well:

``# ipa-healthcheck --source certificate --check expiration``

Output will be a list of sources and checks executed along with the
status.

Running it manually is useful if an administrator is attempting to
correct issues and wants to double-check that something is resolved.



Repairing Issues
----------------------------------------------------------------------------------------------

Repairing an issue involves the administrator making the suggested
changes to their system.

ipa-healthcheck will make some recommendations but it is up to a human
to apply them.

Design
------

Independence
----------------------------------------------------------------------------------------------

The healthcheck tool will reside in its own upstream git repository. It
will import IPA existing modules for LDAP support, certificate handling,
replication topology and communicating with IPA itself. It will be
maintained separately so it can have its own release cycle, increasing
the speed of development. Once the framework is in place a rapid
development/release process can be done. As more plugins or capabilities
are added new releases can be made.

The initial target branches are master (4.8) and ipa-4-7.

Writing a tool that works across versions can be challenging for the
following reasons:

-  Version of python may be limited (e.g. 3.0 was written against Python
   2.6)
-  IPA libraries may be in different locations on different releases
-  IPA libraries may return different data types by version
-  Testing across all releases is a challenge

The IPA server plugin for displaying the data will reside in the FreeIPA
upstream source repository.

It will use similar branching as upstream IPA in order to deal with
differing imports, data types, etc. So there will be an ipa-4-6, ipa-4-7
and master branches eventually.

Errors
----------------------------------------------------------------------------------------------

Error messages should be descriptive without being paragraphs long. It
is very possible that external documentation will be needed to aid a
user in resolving some issues.

Severity
----------------------------------------------------------------------------------------------

Severity of a problem is defined as:

+-------+----------+-------------------------------------------------+
| Value | Severity | Definition                                      |
+=======+==========+=================================================+
| 0     | success  | The check executed and found no issues.         |
+-------+----------+-------------------------------------------------+
| 1     | critical | Something is terribly wrong (e.g. a service is  |
|       |          | not started, certificates are expired, etc).    |
+-------+----------+-------------------------------------------------+
| 2     | error    | Something is wrong but your IPA master is       |
|       |          | probably still working (e.g. replication        |
|       |          | conflict)                                       |
+-------+----------+-------------------------------------------------+
| 3     | warning  | Not an issue yet, but may be (e.g. expiring     |
|       |          | certificate soon)                               |
+-------+----------+-------------------------------------------------+

A success value is reported so an administrator can know that all checks
have executed.

Analysis
----------------------------------------------------------------------------------------------

The main flaw of this decentralized design is that it is decentralized.
For example, we require one and only one CRL generator. There is no way
to enforce this currently via healthcheck. Each master can see if it
should be the master and warn as appropriate but there is no "require
only one" option.

Note that for this particular example, and perhaps for all, we can add a
server role for CRL generator. Every master would be able to see this
role. If it is them then they check the config to confirm they are
configured appropriate. If not they raise an error.

Framework
----------------------------------------------------------------------------------------------

The healthcheck plugin framework will be thin, consisting of:

-  option parser
-  setup logging (just for when running manually)
-  LDAP connection (to be passed to plugins)
-  IPA api will be finalized and run in_server=True
-  plugin loader
-  plugin execution
-  recording results in LDAP

A failure entry will be created if a plugin fails to execute, raises an
exception, and will be cleared if a subsequent run of the plugin is
successful.

Plugins
----------------------------------------------------------------------------------------------

Plugins will define a name to be used to in part to record as the
``ipaErrorSource`` and to select when manually running individual tests
on the command-line. This is called the "source".

The entry point to the plugin is a run() method. This will execute all
of the tests provided by the plugin.

Each test will have a short, unique name known as the "check".

Care will be needed to ensure uniqueness of check names within a given
source. The framework may be able to enforce this.

So: the healthcheck daemon runs sources which executes checks. Failed
checks are stored as errors in LDAP.

Examples of sources and checks:

-  certtool

   -  expired
   -  expiring-soon
   -  tracking

-  replication

   -  sync
   -  conflict

Plugins will execute one or more discrete tests. Each test should be as
atomic as possible. It is better to report:

``File /path/to/foo has incorrect permissions, 0644 and should be 0600``

Rather than

``Files a, b, c, d have incorrect permissions``

Plugins will return an error class containing the name/value pairs of
errors and the severity as an iterator.

Plugins will return () if no errors are found.

**All** errors encountered by a plugin should be reported to the tool
(so aggressive use of try/except is required). The failure of a source
(or check) to execute is a failure that should be reported. There can be
zero chance that a failed check can cause the entire healthcheck command
execution to fail. If executing a source fails then there will be no
value for ipaErrorCheck.

The basic execution will look like:

for source in sources:

| ``   for check in sources.check():``
| ``       check()``

The analysis (deduplicating, writing to LDAP, etc) can be either done
per-source or once globally. It would be fewer LDAP searches to do
globally perhaps but would probably be fine running for each source as
well, at least in the LDAP case.

The initial plugins for the tool are:

IPA
^^^

-  basic service status (are all services running that should)
-  file permission and ownership
-  SELinux contexts
-  hostname sanity
-  disk utilization (may require config to set threshold)

Certificates/CA/KRA
^^^^^^^^^^^^^^^^^^^

-  certificate expiration warnings (may require config to define period)
-  certificate tracking issues
-  NSS trust
-  compare CA entries between dogtag and IPA
-  ensure RA agent cert is working
-  ensure there is a renewal master
-  ensure there is a CRL master
-  certmonger request tracking correctness
-  CA chain validation
-  certificate serial number ranges

Replication
^^^^^^^^^^^

-  replication consistency (are masters missing entries? expensive)
-  replication status
-  replication conflicts (old and new style)
-  DNA ranges
-  Unused RUVs



AD Trust
^^^^^^^^

-  connectivity

Kerberos
^^^^^^^^

-  validate kvno of keytabs

DNS
^^^

-  ???

Topology
^^^^^^^^

-  Check number of agreements per master
-  Find weak points in topology
-  Find single points-of-failure

Custodia
^^^^^^^^

-  verify keys are consistent

Upgrade
^^^^^^^

This test is normally not executed by default. It needs to be requested
on the command-line and is for upgrades only. If any critical failures
are reported then an Upgrade failure is recorded in LDAP and the upgrade
is aborted.

Reporting the error via LDAP would provide at least one window into
alerting users that the IPA upgrade has failed.

Execution
----------------------------------------------------------------------------------------------

As the plugins execute for any given test there will be one of two
outcomes: success or failure. Middle ground may be represented in
ipaErrorLevel. This purpose of this tool is to report errors, not info.

Upon failure:

-  a search for a matching error message and not resolved
-  if no matches, create a new record
-  otherwise continue

Upon success:

-  a search for a matching error message and empty date resolved
-  if found then mark as resolved with the current date

A 5-minute default timeout will wrap plugin execution to ensure
completion (it should be customizable per-plugin).

The definition of *match* here is TBD and depends on localization.
Automatic removal of failures would be done like this:

#. There is an initial set of errors, perhaps 0
#. A run is executed, returning 0 or more errors as the **current** list
   of errors
#. The initial errors are compared to the current errors. Errors in the
   **initial** list which are not in the **current** list are marked as
   fixed
#. Errors in the current run that are **not** in the initial set are
   recorded as new errors

This will automatically account for issues that are fixed either
automatically (e.g. certificate renewal) or as part of a larger effort
to close issues. It is not required for an administrator to mark
anything as fixed. Manually adding a resolved date will make the error
re-appear upon the next run. The exception is if the error is marked as
ignore.

The tool will return 0 if no errors are found, non-zero otherwise.

Configuration
^^^^^^^^^^^^^

The ipa-healthcheck tool will store its configuration in
/etc/ipa/healthcheck.conf. It will be an ini-style config file using the
same config routines as IPA. The format is

| ``[global]``
| ``plugin_timeout=300``

In general it would be best to store configuration in LDAP. For the
purposes of timeout LDAP may not be reachable so needs local
configuration.

Other configuration identified (may be out-of-scope for initial
implementation)

-  disk space threshold
-  days before certificate expiration warnings appear

Operation
----------------------------------------------------------------------------------------------

Kerberos credentials will be required for some operations. Ideally this
can be handled as a bind using the host principal. Bind to LDAP will be
done using ldapi which should provide read access to any data not
available as the host.

Installation
----------------------------------------------------------------------------------------------

The ipa-healthcheck command and plugins will be distributed as a
separate tarball so will be a separate package. The freeipa-server
package will have a dependency on this so it will be included by
default.

The server healthcheck plugin will be delivered in the freeipa-server
package so will be installed by default.

**Note:** there is still some uncertainty about whether ipa-healthcheck
will be a separate upstream project or be included in freeIPA. The
advantage to being separate is that it can be updated much more
frequently. The disadvantage is the additional packaging work. This is
still under discussion but for now it is separate.

Implementation
--------------



Feature Management
------------------

UI

TBD. It may be possible to make the output readable by the UI and
display the exceptions.

CLI

ipa-healthcheck:

=============== ============================================
Command         Options
=============== ============================================
ipa-healthcheck --source execute only a specific set of test
\               --verbose expanded output
\               --failures-only
\               --output-file=FILENAME
=============== ============================================

``$ ipa-healthcheck``

The ipa-healthcheck command return code indicates whether it was able to
run successfully, not if it encountered any issues with the IPA
installation. A 0 means that all sources and checks were executed. A
non-zero means some unrecoverable condition was encountered and needs
further investigation.

The ipa-healthcheck tool does not log to a file by default, it outputs
to stdout. --output-file can be used to write the JSON output to a file.

The output format by default is JSON and will look like:

| `` {``
| ``   "source": "filesystemspace",``
| ``   "check": "FileSystemSpaceCheck",``
| ``   "severity": 0,``
| ``   "uuid": "7bc5e1f1-a67f-4fe4-8eb2-ffba890aa1a7",``
| ``   "when": "20190620171103Z",``
| ``   "duration": null,``
| ``   "kw": {``
| ``     "msg": "/tmp: free space within limits: 1971 MiB >= 512 MiB",``
| ``     "store": "/tmp",``
| ``     "free_space": 1971,``
| ``     "threshold": 512``
| ``   }``



Test Plan
---------

It can be difficult to simulate some issues.

At a minimum it should return 100% success on new installations of the
supported IPA versions.

For testing certificates at least one round of certificate renewals
should be done.