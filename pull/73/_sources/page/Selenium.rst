Selenium
========

Overview
========

This document describes the process to execute and develop test cases
for IPA Web UI using Selenium.

This document assumes that IPA source code is checked out in IPA_SRC
directory and IPA Server is installed on the same machine.

Web UI test files can be found in IPA_SRC/install/ui/test. There are
several subfolders:

-  bin: This folder contains test scripts.
-  conf: This folder contains test configurations.
-  lib: This folder contains test libraries.
-  functional: This folder contains the test cases and suites. Test
   suite file names follow this pattern: -suite.html.
-  results: This folder contains test results. Test results file names
   follow this naming pattern: -results.html.



Installing Selenium Server
==========================

Download `Selenium Server <http://seleniumhq.org/download/>`__ and put
the selenium-server-standalone-xxx.jar file in /usr/share/java. Create a
symbolic link to that file:

::

   % ln -s selenium-server-standalone-2.0b3.jar selenium-server-standalone.jar



Configuring Firefox
===================

The following steps should be done while Firefox is running with the
default profile. This is to make sure the tests can run continuously
without any intervention.

To open Firefox's profile manager:

::

   % firefox -ProfileManager -no-remote



Importing IPA CA Certificate
----------------------------

Import /etc/ipa/ca.crt into Firefox.



Importing Selenium CA Certificate
---------------------------------

Extract cybervillainsCA.cer into a temporary folder:

::

   % mkdir tmp
   % cd tmp
   % jar xvf /usr/share/java/selenium-server-standalone.jar sslSupport/cybervillainsCA.cer

Import sslSupport/cybervillainsCA.cer into Firefox. The temporary folder
can be removed now.



SSL Certificate
---------------

Open http://localhost. Accept and store all untrusted IPA certificates
if asked.



Disabling Add-on Notification
-----------------------------

Open about:config in Firefox, then add the following preference:

-  Name: extensions.newAddons
-  Type: boolean
-  Value: false



Running Tests using Selenium Server
===================================

Selenium Server can be used to run the tests from command line.

Close all Firefox instances, then run one of the following scripts.
These scripts will launch a new Firefox instance with a new profile
copied from the default Firefox profile. There will be 2 Firefox
windows, one for the Selenium, and the other for IPA Web UI. Any
configuration changes made to this instance will not be saved into the
default Firefox profile.

functional.sh
-------------

The functional.sh can be used to run some or all functional test cases.

::

   % cd IPA_SRC/install/ui/test
   % bin/functional.sh [<test suite names>...]

The test suite names should correspond to the test suite files in
IPA_SRC/install/ui/test/functional folder. The names should be separated
by spaces, e.g. user group host. The results are going to be stored in
the IPA_SRC/install/ui/test/results folder.

If no test suite names are specified, this script execute all functional
test suites.

selenium.sh
-----------

The selenium.sh will execute the specified test suite and store the
results in the specified path.

::

   % cd IPA_SRC/install/ui/test
   % bin/selenium.sh <test suite path> <test results path>

The is the path to the HTML test suite. The is the path for the output.



Running Tests using Selenium IDE
================================

Install `Selenium IDE <http://seleniumhq.org/download/>`__ in Firefox.
Open Selenium IDE, then open a test suite (\*-suite.html).



Writing Tests using Selenium IDE
================================

References
==========

-  `Python bindings for
   Selenium <http://pypi.python.org/pypi/selenium>`__
-  `Creating Firefox profile for your Selenium RC
   tests <https://girliemangalo.wordpress.com/2009/02/05/creating-firefox-profile-for-your-selenium-rc-tests/>`__