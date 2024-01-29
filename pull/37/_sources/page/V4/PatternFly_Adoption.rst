PatternFly_Adoption
===================

Overview
--------

At the end of year 2013 a project called RCUE was started. This
project's goal was to create a set of common UX principles and patterns
usable in enterprise open source web applications. This project was then
renamed to PatternFly https://www.patternfly.org/. FreeIPA along with
other open source projects such as Foreman, AeroGear, Cockpit,
KeyCloak... decided to adopt these principles. PatternFly has a
referential implementation which is based on Bootstrap 3 frontend
framework. Using this implementation makes the adoption quick.

One of key Bootstrap 3 features is responsiveness. It allows FreeIPA to
abandon absolute position layout and became usable on devices with
various screen sizes.



Use Cases
---------

Every Web UI access.

Design
------

Adoption parts:



Use PatternFly CSS
----------------------------------------------------------------------------------------------

-  PatternFly and Bootstrap 3 uses Less for CSS build. FreeIPA as well,
   however PF and BS uses NodeJS lessc for the task. FreeIPA uses
   lesscpy which is not able to compile PatternFly CSS at the time of
   writing (there is effort to make it possible). Therefore the CSS
   should be built by a developer and updated as necessary. When
   PatternFly and Bootstrap are packaged to Fedora, FreeIPA should make
   effort to use the system packages.



Fluid layout
----------------------------------------------------------------------------------------------

-  abandon absolute layout which cause elements overlap on non-standard
   browser configuration (locale with longer strings, forced bigger
   font). Such overlap usually made breadcrumb navigation unusable.
-  new fluid layout allows to make better use of space on larger
   screens, and on the other hand, usable apps on smaller screens

Navigation
----------------------------------------------------------------------------------------------

-  use Two-Level navigation with dropdown
   <https://www.patternfly.org/wikis/patterns/navigation/primary-navigation-bar/#drop_down_menu>
-  restructure menu items to have only 7 items top in each visible
   level. It's not a hard rule but a UX recommendation (related to
   number of items human brain can remember together)



Implement Login Page
----------------------------------------------------------------------------------------------

-  https://www.patternfly.org/wikis/patterns/user-authentication/loginlogout/
-  required to implement support for standalone pages
-  old unauthorized dialog is replaced by login facet
-  requires to hide dialogs which don't belong to current facet to
   prevent having dialog over login screen when unauthenticated



Adapt forms
----------------------------------------------------------------------------------------------

-  https://www.patternfly.org/widgets/#forms
-  new styles for almost every element
-  requires changes in widget HTML output - mostly css classes
-  text fields are wider because inputs make use of their container's
   space. Therefore undo, delete buttons and others elements should be
   part of form-group or on a next row
-  majority of link buttons should be converted to buttons (at least
   visually)
-  abandon table layout in sections
-  two columns of form sections can be displayed on large screens



Adapt tables
----------------------------------------------------------------------------------------------

-  https://www.patternfly.org/widgets/#tables
-  add "Last" and "First" buttons into pager to support going to the
   last and first page
-  abandoned calculation of column with, leave it on browser.
   Calculation with fluid layout is very difficult, isn't worth the
   hassle.
-  use horizontal scrolling on small screens
-  enable various styles for odd and even rows
-  hovered row should be highlighted



Page(facet) controls
----------------------------------------------------------------------------------------------

-  convert link buttons to regular buttons
-  reorder input elements and buttons in facet header. Search facets
   should have search related elements on the left, as defined in
   PatternFly, and actions on the right. Details facets should have
   everything on the left.



Move actions into action dropdown
----------------------------------------------------------------------------------------------

-  action dropdown is a dropdown list in controls section of a facet
   header
-  old action list and action panels do not play well with fluid layout.
   They tend to clash with nearby elements. All actions should be moved
   to action list. Therefore action list and nearby buttons are a
   central point of invoking actions related to displayed object or list
   of objects. Since all action can be then invoked by a single click
   (when dropdown menu is visible), all actions should require
   confirmation in order to prevent accidental invocation of an action.



Activity notification
----------------------------------------------------------------------------------------------

-  old activity indicator are not very visible and users tend to
   overlook them because they are positioned in a place different from
   user's focus
-  add new global indicator. It should be displayed in a center of a
   page when an ajax operation is active
-  this behavior could be further extended by providing context related
   activity indicators instead of a global one (future goal)



Event notification
----------------------------------------------------------------------------------------------

-  "error", "success", "warning", "info" notification should use
   consistent style and same implementation across application



Better Error handling
----------------------------------------------------------------------------------------------

-  multivalued widget should highlight only inputs with error and do not
   show duplicate required error message
-  error notification should be displayed after fail validation on
   details page instead of in a modal dialog (user doesn't have to click
   on 'ok' button).
-  first focusable widget with error should be automatically focused.
   User can then quickly correct his mistake.



User menu
----------------------------------------------------------------------------------------------

-  user menu should added to navigation (related to
   `#Navigation <#Navigation>`__)
-  it should allow:

   -  password reset
   -  displaying about dialog with current FreeIPA version
   -  logging out
   -  navigating to user's profile

Dependencies
----------------------------------------------------------------------------------------------

fontawesome-fonts, open-sans-fonts, python-lesscpy



Feature Management
------------------

UI

As described in `#Design <#Design>`__ section.

CLI

no impact



Test Plan
---------

existing CI tests were adapted to match new HTML structure



RFE Author
----------

pvoborni