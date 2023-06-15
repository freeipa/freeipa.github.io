.. _amd_modules_and_web_ui_build:

AMD modules and Web UI build
============================

Overview
--------

One of the most recommended optimization techniques for web pages are
reduction of the number of http request and a minimization of JavaScript
file size. Web UI uses a lot of JavaScript files. These files can be
concatenated into a single file and then optimized by JavaScript
compiler which strips comments, white-spaces and does other
optimizations.

Design
------

The main idea is transform FreeIPA JavaScript files into AMD modules and
build them with a builder into single file, then the file will be
further optimized by JavaScript compiler.

.. _introducing_dojo:

Introducing Dojo
~~~~~~~~~~~~~~~~

This feature has two topics which are closely related: AMD modules with
AMD loader, and a builder. A loader from a dojo/dojo library is used as
a loader and a build tool from dojo/util library as a builder. Including
source codes of both libraries in FreeIPA repository is not desired
because of their number and size. Therefore FreeIPA git repository
contains only their compiled versions. The builder and a Dojo library
are both built by the builder itself.

Following tools were developed to simplify maintenance of Dojo
Libraries:

-  util/prepare-dojo.sh - can checkout dojo/dojo or dojo/util from
   github repositories to a directory outside FreeIPA directory. Then it
   can apply custom patches on a builder and make symbolic links
   src/dojo and src/build which points to the check-outed repositories.
-  util/make-dojo.sh - creates custom build of dojo/dojo. Requires
   util/prepare-dojo.sh to be run before. Then it calls compile.sh to
   minimize the build.
-  util/make-builder.sh - creates custom build of dojo/util/build
   (builder). Requires util/prepare-dojo.sh to be run before. Then it
   calls compile.sh to minimize the build.
-  util/compile.sh - compiles a build made by a builder using a
   uglify.js compiler. Usually no need to use it alone.

Uglify.js
~~~~~~~~~

Builder transforms AMD packages into single or more files and does basic
minimization like stripping of comments. For further minimization a
proper JavaScript compiler has to be used. Uglify.js was chosen because
of it's capabilities and also because it can be run in rhino environment
and thus to avoid problems of not having a common compiler on targeted
platforms. For easier usage a wrapper scripted was developed:
*util/uglifyjs/uglify SOURCE TARGET*. It will be better to run under
node.js when all target platforms will support it.

.. _building_freeipa:

Building FreeIPA
~~~~~~~~~~~~~~~~

Web UI JavaScript files can be divided into three sets: third party
libraries, FreeIPA code and Dojo library. Lets call them layers. FreeIPA
now uses 4 third party libs. 2 of them are already compiled and the rest
has small size. No bigger optimization techniques are required here.
Dojo is already prepared for optimization. The focus of optimization is
therefore on FreeIPA layer.

.. _amd_module_wrapping:

AMD module wrapping
~~~~~~~~~~~~~~~~~~~

Dojo/util/build tool (builder) is used as a builder of both FreeIPA and
Dojo layer. The builder works best with AMD modules. Full modification
of FreeIPA layer into AMD modules would require splitting files by
component and redefining a lot of dependencies which is a huge tasks.
Different approach is taken to avoid this problem at the moment:
Existing files are only wrapped with AMD module definition. They should
be converted to proper modules - with single purpose and clean
dependencies, gradually in a future. Same applies on Web UI tests.

Build
~~~~~

A build of FreeIPA layer is done by calling *util/make-ui.sh*. This
script is run at make-all phase of an RPM build. Make-ui.sh is not
checking errors during the build because we don't provide dojo/dojo
source codes and thus there are dependency errors. Dojo builder can't be
configured to ignore certain dependency errors and also can't be built
against already built library. The layer is built even though there are
dependency errors, it's not a blocker but it will be nice to fix it in a
future (in dojo upstream or by custom patch).

.. _new_directory_structure:

New directory structure
~~~~~~~~~~~~~~~~~~~~~~~

Web UI directory structure was change because the old one didn't suffice
new needs which are:

-  separate layers into own directories - for AMD
-  easy switch between source code version and built version during
   testing

Following directories in a ui directory were created:

-  src - contains all source codes. Special pages (login, logout, reset
   password) JS code was not moved yet. Not used in production.
-  js - a production directory. At production it should contain built
   layers, at development symbolic links to related folders in src or
   build directories.
-  release - working dir at build phase
-  build - contains output of make-ui.sh and make-dojo.sh scripts.

All FreeIPA layer JS sources were move to src/freeipa folder. Libraries
(jquery, jquery ui, bbq, json2) were moved to src/libs.

.. _development_profiles:

Development profiles
~~~~~~~~~~~~~~~~~~~~

Several approaches of debugging of UI can be considered when developing
UI. Developer can either use a machine with installed FreeIPA or he can
start Web UI from a development directory. When the latter case is used,
the Web UI uses JSON files stored in test/data directory which simulates
JSON-RPC communication. Because there is no communication with server,
we can call this approach an 'offline version'.

.. _offline_version:

Offline version
^^^^^^^^^^^^^^^

Offline version uses FreeIPA layer source codes and built version of
dojo/dojo library by default. It can use built version of FreeIPA layer
or source codes of dojo/dojo library by changing of symbolic links in
*js* directory. To make this task easier, an 'util/change-profile.sh'
script was developed. The script can also set \`git update-index
--assume-unchanged\` to prevent git from notifying about this change.

.. _test_server:

Test server
^^^^^^^^^^^

Different approach must be applied when using test server for debugging.
In this case source codes or built layers must be copied to
*/usr/share/ipa* directory on the test server. To make the task easier
*util/sync.sh* script was created. Developer can specify which parts of
the UI in which form he wants to copy. The tool contains directory
mappings so developer doesn't have think about where source codes or
build versions are placed. Look to Use Cases chapter or tool's help for
more information.



Use Cases
---------

Users
~~~~~

None

Developers
~~~~~~~~~~

Note: all commands are run from install/ui directory of FreeIPA source
dir.

.. _make_new_freeipa_layer_build:

Make new FreeIPA layer build
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  only useful for debugging. It's done automatically in make all phase
   of RPM build.
-  run $ util/make-ui.sh

.. _set_environment_to_debug_source_codes_of_freeipa_layer_using_offline_version:

Set environment to debug source codes of FreeIPA layer using offline version
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  it's the default profile after checkout
-  to switch back from other profile:

   -  $ util/change-profile.sh -p source

-  open index.html by a browser using file:// protocol:

   -  file:///home/login/path-to-freeipa/freeipa/install/ui/index.html
   -  when using Chrome, run it with --disable-web-security option
      otherwise XHR won't work

.. _set_environment_to_debug_built_freeipa_layer_using_offline_version:

Set environment to debug built FreeIPA layer using offline version
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  $ util/change-profile.sh -p compiled
-  open browser same way as in previous use case
-  note: this doesn't create the compiled version, you have to run it
   separately

.. _copy_source_codes_of_freeipa_layer_on_test_server:

Copy source codes of FreeIPA layer on test server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  $ util/sync.sh --host root@test.example.com --freeipa
-  you can replace root with any user who can write into /usr/share/ipa
-  add --clean option if you want to delete all files from target dir
-  run $ util/sync.sh --help to get more information about others
   folders (images, root dir, libs) and option shortcuts

.. _copy_built_freeipa_layer_on_test_server:

Copy built FreeIPA layer on test server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  $ util/sync.sh --host root@test.example.com --freeipa --compiled

.. _update_internal.py_strings_for_web_ui_on_test_server:

Update internal.py (strings for Web UI) on test server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  $ util/sync.sh --host root@test.example.com --strings --restart
-  note: --restart restarts httpd on test server (systemctl restart
   httpd.service). Required for changes to take effect.

.. _clone_dojo_repositories:

Clone Dojo repositories
^^^^^^^^^^^^^^^^^^^^^^^

-  should be run only once
-  required only for Dojo and Builder build
-  $ util/prepare-dojo.sh --all

   -  creates dojo folder at the same dir as freeipa dir is. Can be
      changed by --dir option, but it's not well tested.
   -  clones https://github.com/dojo/dojo.git and
      https://github.com/dojo/util.git into dojo folder
   -  checkouts both repos to tag 1.8.1 (should be change later when
      updating dojo)
   -  applies custom patches in util/build/patches dir on dojo/util repo
   -  makes src/dojo and src/build symbolic links

-  can be fine-tuned by running with different options, check --help

.. _make_new_dojo_lib_build:

Make new Dojo lib build
^^^^^^^^^^^^^^^^^^^^^^^

-  required when FreeIPA layer has new dependency
-  requires to have dojo cloned
-  new dependencies should be define in src/dojo.profile.js in
   layer.include list
-  run $ util/make-dojo.sh

.. _make_new_builder_build:

Make new Builder build
^^^^^^^^^^^^^^^^^^^^^^

-  required when a change in a builder is needed. Usually shouldn't be.
-  requires to have dojo cloned
-  recommended workflow:

   -  clone dojo if not done: $ util/prepare-dojo.sh --all
   -  checkout desired version, if needed $ util/prepare-dojo.sh --dojo
      --util --checkout --branch VERSION
   -  make required changes in dojo-root/util/build
   -  run $ util/make-builder.sh
   -  warning: builder is overwritten on successful build, use git reset
      to change it back if needed
   -  if all OK, create a patch file with the changes. Name should be:
      XXX-dojo-build-NAME-YY-commit-message.patch, where XXX is a the
      following number than in the last patch, NAME is your login, nick
      and YY is your patch number starting from 00
   -  store the patch into util/build/patches directory (will require a
      force option on git add)
   -  make patch of all these changes and send it for review

Implementation
--------------

Any additional requirements or changes discovered during the
implementation phase.



Feature Management
------------------

From new user feature POV doesn't affect Web UI or CLI.



Major configuration options and enablement
------------------------------------------

No configuration options.

Replication
-----------

No impact.

Dependencies
------------

rhino 1.7R3 on build machine (minimum version with common JS modules
support)

.. _impact_on_other_development_teams:

Impact on other development teams
---------------------------------

No impact.

.. _impact_to_web_ui_and_other_components:

Impact to Web UI and other components
-------------------------------------

Pure Web UI change. Speeds up Web UI load.



RFE author
----------

`Petr Vobornik <User:Pvoborni>`__
