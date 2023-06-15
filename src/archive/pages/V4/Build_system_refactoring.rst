Overview
--------

The build system in FreeIPA 4.4 has following problems:

-  It is ineffective. This costs developer time every time we build IPA.
   Every developer typically builds IPA several times a day. Also, it
   slows down CI because it needs to build the packages as well.
-  The build system does not directly support rapid development
   approaches: Every patch-copy-test cycle requires either manual work
   (which is error prone) or full build (which is very slow). It slows
   down rapid prototyping.

   -  Repeated build with minimal changes (like changing one file) takes
      long time, which prevents us from doing rapid devel cycle.
   -  Theoretically the process should be very fast as proper Makefile
      would detect what can be skipped.
   -  In reality the build takes 88 % of the time from the first build!

-  Is not reliable. Often, "git clean -xdf" is needed to get the build
   to pass.
-  It is hard to maintain and debug.

The proposal is to re-write the build system using current best
practices and standard tools. Re-write from scratch is deemed to be more
effecient than fixing bugs one by one because parts of current system
are unnecessairly complex or can be replaced with standard solutions.

.. _user_stories:

User stories
------------

Every user natually wants wants reliable build.

-  **Developer** building a development version of FreeIPA:

   -  Expects very fast build
   -  Wants very simple and quick installation of the new version
   -  Needs easy way to extend/modify the build system as components
      change

-  **Tester** building latest version for test execution

   -  Wants easy-to-follow procedure to build distribution packages with
      latest version of FreeIPA

-  **Release engineer** releasing new upstream version

   -  Wants to use standard tools for creating tarball which is then
      consumed by packager

-  **Packager** preparing a package for Linux Distribution

   -  Wants to use standard tools for packaging, without need for hacks

Design
------

The proposed solution. This may include but is not limited to:

-  High Level schema (`Example 1 <V4/OTP>`__, `Example
   2 <V4/Migrating_existing_environments_to_Trust>`__)
-  Information or update workflow
-  Access control (may include `new permissions <V4/Permissions_V2>`__)
-  Compatibility with other (older) version of FreeIPA. Think if the
   feature requires a minimum `Domain level <V4/Domain_Levels>`__.

For other hints what to consider see `general
considerations <General_considerations>`__ page.

Implementation
--------------

-  Use autotools suite to orchestrate the build on the top-level
-  For Python parts: use setuptools instead of distutils
-  For C parts: use autotools
-  For Internationalization support: use standard gettextize framework +
   custom enhancements to support Zanata workflow
-  Web UI build was not touched nad is usin Dojo builder as it was
   before.

-  **Dependencies**:

   -  New build-time depedency gettext-devel is needed for gettextize
      framework.



Feature Management
------------------

CLI
~~~

+----------------+----------------------------------------------------+
| Command        | Desired effect                                     |
+================+====================================================+
| ./configure    | test system configuration and prepare build        |
|                | according to configuration options                 |
+----------------+----------------------------------------------------+
| ./makerpms.sh  | build RPMs from clean Git tree: autoreconf -i &&   |
|                | ./configure && make rpms                           |
+----------------+----------------------------------------------------+
| make all       | build complete FreeIPA                             |
+----------------+----------------------------------------------------+
| make install   | install FreeIPA files into paths so FreeIPA can be |
|                | executed right away                                |
+----------------+----------------------------------------------------+
| make uninstall | uninstall FreeIPA files so the system is clean     |
|                | again and new installation can be done (e.g.       |
|                | installation from different branch)                |
+----------------+----------------------------------------------------+
| make rpms      | create SRPM and RPMs                               |
+----------------+----------------------------------------------------+
| make clean     | erase the files built by make all                  |
+----------------+----------------------------------------------------+
| make           | See `Automake: Standard                            |
|                | Targets <https://www.gnu.org/software/a            |
|                | utomake/manual/html_node/Standard-Targets.html>`__ |
+----------------+----------------------------------------------------+

Configuration
~~~~~~~~~~~~~

configure script will have several configuration options (replacing
RPM-specific SPEC file magic).

+----------------------+----------------------+----------------------+
| Configure option     | Default value        | Desired effect       |
+======================+======================+======================+
| PYTHON (environment  | python in $PATH      | path to Python       |
| variable)            |                      | interpreter (use     |
|                      |                      | this to select       |
|                      |                      | Python 2/3)          |
+----------------------+----------------------+----------------------+
| --disable-i18n-tests | not present (tests   | do not execute       |
|                      | enabled)             | ipatests/i18n.py     |
|                      |                      | (depends on          |
|                      |                      | python-polib)        |
+----------------------+----------------------+----------------------+
| --disable-server     | server build is      | do not build server  |
|                      | enabled              | components           |
+----------------------+----------------------+----------------------+
| --enable-pylint      | don't run pylint     | run pylint on Python |
|                      |                      | packages using       |
|                      |                      | $PYTHON for make     |
|                      |                      | lint                 |
+----------------------+----------------------+----------------------+
| --with-jslint        | jsl in $PATH         | run JS lint          |
+----------------------+----------------------+----------------------+
| --with-ipaplatform   | autodetect           | select Fedora/RHEL   |
|                      |                      | IPA ipaplatform to   |
|                      |                      | build for            |
+----------------------+----------------------+----------------------+
| --without-ipatests   | include ipatests     | don't include        |
|                      |                      | ipatests             |
+----------------------+----------------------+----------------------+
| --with-vendor-suffix | (empty string)       | vendor suffix; used  |
|                      |                      | in VENDOR_VERSION    |
|                      |                      | string stored in     |
|                      |                      | i                    |
|                      |                      | papython/version.py; |
|                      |                      | e.g. "-1.fc24"; this |
|                      |                      | should be used from  |
|                      |                      | SPEC file, not       |
|                      |                      | necessary for        |
|                      |                      | upstream-only builds |
+----------------------+----------------------+----------------------+

Platforms which are missing some of the tools will be able to use
*--without-feature* and *--disable-feature* options to disable part of
the build or check.

Versioning
~~~~~~~~~~

Current versioning scheme is partly responsible for the slow build.
Developer build with IPA_VERSION_IS_GIT_SNAPSHOT=1 changes version
values in version.m4 during each build. As a result, the whole autotools
machinery needs to be re-executed on each build.

Me and jcholast decided to keep this behavior for option
IPA_VERSION_IS_GIT_SNAPSHOT=1. If you want fast build, disable it. The
main reason is that getting rid of this problem would require
significant effort which would include code changes outside of build
system.

For reference, here are pieces of the old build system which concern
versioning:

-  `VERSION <https://git.fedorahosted.org/cgit/freeipa.git/tree/VERSION?id=2b8163ab5dfcf28a9eba319ef685046ae9d8b5e8>`__
   file
-  variables in
   `Makefile <https://git.fedorahosted.org/cgit/freeipa.git/tree/Makefile?id=2b8163ab5dfcf28a9eba319ef685046ae9d8b5e8>`__
-  `SPEC <https://git.fedorahosted.org/cgit/freeipa.git/tree/freeipa.spec.in?id=2b8163ab5dfcf28a9eba319ef685046ae9d8b5e8>`__
   file

Here is plan what we should do with these variables:

+----------------------------------+----------------------------------+
| Variable                         | What to do with it               |
+==================================+==================================+
| IPA_VERSION_MAJOR                | move to VERSION.m4               |
+----------------------------------+----------------------------------+
| IPA_VERSION_MINOR                | move to VERSION.m4               |
+----------------------------------+----------------------------------+
| IPA_VERSION_RELEASE              | move to VERSION.m4               |
+----------------------------------+----------------------------------+
| IPA_VERSION_ALPHA_RELEASE        | merge to IPA_VERSION_PRE_RELEASE |
|                                  | in VERSION.m4                    |
+----------------------------------+----------------------------------+
| IPA_VERSION_BETA_RELEASE         | merge to IPA_VERSION_PRE_RELEASE |
|                                  | in VERSION.m4                    |
+----------------------------------+----------------------------------+
| IPA_VERSION_RC_RELEASE           | merge to IPA_VERSION_PRE_RELEASE |
|                                  | in VERSION.m4                    |
+----------------------------------+----------------------------------+
| IPA_VERSION_PRE_RELEASE          | new variable; string is appended |
|                                  | to .. to form version number     |
|                                  | like "1.0.0rc1"                  |
+----------------------------------+----------------------------------+
| IPA_VERSION_IS_GIT_SNAPSHOT      | move to VERSION.m4               |
+----------------------------------+----------------------------------+
| IPA_DATA_VERSION                 | move to VERSION.m4               |
+----------------------------------+----------------------------------+
| IPA_API_VERSION_MAJOR            | move to VERSION.m4               |
+----------------------------------+----------------------------------+
| IPA_API_VERSION_MINOR            | move to VERSION.m4               |
+----------------------------------+----------------------------------+
| IPA_VENDOR_VERSION_SUFFIX        | move to configure                |
| (currently defined in SPEC)      | --with-vendor-suffix             |
+----------------------------------+----------------------------------+
| IPA_RPM_RELEASE (currently reads | remove, replaced by              |
| RELEASE file)                    | --with-vendor-suffix             |
+----------------------------------+----------------------------------+

When IPA_VERSION_IS_GIT_SNAPSHOT is enabled, the configure script will
touch VERSION.m4 file. On subsequent builds, this will trigger automatic
re-execution of configure script (assuming AM_MAINTAINER_MODE is
disabled).



How to Use
----------

All users can use multiple CPUs by running make with parameters "-j" or
alternativelly "-l". It is handy to specify these parameters in variable
MAKEFLAGS like this:

``$ export MAKEFLAGS="-j16"``

so it applies to all make jobs by default.

Developer
~~~~~~~~~

-  First round - build & install RPMs once to get all the depedencies
   and scriptlets ran:

| ``$ rm Makefile  # if Makefile exists, remove it``
| ``$ ./makerpms.sh  # this runs configure with paths appropriate for subsequent installation``
| ``$ dnf install dist/rpms/*.rpm``

-  Subsequent rapid development:

| 
| ``$ make install``

``make install`` will quickly rebuild files as needed and install new
files onto development system, so the new build can be tested
immediatelly.

-  Installing files to a remote machine:

The install target supports variable ``DESTDIR`` which specifies where
to copy the files. This can be easily used together with SSHfs which
mounts complete root filesystem from a VM to developer's machine:

| ``$ mkdir /tmp/vm``
| ``$ sshfs -o transform_symlinks root@``\ ``:/ /tmp/vm``
| ``$ make install DESTDIR=/tmp/vm``

This snippet will synchronize all files from developer's machine onto a
VM. Just keep in mind that it will not bump version in RPM database and
things depending on this might break.

To remove all files from the latest build, you can use target
``uninstall``:

``$ make uninstall DESTDIR=/tmp/vm``

Uninstallation ensures that there are no leftovers from the current
version so new version can be safely installed. (Again, keep in mind
that this will not touch RPM database.)

As an optimization for lower-bandwidth/high-latency links you can use
``rsync`` instead of ``sshfs``. Is is just additional step after
``make install``:

| ``$ mkdir /tmp/vm``
| ``$ make install DESTDIR=/tmp/vm``
| ``$ rsync -rlK /tmp/vm/ root@``\ ``:/``

Tester
~~~~~~

| ``$ autoreconf -i``
| ``$ ./configure``
| ``$ make rpms``

Or alternatively:

``$ ./makerpms.sh``

will produce RPMs suitable for further FreeIPA testing.

.. _release_engineer:

Release engineer
~~~~~~~~~~~~~~~~

| ``$ autoreconf -i``
| ``$ ./configure``
| ``$ make dist``

will produce version.tar.gz suitable for further packaging

Packager
~~~~~~~~

| ``$ autoreconf -i``
| ``$ ./configure``
| ``$ make install DESTDIR=``

will install FreeIPA into correct paths in build root so it is very easy
to take all installed files and just package them.

.. _packager___client_only_build:

Packager - client only build
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| ``$ autoreconf -i``
| ``$ ./configure --disable-server --without-ipatests``
| ``$ make install DESTDIR=``

will install FreeIPA into correct paths in build root so it is very easy
to take all installed files and just package them.

Note: This use case does not fully work yet. See progress in
`#6417 <https://fedorahosted.org/freeipa/ticket/6517>`__

.. _translation_maintainer:

Translation maintainer
~~~~~~~~~~~~~~~~~~~~~~

-  Generate a new ``.pot`` file for Zanata:

``$ make ipa.pot-update``

-  Strip untranslated strings from ``.po`` files downloaded from Zanata:

``$ make strip-po``

-  Test all strings and translation system:

``$ make polint``



Test Plan
---------

Test scenarios that will be transformed to test cases for FreeIPA
`Continuous Integration <V3/Integration_testing>`__ during
implementation or review phase. This can be also link to `source in
cgit <https://git.fedorahosted.org/cgit/freeipa.git/>`__ with the test,
if appropriate.
