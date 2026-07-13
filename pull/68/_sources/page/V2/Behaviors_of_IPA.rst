Behaviors_of_IPA
================

\__TOC_\_

Back to `Second Round Of UI design <V2/Second_Round_Of_UI_design>`__.



IPA UX Notes and Behaviors for Screens
======================================

-  V1 – July 20, 2010 at 8:45am.



Types of Screens
----------------

So far, these are the following classes of screens:

-  Selection Screens: These are the screens that let a user choose an
   individual object from an entire list. These pages are: 110, 118,
   130, 138, 150 and 160.
-  Element page - Details: This is the first page of editing each of the
   group types, and is generally a freely formatted form. These pages
   are 111, 119, 131, 139, 151 and 161.
-  Element Page - Set Container: These pages allow users to add to or
   remove from a set of objects around a particular criteria that are
   all contained by a particular object – for instance, users in a
   group, host groups in a netgroup, or roles of a user. These pages
   are: 112-2, 120, 121, 139-1, 140, 152, 153, 154, 155 and 157
-  Element Page - Set Contained-within: These pages allow users to add
   or remove from the set of objects of a given type that contain a
   particular object – for instance, the groups that a user is contained
   in, or the host groups that are contained in other host groups. These
   pages are 112, 112-1, 122, 123, 132, 132-1, 141, 142 and 158
-  Create Object Pop Up: This pop up allows the user to create a new
   top-level object, and then to either edit it, add another object, or
   cancel. The pop ups contain only the basic information needed to
   create a record. These pages are 110a, 118b, 130a, 138a, 150a, and
   160a.
-  Add to Set Pop Up - Container: This pop up allows users to add
   multiple objects into the particular object that hosts the pop-up.
   These pages are: 112c, 120a, 121a, 139a, 140a, 152a, 153a, 154a,
   155a, and 157a.
-  Add to Set Pop Up – Contained Within:: This pop up allows users to
   add a given object to many different groups that can contain the
   object. These pages are: 112a, 112b, 122a, 123a, 132a, 132b, 141a,
   142a, and 158a.
-  Alert pop-ups: This pop up shows a warning, forcing the user to
   acknowledge something before continuing. This page is: 118a.
-  Certificate Pop-ups: These pop ups are part of the workflow for
   adding a certificate to an object. They are: 131a, 131b, 131c, 161a,
   161b, and 161c.



Common Actions
--------------

Following are some overall actions that are common through all the site:

-  Page Scrolling: Scrolling will be at a page level. There will be no
   separate scrolling regions within a page.
-  Length of Lists (i.e. number of entries in list): All lists of
   objects will siaplay up to a maximum of number of entries. That
   maximum will either be fixed, or be configurable via some mechanism.
   When a page hits the IPA server for a list, if less than the maximum
   are returned, all are displayed. If the maximum or more are returned,
   a message will appear telling the user that only the first number of
   records are being shown, and will encourage users to refine their
   query.
-  Column Headers: Column headers in lists are clickable, and will sort
   the returned entries in first ascending, then descending order for
   that column. A down arrow is shown for descending order, an up arrow
   for ascending order
-  Exiting a page with “dirty” information: Whenever content on a page
   has changed and the user navigates to a new page, an Alert pop up
   will come up that asks the user if he wants to save changes before
   continuing. There will be 3 options, return to page, discard changes
   and continue, save changes and continue.
-  Find String Wildcaring: Standard wildcards (? and \*) should apply in
   find strings



Common Screen Areas
----------------------------------------------------------------------------------------------

Across the site, there are some areas of the screen that either appear
always, or quite often. Here's a description of some of those areas, and
some of the behaviors they have:

Banner
^^^^^^

Product logo, and an indicator who is logged in.

-  The user name is itself a link, and clicking on that link will take
   the user to their own user identity entry page.



Main Nav
^^^^^^^^

Identities, Policis or IPA Configuration.

-  NB: We need to describe behavior when user clicks on main nav...



Secondary Nav
^^^^^^^^^^^^^

For Identities, the nav is Users, Groups, Hosts, Host Groups, Netgroups,
and Services.

-  Clicking on a secondary nav item brings the user to the Selection
   page for that type of object, subject to the “dirty data” policy
   described later.



Page Title
^^^^^^^^^^

Main page status for user

Help
^^^^

The bottom of every page has page-specific help reminders for functions
on that page.



Element Navigator
^^^^^^^^^^^^^^^^^

All sub-pages to each identity object have at top a ribbon that allow
the user to choose different elements of the object to view and edit.

Clicking on any element in the element navigator will bring the user to
that page. However, a check is first done to see if there is any “dirty”
data on screen. This is described later.



Selection Screens
-----------------



Element Pages: Details
----------------------

Common behaviors

-  Each Details screen has a pair of buttons named Reset and Save
   Changes
-  When the editable content on the screen is “clean”, both Reset and
   Save Changes are grayed out.
-  When the editable content on the screen is “dirty”, both buttons are
   active
-  Reset returns the editable fields to the way they were before they
   were changed
-  Save Changes stores all changes in the fields, making the content now
   “clean”, and therefore disabling the Reset and Save changes buttons
-  Details screens that are defined as “long” get Reset and Save buttons
   on the top right and bottom left of the overall Details region
-  Details screens that are defind as “short” get only one set of
   buttons on the top right.
-  Long screens are: 111, 131, and 161
-  Short screens are 119, 139, and 151



Create Object Pop Ups
---------------------

These pop ups all require only the basic information needed to create an
object.

Common behaviors:

-  On entry, all fields are empty
-  At the bottom, there are 3 buttons, “Cancel”, “Create” object, and
   “Create and Edit” object.
-  Cancel returns to the calling page
-  Create attempts to create the object, and then control stays on the
   Pop Up
-  Create and Edit attempts to create the object, and then control goes
   to the “Details” page of the newly created object.
-  The “Create” and “Create and Edit” buttons should be grayed out until
   sufficient text has been entered or options have been chosen in the
   pop up.
-  If there is a fatal error in creating the object, control returns to
   the calling page, with an error message placed in the calling
   screen's message area.



Create Host and Create Service (130a and 160a)
----------------------------------------------------------------------------------------------

-  When the user clicks on the “Resolve” button, the contents of the
   text field are submitted to be resolved.
-  If the string resolves correctly, a message appears under the text
   box saying that the Host Resolves.
-  If the string doesn't resolve, a message appears under the text box
   saying that the Host can not be resolved.
-  If the user clicks on either “Create” or “Create and Edit”, the
   resolve is done automatically
-  If the host name can not be resolved, the pop-up is replaced by
   another alert popup telling the user the host name can not be
   resolved, and asking the user if he wants to continue anyway.
-  Both Create buttons are grayed out until the user enters a host name
-  For Create Service:

   -  The Create buttons are grayed out until a service is chosen
   -  The text box under the pull-down list is disabled unless the user
      selects, “Other” as the service name
   -  In this case, the Create buttons are grayed out until at least one
      character is entered in the Service box

NB: These instructions may be modified when discussion about the DNS
section of IPA is fleshed out and we decide if/how to allow users to
automatically add new host to DNS



Add to Set Pop Ups
------------------

These pop ups all have two lists – a list of available objects (source
list), and a “bag” of objects to add (target list). Common behaviors are
listed below, behaviors particular to individual screens are listed
after.

Common behaviors:

-  On entry, the source list and the target list are both empty
-  Users type into an empty “find” box and press the adjacent “find”
   button
-  In this state, the user must be able to see the whole pop-up,
   including the instructions
-  At any point, if the user clicks “Cancel”, the pop-up will dismiss
   and control will return to the calling page, with no changes made.
-  When the user clicks “Find”, the string is matched against all
   eligible objects, and:

   -  All matched objects that are not already included in the set are
      presented in the \**Source list with a graphic pointing to the
      Target list.
   -  This graphic should have a tool-tip that says, “Add to prospective
      list”
   -  Matched objects that are already included in the set are presented
      as disabled, and do not have a graphic pointing to the Target
      list.

-  When the user clicks the “Add to prospective list” graphic, the item
   disappears from the Source list and appears in the Target list with a
   graphic pointing back to the Source list.
-  This graphic should have a tool-tip that says, “Remove from
   prospective list”
-  Clicking the “Remove from prospective list” graphic causes the item
   to disappear from the Target list and re-appear in the Source list
-  Messages are displayed in the message area after each find or add or
   remove from prospective list.
-  No more than the maximum number of entries are displayed in the
   Source list
-  The entire pop-up scrolls vertically
-  No action is taken until the user clicks the action button on the
   bottom of the pop-up.
-  After the action button is pressed, all items in the Target list are
   added to the Set
-  The success or failure to do the add should be displayed in the
   Message area of the calling screen.
-  Should any of those individual adds fail because the item is already
   in the set – which can happen only if another user beat this user in
   a race condition to add the item – the message in the calling screen
   should be advisory only, and not report an error.



Add Host to Host Group or Net Group Pop-ups (139a and 154a)
----------------------------------------------------------------------------------------------

-  In a special case, the system will allow the user to add a host as
   “unmanaged”. The case is defined as follows:

   -  There are no wildcards in the find string
   -  There are no exact matches with any existing host

-  In this case, the first entry displayed in the Source list would be
   the exact name of the hose in the search string, with the text
   “(unmanaged)” next to it.
-  There would still be a graphic that would allow the name to be added
   to the Target list, and one in the target list to move it back to the
   Source list.

NB: What do we do if there is an unmanaged host with name foo, and a
user later adds a managed host with the name foo. Do we allow it?



Specific Screens
----------------



Decouple Private Group (118a)
----------------------------------------------------------------------------------------------

-  On entry, the “Decouple” button is disabled and the check-box is
   unchecked
-  When the user checks the box, the “Decouple” button becomes active



Group Details (119)
----------------------------------------------------------------------------------------------

-  This note applies only if no data are stored for POSIX Group and GID
-  POSIX group check-box is unchecked and GID Checkbox and data entry
   are grayed out
-  When a user checks the POSIX group box, the GID line becomes active,
   allowing for a manually entered GID