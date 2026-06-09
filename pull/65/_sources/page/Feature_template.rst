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



How to Use
----------

This a starting point for design discussions.

Easy to follow instructions how to use the new feature according to the
`use cases <#Use_Cases>`__ described above. FreeIPA user needs to be
able to follow the steps and demonstrate the new features.

The chapter may be divided in sub-sections per `Use
Case <#Use_Cases>`__.

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

Any implementation details you would like to spell out. Describe any
technical details here. Make sure you cover

-  **Dependencies**: any new dependencies that FreeIPA project packages
   would gain and that needs to be packaged in distros? The proposal
   needs to be carefully reviewed, so that FreeIPA dependency size does
   not increase without strong justification.
-  **Backup and Restore**: any new file to back up or change required in
   `Backup and Restore <V3/Backup_and_Restore>`__?

If this section is not trivial, move it to ``/Implementation`` sub page
and only include link.



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

Upgrade
-------

Any impact on upgrades? Remove this section if not applicable.



Test Plan
---------

Test scenarios that will be transformed to test cases for FreeIPA
`Continuous Integration <V3/Integration_testing>`__ during
implementation or review phase. This can be also link to `source in
cgit <https://git.fedorahosted.org/cgit/freeipa.git/>`__ with the test,
if appropriate.