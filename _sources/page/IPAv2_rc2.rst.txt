IPAv2_rc2
=========

\__NOTOC_\_



Overview of FreeIPA v2 Release Candidate 2
------------------------------------------

The FreeIPA project team is pleased to announce the availability of the
`Release Candidate 2 release of freeIPA 2.0
server <http://www.freeipa.org/page/Downloads>`__.

-  Binaries are available for F-14 and F-15.
-  Please do not hesitate to share feedback, criticism or bugs with us
   on our mailing list:
   `mailto:freeipa-users@redhat.com <mailto:freeipa-users@redhat.com>`__



Main Highlights of Release Candidate 2
--------------------------------------

This beta has a set of significant improvements across all areas of the
project. Modifications include but are not limited to:

-  Make Indirect membership clearer.
-  Input validation fixes.
-  WebUI improvements.
-  Created default Roles.
-  IPv6 support
-  Documentation updates



Focus of the Release Candidate Testing
--------------------------------------

-  There was a `Fedora test
   day <https://fedoraproject.org/wiki/QA/Fedora_15_test_days>`__ for
   FreeIPA on Feb 15th. These tests are still relevant and feedback
   would be appreciated.
-  The following
   `section <https://fedoraproject.org/wiki/Features/FreeIPAv2#How_To_Test>`__
   outlines the areas that we are mostly interested to test.



Significant Changes Since RC 1
------------------------------

To see all the tickets addressed between the two beta releases see the
following
`link <https://fedorahosted.org/freeipa/query?status=closed&milestone=2.0.2+Bug+fixing+(RC2)>`__.



Repositories and Installation
-----------------------------

-  Use the following
   `repository <http://freeipa.org/downloads/freeipa-devel.repo>`__ to
   install the release candidate packages.
-  FreeIPA relies on the latest versions of the packages currently
   available from the updates-testing repository. Please make sure to
   enable this repository before you proceed with installation.



Known Issues
------------

-  Installation on F-15 with dogtag fails due to
   https://bugzilla.redhat.com/show_bug.cgi?id=676330. Installing with
   the --selfsign option works.
-  Server-generated error messages are not translated yet.
-  The 'ipa help' command does not support localization.

We plan to address all the outstanding tickets before the final 2.0
release. For the complete list see the following
`link <https://fedorahosted.org/freeipa/milestone/2.0.3.%20Bug%20Fixing%20%28GA%29>`__.