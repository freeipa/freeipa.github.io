Overview
--------

FreeIPA bundles a number of different services and each of them has
certain environment or configuration sensitivity points. Although the
basic goal of FreeIPA is to automate the setup as much as possible,
there are external areas, which are not under its control and need to be
checked that they were configured properly. Additionally, there is
possibility of working configuration crumbling over time due to improper
upgrades or unintentional harmful admin intervention.

The purpose of this design is to introduce the ipa diagnostics tool,
which would be able to assert FreeIPA environment and configuration
conditions which are mandatory for proper functioning of IPA setup. This
is the primary goal.

As a secondary goal, much of the code can be reused to add the
capability of inspecting the FreeIPA environment and providing succint,
anonymyzed information bundle of the FreeIPA deployment.



Use Cases
---------

-  Detecting any misconfigurations and environment problems
-  Log analysis of the running IPA instance for any known problems
-  FreeIPA deployment information extraction

Design
------

.. _design_goals:

Design goals
~~~~~~~~~~~~

-  Ability to gather information and problems from the current instance.
-  Ability to gather information from the remote instances (replicas and
   clients).
-  Clear audit trail for the commands issued by the tool.
-  Simple user interface, tool should be intelligent enough to figure
   out where it is (client/server) and what plugins it can run under
   current privileges.
-  Structured output

.. _tool_high_level_overview:

Tool high-level overview
~~~~~~~~~~~~~~~~~~~~~~~~

To meet the high level goals defined above, the tool needs to be able to
**gather information** from the IPA master, **analyze** it and **preform
additional checks** and **produce the results** both in human-friendly
and machine-friendly format.

To encourage both maintainability and community contributions, all the
non-core parts are highly pluggable. The ipa-diagnosis tool consists of
three main parts:

-  The ipa-diagnosis core (plugin handling, communication with other
   replicas, CLI interface)
-  Reporter plugins (gather information about the particular FreeIPA
   replica)
-  Doctor plugins (performs checks on the particular FreeIPA replica)

The simplified overall structure of the diagnosis tool itself is best
described with the following diagram:

.. figure:: Ipa-diagnose-high-level.png
   :alt: ipa-diagnose-high-level.png

   ipa-diagnose-high-level.png

.. _fetching_reports_from_other_replicas:

Fetching reports from other replicas
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

However, in the scenario above, any Doctors that want to check not only
health of one replica, but health of the whole deployment will not have
necessary information. Consider a Doctor plugin which will try to detect
broken replication agreements. Since replication agreement is
bidirectional, such a plugin would need a list of replication agreements
from both the replicas.

Therefore, a mechanism to fetch the reports from replicas needs to be
introduced. Following diagram describes the simplest case with one
master and one replica:

.. figure:: Ipa-diagnose-simple-fetch.png
   :alt: ipa-diagnose-simple-fetch.png

   ipa-diagnose-simple-fetch.png

However, in case of more complicated replication topology, we cannot be
sure that all IPA replicas are accessible from the IPA master we run the
ipa-diagnosis command on. The only guarantee is that replicas which have
established replication agreements are able to connect to each other.
Each replica which has diagnosis performed on itself as a part of the
health check of the whole deployment, needs to be able to request
diagnosis results from other replicas, and convey that information back
to the original master which initiated the check.

In the non-ideal case, the health check of the deployment might look as
in the illustration below:

.. figure:: Ipa-diagnose-complex-fetch.png
   :alt: ipa-diagnose-complex-fetch.png

   ipa-diagnose-complex-fetch.png

Please note, as the example above suggest, that ipa-diagnose is not
limited to IPA replicas, but it can also check health of IPA clients.

.. _connection_fallbacks:

Connection fallbacks
~~~~~~~~~~~~~~~~~~~~

In general, ipa-diagnose will use admin's credentials (kerberos ticket)
to SSH into other hosts. In case of infrastructure failure (KDC not
being available), will downgrade through other auth mechanisms down to
the interactive password authentication.

Implementation
--------------

.. _implementation_goals:

Implementation goals
~~~~~~~~~~~~~~~~~~~~

-  Provide pluggable API for drop-in Doctors and Reporters
-  Tool should be resiliant to plugin failures and missing dependencies

.. _reporter_plugin:

Reporter plugin
~~~~~~~~~~~~~~~

Each reporter is a small modular plugin implementing a simple interface,
which aims to provide single or multiple pieces of information about the
system.

.. _doctor_plugin:

Doctor plugin
~~~~~~~~~~~~~

Each doctor is a small modular plugin which consumes information
produced by the reporters, and performs additional checks on the system.

.. _report_format:

Report format
~~~~~~~~~~~~~



Feature Management
------------------

UI
~~

Both CLI and WebUI interface is available. Web User interface is
provided as a plugin to the Cockpit project.
`1 <http://cockpit-project.org/>`__. Optionally it can be embedded from
the FreIPA WebUI.

CLI
~~~

Overview of the CLI commands. Example:

============ ====================
Command      Options
============ ====================
ipa-diagnose [--help]
\            [--whole-deployment]
\            [--hosts-only]
============ ====================

Configuration
~~~~~~~~~~~~~

Upgrade
-------

There is no impact on upgrades, diagnostics plugins should be able to
work with multiple versions of underlying FreeIPA packages.

.. _how_to_test17:

How to Test
-----------

N/A

.. _test_plan17:

Test Plan
---------

Given the nature of the tool, it should be covered by integration tests,
which would break/misconfigure IPA in particular way, and detect,
whether ipa-diagnose can properly detect / advise / fix the issue.

Author
------

`Tomas Babej <User:Tbabej>`__
