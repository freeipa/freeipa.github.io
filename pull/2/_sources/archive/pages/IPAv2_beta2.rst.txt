\__NOTOC_\_

.. _overview_of_freeipa_v2_beta_2:

Overview of FreeIPA v2 Beta 2
-----------------------------

The FreeIPA project team is pleased to announce the availability of the
`Beta 2 release of freeIPA 2.0
server <http://www.freeipa.org/page/Downloads>`__.

-  Binaries are available for F-14.
-  With the release of this Beta, freeIPA moves into the Release
   Candidate cycle.
-  Please do not hesitate to share feedback, criticism or bugs with us
   on our mailing list:
   `mailto:freeipa-users@redhat.com <mailto:freeipa-users@redhat.com>`__

.. _main_highlights_of_the_beta_2:

Main Highlights of the Beta 2
-----------------------------

This beta has a set of significant improvements across all areas of the
project. Modifications include but are not limited to:

-  Support of the latest Dogtag packages.
-  Installation fixes.
-  Changes in the DIT structure.
-  New permissions defined against different elements of the tree.
-  Better startup and shutdown handling.
-  Replication improvements.
-  Incremental improvements in IPv6 support.
-  DNS improvements.
-  The package name has been changed to "freeipa" to avoid collision
   with IPA v1.x and many others.

.. _focus_of_the_beta_testing:

Focus of the Beta Testing
-------------------------

-  There is a `Fedora test
   day <https://fedoraproject.org/wiki/QA/Fedora_15_test_days>`__ for
   FreeIPA coming on Feb 10th. Please join us in testing FreeIPA.
   The exact instructions will be provided later and will be available
   off the link on the page.
   **NOTE: The test day has been moved to Feb 15th.**

-  The following
   `section <https://fedoraproject.org/wiki/Features/FreeIPAv2#How_To_Test>`__
   outlines the areas that we are mostly interested to test.

.. _significant_changes_since_beta_1:

Significant Changes Since Beta 1
--------------------------------

To see all the tickets addressed between the two beta releases see the
following
`link <https://fedorahosted.org/freeipa/milestone/0.8%20iteration%20-%20January%20%28cleanup%29>`__.

.. _repositories_and_installation:

Repositories and Installation
-----------------------------

-  Use the following
   `repository <http://freeipa.org/downloads/freeipa-devel.repo>`__ to
   install the beta 2 packages.
-  On Fedora-14 FreeIPA relies on the latest versions of the packages
   currently available from the updates-testing repository. Please make
   sure to enable this repository before you proceed with installation.

.. _known_issues:

Known Issues
------------

-  Installation on F-15 fails due to
   `this <https://bugzilla.redhat.com/show_bug.cgi?id=674916>`__ and
   other issues unrelated to FreeIPA.
-  Server-generated error messages are not translated yet.
-  IPv6 support is not complete.
-  The 'ipa help' command does not support localization.

We plan to address all the outstanding tickets before the final 2.0
release. For the complete list see the following
`link <https://fedorahosted.org/freeipa/milestone/2.0.1%20Bug%20fixing>`__.
