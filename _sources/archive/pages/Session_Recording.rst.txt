Session_Recording
=================

Overview
========

Rationale
---------

Many companies in highly regulated industries like financial or health
care have to balance costs of maintaining their IT infrastructure and
implementing security practices required by regulations like SOX, PCI or
HIPAA. To reduce the cost of ownership they frequently subcontract, or
offshore the IT services to other companies and vendors.

To achieve the level of accountability imposed on them by different
regulations and to meet audit requirements they need to implement
constant monitoring of the activities conducted by the contracted
vendors. To do such monitoring they collect logs and record all the
activity initiated by the privileged users on the systems they have
regardless of the operating system that is used. In other words they
store all the sessions for all the privileged users on the mission
critical systems.



What is it about?
-----------------

The solution is aimed at answering the following questions:

-  Who logged into the system?
-  What did the user do while he/she was logged into that system?
-  How a malicious activity by a contractor or employee can be proven?

The tools and capabilities provided by solution should allow auditor or
security officer to detect and prove that malicious activity has
happened in the past, or detect one happening in real time.



User Stories
============



Auditor/Security Officer
------------------------

As an auditor or security officer who is responsible for maintaining
security and preventing attacks, or proving the fact that a malicious
activity has been conducted by an employee or a contractor:

-  I would like every protected client system to record information
   about who have accessed that system, so in case of a security
   incident it is easier to identify the culprit. This information
   should be emitted by the system in such a way that the recorded
   privileged user can't prevent its flow without leaving tracks he/she
   was trying to do so. Supported by `Phase 0 <#phase-0>`__.
-  I would like the information to be consolidated on a central server
   where it can be viewed and correlated with the authentication
   activity collected from the central authentication servers, so that I
   can identify and verify when sessions started and ended and whether
   they were preceded by authentication failures. Supported by `Phase
   1 <#phase-1>`__.
-  I would like every command and operation that the user has ran while
   he/she was logged into the system to be recorded and delivered to the
   central server so that it can be stored safely, viewed and analyzed,
   and correlated with the activity of other users. Supported by `Phase
   1 <#phase-1>`__.
-  I would like the activity information to be recorded and later
   presented in the most intuitive way, to be able to understand what
   the user was doing while logged into the system. The best format for
   such information, as recognized by the IT professionals, is the full
   screen capture of the input and output of the user's terminal
   session. Supported by `Phase 0 <#phase-0>`__.
-  I would like to be able to better rationalize the impact of the
   malicious activity by correlating the session recording with the
   underlying activity (audit trail) recorded by the system. Supported
   by `Phase 1 <#phase-1>`__.
-  I would like to be able to view the recorded sessions in a central
   server via a web interface, since I prefer not to be bound to a
   specific platform and install a thick client. Supported by `Phase
   1 <#phase-1>`__.
-  In some cases I would like to be able to play session back from a
   command line using a simple command line tool, so I can test the
   setup, or act in limited environments, such as system recovery modes.
   Supported by `Phase 0 <#phase-0>`__.
-  I would like to be able to watch the playback of the session, start,
   stop, rewind, scroll forward or back, play faster or slower. I would
   like to be able to pause the playback select the command that the
   user entered and see the recorded trail of the activities happening
   on the system (file system operations, socket requests, started and
   stopped process) that correspond to the selected command. This would
   allow me to understand the sequence of events within the user session
   quicker and easier. Supported by `Phase 2 <#phase-2>`__.
-  I would like to be able to keep the recorded information for playback
   for a period of time defined by my policy and capacity preferences.
   After that period the solution should overwrite, purge or archive the
   collected information depending on the configuration. Supported by
   `Phase 2 <#phase-2>`__.
-  I would like to be able to easily navigate to the recorded session so
   that I can select the right session to play back. I should be able to
   search recorded sessions and related audit data by user, user group,
   system and group of systems as well as narrow search to a specific
   service like SSH or limit the time period I am looking for the
   session to start (or finish). Supported by `Phase 2 <#phase-2>`__.
   For example I should be able to say:

   -  I want to find all the sessions by John Doe in my environment
      during this weekend
   -  I want to find all administrative sessions that were recorded last
      week against my file servers in my New Jersey datacenter.



System Administrator
--------------------

As a system administrator responsible for integrating the session
recording solution into existing or new infrastructure:

-  I would like to be able to control session recording locally on
   specific hosts, for specific users, without requiring SSSD or FreeIPA
   involvement or use at all, so that I can test it out, support simple
   installations, or integrate with other identity management solutions.
   Supported by `Phase 0 <#phase-0>`__.
-  I would like to be able to control session recording locally on
   specific hosts, for specific users, through already deployed SSSD,
   without involving the central identity management solution (FreeIPA
   or other) which controls it, so that I can test it out, or use it for
   just a few systems with lower management cost. Supported by `Phase
   0 <#phase-0>`__.
-  I would like to be able to control session recording centrally via
   FreeIPA for many hosts, host groups, users or groups, so that I can
   save on maintenance costs by having all configuration done through a
   single interface, in one place. Supported by `Phase 1 <#phase-1>`__.
-  I would like the delivery mechanism that is used to send information
   from the client system to the central server to be scalable, reliable
   and support high-availability and load-balancing configurations.
   Supported by `Phase 0 <#phase-0>`__.
-  I would like the information that is sent from the client system, to
   be sent via protected and authenticated channel so that it can't be
   tampered with or disclosed in transit. Supported by `Phase
   0 <#phase-0>`__.

Design
======

Architecture
------------

We record what passes between the user terminal and the shell, what
files the user accesses and what commands he/she executes, and send all
that to a central storage, where it can be searched, correlated and from
where it can be played back. Who, where and how is recorded can be
controlled either centrally or locally.



Client-side components
----------------------------------------------------------------------------------------------

-  **Tlog-rec** – the recording process which is invoked in place of the
   user's shell when he/she logs into the system. The recording process
   creates a pseudo-terminal, starts the actual user shell under it, and
   records everything that passes between the pseudo-terminal (the user
   shell) and the actual user terminal. The user should not be able to
   stop or prevent the recording without leaving evidence of trying to
   do so.
-  **SSSD** – the component which is responsible for telling the system
   which shell should be started for the user, based on the POSIX
   information for the user account. SSSD should inform the system that
   the recording process is the user shell, and pass the real shell to
   the recording process so that it can be started within the recorded
   session.
-  **Auditd** – general system auditing subsystem which collects all the
   activity related to user session in the form of the audit entries
-  **Aushape** – an audit log converter running under Auditd to convert
   audit events to JSON on the fly, passing them to the logging server.
-  **Logging server** - Rsyslog, Fluentd, or Logstash – a collection
   agent, which streams the audit and session recording data from the
   system to the central server.



Server-side components
----------------------------------------------------------------------------------------------

-  **FreeIPA** - the central integrated security management solution,
   which stores and provides information of who, where, and how to
   record.
-  **ElasticSearch** - the data storage where the session recording and
   audit data can be placed and correlated.
-  **Tlog-play** - terminal based playback tool which can be used from
   the command line to recreate the session.
-  **WebUI (TBD)** - playback terminal with the audit trail correlation
   – most likely a custom web UI control.



Control and Data Flow
---------------------
::

   | ````
   | ``           Servers            Network               Clients``
   | ``     _____________________               _______________________________``
   | ``    |  _________________  |             |  ___________________          |``
   | ``    | |                 | |             | |                   |         |``
   | ``    | |     FreeIPA     |====(control)===>|       SSSD        |         |``
   | ``    | |_________________| |             | |___________________|         |``
   | ``    |         /\          |             |     ||          /\            |``
   | ``    |         ||          |             |     ||          ||            |``
   | ``    |      (control)      |             |  (control)   (control)        |``
   | ``    |   ......||.......   |             |     ||  ........||.........   |``
   | ``    |  : Administrator :  |             |     || :   Administrator   :  |``
   | ``    |   '''''''''''''''   |             |     ||  '''||'''''''''||'''   |``
   | ``    |                     |             |     ||  (control)  (control)  |``
   | ``    |   ...............   |             |     ||     ||         ||      |``
   | ``    |  :    Auditor    :  |             |  ___\/_____\/__   ____\/____  |``
   | ``    |   ''/\'''''''/\''   |             | |              | |          | |``
   | ``    |     ||       ||     |             | |   Tlog-rec   | |  Auditd  | |``
   | ``    |   (data)   (data)   |             | |______________| |__________| |``
   | ``    |  ___||__   __||___  |             |        ||             ||      |``
   | ``    | |       | |       | |             |        ||          ___\/____  |``
   | ``    | | Tlog- | | WebUI | |             |        ||         |         | |``
   | ``    | | play  | | (TBD) | |             |        ||         | Aushape | |``
   | ``    | |_______| |_______| |             |        ||         |_________| |``
   | ``    |     /\       /\     |             |        ||             ||      |``
   | ``    |     ||       ||     |             |      (data)         (data)    |``
   | ``    |   (data)   (data)   |             |        ||             ||      |``
   | ``    |  ___||_______||___  |             |     ___\/_____________\/___   |``
   | ``    | |                 | |             |    |                       |  |``
   | ``    | |                 | |             |    |        Rsyslog        |  |``
   | ``    | |                 | |             |    | - - - - - - - - - - - |  |``
   | ``    | |  Elasticsearch  |<====(data)=========|        Fluentd        |  |``
   | ``    | |                 | |             |    | - - - - - - - - - - - |  |``
   | ``    | |                 | |             |    |        Logstash       |  |``
   | ``    | |_________________| |             |    |_______________________|  |``
   | ``    |_____________________|             |_______________________________|``

The control flow differs by case.



Standalone local control
----------------------------------------------------------------------------------------------

The recorded user's POSIX entry is specified as ``tlog-rec``, and the
actual user's shell is set for all users in the global configuration of
tlog-rec. Alternatively, a special per-shell symlink to tlog-rec is set
as the shell for the user (e.g. ``tlog-rec-shell-bash``, or
``tlog-rec-shell-zsh``).

After a user is authenticated, the login program
(login/telnetd/sshd/etc.) starts tlog-rec, as it is specified as the
shell for the user. Tlog-rec checks if it was invoked under the special
name. If it was, it starts the shell extracted from the name. If it
wasn't, it starts the shell specified in the global configuration.



Local control via SSSD
----------------------------------------------------------------------------------------------

SSSD is configured to enable session recording for all, or specific
users and/or groups. This is done by adding a ``session_recording``
section to ``sssd.conf``, containing parameters specifying the above.

After a user is authenticated, the program that logs user in
(login/telnetd/sshd/etc.) queries which shell to start for the user, and
asks the system to set up the session. If SSSD is configured to record
the specific user, it answers that shell should be ``tlog-rec`` and, as
part of session setup, adds a variable to the user environment telling
``tlog-rec`` which actual shell it needs to start. E.g. it sets
``TLOG_REC_SHELL=/bin/bash``.

The login program starts ``tlog-rec``. Tlog-rec reads the environment
variable added by SSSD and starts the actual user shell.



Central control via SSSD and FreeIPA
----------------------------------------------------------------------------------------------

FreeIPA is instructed to add directory entries specifying which hosts
and users should have recording enabled and with which tlog-rec settings
(whether to record input, output, or both, etc.). The schema should be
similar to the SELinux rule schema, but specific design is TBD.

After a user is authenticated, the login program
(login/telnetd/sshd/etc.) queries which shell to start for the user, and
asks the system to set up the session. SSSD queries FreeIPA directory to
check if this user on this machine should be recorded. If it should,
then SSSD answers that shell should be ``tlog-rec``. As part of the
following session setup, SSSD retrieves tlog-rec settings applicable for
this user and machine from FreeIPA directory, and adds an environment
variable containing those settings, along with a variable specifying the
actual user's shell.

E.g. it sets
``TLOG_REC_CONF_TEXT='{"log": {"input": false, "output": true}}'`` and
``TLOG_REC_SHELL=/bin/bash``. This example specifies that input
recording should be off (e.g. to avoid recording of passwords), output
recording should be on, and the actual user's shell should be
``/bin/bash``. All of that would override the global tlog-rec
configuration.

The login program starts ``tlog-rec``. Tlog-rec reads the environment
variables added by SSSD, adjusts its settings and starts the actual user
shell.



Further control and data flow
----------------------------------------------------------------------------------------------

Other tlog-rec configuration, not mentioned above, is specified in
tlog-rec's global configuration file ``/etc/tlog/tlog-rec.conf``.

After tlog-rec starts the actual user shell, it inserts itself between
it and the user terminal and records everything passing in-between. The
recording is formatted as JSON messages and logged using standard syslog
interface.

Rsyslogd takes these messages, strips away syslog formatting, and sends
them to ElasticSearch as pure JSON. ElasticSearch indexes and stores
them.

The administrator configures auditd to record any required audit events.
The audit messages are converted to JSON and are sent to ElasticSearch
(TBD). ElasticSearch indexes and stores them.

An auditor can then search and correlate the data stored in
ElasticSearch using one of the available analytic interfaces, such as
Kibana or Graphana, and can then playback specific sessions on the
command line with ``tlog-play``, or using the special web UI (TBD).



Audit recording details
-----------------------

Control
----------------------------------------------------------------------------------------------

At the moment we leave the task of configuring auditd for tracking of
specific user activities to the administrator, either manually or using
any of the automatic configuration tools. In the future, however, we may
implement some degree of centralized and automated control over what is
recorded.

File accesses (read/write/execute/attribute change) can be captured with
the help of kernel auditing. The events are written to audit.log or
journal.

Executed commands can be captured this way as well: a rule to log every
execve syscall can be added. The command line will be written to
audit.log or journal.

Both of these are configurable at runtime via auditctl(8) or via
audit.rules(7), matching specific user/group and other process
properties. There is also an API for this in libaudit.

Extra session creation data (production/test environment, tenant, etc.)
can be logged directly from PAM modules (e.g. pam_sss) using audit
facilities and will go to audit.log or journal. A few PAM modules log
there already. This is done via libaudit as well.
::

   | ````
   | ``     Kernel                   Userspace``
   | ``    ____________``
   | ``    __________  |                                 | auditd   <== audit.rules``
   | ``              | | Rules and            | Rules    | auditctl <== command line``
   | ``     Audit    | | messages             |<=API=====| SSSD?``
   | ``              |<===netlink==| libaudit |``
   | ``    subsystem | |                      | Messages | auditctl <== command line``
   | ``    __________| |                      |<=API=====| Aplications``
   | ``    ____________|                                 | PAM modules (pam_sss?)``

There's one caveat regarding automatic configuration, though (citing
Miloslav Trmač):

   Note that the really high-security/paranoid setups (which would be
   likely to want to use the session recording) use (auditctl -e 2),
   i.e. locking down audit so that it is not possible to add rules at
   runtime, so dynamically adding audit rules at each login (as the
   above seems to imply) could be problematic.

   ...

   The ability to audit by dynamic IPA group/role membership would
   probably be useful, and perhaps the default path; but there should be
   \_some_, even if laborious, way to run in the lockdown mode (e.g.
   document the rules that should be used for the targeted users,
   allowing the administrator to manually add the rules to audit.conf
   and then to disable the IPA-based run-time audit rule generation.)

Data
----------------------------------------------------------------------------------------------

We need to convert audit data to JSON before storing it in
ElasticSearch.
::

   | ````
   | ``     Kernel                   Userspace``
   | ``    ____________``
   | ``    __________  |``
   | ``              | |           | auditd  => | audit.log``
   | ``     Audit    | | Messages  |            | audispd => | plugin1``
   | ``              |====netlink=>|                         | plugin2``
   | ``    subsystem | |           |                         | pluginN``
   | ``    __________| |           |                         | aushape``
   | ``    ____________|           | systemd => journal``

A new tool called "Aushape" is being developed to support live
conversion of audit events to JSON, while running as an audispd plugin.
For the moment, to reach the logging server, it logs those events via
syslog(3), but will likely support journal inteface as well. The
development is done in cooperation with auditd developers and aushape is
intended to be a part of future auditd releases.

Playback
--------

Tlog-play provides basic session playback support on the terminal,
either from a plain text file containing JSON messages (can be created
by tlog-rec), or from ElasticSearch directly.

A web UI needs to be implemented as a reusable component, which can then
be embedded into other web UI's as necessary. The possible targets are
FreeIPA web UI, Cockpit, CloudForms and Satellite.

The UI component will fetch JSON documents (terminal I/O and audit log
entries) from ElasticSearch with a RESTful API, and display them in
various ways. Fetching needs to be random access and eventually
asynchronous. What and where to fetch from will be specified from the
outside.

The actual session recording will need to be displayed in a
JavaScript-based terminal emulator (such as xterm.js). The component
will need to support random positioning and variable speed playback of
the terminal session (à la https://showterm.io/ and
https://asciinema.org/). The session terminal I/O will need to be
augmented by audit records, such as files accessed, processes executed,
etc. All of them searchable and correlated in the same UI.
::

   | ````
   | ``   __-----------__                 ,----------------------------.``
   | ``  |--___________--|   Queries      |          Browser           |``
   | ``  |               | <------------- |----------------------------|``
   | ``  | ElasticSearch | I/O and audit  | Search I/O and audit       |``
   | ``  |               | -------------> | Playback I/O and audit     |``
   | :literal:`  `--___________--'                | Rewind to time and matches \|`
   | :literal:`                                   `----------------------------'`



Development Plan
================



Phase 0
-------

Phase 0 is completed with a public technology preview release, including
a tlog pre-release and a minor release of SSSD, containing the features
below.

Features
----------------------------------------------------------------------------------------------

-  Local configuration of session recording via SSSD.
-  No integration with FreeIPA.
-  Terminal I/O recording, with logging into the standard syslog
   interface.
-  Only UTF-8 terminal charset is supported for recording.
-  Terminal recording is not disabled under X sessions. The user will be
   able to have at most one terminal session recorded under X sessions.
-  No official jump-server solution. A jump-server setup will likely be
   possible, but a recommended way will not be described.
-  No I/O recording throttling. Users will be able to overwhelm the
   system with excessive terminal input or output. Throttling will still
   be possible at the logging server level.
-  No audit data delivery. Conversion of audit messages to JSON won't be
   implemented.
-  A basic command-line terminal session playback tool will be
   available.



Phase 1
-------

First production-ready release of tlog and minor releases of FreeIPA,
SSSD, and auditd, containing the features below, mark completion of
Phase 1.



Features
----------------------------------------------------------------------------------------------

-  Non-UTF-8 terminal charsets supported by converting them to UTF-8
   within JSON messages.
-  Terminal recording is disabled under X sessions. Graphical sessions
   are detected and terminal I/O recording is not attempted. The
   graphical sessions should be recorded by other means, as a whole.
-  Official jump-server solution described. A HOWTO for recommended
   setup is provided. If necessary, additional tlog and/or SSSD features
   are implemented.
-  I/O recording throttling is configurable and enforced in tlog-rec.
-  Audit data can be converted to JSON and delivered to ElasticSearch.
   Required auditd features are implemented and included into a release.
-  Basic ability to find sessions to playback in FreeIPA. The FreeIPA
   web UI provides a way for finding recorded sessions by minimal
   criteria, such as user and host names.
-  Basic playback and correlation web UI component, embedded into
   FreeIPA UI. The FreeIPA web UI provides a way to playback and rewind
   terminal I/O recordings, synchronized with audit data.



Phase 2
-------

Phase 2 is completed by finishing second production-ready release of
tlog, and major releases of FreeIPA and SSSD, containing the features
below.



Features
----------------------------------------------------------------------------------------------

-  Central configuration of session recording in FreeIPA: who, where and
   how to record. Tlog-rec configuration LDAP entries can be tied to
   HBAC entries in FreeIPA and observed by SSSD.
-  Central configuration includes selection of auditing level: no audit,
   login information, session audit. Basic auditing configuration can be
   tied to HBAC entries in FreeIPA. The specific client-side
   implementation is to be determined.
-  Central configuration in FreeIPA of how long recordings are stored
   before being purged. Interfacing with storage is to be implemented in
   a separate project.
-  Ability to purge recordings via FreeIPA UI. Interfacing with storage
   is to be implemented in a separate project.
-  Central configuration of permissions to access recordings.
   Interfacing with storage is to be implemented in a separate project.
-  Advanced session search features, including more flexible search
   criteria.
-  Advanced playback and correlation web UI, including variable speed
   playback, searching through most of the data and rewinding to
   results.



User stories
----------------------------------------------------------------------------------------------

Specific user stories handled by this phase include the following.

-  As an IdM administrator I would like to be able to define for whom
   session recording should be performed. This depends on the
   combination of a group of users and which host or group of hosts they
   access. Another factor can be the type of session meaning whether it
   is ssh, scp, su or sudo session. I want to be able to control this
   centrally via IdM UI, CLI and API interfaces.
-  As an IdM administrator I want to control the level and/or classes of
   the audit information I am interested in collecting from a system
   during session recording. It is sufficient to start with the
   following options. How the options are presented is an implementation
   detail. Other options might be added as a part of the design and
   investigation of this feature.

   -  “no audit” – no audit information is collected
   -  “login information” - information about session start, stop and
      authentication
   -  “session audit” - record all audit activity associated with the
      session being recorded.

-  As an administrator I want to be able to define policies related to
   session recording data. For example for how long the data is stored
   in the warehouse before it is purged or overwritten. This should be
   manageable centrally via standard IdM interfaces. Ability to trigger
   a purge from UI, CLI and API would be a nice to have feature.
-  As an administrator I would like to define access control policies
   around the recorded sessions, so that only authorized users can
   playback sessions of other users.