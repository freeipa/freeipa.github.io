Overview
========

This page contains the feedback the IPA team got during usability
testing of the proposed UI for the IPA v2 system.

.. _define_a_configuration_policy:

Define a Configuration Policy
=============================

-  `Task we
   used <http://www.freeipa.org/wiki/images/b/b6/Task_Config_Policy.pdf>`__
-  `Screens for the
   task <http://www.freeipa.org/wiki/images/2/28/Config_Policy.pdf>`__

Instances
---------

-  Unclear what the term "Instance" means or represents.
-  State labels (Applied, Edited, Superseded) are unclear, but
   understood once explained.
-  “Perhaps use Active, Inactive and Retired. Grey out the Retired
   state.”
-  “Perhaps use Unapplied instead of Superseded.”
-  “Perhaps use Deployed instead of Applied.”
-  The Undo feature makes sense.
-  The Roll Back To button makes sense.

.. _branching_policy_edits_and_cloning_policies_into_different_groups:

Branching Policy Edits and Cloning Policies Into Different Groups
-----------------------------------------------------------------

-  Users don't want to maintain several versions of a single policy and
   worry about maintaining duplicate data. From a configuration
   management perspective they would prefer to have the data in one spot
   to be used anywhere.
-  "We have thousands of machines. When Help Desk does Level 1 system
   work, give them access to a machine without rebuilding the policy. We
   don't want to maintain lots of copies of policies on the host level."
-  "Machines will be in disjointed states with data spread out all
   over."
-  "An administrative nightmare."
-  Users prefer that the configuration management tool is always right
   and assume local changes are wrong and need correcting.

.. _association_between_policy_groups_and_hosts:

Association Between Policy Groups and Hosts
-------------------------------------------

-  “I'd like to select a policy to see its associated hosts, and vice
   versa.”

.. _link_to_host_associations_screen:

Link to “Host Associations” Screen
----------------------------------

-  “It's logical. I'd use it to define groups of hosts [on production,
   staging, etc.].”

.. _prioritizing_policies_and_policy_groups:

Prioritizing Policies and Policy Groups
---------------------------------------

-  Users generally expect higher priority to be on top.
-  Perhaps include annotation indicating the order.
-  Managing order with the up and down arrow buttons is intuitive.
-  The up and down arrow buttons were overlooked, but seem fine once
   explained.

.. _policies_preview:

Policies Preview
----------------

-  Some confusion over how to use Policies Preview.
-  Choosing the application, template and host relationship is
   intuitive.
-  Users would like attributes (Application, Template, etc.) to
   automatically reflect the policy currently being edited. Though
   perhaps make this an option.
-  Users would like to have text files of a generated policy to see any
   conflicts.
-  Users would like a diff format to compare specific Boolean changes in
   a previewed policy VS. current state.
-  "If a policy creates a file from two or more policies, I need to see
   that file from a system perspective compared to an unapplied state."
-  “Verifying syntax would be very helpful.” Use a green check mark if
   OK, indicate lines with errors.

.. _merge_log:

Merge Log
---------

-  Helpful.
-  "Marginally helpful." Would prefer something like a diff for
   individual files: compare before and after, and what changed. A text
   file would be useful. Would like comments within the policy
   iteration, and the ability to generate a system log message any time
   something happens on a machine.
-  “The more info the better.”
-  “We need to find where an individual line in a policy file is coming
   from instead of sorting through each policy.” For example, to see why
   someone has access who shouldn't.

.. _general_comments:

General Comments
----------------

-  It's unclear if the “Edit Properties” link affects instance states or
   something else.
-  Can/should we avoid duplication of data?
-  Show a preview of a complete dump for a machine.
-  “It's difficult to write an app configuration policy to give someone
   who runs it a different way.”
-  “I don't want to manually configure monitoring for every app.”
-  “I don't want to manually enter hosts in a GUI for proxy. People
   forget to do that. It should be automatically detected.”
-  “Configuration management systems can be very powerful if you can
   write them yourself and aren't constrained to a form.”
-  “It would take me longer to get familiar with a form than to directly
   write or read code.”
-  “We try to write generic policies so we can change data around them
   instead of having to change a template.” Only one change is needed,
   not multiple instances.
-  “Different people could simultaneously work on different parts of a
   policy.”
-  A text box inside the GUI for direct editing of a file would be nice.
-  Perhaps enter the host to get a list of applications and files
   instead of selecting things in the current order.
-  The “Reset All Policies Preview to Applied Instances” button is
   confusing. Perhaps change to “Revert All Policies Preview”.
-  Add a "Save Text" button to the Policies Preview window.

.. _host_based_access_control_rules:

Host-Based Access Control Rules
===============================

-  `Task we
   used <http://www.freeipa.org/wiki/images/b/b6/Task_HBAC.pdf>`__
-  `Screens for the
   task <http://www.freeipa.org/wiki/images/d/d0/Hbac.pdf>`__
-  `Additional screens for the rule's time range management
   tasks <http://www.freeipa.org/wiki/images/4/4e/Time.pdf>`__

.. _ordering_of_deny_and_allow_rules:

Ordering of Deny and Allow Rules
--------------------------------

-  Deny before Allow makes sense.
-  Having both panels on one page is fine.
-  Prefer to see Allow rules listed first, even if they don't follow
   that order under the hood.

.. _on_hosts_and_from_hosts_steps:

“On Hosts” and “From Hosts” Steps
---------------------------------

-  There is some initial confusion between the addition of hosts to “On
   Hosts” and the addition of hosts to “From Hosts.” We may need to make
   the separation of steps more explicit.
-  No confusion differentiating steps.
-  Not clear how to manage non-IPA managed hosts. Expect to handle all
   hosts, not just IPA managed hosts.
-  From hosts should allow specifying the name, domain, mask or range.
   We should support something like 192.168.0/24 syntax.

.. _changing_rule_attributes:

Changing Rule Attributes
------------------------

-  Adding additional attributes seems intuitive.
-  There is some confusion over how to edit a rule. Individual attribute
   columns may need an “Edit” button or link.
-  Different editing interfaces per attribute is fine. “Attribute types
   are different enough to merit different editing features.”
-  Tool tips would help during editing.

.. _deleting_all_rule_attributes:

Deleting All Rule Attributes
----------------------------

-  In IPA does the object class allow the removal of all user
   attributes? - Dmitri checked, it does.
-  Expect an error or removal of the rule when all meaningful attributes
   are removed.
-  Perhaps highlight (in red) an undefined policy in the list of rules,
   and highlight (in red) specific missing attributes in the rule
   itself.
-  Having to narrow down from the default “All Users/Hosts/etc.” is
   reasonable. Defaulting to "All..." makes sense for a Deny rule.
-  Expect no attributes by default.

.. _deny_all_access_rule:

“Deny All Access” Rule
----------------------

-  The concept and creation of a “Deny All Access” rule makes sense to
   everyone.
-  It's good and prevents having to remove all defined Allow rules.
-  Perhaps provide one as a disabled default rule to be enabled when
   needed. Use cases: security breach, maintenance, pissing off people
   on last day at work. :)

Services
--------

-  Display a service as an application identifies it.
-  Perhaps include “Unspecified” ("Unidentified" - blank service name)
   and/or “Unknown” (identified but not listed in IPA) as a Service
   type.
-  Expect the ability to add services to the list as needed.
-  Expect unknown services to be denied by default.

.. _time_ranges:

Time Ranges
-----------

-  Pretty straightforward. The first drop down is intuitive.
-  Weekly: don't have any days checked by default.
-  Deleting a time range and adding a new one is fine (as opposed to
   editing one).
-  Expect a more direct syntax built with drop downs, but understands
   that would prevent more flexible ranges.
-  Would like any overlapping time range attributes within a rule to be
   merged.

.. _general_comments_1:

General Comments
----------------

-  The UI is generally straightforward.
-  Expect a rule to be applied upon creation.
-  Expect to handle multiple networks.
-  Not clear how rules interact, their priority, or what to expect from
   a host running given rules.
-  Not clear how rules work from a network (VS. host) perspective.
-  How modular can we make it? Can users define rules in alternate ways
   (ie, as a regular expression) to control multiple and/or custom
   attributes? (Maybe just alter the configuration file for now, outside
   of the UI.)
-  Can we expressly indicate an exception to a rule? (This would
   apparently be very difficult.)
-  Analysis of how rules are nested or combined would be helpful.
   Perhaps as a log entry, or a fake session to see and test rules
   before applying them.
-  Statistics on how often a rule is applied could help identify
   redundant, broken or unused rules.
-  Add tip(s) indicating that Deny and Allow rules will be applied in
   the order listed.

.. _cloning_roles:

Cloning Roles
=============

-  `Task we
   used <http://www.freeipa.org/wiki/images/b/bc/Task_Clone_Role.pdf>`__
-  `Screens for the
   task <http://www.freeipa.org/wiki/images/b/bb/Clone_Role.pdf>`__

Roles
-----

-  Undefined role icon: The “?” emblem is not intuitive. Suggestion: use
   an open lock, or a plug with/without an outlet.
-  Add and Delete buttons are expected in the column header like on
   other screens.
-  “If I don't have access to manage a role then don't show it.”

.. _ordering_role_priority:

Ordering Role Priority
----------------------

-  Drag-and-drop ordering is preferred.
-  The reason for ordering roles is unclear. Perhaps provide a tip when
   ordering is necessary (per app).

.. _role_relations:

Role Relations
--------------

-  The term “Relations” is confusing until explained. “I expect people
   will think of them as groups.”
-  Perhaps allow a bulk operation to change all relations.
-  The placement of the Add, Clone and Delete buttons above the
   relations is confusing. (But they can apply to several relations in
   that spot.)

.. _cloning_role_relations:

Cloning Role Relations
----------------------

-  Everyone agrees the Clone button is bad. Find out what a common icon
   is (overlapping pages?) or use a text button or link.

.. _adding_an_undefined_role_name:

Adding an Undefined Role Name
-----------------------------

-  Most users did not realize that an unmanaged role name could be
   entered in the combo box. (Could be due to printed screens.)
-  Some users expect to create an unmanaged role in the Roles column
   before dealing with a relation.
-  Users understand that unmanaged roles need to be linked elsewhere.
-  Suggestion: Add the choice “Add new role name...” to the menu.
-  Suggestion: use explicit or hover tips giving an explanation.
-  Suggestion: Include a description in the actual relation saying it's
   not linked.

.. _renaming_roles:

Renaming Roles
--------------

-  If role is renamed or removed they expect the relations to become
   linked to a new name rather than become disconnected and point to
   non-exiting role. (DP: I doubt this can be fixed).

.. _general_comments_2:

General Comments
----------------

-  “If I rename something someplace else it should be reflected here.”
-  Add more text for more direction and explaining what things are for.
-  Perhaps add an icon key.
-  It's intuitive despite problems mentioned above.

Groups
======

-  `Task we
   used <http://www.freeipa.org/wiki/images/b/b1/Task_Groups.pdf>`__
-  `Screens for the
   task <http://www.freeipa.org/wiki/images/3/33/Groups.pdf>`__

.. _list_of_groups:

List of Groups
--------------

-  Prefer ability to search and sort (alpha-numeric, group type).
-  Would like an indication of which groups have child/parent
   relationships. Perhaps use a tree, indented list or drop down box of
   relationships if possible.

.. _making_a_child_or_parent_group:

Making a Child or Parent Group
------------------------------

-  This function was clear and intuitive.
-  Some people expected an opposite action when clicking Add Under Child
   Groups/Parent Groups. Could require a language change or better tips.

.. _group_id_or_gid:

"Group ID” or “GID”?
--------------------

-  “Group ID” (though familiar with GID).
-  “I read GID as Group ID when I see it.”
-  “Either is fine. I prefer GID.”
-  General consensus: use GID.

“POSIX”?
--------

-  POSIX is fine. “It's pretty universal.”

Users
-----

-  Beyond user name, a secondary or even tertiary identifier would be
   nice. They could be toggled on and off or preferred fields could be
   configured.

.. _general_comments_3:

General Comments
----------------

-  “The UI looks slick.”
-  “Pretty straightforward.”
-  “You wouldn't need documentation to use it. You could click around to
   figure it out.”

Netgroups
=========

-  `Task we
   used <http://www.freeipa.org/wiki/images/c/ce/Task_netgroups.pdf>`__
-  `Screens for the
   task <http://www.freeipa.org/wiki/images/0/0f/Netgroups.pdf>`__
-  `Additional
   screens <http://www.freeipa.org/wiki/images/9/96/Netgroups_addon.pdf>`__

.. _adding_all_hosts:

Adding “All Hosts”
------------------

-  “All Hosts” eclipsing individual hosts makes sense.
-  Perhaps make eclipsed individual host names italic and gray.
-  Perhaps add a tip that individual hosts are already included in “All
   Hosts.”
-  In a search and add pop-up, expect eclipsed hosts to reappear if “All
   Hosts” is removed.
-  Expect that eclipsed hosts would not be preserved if “All Hosts” is
   removed from a live list. Perhaps provide a warning before discarding
   eclipsed hosts.
-  Would prefer an option to preserve eclipsed hosts for later use.

.. _managed_and_unmanaged_hosts:

Managed and Unmanaged Hosts
---------------------------

-  “All IPA Managed Hosts” makes sense to everyone.
-  “Individual Hosts” should search IPA managed hosts.
-  Using the check box to add unmanaged hosts checkbox makes sense to
   everyone.
-  Expect a host's icon to change as its state changes
   (managed/unmanaged).
-  An unmanaged host changed to a managed host should be eclipsed by
   “All Hosts.”
-  The possibility of duplicate host names is fine. Perhaps use a
   secondary identifier.

.. _pre_populated_domain_name:

Pre-Populated Domain Name
-------------------------

-  Would be convenient.
-  Perhaps provide a list of commonly used domains to choose from.
-  Do not need to pre-fill.

Automount
=========

-  `Task we
   used <http://www.freeipa.org/wiki/images/c/cc/Task_Automount.pdf>`__
-  `Screens for the
   task <http://www.freeipa.org/wiki/images/8/84/Automount.pdf>`__

.. _separate_screens_for_direct_and_indirect_maps:

Separate Screens for Direct and Indirect Maps
---------------------------------------------

-  Generally intuitive.
-  “The delineation of the two is obvious.”
-  Not obvious on first glance. Didn't expect two separate screens.
   First inclination was to start working on the default page shown, but
   would have figured it out quickly.

Options
-------

-  Make the column wider. “Usually twice as many options.”
-  Expect options to truncate (“...”) if too long for the column row.
-  “You don't need to see all of the options all the time. Options are
   usually standard.”
-  Users would like a quick way of entering options. Perhaps provide a
   menu of frequently used attributes.
-  Look at real world examples of options for better reference.

.. _would_different_mounts_for_different_hosts_be_useful:

Would Different Mounts for Different Hosts be Useful?
-----------------------------------------------------

-  Yes.

.. _editing_mount_points:

Editing Mount Points
--------------------

-  Some confusion over which “Edit” link to click. Perhaps move “Edit
   Properties” closer to Mount Points, or make mount points links.

.. _general_comments_4:

General Comments
----------------

-  Sorting by Source or Options would be helpful.
-  Would like to include inactive mounts in a list.
-  There could be a “scary” amount of mount points in a list.
-  “That was stupid easy.”

.. _help_desk_update_user:

Help Desk: Update User
======================

-  `Task we
   used <http://www.freeipa.org/wiki/images/4/42/Task_Update_User.pdf>`__
-  `Screens for the task (Part
   1) <http://www.freeipa.org/wiki/images/6/6a/Update_user1.pdf>`__
-  `Screens for the task (Part
   2) <http://www.freeipa.org/wiki/images/6/6b/Update_user2.pdf>`__

.. _edit_protected_fields_checkbox:

“Edit Protected Fields” Checkbox
--------------------------------

-  Hard to find. (Could be due to printed screen.)
-  Not hard to find.
-  Change color or make more pronounced.
-  Reset to unchecked state after updating.

.. _link_to_associations_screen:

Link to “Associations” Screen
-----------------------------

-  Not clear at first what will be on that page, but it makes sense once
   you know.
-  Change the name to “Memberships.”

.. _one_vs._two_column_layout:

One VS. Two Column Layout
-------------------------

-  Everyone prefers a single column.
-  Nobody prefers expandable sections. Scrolling is fine.

.. _user_update_my_account:

User: Update My Account
=======================

-  `Task we
   used <http://www.freeipa.org/wiki/images/7/74/Task_Update_My_Account.pdf>`__
-  `Screens for the task (Part
   1) <http://www.freeipa.org/wiki/images/2/29/Update_my1.pdf>`__
-  `Screens for the task (Part
   2) <http://www.freeipa.org/wiki/images/6/61/Update_my2.pdf>`__

.. _non_editable_info_fields:

Non-Editable Info Fields
------------------------

-  “If I can't edit something just don't show it.”
-  “Users don't care about Home Directory.”

.. _general_comments_5:

General Comments
----------------

-  Layout is fairly standard and easy to use.

.. _navigation_tabs:

Navigation Tabs
===============

-  Compact and minimal tabs and toolbars are preferred.
-  The organization of items under tabs was often unexpected, but
   generally made sense once explained or displayed.
-  Expect to find HBAC Rules under the Policies tab. (Correct.)
-  Expect to find Define a Configuration Policy under the Policies tab.
   (Correct.)
-  Expect to find Roles under the Identities tab instead of Policies.
-  Expect to find Groups under the System tab instead of Identities.
-  Expect to find Groups under the Policies tab instead of Identities.
-  Expect to find Netgroups under the Identities tab. (Correct.)
-  Expect to find Netgroups under the Policies tab instead of
   Identities.
-  Expect to find Automount under the System tab instead of Policies. “I
   think of (physically adding) resources instead of policies.”
-  Suggestion: differentiate the System and My Account tabs from the I.
   P. A. tabs. Move them away or change color.
-  Perhaps rename System. “I thought it meant external system (as
   opposed to IPA itself).”

.. _pop_ups:

Pop-ups
=======

-  Using “X” to clear fields: “Xs scare me. I'm worried the window will
   go away.”
-  Using blue arrow icon buttons for adding hosts or users is not clear
   to everyone. “It looks more like a notation.” Perhaps make it more
   like a button and/or add a user or host when clicking either the name
   or the icon.
-  Everyone immediately understood how to search for and add users and
   hosts.
-  Users expect to use the control key to select multiple results, or
   else to see results added to the “Add to...” column as they are
   clicked.
-  Change search functionality to allow wider searches that are then
   filtered. Instead of radio buttons have two checkboxes (Individual
   Users/Hosts, User/Host Groups) and the search field. Always return
   “All Users/Hosts” as a result. (Should “All Users/Hosts” be presented
   a selectable result before performing a search?)

.. _in_different_lists_what_should_the_contents_link_to:

In Different Lists What Should the Contents Link to?
====================================================

-  Expect to see and/or edit user properties.
-  Expect to see extended info on a user's groups, account settings,
   etc.
-  Expect a hover box displaying some info or a blurb, with an Edit
   link.
-  Expect to use a separate tab, screen and/or session.
-  Wouldn't mind using the same screen and session if the Back button
   was smart and work was retained. If not, use a new window or tab.

.. _thoughts_on_puppet:

Thoughts on Puppet
==================

Pros
----

-  Everyone uses Puppet.
-  You can deal with it programatically.
-  It provides “sanity checks.”
-  It's unlikely to syntactically break something.
-  It has the flexibility to accomplish pretty much anything you need to
   do.
-  It can manage configuration of every app in a consistent, repeatable
   manner.
-  You can dynamically add users to different policies.
-  It runs Ruby inside of templates. “Ruby is significantly easier to
   write than XSLT.”
-  It creates an audit trail to ensure consistent changes.
-  It checks hosts, builds its own configuration files, and doesn't
   require manual updates.
-  It compiles updates and applies changes once at the end instead of
   per change.
-  It can create code within GIT repository and push to Puppet Master
   via scripts.
-  It checks to see and alerts you if hosts are up, not just running
   apps.
-  You can disable a service on a machine without stopping Puppet. (It
   must then be manually enabled.)
-  It minimizes work by auto-populating fields as much as possible (90%
   best guess).

Cons
----

-  Not immediately intuitive.
-  More flexibility means greater complexity, which is more prone to
   failures.
-  It's hard to manage a large number of app nodes.
-  It doesn't come with the modules IPA has.
-  It doesn't always do good clean up of access files, flat files, etc.

.. _general_comments_6:

General Comments
----------------

-  Several people suggested using Puppet and IPA together, or
   integrating IPA with other existing apps.
-  IPA could potentially be more useful (than Puppet) if flexible
   enough.

.. _general_comments_and_suggestions:

General Comments and Suggestions
--------------------------------

-  Everyone prefers a command line interface, but many would use
   whatever is fastest and least obtrusive. “UIs are counterintuitive to
   me.” “I worry about what's happening underneath the UI.” “Zimbra's
   interface is ridiculous.”
-  Users worry about having to manually write or manage RNG, XML and
   XSLT files.
-  Everyone would prefer to not spend time configuring things
   day-to-day.
-  Users like to view lots of data at once. The more info on a user the
   better.
-  Users like an uncluttered interface.
-  Users prefer auto-populated and default attributes to reduce
   unnecessary work.
-  Users generally don't read tips.
-  Checking syntax of meta data would be good.
-  Rolling out an app in pieces is preferred to requiring the use of all
   its functionality at once.
-  It would be nice to have a development kit to help in creating or
   translating templates.
-  “The core functionality is definitely there.”
-  “I'm hoping this is something we can implement.”

.. _quick_ui_fixes_and_concerns:

Quick UI Fixes and Concerns
===========================

-  Change Group ID to GID.
-  Add a Logout link next to logged in user name.
-  Add Help as an icon or link to an external documentation or public
   support. For IPA in general, not per function.
-  Account page: change the “Associations” link to “Memberships.”
-  Account page: “Edit Protected Fields” checkbox: change color or make
   more pronounced.
-  Change the “?” emblem in “undefined” icons to an open lock or a plug
   and outlet.
-  Define a Configuration Policy: add a "Save Text" button to the
   Policies Preview window.
-  Define a Configuration Policy: the “Reset All Policies Preview to
   Applied Instances” button is confusing. Perhaps change to “Revert All
   Policies Preview.”
-  Users occasionally expect “Edit Properties” to work as individual
   “Edit” links would, instead of affecting units within a
   group/relation/collection.
-  Differentiate the System and My Account tabs from the I. P. A. tabs.
   Move them away or change color. Perhaps rename “System.”
-  Add tip(s) indicating that Deny and Allow rules will be applied in
   the order listed.
-  Make entries in the lists be links to corresponding objects. This
   applicable to Groups, Netgroups, HBAC, relations etc.

.. _other_ui_changes:

Other UI Changes
================

-  Split lists consisting of multiple columns to columns so size of
   columns can be changed.
-  Allow the selection of users and hosts.
-  Add sorting capabilities by column.
-  Add ability to filter groups by name.
-  Allow dragging item and carrying it around to change order/priority.
-  Add more intuitive automount option management.

.. _best_comment:

Best Comment
============

-  “If we have to use a GUI we'll kill ourselves.”
