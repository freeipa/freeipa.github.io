General_considerations
======================

This page covers questions that plugin developers that want to extend
FreeIPA should consider.

Please use `Design page template <Feature_template>`__ for your
proposal. Following list of questions can help you to find holes in the
design. The proposed design should be linked from `V4
Proposals <V4_Proposals>`__ section.



LDAP Related Questions
----------------------

-  Are you planning to add new objects or extend existing ones?
-  What attributes and objectclasses would you need to add to the
   schema?
-  Are those objectclasses ABSTRACT, AUXILIARY or STRUCTURAL?
-  What is the name-prefix you are going to use for your objectclasses
   and attributes in order to define a namespace?
-  What is the base OID and specific OIDs that you are going to use for
   your attributes and objectclasses?
-  Where in the tree will your entries be located?
-  Do you have any configuration entries that you plan to store in
   `Directory Server <Directory_Server>`__?
-  Do you have any entries that need to be preloaded?
-  What are the read/write/... permissions for your entries?
-  Are you planning to also import schema defined by a third party that
   is not yet a part of FreeIPA?



Integration and source control related questions
------------------------------------------------

-  Do you see your solution to become a part of the core FreeIPA over
   time or as a stand alone independent offering?
-  What source tree repository you are going to use: FreeIPA or an
   external independent repo?
-  Do you plan to develop your features/extension/plugin in an
   independent project tree on fedorahosted or github or do you want
   your code to be a part of the core FreeIPA repo from the beginning?
-  How you see the evolution of you project over time in context of the
   FreeIPA releases and milestones?
-  What tool are you going to use for source control? GIT or something
   else?
-  Who will have commit privileges to the code?



Process related questions
-------------------------

*If answer is NO to any of the questions in this section you project
can't be a part of the FreeIPA tree and most likely would not be
integrated and supported as a part of FreeIPA core functionality.*

-  Are you going to follow the same `coding style <Coding_Style>`__?
-  Are you planning to use same or similar `bug tracking
   system <https://fedorahosted.org/freeipa/>`__?
-  Are you going to follow patch review process and patch naming
   conventions?
-  Are you going to publish and review designs? Where?



Implementation related questions
--------------------------------

-  What are the CLI commands you are going to provide?
-  Are they written following the same style the 'ipa' command uses ?
-  Is help system implemented in the same way it is done in the 'ipa'
   command?
-  Do you provide command line utilities to configure or enable/disable
   your solution?
-  Do you provide man pages? Do man pages follow the style of the rest
   of the FreeIPA project?
-  How is your solution installed?
-  Is it installed on every replica or only on some?
-  How you deal with upgrades?
-  Do you provide a special upgrade script?
-  How is your solution packaged?
-  How is your solution uninstalled?
-  Have you considered different replication configurations and
   topologies?
-  Do you plan to implement unit tests? Do they provide sufficient
   coverage?
-  How you plan to integrate with web UI?
-  Do you plan to provide user and administrator documentation for your
   solution?
-  Do you plan to provide DS plugins?
-  What additional dependencies for the project does your new
   functionality add?

Security
--------

-  Does your solution introduce any security risk to the overall
   product?
-  Are ACIs well thought through and properly implemented?
-  Do you need to use clear text passwords, or otherwise any different
   method to perform authentication or store password hashes than what
   is already available in ipa ? Do you need to give clients access to
   the directory or other IPA controlled services using these
   alternative credentials or authentication methods? Why ?
-  Do you need to implement special new crypto functions? Why?
-  Does your plugin work in FIPS mode? (not a requirement at the moment)