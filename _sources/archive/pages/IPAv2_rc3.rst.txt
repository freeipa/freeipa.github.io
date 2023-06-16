\__NOTOC_\_

.. _overview_of_freeipa_v2_release_candidate_2:

Overview of FreeIPA v2 Release Candidate 2
------------------------------------------

The FreeIPA project team is pleased to announce the availability of the
`Release Candidate 3 release of freeIPA 2.0
server <http://www.freeipa.org/page/Downloads>`__. If no critical
problems are found this will become the final release.

-  Binaries are available for F-14 and F-15.
-  Please do not hesitate to share feedback, criticism or bugs with us
   on our mailing list:
   `mailto:freeipa-users@redhat.com <mailto:freeipa-users@redhat.com>`__

.. _main_highlights_of_release_candidate_3:

Main Highlights of Release Candidate 3
--------------------------------------

This release candidate has a set of significant improvements across all
areas of the project. Modifications include but are not limited to:

-  i18n improvements
-  Fixed the self-service page in the WebUI
-  Use TLS for CA replication
-  Setting up Winsync agreements has been fixed

.. _focus_of_the_release_candidate_testing:

Focus of the Release Candidate Testing
--------------------------------------

-  There was a `Fedora test
   day <https://fedoraproject.org/wiki/QA/Fedora_15_test_days>`__ for
   FreeIPA on Feb 15th. These tests are still relevant and feedback
   would be appreciated.
-  The following
   `section <https://fedoraproject.org/wiki/Features/FreeIPAv2#How_To_Test>`__
   outlines the areas that we are mostly interested to test.

.. _significant_changes_since_rc_2:

Significant Changes Since RC 2
------------------------------

To see all the tickets addressed since rc2 see the following
`1 <https://fedorahosted.org/freeipa/milestone/2.0.3.%20Bug%20Fixing%20%28GA%29link>`__.

.. _repositories_and_installation:

Repositories and Installation
-----------------------------

-  Use the following
   `repository <http://freeipa.org/downloads/freeipa-devel.repo>`__ to
   install the release candidate packages.
-  FreeIPA relies on the latest versions of the packages currently
   available from the updates-testing repository. Please make sure to
   enable this repository before you proceed with installation.

.. _known_issues:

Known Issues
------------

-  Installing IPA on Fedora-15 works but can take more time than Fedora
   14 due to systemd. It is not recognizing some restarts as being
   successful so only continues after a 3-minute timeout. We are working
   on a solution.
