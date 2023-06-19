General
-------

We tend to primarily develop by building rpms and installing them rather
than 'make install'. The make install method should work but it isn't
tested extensively.

The spec files within this repo are similar but not the same as in the
Fedora or RHEL repo.

Details about build system can be found in the `V4/Build system
refactoring <V4/Build_system_refactoring>`__ design page.

.. _the_version_file:

The VERSION file
----------------

The version of IPA is mostly controlled in the file VERSION.m4 in the
root of the IPA git repo. This file is generally self-explanatory but it
does distinguish between developer builds and final releases. This is
really only important if you are building RPMs, it puts the GIT version
into the release so you can easily tell where in the revision history
you are working from.

To build a release, define ``IPA_VERSION_IS_GIT_SNAPSHOT`` as ``no`` in
VERSION.m4 file before you execute the build.

.. _building_from_the_git:

Building from the git
---------------------

First, clone our git tree of your project:

``$ git clone ``\ ```https://pagure.io/freeipa.git`` <https://pagure.io/freeipa.git>`__

Next, we will install all packages needed to build FreeIPA as they do
not have to be installed on your system. You can install them easily by
following instructions:

| ``$ cd freeipa``
| ``$ cp freeipa.spec.in freeipa-builddep.spec # this is workaround for yum-builddep: file name has to end with ".spec"``
| ``$ sudo yum-builddep freeipa-builddep.spec``

or with dnf (and builddep plugin from ``dnf-plugins-core``:

| ``$ sudo dnf install rpm-build``
| ``$ sudo dnf builddep -b -D "with_wheels 1" -D "with_lint 1" --spec \``
| ``freeipa.spec.in --best --allowerasing --setopt=install_weak_deps=False``

A dose of ``--enablerepo=updates-testing`` may at times be needed.

Depending on the git branch which we want to build, Fedora official
repositories don't always contain versions of packages required for the
build and installation. These requirements are located in
`COPR <https://copr.fedoraproject.org/>`__ repositories. Current ones
are:

-  master branch:
   `@freeipa/freeipa-master <https://copr.fedorainfracloud.org/coprs/g/freeipa/freeipa-master/>`__
-  ipa-4-6 branch for Fedora 26 and Fedora 27:
   `@freeipa/freeipa-4-6 <https://copr.fedorainfracloud.org/coprs/g/freeipa/freeipa-4-6/>`__
-  ipa-4-5 branch for Fedora 25 and Fedora 26:
   `@freeipa/freeipa-4-5 <https://copr.fedorainfracloud.org/coprs/g/freeipa/freeipa-4-5/>`__
-  ipa-4-4 branch for Fedora 24:
   `@freeipa/freeipa-4-4 <https://copr.fedorainfracloud.org/coprs/g/freeipa/freeipa-4-4/>`__
-  ipa-4-3 branch:
   `@freeipa/freeipa-4-3 <https://copr.fedorainfracloud.org/coprs/g/freeipa/freeipa-4-3/>`__
   (*abandoned*)
-  ipa-4-2 branch:
   `mkosek/freeipa-4.2 <https://copr.fedoraproject.org/coprs/mkosek/freeipa-4.2/>`__
   (*abandoned*)
-  ipa-4-1 branch:
   `mkosek/freeipa-4.1 <https://copr.fedoraproject.org/coprs/mkosek/freeipa-4.1/>`__
   (*abandoned*)
-  ipa-4-1 branch for Fedora 20 and CentOS:
   `mkosek/freeipa <https://copr.fedoraproject.org/coprs/mkosek/freeipa/>`__
   (*abandoned*)

To use the repo run (requires **dnf-plugins-core** package installed):

`` $ sudo dnf copr enable @freeipa/freeipa-master``

When all packages required for the build are installed, you can start
building FreeIPA. The result will be a tarball with sources, SRPM and
RPMs that can be installed on a system:

| ``$ ./makerpms.sh``
| ``$ sudo yum localinstall dist/rpms/*.rpm``

or with dnf

``$ sudo dnf install dist/rpms/*.rpm``

Sometimes the build can fail because of garbage from branch switching or
previous builds. To avoid that, clean the git repository, but make sure
that all your changes are committed.

`` $ git clean -dfx``

.. _running_freeipa_in_tree:

Running FreeIPA In-tree
-----------------------

If you have IPA installed on your development system you can do some
limited in-tree development of management plugins. To do this:

Server setup:

-  As root user, install IPA using ipa-server-install
-  Create ~/.ipa/alias/.pwd and enter the admin password
-  Run kinit admin
-  To run the server, execute python lite-server.py

Client setup:

-  Copy /etc/ipa/default.conf into ~/.ipa/default.conf
-  Replace xmlrpc_uri with http://127.0.0.1:8888/ipa/xml
-  To run the CLI, execute ./ipa

FreeIPA will detect that it is running in-tree and will use the port and
XML-RPC location that ``lite-server.py`` is listening only. If you make
changes to the server-side of a plugin you'll need to restart
``lite-server.py``.

`Category:ImproveOrRemove <Category:ImproveOrRemove>`__
