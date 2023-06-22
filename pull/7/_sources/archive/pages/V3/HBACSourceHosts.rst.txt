\__NOTOC_\_

Overview
========

Related ticket: `#3528 <https://fedorahosted.org/freeipa/ticket/3528>`__

The original set of requirements for IPA and SSSD included the
requirement to factor in the source host of the connection in the host
based access control decisions. Unfortunately the actual implementation
showed that it can't be done reliably because source host information is
not consistently passed through the stack to the component that needs to
make a decision. As a result the implementation suffered failures that
were hard to troubleshoot and/or avoid. As a result the client side SSSD
made a decision to deprecate the support of source host. However the
server side - IPA still allows defining rules that include source host.

Commands and command options that include source hosts will be
completely removed from the web UI. In the CLI they will be hidden and
marked as deprecated. These commands and options will not show up in the
help and it will not be possible to use them from the CLI. They will
however remain in the API and when called directly using the API, will
raise appropriate exceptions informing the user about their deprecation.

.. _use_cases118:

Use Cases
=========

-  User opens the FreeIPA web UI and navigates to Policy -> Host Based
   Access Control -> HBAC rules -> any rule. 'From' section does not
   appear as it has been removed.
-  User opens the FreeIPA web UI and navigates to Policy -> Host Based
   Access Control -> HBAC test. 'From' section does not appear as it has
   been removed.
-  User tries to call the deprecated ``hbacrule-add-sourcehost`` command
   from the CLI. The command is not recognized as a valid ipa command
   and the error "unknown command" is returned.
-  User tries to call the deprecated ``hbacrule-add-sourcehost`` command
   from the API. An exception is raised informing the user that the
   command has been deprecated.
-  User tries to call the command ``hbacrule-add`` with the deprecated
   option ``--srchostcat`` from the CLI. The option is not recognized as
   a valid option for this command and the error "no such option" is
   returned.
-  User tries to call the command ``hbacrule-add`` with the deprecated
   option ``--srchostcat`` from the API. An exception is raised
   informing the user that the option has been deprecated.
-  User opens the help for HBAC related commands using the
   ``ipa help ...`` or ``ipa command -h``. No mention of source hosts or
   source host groups is present in help.

Design
======

Remove any reference to source hosts and hostgroups from the help
docstrings.

Remove 'From' section from the HBAC rule details page and from the HBAC
test page in the web UI.

Add new exception type to ``ipalib/errors``: ``DeprecationError``.

Modify the commands ``hbacrule-add-sourcehost`` and
``hbacrule-remove-sourcehost``:

-  ``NO_CLI`` flag will be set
-  validate() method will raise the DeprecationError

Modify the options: ``sourcehostcategory``, ``sourcehost_host``,
``sourcehost_hostgroup`` of the ``hbacrule`` class and ``sourcehost`` of
the ``hbactest`` class:

-  ``no_option`` flag will be set
-  ``_rule_deprecate()`` method will return a deprecation message

Add new unit tests to ensure the appropriate exceptions are raised on
invocation of these commands and options.

Modify existing unit tests to take into account the new behavior of
these commands and options.

.. _feature_managment:

Feature Managment
=================

UI
~~

N/A

CLI
~~~

N/A



Major configuration options and enablement
==========================================

N/A

Replication
===========

N/A



Updates and Upgrades
====================

N/A

Dependencies
============

N/A



External Impact
===============

N/A



RFE Author
==========

`akrivoka <User:Akrivoka>`__ 08:40, 10 April 2013 (EDT)
