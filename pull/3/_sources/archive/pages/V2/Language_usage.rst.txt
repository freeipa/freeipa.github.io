\__TOC_\_

Back to `Second Round Of UI design <V2/Second_Round_Of_UI_design>`__.

.. _general_language_usage:

General Language Usage
----------------------

.. _containing_vs._contained:

Containing vs. Contained
------------------------

One of the most critical possible points of user confusion will have to
do with the difference between the concept of an object (user, group,
netgroup, etc) as a \*container\* holding other objects and the concept
of an object being \*contained\* by another group.

To design against this confusion, I would like to see us put in as many
points of disambiguation as possible. These include differentiation in
icons, perhaps colors, and, most important, in language.

In particular - it is important the the term referring the "containing
others" and the term referring to "contained by others" be
linguistically different from each other. The terms "Member" and
"Membership", for instance, are too similar to provide much
disambiguation. In the `July 7 <Media:July_7.pdf>`__ set of skeletons, I
introduced a consistent terminology in which these terms were
linguistically different. Dmitry pointed out to me yet another cause of
confusion that my terms added. After some discussion, we came up with a
different set of terms, that I would like to get some feedback on.

Terms
~~~~~

From the point of view of any given object:

-  The objects it contains are \*Members\*
-  The relationships to objects that contain it are \*Enrollments\*

The actions related to them:

-  Contained objects are \*added\* and \*removed\* -- i.e. Members
-  Objects are \*enrolled in\* and \*withdrawn from\* containing objects
   -- i.e. Enrollments

Objects:

-  To create an object, use \*New\*
-  To delete an object, use \*Delete\*

These terms form the basis of the grammar I used in the `July
7 <Media:July_7.pdf>`__ skeletons, and commands, help strings, labels,
and titles are derived from these.

There are several other clues that are designed in as well:

-  The general action paradigm is to:

   -  find one object out of many;
   -  navigate to the proper element of that object;
   -  either add to or delete from the list of objects stored in that
      element

-  For adding Members, I title the screen in the form:

"Add object(s) to thisObject itsName" (e.g. "Add Host Group(s) to
Netgroup foo")

-  For adding Enrollments, I title the screen in a similar, but
   (hopefully) disambiguated way:

   -  "Enroll thisObject itsName in object(s)" (e.g. "Enroll User foo in
      Netgroup(s)")

-  Just because it seems extra weird to enroll an object in its own kind
   of object, I add the word "other" in this case:

   -  "Enroll Netgroup Foo in other Netgroup(s)"

-  The navigation tabs to the separate elements in an object are also
   based on this formula, and disambiguated by whether the tab
   represents objects "contained" in this one, or pointers to other
   "containing" object.
-  For objects "contained" in this one, I use the form:

   -  "Member objects" (e.g. "Member Hosts" or "Member Groups")

-  For pointers to objects "containing" this, I use the form:

   -  "Enrollment in objects" (e.g. "Enrollment in Host Groups" or
      "Enrollment in Groups")

-  I also use the word other when the relationship is for the same kind
   of object:

   -  "Enrollment in other Netgroups"

-  For elements that do not present the ambiguity of contained vs.
   containing, I just use the word: e.g. Details, or Roles.
-  At the head of the list of tabs, I use the type of the current object
   with a colon, suggesting a phase completion.
-  For instance, on the element pages for the Group object, the word
   "Group:" is at the head of the tabs. Possible completions (with the
   tab names) are:

   -  Group: Member Users
   -  Group: Member Groups
   -  Group: Enrollment in other Groups
   -  Group: Enrollment in Netgroups
   -  Group: Details

.. _enrollment_or_not_enrollment:

Enrollment? Or Not Enrollment?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

So here's the challenge: The term "enroll" is overloaded. There are
others that the thesaurus has that are workable: Enlistment ,
Association and Membership

We also have to choose words that fit in the various parts of speech in
which the word is used in the UI. For instance, "Join" is great as a
verb, but there is no noun to describe it (Joinment?).

(As a reference, I've put the list of all phrases that use these related
terms at the end of this note.)

If we choose "Membership", then we need to change the notion of
"Member". Not that it's wrong -- but I want to use another term that is
clearly linguistically different than Membership -- to disambiguate.
Possible words are: affiliate, associate, and constituent.

Ideas
^^^^^

Do we use:

   Member(s) and Enrollment (my first attempt, but conflicts with other
   meanings of Enroll)
   Member(s) and Enlistment
   Member(s) and Association

   Associate(s) and Membership
   Affilate(s) and Membership
   Constituent(s) and Membership

.. _phrases_from_7_july_skeletons:

Phrases from 7-July Skeletons
-----------------------------

-  Enroll Group Foo in Netgroup(s)
-  Enroll Group Foo in other Group(s)
-  Enroll Host Group Foo in Netgroup(s)
-  Enroll Host Group Foo in other Host Group(s)
-  Enroll Netgroup Foo in other Netgroup(s)

-  Enroll Host Foo in Host Group s)
-  Enroll Host Foo in Netgroup(s)
-  Enroll User Pat D. Bunny in Group (s)
-  Enroll User Pat D. Bunny in Netgroup(s)

-  Enroll Host
-  Enroll Host Group
-  Enroll User
-  Enroll Group
-  Enroll Netgroup

-  Enroll in Host Group(s)
-  Enroll in other Host Group(s)
-  Enroll in Group(s)
-  Enroll in other Group(s)
-  Enroll in Netgroup(s)
-  Enroll in other Netgroup(s)

-  Enrollment in Groups
-  Enrollment in other Groups
-  Enrollment in Host Groups
-  Enrollment in other Host Groups
-  Enrollment in Netgroups
-  Enrollment in other Netgroups

-  Prospective Group Enrollment(s)
-  Prospective Host Group Enrollment(s)
-  Prospective Netgroup Enrollment(s)

-  Status\* Enrolled, Kerberos Key Present
-  Enroll via One-Time-Password\*
-  Enrolled By\*
-  Enrolled?
-  Delete Key, Unenroll

-  Add Group(s) to Group Foo
-  Add Group(s) to Netgroup Foo
-  Add Host Group(s) to Host Group Foo
-  Add Host Group(s) to Netgroup Foo
-  Add Host(s) to Host Group Foo
-  Add Host(s) to Netgroup Foo
-  Add Role(s) to User Pat D. Bunny
-  Add User(s) to Group Foo
-  Add User(s) to Netgroup Foo
-  Add Role(s) to User Pat D. Bunny
-  Add Netgroup(s) to Netgroup Foo

-  Add User(s)
-  Add Group(s)
-  Add Host(s)
-  Add Host Group(s)
-  Add Role (s)

-  Remove User(s)
-  Remove Group(s)
-  Remove Role(s)
-  Remove Host Group(s)
-  Remove Host(s)

-  Member Groups
-  Member Host Groups
-  Member Hosts
-  Member Netgroups
-  Member Users

-  Withdraw from Host Group(s)
-  Withdraw from Netgroup(s)
-  Withdraw from Group(s)

-  New User
-  New Group
-  New Service
-  New Netgroup
-  New Host Group
-  New Host
-  New Certificate

-  Delete User (s)
-  Delete Group(s)
-  Delete Service(s)
-  Delete Netgroup(s)
-  Delete Host Group(s)
-  Delete Host(s)
-  Delete Key, Unenroll
-  Delete Key, Unprovision

-  User Details
-  Group Details
-  Netgroup Details
-  Host Group Details
-  Host Details
-  Service Details

-  Issue New Certificate for Host foo
-  Issue New Certificate for Service foo
