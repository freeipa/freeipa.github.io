Integration_testing
===================

\__NOTOC_\_

Overview
========

Ticket: `#3621 <https://fedorahosted.org/freeipa/ticket/3621>`__
Integrated Automation Testing.

Make it possible to write and run multi-host integration tests (such as:
install master & replica, add user on replica, verify it's added on
master).

These tests will be run from continuous integration. Any developer can
also run them manually.



Use Cases
=========



Continuous integration
----------------------

The developer team at Red Hat will run a Jenkins continuous integration
server that will run the tests automatically (after each commit if
resources are available).

The CI results will be posted publicly.



Developer testing
-----------------

Anyone is be able to run integration tests without advanced
infrastructure, only a number of virtual machines to run the tests on is
needed.



Beaker integration
------------------

The tests will run seamlessly inside
`Beaker <http://beaker-project.org/>`__/`RHTS <https://fedoraproject.org/wiki/QA/RHTS>`__.
A special option enables reporting via BeakerLib.



Non-goals
=========

A complete testing/continuous integration setup needs some steps that
will not be included in IPA's test suite:

-  Building the code
-  VM provisioning

-  Configuring the basic system, installing the packages

Design
======

The Python package with the IPA test suite is renamed to ``ipatests``,
and packaged for RPM-based systems as ``freeipa-tests``. Eventually the
package will be included in Fedora.

Integration tests will be controlled from a single machine, and executed
on a number of "remote" machines that act as servers, replicas, clients,
etc. The controlling machine communicates with the others via the SSH
protocol. (The controlling machine may be the same as one of the
"remote" ones.)

Integration tests are included in the main IPA set suite, and configured
using environment variables. If the variables are missing, all
integration tests are skipped. If an insufficient number of hosts is
configured for a test, the individiual test will be skipped.

A tool is provided to run installed tests.

The remote machines used for integration testing are required to have
relevant IPA packages installed, firewall opened up, any needed
workarounds applied (RPM downgrades, SELinux mode,...), and sshd set up
to allow root login. The test runner will connect to these machines,
install IPA, perform the tests, and then uninstall IPA & return the
systems to their previous state.

A plugin for integration with BeakerLib is provided.



Test configuration
==================

Tests are configured using these environment variables. For any
additional information, see man ipa-test-config.



Host configuration
------------------

$MASTER
   FQDN of the first IPA server
$REPLICA
   FQDNs of other IPA servers (space-separated)
$CLIENT
   FQDNs of IPA clients (space-separated)
$MASTER_env2, $REPLICA_env2, $CLIENT_env2, $MASTER_env3, ...
   can be used for additional domains when needed
$BEAKER_env, e.g. $BEAKERREPLICA1_env1
   the external hostname of the given host; the test framework will
   connect using this name
   Default: same as the internal FQDN
$BEAKER_IP_env, e.g. $BEAKERREPLICA1_IP_env1
   the IP address of the given host
   Default: resolved via ``gethostbyname`` (or DNS if $IPv6SETUP is set)
$AD_env1, $AD_env2, $AD_env3, $AD_env4, ...
   can be used to define Active Directory domains. Please note that
   these
   domains are not treated as separate from the IPA domains, so please
   use an
   unique environment suffix for each of your Active Directory domains,
   e.g. MASTER_env1 and AD_env2

DNS needs to be set up so that IP addresses can be obtained for these
hosts.



Additional roles
----------------------------------------------------------------------------------------------

You can define any additional custom roles that are required by the
tests using the following environment variables.

$TESTHOST__env, e.g. $TESTHOST_LEGACY_CLIENT_env1
   FQDN(s) of the -th machine with the extra role
$BEAKER_env, e.g. $BEAKERLEGACY_CLIENT_env1
   the external hostname(s)
$BEAKER_IP_env, e.g. $BEAKERLEGACY_CLIENT_IP_env1
   IP address(es) of the machine of the extra role

If multiple hosts of the same role are needed, separate the values with
a space, e.g.
``TESTHOST_LEGACY_CLIENT_env1='lc1.ipa.example.com lc2.ipa.example.com'``.

The role name may not end with a number.



Basic configuration
-------------------

$IPATEST_DIR
   Directory for test data on the remote hosts
   Default: /root/ipatests
$DNSFORWARD
   IP of a DNS forwarder
   Default: 8.8.8.8
$IPA_ROOT_SSH_PASSWORD
   root password for the remote machines
   Used if ``$IPA_ROOT_SSH_KEY`` is not set.
$IPA_ROOT_SSH_KEY
   name of a file containing the private RSA key for root on the remote
   machines
   Default: ~/.ssh/id_rsa



Test customization
------------------

$DOMAIN
   IPA domain name
   Default: taken from $MASTER
$NISDOMAIN
   NIS domain name
   Default: ipatest
$NTPSERVER
   NIS domain name
   Default: ipatest
$IPv6SETUP
   Set to TRUE for IPv6-only connectivity
$IPADEBUG
   Set to enable test debugging

$ADMINID
   Admin username
   Default: admin
$ADMINPW
   Admin user password
   Default: Secret123
$ADADMINID
   Active Directory Administrator username
   Default: Administrator
$ADADMINPW
   Active Directory Administrator password
   Default: Secret123
$ROOTDN
   Directory manager DN
   Default: cn=Directory Manager
$ROOTDNPWD
   Directory manager password
   Default: Secret123



Supporting tools
================



ipa-test-config
---------------

This tool reads the configuration variables above and outputs a Bash
script that sets a much more complete set of variables for easy
shell-based testing or test set-up.

Without arguments, ``ipa-test-config`` outputs information specific to
the host it is run on. When given a hostname, it prints config for that
host. With the ``--global`` flag, it outputs configuration common to all
hosts.



ipa-run-tests
-------------

This tool is a wrapper arount ``nosetests`` and accepts the same
arguments as Nose. It loads any additional plugins and runs tests from
the system-installed IPA test suite.



ipa-test-task
-------------

Run a test task, respecting to the configuration variables listed above.

The tasks this tool can perform are:

-  ``install-topo``: install IPA on the configured hosts in a chosen
   topology
-  ``list-topos``: list available topologies for ``install-topo``
-  ``install-master``: install an initial IPA master *(invokes
   ``ipa-server-install`` on the master)*
-  ``install-replica``: install a IPA replica *(invokes
   ``ipa-replica-install`` on a given replica or all replicas)*
-  ``install-client``: install a IPA client *(invokes
   ``ipa-client-install`` on a given client or all clients)*
-  ``uninstall-server``: uninstall an IPA master or replica (invokes
   ``ipa-server-install --uninstall`` on a given server or all
   servers)''
-  ``uninstall-client``: uninstall an IPA client *(invokes
   ``ipa-client-install --uninstall`` on a given client or all clients)*
-  ``uninstall-all``: uninstall all servers and clients + clean up
-  ``connect-replica``: connect two IPA replicas *(invokes
   ``ipa-replica-manage connect`` on a given replica)*
-  ``disconnect-replica``: disconnect two IPA replicas *(invokes
   ``ipa-replica-manage disconnect`` on a given replica)*
-  ``cleanup``: clean up a host -- restore hostname, resolv.conf, etc.
   *(this is also called by the ``uninstall-*`` subcommands)*
-  ``others``: see man ipa-test-task for the full list

Like ``ipa-run-tests``, the tool should be run from a dedicated machine
(or "master").

The ``--with-beakerlib`` option turns on BeakerLib logging, similar to
``ipa-run-tests``.

Implementation
==============

Test cases are implemented as Nose test classes, with
installation/uninstallation as class setup/teardown.

A BeakerLib plugin is provided that starts/ends Beaker phases for Nose
test contexts and cases, issues a Beaker assertion (rlPass/rlFail) for
each test case, and collects and submits relevant logs.

A separate plugin will be provided to collect logs outside of a Beaker
environment.



Ordering of the tests
=====================

The Nose test classes are by default executed in the alphabetical order.

The test methods within each class are executed in the order they were
defined (the integration testing framework makes use of OrderedTests
nose plugin that achieves this). This gets more complicated with
inheritance. To allow creating new test classes by overriding a few
selected method of the parent class, the methods within one class (this
includes both over-ridden and inherited methods) are executed in the
order of the classes they were first defined in the inheritance chain.
Methods that were introduced in the same class, are executed in the
order they were defined within that class.



Example instructions
====================

To run the test called ``test_integration/test_simple_replication.py``,
which needs to run with two masters, follow these instructions.

Install the IPA server packages on two machines, and do any preparations
necessary to install IPA (e.g. configure/disable the firewall).

Then, install the ``freeipa-tests`` package on the machine that will run
the tests (this may be one of the machines above, or preferably a
different one). Set MASTER and REPLICA environment variables to the
fully qualified hostnames of the two machines prepared earlier. Also set
any other relevant variables listed in `Test
configuration <#Test_configuration>`__. You may run
``ipa-test-config --global`` to verify how the test configuration will
be handled.

The next steps depend on whether the test will run within a BeakerLib
session or not.



With BeakerLib
--------------

Set up a BeakerLib test (e.g. ``rlJournalStart``), and run:

``   ipa-run-tests --with-beakerlib --no-skip test_integration/test_simple_replication.py``

The output is somewhat messy as BeakerLib logs are printed to standard
error. Not that output from external hosts is buffered, so installation
may appear hung.

Archive any relevant data (e.g. with ``rlJournalPrintText``), and end
the BeakerLib session (``rlJournalEnd``).



Without BeakerLib
-----------------

Run:

``   ipa-run-tests test_integration/test_simple_replication.py``

As with other Nose tests, no output is shown for test setup
(installation) if nothing goes wrong, so there may be a long time
without output. A summary is printed at the end of the test run.



Feature Managment
=================

UI

N/A

CLI

See above



Major configuration options and enablement
==========================================

See instructions above.

Replication
===========

N/A



Updates and Upgrades
====================

N/A

(Note: The tests can theoretically be used to drive hosts with other
versions of IPA packages to test backward/forward compatibility.)

Dependencies
============

The freeipa-test package will depend on some libraries that are already
used for unit tests and other test-related tasks:

-  python-nose
-  python-paste
-  python-coverage
-  python-polib

Integration testing brings in a dependency on a library for the SSH
protocol:

-  python-paramiko

Naturally, the new dependencies are not needed in a production
environment.



External Impact
===============

Cooperation with QE is underway.



AD integration
==============

A subpage dedicated to the AD integration testing can be found here:
`V3/Integration_testing/AD <V3/Integration_testing/AD>`__



Design author
=============

`Petr Viktorin <User:pviktorin>`__