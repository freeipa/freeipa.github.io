Repository
==========

FreeIPA project is an open source and as thus offers a free access to
it's source codes as well as transparent policies how the changes to the
code are `planned <Roadmap>`__, `executed <Contribute/Code>`__ and
`released <Release>`__.

Repositories
============

FreeIPA project has 1 active repository:

-  **Main code repository**: FreeIPA project code and tests

   -  Link:
      ```https://pagure.io/freeipa.git`` <https://pagure.io/freeipa.git>`__
   -  `Code browser <https://pagure.io/freeipa/commits/master>`__
   -  `Contribution howto <Contribute/Code>`__



Read access
===========

As said in the beginning, the code can be freely retrieved, feel free to
`build <build>`__ and `test <Testing>`__ and
`contribute <contribute>`__!



Write access
============

Write access is allowed only to a limited list of upstream core
developers (`member list <https://pagure.io/group/freeipa>`__) who are
contributing to the project success, keep the code base clean and
functional. The list is kept short to make sure that all committers know
what should be pushed, when, to which branch(es) and with appropriate
bookkeeping in bug trackers.



Push policy
-----------

After a patch is pushed, committer needs to do the following:

-  Add a name and a mail of the reviewer to the end of the commit
   message in Reviewed-By tag - it is later used for statistics.
   Example:

``Reviewed-By: Joe Hacker <joe@hacker.test>``

-  Confirm that if the patch has a valid ticket it's URL is referenced
   in the patch description
-  Close the associated ticket(s). Include the commit hash for each
   branch. It is equally acceptable to include only the hash or a full
   URL to the upstream change.
-  Add the hash to any associated Bugzillas. If this corrects the
   problem (i.e., all tickets linked to Bugzilla are closed) put the
   Bugzilla into POST state so that downstream is notified about the
   finished work.



Process tools
-------------

To make the committer's life a little bit easier, we developed an
`ipatool <https://github.com/freeipa/freeipa-tools/blob/master/ipatool>`__
which makes sure that the right patches are pushed to the right branches
and that all mandatory patch requirements are fulfilled (like setting
the ``Reviewed-By`` field). It also prepares text snippets with commit
IDs to be added to mail response, Trac ticket or Bugzilla.