\__NOTOC_\_

Test stageuser

Implemented in
``ipatests.test_xmlrpc.test_stageuser_plugin.test_stageuser``

Like other tests in the test_xmlrpc suite, these tests should run on a
clean IPA installation, or possibly after other similar tests.

.. _section-1:

Cleanup

| ``ipa stageuser_del tuser1 --continue``
| ``ipa stageuser_del tuser2 --continue``
| ``ipa stageuser_del tuser --continue``
| ``ipa user_del active_tuser1 --continue``
| ``ipa user_del new_active_tuser1 --continue``
| ``ipa automember_default_group_remove --type=group``

`Category:FreeIPA V4 Test Plan <Category:FreeIPA_V4_Test_Plan>`__
`Category:FreeIPA Test Plan <Category:FreeIPA_Test_Plan>`__
