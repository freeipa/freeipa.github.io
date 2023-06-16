.. _how_to_start:

How to start
------------

.. _find_something_to_start_with:

Find something to start with
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can either add a new test case that you think is missing in an
existing plan, or you can develop new automated test for a feature or
its aspect that is not covered yet.

Some tips as to what tests would come in handy can be found on `this
Trac page <https://fedorahosted.org/freeipa/report/44>`__ listed under
"Tests" component.

You can also create a test plan for a new feature - to do that, just
pick an RFE ticket and check whether new test plan is needed. To check
this, there are test related fields that indicate the status:

-  Test case - links to a test plan/case in case it is already done
-  Test by - QA contact the issue is assigned to
-  Test coverage - can have four values:

   -  yes - the test is already completed, the Test case and Test by
      fields should already be filled
   -  no - the test is deemed unnecessary or unreasonably laborious to
      create (but you can still create it, if you really want to)
   -  wanted - the test is deemed necessary, but not done yet. Check the
      Test by field - if it's filled, the test plan is in progress, if
      it's empty, it's free to take
   -  empty field - this issue needs review

.. _update_trac_ticket:

Update Trac ticket
~~~~~~~~~~~~~~~~~~

Should you choose to contribute a test with an existing ticket, click
*Modify ticket* and select *accept* in box Action. The owner of a ticket
will be changed to your name and next status of the ticket will be
*assigned*.

If you're working on yet unassigned test for RFE ticket, simply fill out
the *Test by* field.

If you prefer to work on a test with no existing ticket, create a new
ticket for the purpose.

.. _get_the_source:

Get the source
--------------

See `Contribute code <Contribute/Code#Get_the_source>`__ section.

.. _design_a_test_or_a_test_plan:

Design a test or a test plan
----------------------------

The first step to create a new test is designing a test plan. Prepare a
test plan proposal using the `test plan
template <Test_plan_template>`__. If the test plan is designed for a new
feature (RFE ticket), publish the test plan on
`www.freeipa.org/page/V4/feature_name/Test_plan <www.freeipa.org/page/V4/feature_name/Test_plan>`__,
where *feature_name* is the name of the new feature you're testing. Then
send the test plan for comments and approval to
`freeipa-devel <http://www.redhat.com/mailman/listinfo/freeipa-devel>`__
mailing list.

.. _create_a_test_or_test_plan:

Create a test or test plan
--------------------------

The test should be written in Python and using
`pytest <http://pytest.org>`__.

When creating a test, you can follow

-  `Python coding style guide <Python_Coding_Style>`__
-  `Coding best Practices <Coding_Best_Practices>`__

.. _prior_to_starting_the_implementation:

Prior to starting the implementation
------------------------------------

-  Check if the patch/test effort is not duplicated by someone else
   and/or already covered
-  Check if the patch/test requires coverage in other component areas,
   e.g WebUI, upgrade, migration, backup-restore, Trust, DNS,
   external-CA, Multiarch, etc.
-  Check if the change take into consideration availability of packages
   for all distributions

.. _prior_to_committing:

Prior to committing
-------------------

#. Check the logic of the patch/test and its relevance to the original
   issue
#. Check for spelling errors and common mistakes like (not exhaustive):

   #. Exception handling (too broad or too specific)
   #. Unconditional xfails
   #. Hardcoded values
   #. Relying on sleep when there are other mechanism to determine
      readiness (e.g. Selenium’s WebDriverWait)
   #. Checking the proper logs using their respective paths
   #. Commented out code

#. If needed, include PR-CI definitions
#. Execute the bundled code checks using make fastlint. This will check
   for linting errors and PEP8 standards

.. _submit_a_patch:

Submit a patch
--------------

When the test is completed, `submit a
patch <Contribute/Code#Submit_a_patch>`__ much the same way as when
contributing code. Then the patch goes through `code review
process <Contribute/Code#Work_through_Code_Review_process>`__. If the
changes in the patch are approved, they get pushed to the source
repository. The process is also summarized bellow.

.. _committing_doesnt_apply_for_the_temp_commit:

Committing (doesn’t apply for the temp commit)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Follow the commit message guidelines listed in `Commit message
   requirements <Contribute/Code#Commit_message_requirements>`__
#. Signoff and sign your commit
#. All commits must include Pagure link
#. Include separate temp commit (see below)

| ``(lowercase) ipatests : short summary within 80 chars (no dot)``
| ``This is the particular reason for a change and or addition of a new test. Be as specific as possible. Use imperative language (fix bug, not fixed bug nor fixes bug) and present time.``
| ``Fixes (or Related): Pagure ticket link``
| ``Signed off by: (use –signoff and -S when committing)``

Note: “Fixes” is used when providing a fix for the raised issue.
“Related” is used when providing a test for the fix.

.. _temp_commit:

Temp commit
~~~~~~~~~~~

When making changes to the tests, also include a separate temp commit.
The purpose of the temp commit is to execute the new changes in the
PR-CI.

The template of the temp commit is available at
`temp_commit.yaml <https://github.com/freeipa/freeipa/blob/master/ipatests/prci_definitions/temp_commit.yaml>`__

.. _integration_tests:

Integration tests
^^^^^^^^^^^^^^^^^

When changing e.g. test_sudo.py, the author would need to edit
`Line#71 <https://github.com/freeipa/freeipa/blob/master/ipatests/prci_definitions/temp_commit.yaml#L71>`__
to read

``test_integration/test_sudo.py``

`Line#L68 <https://github.com/freeipa/freeipa/blob/master/ipatests/prci_definitions/temp_commit.yaml#L68>`__
should read

``RunPytest ``

.. _webui_tests:

WebUI tests
^^^^^^^^^^^

When changing, e.g. test_loginscreen.py, edit
`Line#71 <https://github.com/freeipa/freeipa/blob/master/ipatests/prci_definitions/temp_commit.yaml#L71>`__
to read

``test_webui/test_loginscreen.py ``

and change
`Line#L68 <https://github.com/freeipa/freeipa/blob/master/ipatests/prci_definitions/temp_commit.yaml#L68>`__
to read

``RunWebuiTests``

Also make sure, the topology selected reflects the topology needed for
the test (e.g.
`Line#L87 <https://github.com/freeipa/freeipa/blob/master/ipatests/prci_definitions/temp_commit.yaml#L87>`__
requires 3 replicas, therefore the topology would be

``*master_3repl_1client``

You can see the list of available topologies at the top of
`temp_commit.yaml <https://github.com/freeipa/freeipa/blob/master/ipatests/prci_definitions/temp_commit.yaml>`__

Last but not least, link the PR-CI definition with the temp commit
definition, i.e.

``$ ln -sf ipatests/prci_definitions/temp_commit.yaml .freeipa-pr-ci.yaml``

executed from the repository root

Note: Don’t execute just the changed test case/test class, but rather
the whole suite.

.. _creating_pull_request:

Creating Pull Request
~~~~~~~~~~~~~~~~~~~~~

#. Provide the same title and summary as in the commit message
#. Add yourself to the list of Assignees
#. Set up the proper labels for backports and state of the work (WIP,
   Needs review)

   #. Ask for a review by assigning a reviewer, if known in advance

.. _reviewing_the_pull_request:

Reviewing the pull request
~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Review all the steps from *Prior to starting the implementation* and
   *Prior to committing*
#. Check the results of PR-CI and make sure the intended test scenario
   was actually executed by checking the tests and the output of
   executed commands.
#. Provide comment/s with your suggestions and/or final statement. Be
   professional and respectful in your comments. When the # review is
   finalized, provide an appropriate label (e.g. ACK).

Merging
~~~~~~~

#. Copy the link to the successful temp commit and include it in the
   comments
#. Delete the temp commit within the PR, so that only the main commit
   with the patch remains.
#. **Don’t use the Merge button within GitHub!** If you have the
   appropriate project permissions, use the `ipa
   tool <https://github.com/freeipa/freeipa-tools>`__ to merge the pull
   request, e.g.

``ipatool pr-push 3406 -r reviewer1 -r reviewer2  -B ipa-4-8 -B ipa-4-7 ``

If you don’t have the permissions, e.g. as an external contributor,
merging will be taken care of, usually by the reviewer.
