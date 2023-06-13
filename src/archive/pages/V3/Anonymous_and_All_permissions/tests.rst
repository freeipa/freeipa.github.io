\__NOTOC_\_

Test permission bindtype

Implemented in
``ipatests.test_xmlrpc.test_permission_plugin.test_permission_bindtype``

Like other tests in the test_xmlrpc suite, these tests should run on a
clean IPA installation, or possibly after other similar tests.

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermBindRuleType: anonymous``
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

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermBindRuleType: all``
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

| ``dn: cn=testperm1_rn,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm1_rn``
| ``ipaPermBindRuleType: all``
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

| ``dn: cn=testperm1_rn,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm1_rn``
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

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
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
| ``ipa permission_del testperm1_rn --force``
| ``ipa privilege_del testpriv1``
