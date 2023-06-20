Refactorings
============



Refactorings and infrastructure improvements
============================================

These are long-term efforts without (many) immediately visible effects.
As a result they often have low priority and will take some time to
finish.

Each entry has a contact person who has either started working on it, or
just knows about the topic or can refer you to someone who does.

Ongoing
=======



LDAP code rework
----------------

Currently, we use 2 to 4 APIs for LDAP: IPAdmin (Entity, Entry), ldap2,
and raw python-ldap. Consolidate them into a single better tool.

Ticket: `#2660 installer code should use
ldap2 <https://fedorahosted.org/freeipa/ticket/2660>`__

Contact: pviktori (installers), jcholast (plugins)



Admin tool framework
--------------------

Our installation and management tools use too much copy-pasted spaghetti
code. Fit them within a framework and share the common parts.

Ticket: `#2652 Framework for admin/install
tools <https://fedorahosted.org/freeipa/ticket/2652>`__

Contact: pviktori



i18n improvements
-----------------

-  Use fake translations for tests
-  Split up huge strings so the entire text doesn't have to be
   retranslated each time something changes/is added (done, see `Coding
   Best Practices#Split long translatable
   strings <Coding_Best_Practices#Split_long_translatable_strings>`__)
-  Keep a history/repo of the translations, since Transifex only stores
   the latest version
-  Update the source strings on Transifex more often (ideally as soon as
   patches are pushed)
-  Break Git dependencies: make it possible generate the POT in an
   unpacked tarball
-  Figure out how to best share messages across versions (2.x vs. 3.x)
   so they only have to be translated once
-  Clean up checked-in PO files even more, for nicer diffs
-  Automate & document the process so any dev can do it

Discussion:
http://www.redhat.com/archives/freeipa-devel/2013-January/msg00063.html

Contact: pviktori



Web UI extensibility
--------------------

Ongoing refactoring to support better extensibility:
`#3235 <https://fedorahosted.org/freeipa/ticket/3235>`__
`#3236 <https://fedorahosted.org/freeipa/ticket/3236>`__

-  navigation: router, menu
-  application controller
-  extension registration
-  builders, object definition and alternation by plugins during app
   start-up
-  test impact: different URLs (hash content)

Contact: pvoborni



Remove global API object
------------------------

The first step here is to allow plugin registration on any API object.
The syntax used for this in the plugins is already decided, see `Coding
Best Practices#Decorator-based plugin
registration <Coding_Best_Practices#Decorator-based_plugin_registration>`__.

contact: pviktori

Proposed/Planned
================



Mutable Command objects
-----------------------

The Commands should be instantiated every time they're called, so that
we can set attributes on them to share data between pre/post-callbacks.

``api.Command['user_mod']`` instantiates a new Command

contact: pviktori



Index plugin Namespaces by classes
----------------------------------

Ticket: `#4185 <https://fedorahosted.org/freeipa/ticket/4185>`__ Index
plugin namespaces by classes

Instead of api.Command['obj_cmd'], we should allow and prefer
api.Command[obj_cmd] -- indexing with the actual class, which needs to
be imported, and thus it's clear (to us and to pylint) it's a
dependency.

contact: pviktori



Separate ipapython & ipalib
---------------------------

contact: pviktori



Switch test suite to pytest
---------------------------

contact: pviktori

Benefits
----------------------------------------------------------------------------------------------

-  It would be easier to add classes to the tests and configure our CI
   to run only selected class of tests and for example avoid running
   really long tests (like CA-less) for every commit.
-  Fixtures would allow easy sharing data between tests -- e.g. create
   an OTP, use its ID in the next tests. (Currently this needs to be
   hacked by storing data in class attributes, breaking isolation.)

Past
====

See `refactorings page for the older version <V3/Refactorings>`__.