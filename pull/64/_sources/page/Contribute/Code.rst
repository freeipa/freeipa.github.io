Code
====

This article should serve both as a reference article for already
seasoned FreeIPA developers, but also as a guide for new people who
would like to contribute. You can fix an open bug described in chosen
`Pagure ticket <https://pagure.io/freeipa/issues>`__ or implement a
whole new feature! The steps below should help get you started.

If you need to learn more about FreeIPA itself, visit the `main
page <Main_Page>`__.

The Modern WebUI is a separate project that is stored in a submodule of
the FreeIPA repository. It is available at
`freeipa-webui <https://github.com/freeipa/freeipa-webui>`__ and
installed in the directory ``install/freeipa-webui``. You can see the 
issues for Modern WebUI at 
`freeipa-webui issues <https://github.com/freeipa/freeipa-webui/issues>`__.

Most of the development for Modern WebUI is similar to the development of
FreeIPA, for more specifics, please refer to the links above.



Find something to start with
----------------------------

If you want to start with something (relatively) easy please take a look
at `list of "easyfix"
tickets <https://pagure.io/freeipa/issues?status=Open&tags=easyfix>`__
in our Pagure. It is always better to start with something simple :-)

Do not hesitate to `contact us <Contribute#Communication>`__ with any
question or idea!

Prepare
-------

If you are just fixing a simple bug, you can easily continue with the
next steps. However, if you want to work on a bigger feature, a
preparation is advisable:

#. See `General considerations <https://www.freeipa.org/page/General_considerations>`__ article to be
   able to evaluate the scope of changes needed to be done to get a
   bigger feature to FreeIPA
#. Read the `Extend FreeIPA
   guide <http://abbra.fedorapeople.org/guide.html>`__ to see how the
   FreeIPA can be extended
#. Prepare a feature design using the `Feature
   template <https://www.freeipa.org/page/Feature_template>`__ and send it for comments to
   `freeipa-devel <https://lists.fedoraproject.org/archives/list/freeipa-devel@lists.fedorahosted.org/>`__
   list. A feedback from experienced FreeIPA developers will help to
   avoid running into dead ends and also informing the developers about
   your plans. See currently `accepted <https://www.freeipa.org/page/V4_Designs>`__ or `proposed
   designs <https://www.freeipa.org/page/V4_Proposals>`__ for inspiration



Understanding FreeIPA
---------------------

To get started and understand more about the project, you can check the
`freeipa-workshop <https://github.com/freeipa/freeipa-workshop>`__. Is a
guided tour through the core functionalities and how to setup a FreeIPA
environment.



Understanding the source code
----------------------------------------------------------------------------------------------

To have a better understanding of how the code works (how the pieces fit
together, etc), you can read the `"Extending FreeIPA Guide"
here <https://www.freeipa.org/page/Documentation#additional-resources>`__. It's outdated, however,
is still a good source of information.



Get the source
--------------

The source `repository <https://www.freeipa.org/page/Repository>`__ for FreeIPA is stored
in git. Retrieve it and its submodules with:

::

    git clone --recurse-submodules https://pagure.io/freeipa.git

The code can also be browsed online
`here <https://pagure.io/freeipa/commits>`__

Note: to avoid pushing by mistake to pagure.io, you can replace the (push) url
with the following command:

::

   git remote set-url --push origin "You shall not push"


Change the code
---------------

This is the best part - do the changes to FreeIPA code!

Debugging
----------------------------------------------------------------------------------------------

To debug the code you can do:

::

   python -m pdb /usr/bin/ipa <command>

However, sometimes you will need to debug with the server context. Then,
you can use a script like this:

::

   # debug_ipa.py
   from ipalib import api
   from ipapython import ipaldap

   kinit_path = '/home/vagrant/.ccache'  # change to your path here
   api.bootstrap(in_server=True, log=None, context='server')
   api.finalize()
   api.Backend.ldap2.connect(autobind=ipaldap.AUTOBIND_DISABLED, ccache=kinit_path)

   result = api.Command["<command name>"](<command parameters>)

Then, you just need to run:

::

   $ kinit admin -c <path in kinit_path>
   $ sudo python -m pdb debug_ipa.py



Update Pagure ticket
----------------------------------------------------------------------------------------------

We use Pagure for work coordination. Please be so kind and assign the
ticket you are working on to yourself. It is just a few clicks and it
prevents people from working on the same thing at the same time without
knowing about each other.

-  `Find a ticket <https://pagure.io/freeipa/issues>`__ related to your
   work.

   -  Maybe the ticket does not exist. If you consider the code change
      small, just send the patch and do not waste time with paperwork.
      Please create a new ticket if the code change is big (like a new
      feature, rework of existing component etc.) or it needs a backport
      for an older branch. It eases coordination, we can discuss
      different approaches etc.

-  In the particular ticket, assign the ticket to yourself by clicking
   on the "Take" button (note: you must be logged in
   `pagure.io <https://pagure.io/login/?next=https://pagure.io/freeipa/>`__,
   see `Login to pagure or create your
   account <https://docs.pagure.org/pagure/usage/first_steps.html#login-to-pagure-or-create-your-account>`__)



Update code
----------------------------------------------------------------------------------------------

Before completing the patch, please just make sure your contribution
complies with couple simple rules.



Code quality
^^^^^^^^^^^^

-  `C Coding Style Guide <https://www.freeipa.org/page/Coding_Style>`__: Our style guide for C
-  `Python Coding Style Guide <https://www.freeipa.org/page/Python_Coding_Style>`__: Our style guide
   for Python
-  `Coding Best Practices <https://www.freeipa.org/page/Coding_Best_Practices>`__: List of
   IPA-specific best practices that all existing code doesn't yet follow
-  `Schema Handling <https://www.freeipa.org/page/Schema_Handling>`__: How to handle new schema
-  **Comments**: keep in mind that your code will be read and modified
   by other developers. It is important to add meaningful comments that
   will make your intent clear (the code tells us *how* but the comment
   tells us *why*...)



Changes to requirements
^^^^^^^^^^^^^^^^^^^^^^^

If your change requires a new version of our dependency (like
389-ds-base, pki-ca or httpd), make sure that ``freeipa.spec.in`` is
updated and patch descriptions justifies a reason for the change. The
description helps later in analyzing and tracking requirements
differences between FreeIPA versions.

Change to project requirements may also mean a need to regenerate
development repositories, so make sure other developers know about it.



Update documentation
----------------------------------------------------------------------------------------------

Note that if your code change warrants an update in upstream
documentation (especially if the related Trac ticket had *Affects
Documentation* flag checked) you are required to update it as well. See
`Contributing Documentation <https://www.freeipa.org/page/Documentation>`__ page for
details.

Build
-----

See `Build <https://www.freeipa.org/page/Build>`__ for instructions on how to build FreeIPA from
source

Test
----

-  The contribution should not break any existing `tests <https://www.freeipa.org/page/Testing>`__.

-  FreeIPA Makefile provides targets allowing to perform basic tests and
   these tests must be successful:

   -  ``make fastlint`` (pycodestyle and pylint)
   -  ``make fasttest`` (tests which don't require access to IPA API)

For convenience the 'fastcheck' runs 'fasttest' and 'fastlint' with
Python 2 and 3 in one go. A fastcheck takes about half a minute to a
minute to execute: ``make fastcheck``. Please see
`Testing#Fast_test <https://www.freeipa.org/page/Testing#for-developers>`__ for instructions.

-  Important rule: For each ticket, a corresponding test must be
   written.



Create pull request on Github
-----------------------------

Create pull request against **freeipa/freeipa** on
`Github <https://github.com/freeipa/freeipa>`__. Please follow steps
listed here `Pull request on Github <https://www.freeipa.org/page/Pull_request_on_Github>`__ if you
are not sure how to work with pull requests.

Please try to keep commits limited to a single logical change. Multiple
commits in the same pull request are preferred because they allow to
demonstrate the logic and isolate changes.

All pull requests need an associated pagure ticket unless they are
trivial.



Commit message requirements
----------------------------------------------------------------------------------------------

The pull request must contain a commit message following the template
present in the source tree (.git-commit-template):

::

  component: Subject

  Explanation

  Fixes: https://pagure.io/freeipa/issue/XXXX
  or
  Related: https://pagure.io/freeipa/issue/XXXX
  Signed-off-by: Full Name <email@yourmail.com>


-  *component: Subject* is a single-line summary
-  *Explanation* must describe the fix or feature + the method chosen to
   implement it, and can span across multiple lines.

   -  A good commit message allows understanding whether it is related
      to a new feature, an enhancement, a code refactoring or a bugfix.
   -  It needs to provide a context (for instance the issue happens when
      command xx is called) and a high-level description of the approach
      to address the issue, along with potential side-effects.

-  *Fixes:* means that the commit fixes the referenced issue(s).
-  *Related:* means that the commit is related to the issue(s) in some
   way, but does not resolve it/them.
-  *Signed-off-by:* contains your email, will be credited in the Contributors and Release Notes

When a pull request is created, please update Pagure ticket with link to
the pull request (in the ticket, click on "Edit Metadata" and update the
field "on_review" with the link to your PR, for instance
https://github.com/freeipa/freeipa/pull/1234).

The PR will trigger only a subset of the tests. Please keep in mind
that, due to resource limitations, all the tests from the source tree
will not be executed.

We expect you to check if the PR-CI tests are indeed testing your fix.
If some parts of your code are not executed by the PR-CI, then you need
to:

-  check if there are tests in ipatests/ which validate your fix
-  run these tests using the instructions in
   `Testing <https://www.freeipa.org/page/Testing>`__
-  list these tests in your PR
-  mention which commands or scenarios should be thoroughfully checked
   by the reviewer
-  describe the manual tests than you have run

A good habit is also to try to reproduce the issue first: build a
scenario that consistently shows the issue, then implement the fix, and
re-run the same scenario to make sure that the fix is correct.

Once the review is in progress, please remember that the fix is still
under your responsibility as long as no ACK has been given. This means
that you should answer to questions or requests for modifications and
update your PR by taking into accounts all the comments.

If the PR does not progress for a while, you can ask assign the review
to a developer by setting an Assignee (in the PR page, click on
Assignees and pick a reviewer).



Work through Code Review process
--------------------------------



Tracking reviews
----------------------------------------------------------------------------------------------

All review related information is tracked in `pull request
queue <https://github.com/freeipa/freeipa/pulls>`__



PR Submitter responsibilities
----------------------------------------------------------------------------------------------

A patch may not be merged upstream until it has received an approval -
ACK label and passes all mandatory pull request CI checks.

There is an exception to this rule called the "One Liner" rule. When you
have a write access to FreeIPA code base and the patch is trivial (e.g.
only one line changed), it can be pushed upstream without a review, but
it still must pass CI checks. What means trivial? Use your best
judgment.

After a pull request is created, FreeIPA developers will take a look at
it and report any concerns they have. The developer starting a review of
your patch should add his name to *Assigned* field in the pull request
to keep track of the process.

When the reviewer pass a feedback, patch should be then updated to clear
all concerns and thus be ready to be merged. See `Pull request on
Github <https://www.freeipa.org/page/Pull_request_on_Github>`__ for advice how to update a pull
request. No changes in Pagure are needed when a reviewer rejects the
patch or submitter sends a new version of the patch.



PR Reviewer responsibilities
----------------------------------------------------------------------------------------------

You can also contribute to FreeIPA by reviewing pull requests. When you
start reviewing a PR, add your name to the Assignees list in order to
avoid duplication of effort.

The reviewer responsibilities include the following steps:

-  ensure that the `#Code quality <https://www.freeipa.org/page/Contribute/Code#code-quality>`__ requirements are
   met
-  check that the PR-CI was successful (otherwise comment the PR asking
   for a fix, for instance because lint failed ...)
-  check that the PR includes a corresponding test and that the test
   scenario exhibits the issue
-  build with the patch
-  install and run
-  try to consider how you would have fixed the issue and compare with
   the proposed fix if a different strategy was picked
-  try to consider the potential regressions (for instance if a method
   is modified, identify which parts of the code are using this method,
   and check whether they are tested)
-  if the PR-CI does not validate the fix, check if there are existing
   tests that are relevant and launch them, or perform a manual
   verification.

Remember that a reviewer also has a teaching or guiding role: suggest
enhancements or point to existing portions of code that could be reused,
promote good practices and always assume good intent. The PR submitter
may be new to FreeIPA or even new to python development and is certainly
willing to improve and learn from others.

In the review comments, list the checks that you manually did or the
scenario that you tested. Make objective and argumented comments (avoid
"I don't like that..." but rather explain "this should not be done this
way because...") and suggest improvements or alternate strategies when
you request a change.

Finally, when you are OK with the fix, give the ACK label to the PR so
that the fix can be pushed by one of the FreeIPA members.



Enjoy the benefits
------------------

When your contribution succeeds in the code review, it is pushed to our
upstream `repository <Contribute/Repository>`__ and will be part of our
next release. See our `Roadmap <https://www.freeipa.org/page/Roadmap>`__ for details.