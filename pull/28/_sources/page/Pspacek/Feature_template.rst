Feature_template
================

Overview
--------

Short overview of the problem set and any background material or
references one would need to understand the details.



Use Cases
---------

Walk through one or more full examples of how the feature will be used.
These should not all be the simplest cases.

-  Explicitly list use cases which were considered but will not
   supported for some reason. Include the reason, too ;-)



Feature Management
------------------

UI

How the feature will be managed via the Web UI.

CLI

Overview of the CLI commands. Example:

+------------+--------------------------------------------------------+
| Command    | Options                                                |
+============+========================================================+
| config-mod | --user-auth-type=password/otp/radius                   |
+------------+--------------------------------------------------------+
| user-mod   | --user-auth-type=password/otp/radius --radius=STR      |
|            | --radius-username=STR                                  |
+------------+--------------------------------------------------------+

Configuration
----------------------------------------------------------------------------------------------

Any configuration options? Any commands to enable/disable the feature or
turn on/off its parts?

Design
------

The proposed solution. This may include but is not limited to:

-  High Level schema (`Example 1 <V4/OTP>`__, `Example
   2 <V4/Migrating_existing_environments_to_Trust>`__)
-  Information or update workflow
-  Access control (may include `new permissions <V4/Permissions_V2>`__)
-  Compatibility with other (older) version of FreeIPA. Think if the
   feature requires a minimum `Domain level <V4/Domain_Levels>`__.



Design Assumptions
----------------------------------------------------------------------------------------------

-  What conditions have to be fulfilled to make the feature functional
   and actually useful?
-  This should include assumptions about the clients and surrounding
   environment, too.
-  The text should answer questions like "When the feature will fail and
   why?", "why we did it *this* way and not *that* way?"

   -  Example 1: Feature is "Support for internationalized principal
      names in Kerberos." => Assumption: "Kerberos client can encode the
      internationalized names in the same way as KDC expects." => sanity
      check: "In beginning of 2016 there is no standard for this. Does
      the feature make sense without doing standardization work first?"
   -  Example 2: Feature is "Support for this new fancy DNS magic." =>
      Assumption: "IPA DNS is the only DNS server used in the network."
      => Sanity check: "A lot of users already have DNS infrastructure
      so the feature will not be available to them. Does it make sense
      to implement it?"

For other hints what to consider see `general
considerations <General_considerations>`__ page.

Implementation
--------------

Any implementation details you would like to spell out. Describe any
technical details here. Make sure you cover

-  **Dependencies**: any new dependencies that FreeIPA project or it's
   part would gain and that needs to be packaged in distros?
-  **Backup and Restore**: any new file to back up or change required in
   `Backup and Restore <V3/Backup_and_Restore>`__?

If this section is not trivial, move it to ``/Implementation`` sub page
and only include link.

Upgrade
-------

Any impact on upgrades? Remove this section if not applicable.



How to Test
-----------

Easy to follow instructions how to test the new feature. FreeIPA user
needs to be able to follow the steps and demonstrate the new features.

The chapter may be divided in sub-sections per `Use
Case <#Use_Cases>`__.



Test Plan
---------

Test scenarios that will be transformed to test cases for FreeIPA
`Continuous Integration <V3/Integration_testing>`__ during
implementation or review phase. This can be also link to `source in
cgit <https://git.fedorahosted.org/cgit/freeipa.git/>`__ with the test,
if appropriate.