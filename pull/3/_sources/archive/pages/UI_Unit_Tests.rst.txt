Overview
========

The UI unit tests are used to validate the basic functionalities of the
the client-side components of the IPA UI. The automation is implemented
using QUnit framework. The tests are limited to functionalities that can
be tested programmatically via JavaScript/JQuery. Other aspects of the
UI will still require a visual validation.

.. _running_unit_tests:

Running Unit Tests
==================

The test files are stored in IPA's git repository. To checkout IPA
source code execute the following command:

::

   git clone git://git.fedorahosted.org/git/freeipa.git

.. _necessary_steps_for_running_tests:

Necessary steps for running tests
---------------------------------

#. Run ``autoreconf`` command in the root of the repository
#. Run ``./configure`` in the root of the repository
#. Go to ``install/ui/libs/`` directory
#. Run ``make`` - this step generates loader.js which is necessary for
   load current api version into WebUI. This version is necessary for
   checking response of each API call.
#. Go to ``install/ui/test``
#. Run ``firefox index.html``

Only Firefox browser is supported, because Google Chrome has stricter
checks for cross domain requests (does not allow to fetch files using
file:// protocol).

To run all tests at once click "Complete Test Suite". To run each
component individually, click one of the available test suites:

-  Core Test Suite
-  Entity Test Suite
-  Details Test Suite
-  Association Test Suite
-  Navigation Test Suite

The results for each test case will be shown next to the test case
description: (, , ).

The results for each validation step can be viewed by clicking the test
case description.

The summary of the results will be displayed at the bottom of the page.

.. _writing_unit_tests:

Writing Unit Tests
==================

All test files are stored in the install/static/test folder.

.. _test_data:

Test Data
---------

Due to browser's security restriction, AJAX can only retrieve files in
the same folder or any sub folder underneath it. Currently all JSON test
data are now stored in install/static/test/data, but future test cases
may need separate folders.

JSON test data can be created by capturing the output of the API invoked
using curl. The following example invokes user-find command:

::

   curl\
     -H "Content-Type:application/json"\
     -H "Accept:applicaton/json"\
     --negotiate -u :\
     --cacert /etc/ipa/ca.crt\
     -d '{"method":"user_find","params":[[""],{}],"id":0}'\
     -X POST\
     https://dev.example.com/ipa/json

The output should be stored in a file named according to the command
name (e.g. user_find.json).

.. _test_script:

Test Script
-----------

Test cases are stored in a JavaScript file (e.g. ipa_tests.js). Each
test case is usually defined as follows:

::

   test(<description>, <implementation>);

The implementation is a function which prepares the environment,
executes the operation to be tested, then performs validations, and
cleans up if necessary.

See the following example:

::

   test("Testing ipa_init().", function() {

       expect(1);

       ipa_ajax_options["async"] = false;

       ipa_init(
           "data",
           true,
           function(data, text_status, xhr) {
               ok(true, "ipa_init() succeeded.");
           },
           function(xhr, text_status, error_thrown) {
               ok(false, "ipa_init() failed: "+error_thrown);
           }
       );
   });

The expect() is used to specify the number of validations to be executed
in this test case.

The ok() is used to perform validation. Other validation functions are
equals() and same().

See QUnit documentation for more info.

.. _test_page:

Test Page
---------

The test cases are executed by opening the test page (e.g.
ipa_tests.html) in the browser. The test page should load all necessary
libraries used by the test script and the test script itself.

See the following example:

::

   <!DOCTYPE html>
   <html>
   <head>
       <title>Core Test Suite</title>
       <link rel="stylesheet" href="qunit.css" type="text/css" media="screen">
       <link rel="stylesheet" type="text/css" href="../jquery-ui.css" />

       <script type="text/javascript" src="../js/libs/loader.js"></script>
       <script type="text/javascript" src="qunit.js"></script>
       <script type="text/javascript" src="../jquery.js"></script>
       <script type="text/javascript" src="../jquery-ui.js"></script>
       <script type="text/javascript" src="../ipa.js"></script>
       <script type="text/javascript" src="ipa_tests.js"></script>
   </head>
   <body>
       <h1 id="qunit-header">Core Test Suite</h1>
       <h2 id="qunit-banner"></h2>
       <div id="qunit-testrunner-toolbar"></div>
       <h2 id="qunit-userAgent"></h2>
       <ol id="qunit-tests"></ol>
       <div id="qunit-fixture"></div>
   </body>
   </html>

.. _complete_test:

Complete Test
-------------

The all_tests.html is similar to a regular test page, but it includes
all test scripts and all required libraries. This page can be used to
quickly check for regressions.

.. _test_main_page:

Test Main Page
--------------

The index.html contains references to all available test suites. New
test suite should be added to the list below:

::

   <div id="content">
       <a href="all_tests.html">Complete Test Suite</a>
       <ul>
       <li><a href="ipa_tests.html">Core Test Suite</a>
       <li><a href="entity_tests.html">Entity Test Suite</a>
       <li><a href="details_tests.html">Details Test Suite</a>
       <li><a href="association_tests.html">Association Test Suite</a>
       <li><a href="navigation_tests.html">Navigation Test Suite</a>
       </ul>
   </div>

References
==========

-  `QUnit <http://docs.jquery.com/Qunit>`__
-  `Development with jQuery &
   Qunit <http://www.swift-lizard.com/2009/11/24/test-driven-development-with-jquery-qunit/>`__
-  `Talking to FreeIPA JSON web API via
   curl <http://adam.younglogic.com/2010/07/talking-to-freeipa-json-web-api-via-curl/>`__
-  `JavaScript Coding
   Standards <https://fedorahosted.org/freeipa/wiki/Javascript_Coding_Standards>`__
