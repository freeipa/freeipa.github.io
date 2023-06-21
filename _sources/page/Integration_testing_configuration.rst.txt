Integration_testing_configuration
=================================

Overview
--------

It is necessary to configure the environment according to the procedures
described below, otherwise the integration tests will not work.

To simulate real environment, integration tests are run using multiple
hosts. The configuration ensures that the real environment is properly
simulated. Proper simulation allows testing advanced functionality such
as replication or integration with other products.

The tests are executed by having a *controller machine* establish an SSH
connection with each of the hosts and running commands remotely. The
*controller machine* is the one you run the ipa-run-tests command on,
and ideally should not be one of the hosts that are used for the
testing.

Each test machine must be configured to allow root SSH access, either
via public key or a password. Make sure that root's profile scripts do
not fail for a login session without a TTY attached. (Older RHEL/Fedora
machines invoke ``clear`` from ``/root/.bash_logout``, this will fail
without a TTY. Remove the ``clear`` command before running the tests.)



Configuring the Test Environment
--------------------------------

The configuration of your environment used for the testing can be done
in two ways:

-  `a YAML/JSON configuration
   file <Integration_testing_configuration#Using_YAML.2FJSON_configuration_file>`__
-  `environment
   variables <Integration_testing_configuration#Setting_Environment_Variables>`__



Using YAML/JSON configuration file
----------------------------------------------------------------------------------------------

The configuration file defines a few globals, and a list of *domains*
that consist of *hosts*. Each host entry corresponds to a machine with a
certain set of packages installed. Which packages are expected is
loosely determined by the "role" of the host: typically this is
'master', 'replica', or 'client', which need the appropriate IPA
packages installed; or 'ad', 'ad_subdomain', 'ad_treedomain' which name
a specially configured Active Directory servers (see
`V3 <V3/Integration_testing/AD>`__ or
`V4 <V4/AD_configuration_for_testing>`__ configuration page). Other
roles may be used when needed by individual tests. If not enough hosts
of a needed role are available for a test, that test is skipped.

An example YAML configuration file follows:

| ``ad_admin_name: Administrator``
| ``ad_admin_password: Secret123456``
| ``admin_name: admin``
| ``admin_password: Secret123``
| ``debug: false``
| ``dirman_dn: cn=Directory Manager``
| ``dirman_password: Secret123``
| ``dns_forwarder: 203.0.113.123``
| ``nis_domain: ipatest``
| ``domain_level: 0``
| ``ntp_server: 1.pool.ntp.org``
| ``root_ssh_key_filename: ~/.ssh/id_rsa``
| ``test_dir: /root/ipatests``
| ``domains:``
| ``- name: dom203.ipa.test``
| ``  type: IPA``
| ``  hosts:``
| ``  - name: vm-203.dom203.ipa.test.``
| ``    external_hostname: vm-203.dom203.ipa.test``
| ``    ip: 192.0.2.203``
| ``    role: master``
| ``  - name: vm-204.dom203.ipa.test.``
| ``    external_hostname: vm-204.dom203.ipa.test``
| ``    ip: 192.0.2.204``
| ``    role: replica``
| ``  - name: vm-205.dom203.ipa.test.``
| ``    external_hostname: vm-205.dom203.ipa.test``
| ``    ip: 192.0.2.205``
| ``    role: replica``
| ``  - name: vm-214.dom203.ipa.test.``
| ``    external_hostname: 192.0.2.214``
| ``    ip: 192.0.2.214``
| ``    role: legacy_client_sssd_redhat``
| ``- name: ad.test``
| ``  type: AD``
| ``  hosts:``
| ``  - name: ad1.ad.test.``
| ``    external_hostname: 198.51.100.1``
| ``    ip: 198.51.100.1``
| ``    role: ad``
| ``    username: Administrator``
| ``    password: Secret123``
| ``- name: child.ad.test``
| ``  type: AD_SUBDOMAIN``
| ``  hosts:``
| ``  - name: ad2.child.ad.test.``
| ``    external_hostname: 198.51.100.2``
| ``    ip: 195.51.100.2``
| ``    role: ad_subdomain``
| ``    username: Administrator``
| ``    password: Secret123``
| ``- name: adtree.test``
| ``  type: AD_SUBDOMAIN``
| ``  hosts:``
| ``  - name: ad3.adtree.test.``
| ``    external_hostname: 198.51.100.3``
| ``    ip: 195.51.100.2``
| ``    role: ad_treedomain``
| ``    username: Administrator``
| ``    password: Secret123``

To use the configuration, first install PyYAML

``yum install PyYAML``

then set ``$IPATEST_YAML_CONFIG`` to the name of the YAML file, e.g.

``export IPATEST_YAML_CONFIG=~/ipa-test-config.yaml``

or for a single run,

``IPATEST_YAML_CONFIG=~/ipa-test-config.yaml ipa-run-tests test_integration/test_simple_replication.py``

To use JSON configuration, prepare a JSON file with the same contents
and set ``$IPATEST_JSON_CONFIG`` instead.

To convert between YAML-, JSON- and environment-based configuration, use
the ``ipa-test-config`` command:

| ``ipa-test-config --yaml     # output current configuration as YAML``
| ``ipa-test-config --json     # output current configuration as JSON``
| ``ipa-test-config --global   # output current configuration as environment variables``



Setting Environment Variables
----------------------------------------------------------------------------------------------

For compatibility with existing tests, configuration may be passed via
environment variables. Let's dive into simplest possible
self-explanatory example:

``~/.bashrc``:

``export MASTER_env1=vm-203.dom203.ipa.test``

This environment variable defines a IPA master. The first part of the
variable defines the **role**, and the second part defines the **domain
suffix**. Please note that *vm-203.dom203.ipa.test* should be different
from the machine that we're running the tests on (as mentioned
previously).

Each test requires a minimal number of available resources (we think of
a host with a defined role as of resource - e.g., from previous example,
vm-203.dom203.ipa.test is master resource) that it needs for its run. If
the resource demand is not met, this particular test is skipped.

Other hosts for pre-defined roles (such as REPLICA, AD or CLIENT) can be
defined in a similar way, in *~/.bashrc*:

| ``export MASTER_env1=vm-203.dom203.ipa.test``
| ``export REPLICA_env1=vm-204.dom203.ipa.test vm-205.dom203.ipa.test``

This defines a testing environment with one IPA master and two replicas,
suitable for running e.g. an replication test.

For tests that need to operate with Active Directory, you need to define
an host of AD role. This works as expected and described above. However,
there is one catch, since IPA and AD do not share the same domain,
please make sure to use **different** domain suffixes for IPA master and
AD:

| ``export MASTER_env1=vm-203.dom203.ipa.test``
| ``export REPLICA_env1=vm-204.dom203.ipa.test vm-205.dom203.ipa.test``
| ``export AD_env2=ad.addomain.ipa.test``

Note the usage of **\_env1** and **\_env2** in the example above.

Also, you can use the ADADMINPW environment variable to define the
password of the AD's Administrator's account. (by default, this is set
to Secret123)

| ``export MASTER_env1=vm-203.dom203.ipa.test``
| ``export REPLICA_env1=vm-204.dom203.ipa.test vm-205.dom203.ipa.test``
| ``export AD_env2=ad.addomain.ipa.test``
| ``export ADADMINPW=Secret123456``

Some tests need to operate with machines that have custom configuration
and cannot be referred to as a general client or replica. A particular
example of such a test are legacy client tests, when we need to make
sure, that the client we're using for the testing is indeed a legacy one
(and not an up-to-date Fedora).

To support this use case, such tests require custom roles. To define a
custom role host, you need to define an environment variable that starts
with **TESTHOST\_** prefix (this prefix is what the framework uses to
make a difference between a normal environment variable and a one that
defines a custom role). The rest of the environment variable is
consistent with the examples above, so e.g.:

| ``export MASTER_env1=vm-203.dom203.ipa.test``
| ``export TESTHOST_LEGACY_CLIENT_SSSD_REDHAT_env1=vm-214.dom203.ipa.test``

will create a custom role under the name of "legacy_client_sssd_redhat".

To see what resources a test requires, you can have a peek into its
implementation:

| ``$ vim test_legacy_clients.py``
| ``class TestLegacySSSDBefore19RedHat(BaseTestLegacyClient):``
| ``   advice_id = 'config-redhat-sssd-before-1-9'``
| ``   required_extra_roles = ['legacy_client_sssd_redhat']``

| ``$ vim test_simple_replication.py``
| ``class TestSimpleReplication(IntegrationTest):``
| ``    """Simple replication test``
| ``    Install a server and a replica, then add an user on one host and ensure``
| ``    it is also present on the other one.``
| ``    """``
| ``    num_replicas = 1``

Additionally, if a test is skipped due to the insufficient resources
available, the exception contains information about what resources were
not available.



Further information
-------------------

For more information about the configuration options, see the manual
pages for the ipa-test-config.

``$ man ipa-test-config``