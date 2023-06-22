Overview
--------

**Local installation and configuration of FreeIPA client libaries.**

As of now it is rather hard to use FreeIPA's client API or command line
programs outside of a regular, system-wide installation. FreeIPA doesn't
follow best practices for Python packaging. It's a side effect if
FreeIPA being a single vendor Open Source project.

#. Despite the fact that the client libraries ``ipaclient`` and
   ``ipalib`` are written in Python and properly packaged with
   setuptools, FreeIPA client is not distributable and installable with
   pip. System-wide installation with a package manager is not an option
   for local installations inside a virtual env.
#. FreeIPA assumes that the host has been enrolled by root with
   ``ipa-client-install``. System-wide configuration as root is not an
   option for integrations. For instance for automation with Ansible it
   is preferable to have a local configuration that does not require
   host enrollment. The local of configuration files can be overriden
   already but FreeIPA lacks a simple and official API.
#. The ``ipaplatform`` package contains platform and distribution
   specific configuration like service names, paths and constants. The
   package is configured on build time. It does not support runtime
   detection of platforms.

I'm proposing a series of small changes and improvements to FreeIPA's
configuration and packaging in order to enable local installation and
configuration of client-side tools. Once all improvements are in place,
FreeIPA's client tools can be installed and configured in an isolated
virtualenv. Multiple configurations and even different versions of
FreeIPA's client libraries can be installed on a single machine. Admin
privileges are only required to install compiler and some build
dependencies, but not for virtualenvs and configuration.

Scope
~~~~~

The scope of the proposal is limited to Python client libraries. The
Python packages and ``IPA_CONFDIR`` env var neither include FreeIPA
server libraries (``ipaserver``, web interface, supporting daemons) nor
enrolment of clients. A local installation just contains the Python code
and the ``ipa`` command. First of all, the proposal will enable
developers to use ``ipalib`` in 3rd party code to call FreeIPA plugins
remotely over JSON or XML-RPC. For now, advanced features like keytabs
and DNS auto-discovery are deliberately not included.

For now integration is limited to supported platforms: Fedora, RHEL and
Debian-like platforms with systemd. Other platforms like Windows, BSD,
macOS, and Linux distributions with System V Init are out of scope.

Example
~~~~~~~

Example:

::

   # create and activate a virtual env
   $ virtualenv /tmp/ipaenv
   $ source /tmp/ipaenv/bin/activate

   # install ipaclient and dependencies locally
   (ipaenv) $ pip install ipaclient

   # create and activate a local configuration
   (ipaenv) $ ipa-client-gencfg /tmp/ipaenv/config master.ipa.example
   (ipaenv) $ export IPA_CONFDIR=/tmp/ipaenv/config
   (ipaenv) $ export KRB5_CONFIG=$IPA_CONFDIR/krb5.conf
   (ipaenv) $ ls /tmp/ipaenv/config/
   ca.crt  default.conf  krb5.conf  nssdb

   # use local configuration to acquire TGT and talk to a server
   (ipaenv) $ kinit admin
   (ipaenv) $ ipa ping

The ``ipa-client-gencfg`` command is just an example.



Use Cases
---------

-  As a developer of a third-party Python library or application I like
   to use ``ipalib`` to perform RPC plugin calls. I also like to follow
   common best practices for Python packages to install ``ipalib`` and
   its dependencies. These best practices pip install-able packages,
   virtual envs and tox.
-  As a developer of an OpenStack project I like to have a simple way to
   install and use FreeIPA client libraries on a variety of
   distributions.
-  As a developer of an OpenStack project unit and functional tests
   executed by tox in a venv are mandatory so the minimum is to be able
   to import ipalib/ipaplatform. The backend may be mocked or otherwise
   worked around but not being able to import code using IPA is a
   show-stopper.
-  As an ordinary user without root privileges I like to install and
   configure a local copy of the ``ipa`` command. I can acquire a
   Kerberos TGT and know the hostname of a FreeIPA server but can neiter
   install software with a package manager nor enroll my machine.
-  For an Ansible playbook I like to have a simple and consistent way to
   point ``ipa`` command and ``ipalib.api`` to a local FreeIPA config
   file ``krb5.conf`` and ``nssdb``. The files can't be global and must
   be part of the current Ansible inventory.
-  As a maintainer of a project that uses ``ipaclient`` I like to verify
   compatibility with multiple versions of FreeIPA without resorting to
   heavy weight solutions like containers or virtual machines. Instead I
   prefer virtual envs with pinned package versions.

.. _lead_engineers:

Lead Engineers
~~~~~~~~~~~~~~

-  Christian Heimes (IPA/CS)

.. _supporting_engineers:

Supporting Engineers
~~~~~~~~~~~~~~~~~~~~

-  Adam Young (OpenStack Platform)
-  Rob Crittenden (OpenStack Platform)

.. _design_implementation:

Design & Implementation
-----------------------

.. _api_for_local_configuration_directory:

API for local configuration directory
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

At the moment FreeIPA has multuple ways to load a different config file
than the default ``default.conf`` from ``/etc/ipa``.

-  For ``ipa`` command, config options can be overriden with the ``-e``
   flag. For example
   ``ipa -e confdir=/path/to/alternative/directory ping`` loads
   ``default.conf`` from an alternative directory.
-  Python application can override settings during the initialization
   phase of FreeIPA API, e.g.
   ``ipalib.api.bootstrap(confdir='/path/to/alternative/directory')``.
   Once ``ipalib.api`` is fully configured, all settings are read-only.

Both approaches have some disadvantages. A user must repeat the ``-e``
option in every call to ``ipa`` or create a shell alias. It's both
tedious and error-prone. Some scripts don't have an override, e.g.
``ipa-client-automount``.

I propose a ``IPA_CONFDIR`` environment variable that works comparable
with MIT Kerberos' ``KRB5_CONFIG`` environment variable,
https://web.mit.edu/kerberos/krb5-1.14/doc/admin/env_variables.html . In
presence of the environment variable, FreeIPA API will use the value of
the environment variable as path for ``confdir``. An explicit ``-e``
option or ``api.bootstrap()`` argument takes precedence over the
environment variable. Some contexts (e.g. server, installers, update)
will still depend on global setting and system file. Therefore they
won't support the env var and refuse to initialize the API.

Precedents
^^^^^^^^^^

-  MIT KRB5 has ``KRB5_CONFIG``,
   https://web.mit.edu/kerberos/krb5-1.14/doc/admin/env_variables.html
-  freedesktop.org has ``XDG_CONFIG_HOME``,
   https://specifications.freedesktop.org/basedir-spec/latest/ar01s03.html
-  Python has multiple env vars like ``PYTHONHOME``,
   https://docs.python.org/2/using/cmdline.html#environment-variables
-  pip uses ``PIP_*``,
   https://pip.pypa.io/en/stable/user_guide/#environment-variables
-  Wikipedia defines: *Environment variables are a set of dynamic named
   values that can affect the way running processes will behave on a
   computer.*

Pros
^^^^

-  ``IPA_CONFDIR`` works similar to MIT KRB5's ``KRB5_CONFIG``.
-  Local configuration for ``ipa`` command and ``ipalib`` becomes easy.
   A user or program just has to set the environment variables
   ``IPA_CONFDIR`` and ``KRB5_CONFIG`` to local configuration files. All
   API calls automatically pick up the right configuration in the
   current shell session.

Cons
^^^^

-  It's yet another way to set the ``confdir`` option.

.. _tickets_prs:

Tickets / PRs
^^^^^^^^^^^^^

-  Allow client commands without enrolling

   -  https://fedorahosted.org/freeipa/ticket/6389

-  Use env var ``IPA_CONFDIR`` to get confdir for cli context

   -  https://github.com/freeipa/freeipa/pull/182

-  ``ipalib.api.finalize()`` requires Kerberos credentials

   -  https://fedorahosted.org/freeipa/ticket/6408

-  Use ``api.env.nss_dir`` instead of ``paths.IPA_NSSDB_DIR``

   -  https://fedorahosted.org/freeipa/ticket/6386
   -  https://github.com/freeipa/freeipa/pull/143

-  Make ``api.env.nss_dir`` relative to ``api.env.confdir``

   -  https://github.com/freeipa/freeipa/pull/180

.. _add_package_dependencies_for_distribution_with_pip:

Add package dependencies for distribution with pip
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

With FreeIPA's recent move to setuptools, the Python build system
supports wheels. Wheels https://wheel.readthedocs.io/en/latest/ is the
new and recommended packaging format for Python libraries. In order to
make FreeIPA's client packages easily install-able with pip, the
packages need to provide a list of install requirements. Setuptools
include the requirements in the packages'. Pip downloads and install the
requirements automatically.

.. _build_and_runtime_requirements:

Build and runtime requirements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

FreeIPA no longer contains C extensions. It depends on a couple of
packages with C extensions that require a compiler, libraries and
headers at build time. Python packages with C code are cffi,
cryptography, gssapi, pyldap/python-ldap, python-nss and
[STRIKEOUT:lxml]. FreeIPA also uses some external programs like openssl,
kinit and certutil from nss-tools.

ipapython's dependencies on libxml, libxslt and lmxl could be dropped
replacing lxml with Python stdlib's xml.etree package. The XML parser in
the standard library is built on top of libexpat. xml.etree does not
provide all features of lxml.etree. That's not a problem since FreeIPA
uses only basic features and no advanced features like XSLT or complex
XPath. **FIXED** ipaclient no longer imports lxml.

.. _fedora_rhel:

Fedora / RHEL
'''''''''''''

On Fedora and RHEL the runtime dependencies are provided by:

-  python-pip
-  keyutils
-  krb5-workstation
-  openssl
-  openldap-clients
-  nss-tools
-  libffi

Further more build time dependencies are:

-  python-wheel
-  gcc
-  krb5-devel
-  libffi-devel
-  nss-devel
-  openldap-devel
-  openssl-devel

Debian
''''''

Runtime dependencies:

-  python-pip
-  krb5-user
-  openssl
-  ldap-utils
-  libnss3-tools
-  libffi6
-  [STRIKEOUT:libxml2]
-  [STRIKEOUT:libxslt1.1]

Build dependencies:

-  python-dev
-  python-wheel
-  build-essential
-  pkg-config
-  libkrb5-dev
-  libffi-dev
-  libnss3-dev
-  libldap2-dev
-  libsasl2-dev
-  libssl-dev
-  [STRIKEOUT:libxml2-dev]
-  [STRIKEOUT:libxslt1-dev]

.. _pros_1:

Pros
^^^^

-  FreeIPA's client libraries become easy installable in a virtual env.

.. _cons_1:

Cons
^^^^

-  Package requirements from ``freeipa.spec`` are duplicated in
   ``setup.py`` files. It increases package maintenance slightly.

Remarks
^^^^^^^

python-nss does not support wheels yet,
https://bugzilla.redhat.com/show_bug.cgi?id=1389739

.. _tickets_prs_1:

Tickets / PRs
^^^^^^^^^^^^^

-  Make ipaclient pip install-able
   https://fedorahosted.org/freeipa/ticket/6468
-  ` <https://github.com/tiran/freeipa/commits/python\_requirements>`__\ `https://github.com/tiran/freeipa/commits/python\_requirements <https://github.com/tiran/freeipa/commits/python\_requirements>`__
-  Make :literal:`\`setup.py`\ \` files PyPI compatible
   https://github.com/freeipa/freeipa/pull/197
-  Use xml.etree instead of lxml in odsmgr.py
   https://fedorahosted.org/freeipa/ticket/6469

.. _ipaplatform_auto_configuration:

ipaplatform auto-configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``ipaplatform`` package is an abstraction layer for platform and
distribution specific settings and services. The other FreeIPA packages
use ``ipaplatform.constants``, ``ipaplatform.paths``,
``ipaplatform.services``, and ``ipaplatform.tasks``. Internally the
modules are aliases, e.g. on Fedora ``ipaplatform.paths`` is an alias
for ``ipaplatform.fedora.paths``. As of now the aliases are implemented
with symlinks. The symlinks are created at build time by autoconf. The
approach is not compatible with Python wheels and pip. FreeIPA packages
might be build on Fedora but installed on CentOS.

I'm proposing two changes:

-  Platform selection shall become an import time decision. The platform
   id is read from ``/etc/os-release``. The file is available on all
   relevant platform. It also contains an ordered list of similar
   platforms. For example on CentOS the *ID* is ``centos`` and *ID_LIKE*
   are ``rhel`` and ``fedora`` in that order. Since ``ipaplatform`` does
   not provide a ``ipaplatform.centos`` sub-package, it will
   automatically select ``ipaplatform.rhel`` as platform provider for
   CentOS.
-  ``ipaplatform`` is turned into a namespace package. A namespace
   package allows third parties to provide external packages with
   platform definitions, e.g. a ``ipaplatform.debian`` package.

.. _pros_2:

Pros
^^^^

-  The platform reflects the actual platform that FreeIPA is running on.
-  The platform selector falls back to related platform identifiers.
-  Third parties can provide pip install-able platform modules.

.. _cons_2:

Cons
^^^^

-  The implementation becomes a bit more complicated.

.. _remarks_1:

Remarks
^^^^^^^

The ``__path__`` trick is not compatible with namespace packages.
``ipaplatform.__init__`` cannot contain any code.

pylint is not able to understand meta import hooks. An AstroidBuilder
plugin for pylint turned out to be too fragile. My new implementation
uses a facade module that is replaced with the actual module.

.. _tickets_prs_2:

Tickets / PRs
^^^^^^^^^^^^^

-  https://github.com/tiran/freeipa/commits/ipaplatform_detect
-  Select ipaplatform at runtime
   https://fedorahosted.org/freeipa/ticket/6474

.. _ipaplatform_debian_support:

ipaplatform Debian support
~~~~~~~~~~~~~~~~~~~~~~~~~~

FreeIPA upstream does not include platform configuration for
Debian-based distributions. In order to support development and
deployment on other distributions, FreeIPA should include Timo Aalton's
patch for ``ipaplatform.debian``. There is demand for Debian support
from OpenStack side.

.. _tickets_prs_3:

Tickets / PRs
^^^^^^^^^^^^^

-  Include ipaplatform.debian
   https://fedorahosted.org/freeipa/ticket/6475

.. _pypi_packages:

PyPI packages
~~~~~~~~~~~~~

**details TBD**

I have reservered four package names on PyPI:

-  ipaclient
-  ipalib
-  ipaplatform
-  ipapython

Further more I have registered four additional packages to prevent name
squatting attacks. The ``ipa`` and ``freeipa`` packages just contain
metadata (dependency on ``ipalib``) and no code. The ``ipaserver`` and
``ipatests`` packages have no release at all.

-  ipaserver
-  ipatests
-  ipa
-  freeipa

Upgrade
~~~~~~~

Package dependencies must be synced between RPM spec and setup.py /
ipasetup.py.



How to Use
----------

**TBW**



Test Plan
---------

**TBW**
