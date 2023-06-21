Testing
=======



Quick Start
-----------

IPA provides an ipatests package which contains all of the tests and
tools needed to run the unit and IPA integration tests.

Requirements for 100% test passing:

-  --setup-dns option must be passed to ipa-server-install (or run
   ipa-dns-install afterward)
-  freeipa-server-trust-ad must be installed (AD trust is not necessary)

Setup
----------------------------------------------------------------------------------------------

The tests allow customization through the use of a required, local
configuration directory, ~/.ipa. This can be created with:

| ``$ mkdir ~/.ipa``
| ``$ ln -s  /etc/ipa/default.conf ~/.ipa/default.conf``
| ``$ ln -s /etc/ipa/ca.crt ~/.ipa/ca.crt``

**Note:** you can instead copy those files but you'll need to update
them again if re-install IPA.

To run the whole integration test suite, run the following:

| ``$ kinit admin``
| ``$ ipa-run-tests``

For additional options or to run the integration tests see below.

The certificate test *test_0027_sizelimit_zero* will fail after the
first execution of ipa-run-tests. It makes assumptions about the number
of certificates issued. New certificates are issued as part of test
execution which it is not expecting.



For Developers
--------------



Fast test
~

Often during development phase one needs to quickly check whether Python
code is OK and whether basic tests (which don't require access to IPA
API) are working. To do so, freeIPA 4.6.2 introduces 'fasttest' target:

``$ make fasttest``

The 'fastlint' target allows to quickly check pylint of modified Python
files and pycodestyle checks of code diff. The target automatically
determinates difference between current working tree and the origin
branch (e.g. master or ipa-4-6):

``$ make fastlint``

For convenience the 'fastcheck' runs 'fasttest' and 'fastlint' with
Python 2 and 3 in one go. A fastcheck takes about half a minute to a
minute to execute:

``$ make fastcheck``

Note: in order to install all the dependencies for fastcheck, you can
run

| ``$ sudo dnf builddep -b -D "with_python3 1" -D "with_wheels 1" -D "with_lint 1" --spec freeipa.spec.in --allowerasing``
| ``$ ./autogen.sh``
| ``$ make fastcheck``



Full test run
^^^^^^^^^^^^^

You can run the full tests from inside a source tree, but some template
files must be generated first.

``$ make version-update``

You also need to run tests with admin ticket:

``$ kinit admin``

Run the test script:

``./make-test``

This command executes all the tests in ipatests folder.

You can also use this to run specific tests. Here is how you would test
the user plugin:

``./make-test ipatests/test_xmlrpc/test_user_plugin.py``

Or you can run only tests from specific test class...

``./make-test ipatests/test_xmlrpc/test_user_plugin.py::TestUser``

... or even one particular test case (applies for non-declarative tests
only):

``./make-test ipatests/test_xmlrpc/test_user_plugin.py::TestUser::test_retrieve``

Note that some of the tests make certain assumptions about the data on
the server. Some tests, for example, pull all entries and expect a
certain number to be returned and may raise an error.



In-tree testing
----------------------------------------------------------------------------------------------

It is possible to execute the tests inside the tree rather than from the
packaged files. This can make it faster to develop tests or test changes
without having to do a full rebuild, install cycle.

Install
^^^^^^^

You need to have already executed ipa-server-install. This provides the
Kerberos and LDAP servers needed for IPA testing.

Configure
^^^^^^^^^

You also need to configure IPA for your local installation.

When running the server in-tree we will use ~/.ipa/ instead of /etc/ipa
to look for configuration files. You need to create ~/.ipa/default.conf.
You can copy this from /etc/ipa/default.conf.

| ``$ mkdir -p ~/.ipa``
| ``$ cp /etc/ipa/default.conf ~/.ipa/``
| ``$ cp /etc/ipa/ca.crt ~/.ipa/``



Run the tests
'''''''''''''

| ``$ kinit admin``
| ``$ PYTHONPATH=. python3 ipatests/ipa-run-tests``



Optional tests
''''''''''''''

-  In order for the certificate tests to pass you'll also need to copy
   the Dogtag agent certificate and private key from /var/lib/ipa/. Be
   sure to change ownership of these files too. That should do it.

| ``$ cp /var/lib/ipa/ra-agent.{pem,key} ~/.ipa/``
| ``$ chown $USER ~/.ipa/ra-agent.{pem,key}``

-  To test the ldap updater you need to store password for Directory
   Managed to ~/.ipa/.dmpw file.



Lite server
----------------------------------------------------------------------------------------------

The lite-server is a lightweight WSGI server that can be used to
simplify web framework debugging in the source tree. Lite server info
can be found `here <http://www.freeipa.org/page/Testing/Lite_server>`__.



Remote testing
--------------

You can also test against an IPA installation on another machine, it
just requires a bit more configuration.

You first need to update ~/.ipa/default.conf to point to the remote
machine. My test machine is ipa.example.com, here is my configuration:

| ``[global]``
| ``domain=example.com``
| ``realm=EXAMPLE.COM``
| ``basedn=dc=example,dc=com``
| ``server=ipa.example.com``
| ``enable_ra=True``
| ``xmlrpc_uri=``\ ```https://ipa.example.com/ipa/xml`` <https://ipa.example.com/ipa/xml>`__

If you don't want your development machine to be enrolled as a client of
the remote IPA master you can grab the remote krb5.conf and use that:

| ``$ scp ipa.example.com:/etc/krb5.conf lion-krb5.conf``
| :literal:`$ export KRB5_CONFIG=`pwd`/lion-krb5.conf`
| ``$ kinit admin``

Finally you need to retrieve the CA from the IPA master and put it into
~/.ipa/ca.crt

``$ wget  -O ~/.ipa/ca.crt. ``\ ```http://ipa.example.com/ipa/config/ca.crt`` <http://ipa.example.com/ipa/config/ca.crt>`__

Now you should be good-to-go to run the XML-RPC tests against a remote
server.



Web UI testing
--------------

Web UI testing is covered by `unit tests <FreeIPAv2:UI_Unit_Tests>`__
and `integration tests <Web_UI_Integration_Tests>`__.



Integration tests
-----------------

Integration tests test IPA installations in multiple configurations
across potentially multiple virtual machines.



Install
----------------------------------------------------------------------------------------------

To run the `integration tests <V3/Integration_testing>`__ you need to
have the pythonX-ipatests package installed.

``# dnf install python2-ipatests``

or (preferred):

``# dnf install python3-ipatests``

All the files containing actual test implementations are located in the
*$PYTHON_SITELIB/ipatests/test_integration/* directory and start with a
*test\_* prefix.

Configuration
----------------------------------------------------------------------------------------------

To properly configure the environment, see `integration testing
configuration page <Integration_testing_configuration>`__.

Particularly, the configuration of your environment used for the testing
can be done in two ways:

-  `a YAML/JSON configuration
   file <Integration_testing_configuration#Using_YAML.2FJSON_configuration_file>`__
-  `environment
   variables <Integration_testing_configuration#Setting_Environment_Variables>`__



Run tests
----------------------------------------------------------------------------------------------

To run the whole integration test suite, run the following:

``$ ipa-run-tests``

To run only tests from a specific file, run the following:

``$ ipa-run-tests test_integration/test_simple_replication.py``

To run tests from specific class, run:

``$ ipa-run-tests test_integration/test_simple_replication.py::TestSimpleReplication``

To run only one specific test, run:

``$ ipa-run-tests test_integration/test_simple_replication.py::TestSimpleReplication::test_user_replication_to_master``

Please note that you need to specify a whole path **relative** to the
python's *site-packages/ipatests/* directory.