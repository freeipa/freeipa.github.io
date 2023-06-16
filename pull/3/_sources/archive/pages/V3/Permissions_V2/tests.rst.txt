\__NOTOC_\_

Test permission

Implemented in
``ipatests.test_xmlrpc.test_permission_plugin.test_permission``

Misc. tests for the permission plugin

Like other tests in the test_xmlrpc suite, these tests should run on a
clean IPA installation, or possibly after other similar tests.

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm2,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm2``
| ``ipaPermAllowedAttr: cn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``
| ``owner: cn=test``
| ``owner: cn=test2``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: read``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermTargetFilter: (memberOf=cn=ipausers,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``member: cn=testpriv1,cn=privileges,cn=pbac,$SUFFIX``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``
| ``owner: cn=other-test``
| ``owner: cn=other-test2``

Note: the permission entry cn=testperm,cn=permissions,cn=pbac,$SUFFIX
will not be present

Note: the permission entry will look like this:

| ``dn: cn=testperm1_rn,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm1_rn``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: all``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermTargetFilter: (memberOf=cn=ipausers,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``member: cn=testpriv1,cn=privileges,cn=pbac,$SUFFIX``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``
| ``owner: cn=other-test``
| ``owner: cn=other-test2``

Note: the permission entry
cn=testperm1_rn,cn=permissions,cn=pbac,$SUFFIX will not be present

Note: the permission entry will look like this:

| ``dn: cn=Testperm_RN,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: Testperm_RN``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermTargetFilter: (memberOf=cn=ipausers,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``member: cn=testpriv1,cn=privileges,cn=pbac,$SUFFIX``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``
| ``owner: cn=other-test``
| ``owner: cn=other-test2``

Note: the permission entry will look like this:

| ``dn: cn=Testperm_RN,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: Testperm_RN``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTargetFilter: (memberOf=cn=ipausers,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``member: cn=testpriv1,cn=privileges,cn=pbac,$SUFFIX``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``
| ``owner: cn=other-test``
| ``owner: cn=other-test2``

Note: the permission entry will look like this:

| ``dn: cn=testperm2,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm2``
| ``ipaPermAllowedAttr: cn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``
| ``owner: cn=test``
| ``owner: cn=test2``

Note: the permission entry cn=Testperm_RN,cn=permissions,cn=pbac,$SUFFIX
will not be present

Note: the permission entry cn=testperm2,cn=permissions,cn=pbac,$SUFFIX
will not be present

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermTargetFilter: (memberOf=cn=editors,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermTargetFilter: (memberOf=cn=admins,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry cn=testperm,cn=permissions,cn=pbac,$SUFFIX
will not be present

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermRight: write``
| ``ipaPermTarget: cn=editors,cn=groups,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm3,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm3``
| ``ipaPermAllowedAttr: cn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm3,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm3``
| ``ipaPermAllowedAttr: cn``
| ``ipaPermAllowedAttr: uid``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

.. _section-1:

Cleanup

| ``ipa permission_del testperm --force``
| ``ipa permission_del testperm2 --force``
| ``ipa permission_del testperm3 --force``
| ``ipa permission_del testperm1_rn --force``
| ``ipa permission_del Testperm_RN --force``
| ``ipa privilege_del testpriv1``

.. _section-2:

Test permission rollback

Implemented in
``ipatests.test_xmlrpc.test_permission_plugin.test_permission_rollback``

Test rolling back changes after failed update

Like other tests in the test_xmlrpc suite, these tests should run on a
clean IPA installation, or possibly after other similar tests.

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=admin,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=admin,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=admin,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=admin,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=admin,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=admin,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

.. _section-3:

Cleanup

``ipa permission_del testperm --force``

.. _section-4:

Test permission sync attributes

Implemented in
``ipatests.test_xmlrpc.test_permission_plugin.test_permission_sync_attributes``

Test the effects of setting permission attributes

Like other tests in the test_xmlrpc suite, these tests should run on a
clean IPA installation, or possibly after other similar tests.

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermTargetFilter: (memberOf=cn=admins,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermTargetFilter: (memberOf=cn=admins,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermTargetFilter: (memberOf=cn=admins,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTargetFilter: (memberOf=cn=admins,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=groups,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: cn=*,cn=groups,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=groups,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: cn=editors,cn=groups,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

.. _section-5:

Cleanup

``ipa permission_del testperm --force``

.. _section-6:

Test permission sync nice

Implemented in
``ipatests.test_xmlrpc.test_permission_plugin.test_permission_sync_nice``

Test the effects of setting convenience options on permissions

Like other tests in the test_xmlrpc suite, these tests should run on a
clean IPA installation, or possibly after other similar tests.

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermTargetFilter: (memberOf=cn=admins,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermRight: write``
| ``ipaPermTargetFilter: (memberOf=cn=admins,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermRight: write``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=groups,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: cn=*,cn=groups,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermAllowedAttr: sn``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=groups,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: cn=editors,cn=groups,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

.. _section-7:

Cleanup

``ipa permission_del testperm --force``

.. _section-8:

Test permission flags

Implemented in
``ipatests.test_xmlrpc.test_permission_plugin.test_permission_flags``

Test that permission flags are handled correctly

Like other tests in the test_xmlrpc suite, these tests should run on a
clean IPA installation, or possibly after other similar tests.

.. _section-9:

Cleanup

``ipa permission_del testperm --force``
