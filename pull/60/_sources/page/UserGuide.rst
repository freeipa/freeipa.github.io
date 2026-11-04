UserGuide
=========

Introduction
============

IPA provides both command-line and browser-based interfaces to the IPA
server. You can use these to manage various aspects of your own account,
and to search for other IPA users and groups. Depending on the
permissions that have been specified by IPA server administrators, you
can also perform more extensive operations, such as modifying other
user's account details.

Before you can log in to the IPA server, the administrator must create
your Kerberos account and provide an initial password. You can then use
these Kerberos credentials to log in to the IPA server from any machine
that has been correctly configured.



Logging in to IPA
-----------------

IPA uses the Kerberos credentials that you provide when you log in to
your machine. To connect to the IPA server, enter the server's address
in your browser. For example, http://myIPAserver.example.com



Managing Your Account
=====================

You can use the IPA Self Service facility to update your own account
information, including your name, display name, password, contact
details, etc. You can also add and remove yourself from groups,
according to the permissions that have been set by the IPA
administrator.



Updating Your Personal Information (Self Service)
-------------------------------------------------

When you first log in to the IPA server, the IPA home page displays.

**To update your personal information:**

#. Click the "Self Service" link in the **Tasks** list on the right side
   of the page to display the **Edit User** page.
#. Update your personal information as required. If you want to change
   your password, select the **edit protected fields** box at the top of
   the page to enable the Password fields.
#. Click **Update User** to save your changes.

A status message is displayed at the top of the page indicating the
success or failure of the update.



Editing Protected Fields
------------------------

Not all of the fields in your account can be edited, and restrictions
may apply to other fields. For example, unless you are a member of the
*admin* or *editors* groups, you cannot change your User Login, Account
Status, or Manager. Password changes are subject to the restrictions
specified in the `Policy
Requirements <Obsolete:Administrators_Guide#Specifying_the_Password_Policy>`__,
which are maintained by the IPA administrator. This helps to maintain
the security of the system by ensuring that you only use strong
passwords, that they are changed regularly, etc.

For advice on how to create strong passwords, refer to `Creating Srong
Passwords <http://www.redhat.com/docs/manuals/enterprise/RHEL-5-manual/en-US/RHEL510/Deployment_Guide/s2-wstation-pass-create.html>`__



Searching for Users and Groups
==============================

IPA provides extensive search capabilities, which enable you to perform
simple and partial-match searches on Given name, userID, phone number,
sn, ou, title and a range of other attributes.

Searches are case-insensitive, and automatically search across multiple
fields. Search results are displayed with exact matches listed first,
followed by partial matches.

You can sort the search results by any column. The default display lists
users in alphabetical order. Click any column title to sort that column
in alphabetical or numerical order. Click the column title again to sort
in reverse order. The sort order is indicated by an icon next to the
title.

Not all fields are indexed for searching. For example, you cannot search
on the following user details:

-  initials
-  account status
-  home directory
-  login shell
-  gecos
-  home page

**To search for users:**

#. Click the "Find User" link in the **Tasks** list on the right side of
   the page to display the **Find User** page.
#. Enter the keywords that you want to search for, and click **Find
   User**.

For example:

-  To find Joe Blake in the Research department, enter "joe blake
   research" (without the quotes).
-  To find who has a particular telephone extension, enter part or all
   of the extension.

**To search for groups:**

#. Click the "Find Group" link in the **Tasks** list on the right side
   of the page to display the **Find Group** page.
#. Enter the keywords that you want to search for, and click **Find
   Group**.

..

   |Note.png| **Note:**

      You cannot search on quoted strings. For example, you cannot
      search for an exact match on "Engineering Group Members".

The following is an example of the results returned by a partial search
by phone number:

.. figure:: FindUserResultsPage.png
   :alt: Results of a partial search by phone number

   Results of a partial search by phone number

.. |Note.png| image:: Note.png