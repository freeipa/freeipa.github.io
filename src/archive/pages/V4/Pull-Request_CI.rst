Overview
--------

During FreeIPA development, existing test cases are often broken by
seemingly unrelated changes in the code. Manual testing of all use cases
isn't feasible. Running the integration test suite manually is not
simple and very time consuming, and thus often skipped. This causes
frequent regressions and malfunctioning code in the master branch.

A proposed solution is to implement on pull-request (PR) continuous
integration (CI) that will run a large portion of the existing test
suite. Currently, we are running single host test on every pull request.
The goal of this effort is to extend our on-PR CI to also execute
multi-host tests. Running the extended test suite should help us to
discover broken test cases early on and stabilize our master branch.



Use Cases
---------

-  As a FreeIPA contributor,

   -  I open a pull request on GitHub against the project. FreeIPA core
      developer verifies no malicious code will be executed during the
      test runs and approves it for testing and PR is automatically
      queued for test execution.
   -  I update the pull-request. The changes have to be approved by a
      core developer once again before the code is queued for test
      execution.

-  As a white-listed FreeIPA contributor,

   -  I open a pull request on GitHub against the project. This PR is
      automatically queued for test execution.
   -  I update the pull request's code. This PR is automatically queued
      for test execution.

Once the test execution finishes, I can see the test results directly in
the PR along with links to publicly accessible logs I can inspect to
debug failures.

Design
------

The main goal of this feature is to enable execution of multi-host tests
on GitHub PRs. The following has to be addressed in this design:

-  Multi-host environment provisioning
-  GitHub integration
-  Task execution
-  Publishing test results and artifcats
-  Deployment of test runners

Requirements
~~~~~~~~~~~~

-  Test results and logs are easily accessible to developers and
   community

   -  It is crucial to be able to track down which test case has failed
      and why
   -  The results must be publicly accessible

-  All test results are available under 1.5 hours

   -  Rapid feedback is a priority, even if that results in a smaller
      test-suite
   -  Up to 3 PRs can be tested simultaneously. Additional PRs may
      increase the time.

.. _overview_1:

Overview
~~~~~~~~

Collection of machines called *test runners* are monitoring GitHub PRs.
Test runners can be running anywhere, they just need to have credentials
for GitHub, access to the internet and must support virtualization. If
the test runner itself is a virtual machine, it has to support nested
virtualization.

A prioritized queue of tasks is constructed from the PRs. These tasks
are builds and individual test suites that are executed atomically. Test
runner picks up a task with the highest priority and starts executing
it. This is announced through `GitHub commit
status <https://developer.github.com/v3/repos/statuses/>`__ update to
avoid collisions with other test runners.

Once the task is finished, a publicly accessible URL with links to the
artifacts (rpms, logs) is announced along with the task result in a
commit status.

.. _multi_host_environment_provisioning:

Multi-host environment provisioning
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hosts are provisioned as virtual machines (VM). To speed up the
provisioning and to alleviate issues with broken dependencies, VM
templates will be created and kept up-to-date. libvirt was chosen as the
provider and the templates themselves will be vargant boxes. The boxes
are published and accessible on `HashiCorp
Atlas <https://atlas.hashicorp.com/freeipa>`__.

.. _github_integration:

GitHub integration
~~~~~~~~~~~~~~~~~~

Test runners communicate through with GitHub through its API. Each pull
request has a collection of `GitHub commit
statuses <https://developer.github.com/v3/repos/statuses/>`__ associated
with it. These are used for all communication. This was inspired by the
`Cockpit
project <https://github.com/cockpit-project/cockpit/tree/master/test>`__.

.. _task_execution:

Task execution
~~~~~~~~~~~~~~

For each PR, a build and a series of test suites have to be executed.
The test runners use a common algorithm to determine the priority of all
waiting tasks for all pull requests. Idle test runner picks the
top-priority task, starts executing it, and marks it in progress.

.. _publishing_test_results_and_artifacts:

Publishing test results and artifacts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once the task finishes, the artifacts (rpms, logs) are uploaded and a
URL is created. Currently, the artifacts are uploaded and stored on
`fedorapeople.org <https://fedorapeople.org/groups/freeipa/>`__,
although any publicly accessible storage can be used.

The produced URL and the test result is announced by the test runner in
the GitHub commit status.

Implementation
--------------

For implementation details, see the `Developer
Documentation <https://github.com/freeipa/freeipa-pr-ci/blob/master/doc/README.md>`__.
