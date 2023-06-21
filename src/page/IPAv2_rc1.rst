IPAv2_rc1
=========

\__NOTOC_\_



Overview of FreeIPA v2 Release Candidate 1
------------------------------------------

The FreeIPA project team is pleased to announce the availability of the
`Beta 2 release of freeIPA 2.0
server <http://www.freeipa.org/page/Downloads>`__.

-  Binaries are available for F-14 and F-15.
-  Please do not hesitate to share feedback, criticism or bugs with us
   on our mailing list:
   `mailto:freeipa-users@redhat.com <mailto:freeipa-users@redhat.com>`__



Main Highlights of Release Candidate 1
--------------------------------------

This beta has a set of significant improvements across all areas of the
project. Modifications include but are not limited to:

-  Installation fixes.
-  Changes in the DIT structure.
-  WebUI improvements.



Focus of the Release Candidate Testing
--------------------------------------

-  There is a `Fedora test
   day <https://fedoraproject.org/wiki/QA/Fedora_15_test_days>`__ for
   FreeIPA coming on Feb 15th. Please join us in testing FreeIPA.
   The exact instructions will be provided later and will be available
   off the link on the page.
-  The following
   `section <https://fedoraproject.org/wiki/Features/FreeIPAv2#How_To_Test>`__
   outlines the areas that we are mostly interested to test.



Significant Changes Since Beta 2
--------------------------------

To see all the tickets addressed between the two beta releases see the
following
`link <https://fedorahosted.org/freeipa/query?status=closed&milestone=2.0.1+Bug+fixing+(RC)>`__.



Repositories and Installation
-----------------------------

-  Use the following
   `repository <http://freeipa.org/downloads/freeipa-devel.repo>`__ to
   install the release candidate packages.
-  On Fedora-14 FreeIPA relies on the latest versions of the packages
   currently available from the updates-testing repository. Please make
   sure to enable this repository before you proceed with installation.



Known Issues
------------

-  Installation on F-15 with dogtag fails due to
   https://bugzilla.redhat.com/show_bug.cgi?id=676330. Installing with
   the --selfsign option works.
-  Server-generated error messages are not translated yet.
-  IPv6 support is not complete.
-  The 'ipa help' command does not support localization.

We plan to address all the outstanding tickets before the final 2.0
release. For the complete list see the following
`link <https://fedorahosted.org/freeipa/milestone/2.0.2%20Bug%20fixing%20%28RC2%29>`__.