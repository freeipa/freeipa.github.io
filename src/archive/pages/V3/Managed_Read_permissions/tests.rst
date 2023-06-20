tests
=====

\__NOTOC_\_

Test managed permissions

Implemented in
``ipatests.test_xmlrpc.test_permission_plugin.test_managed_permissions``

Like other tests in the test_xmlrpc suite, these tests should run on a
clean IPA installation, or possibly after other similar tests.

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermBindRuleType: permission``
| ``ipaPermDefaultAttr: cn``
| ``ipaPermDefaultAttr: l``
| ``ipaPermDefaultAttr: o``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: MANAGED``
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
| ``ipaPermDefaultAttr: cn``
| ``ipaPermDefaultAttr: l``
| ``ipaPermDefaultAttr: o``
| ``ipaPermExcludedAttr: cn``
| ``ipaPermIncludedAttr: dc``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: MANAGED``
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
| ``ipaPermDefaultAttr: cn``
| ``ipaPermDefaultAttr: l``
| ``ipaPermDefaultAttr: o``
| ``ipaPermExcludedAttr: cn``
| ``ipaPermIncludedAttr: cn``
| ``ipaPermIncludedAttr: sn``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: MANAGED``
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
| ``ipaPermDefaultAttr: cn``
| ``ipaPermDefaultAttr: l``
| ``ipaPermDefaultAttr: o``
| ``ipaPermExcludedAttr: cn``
| ``ipaPermIncludedAttr: cn``
| ``ipaPermIncludedAttr: o``
| ``ipaPermIncludedAttr: sn``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: MANAGED``
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
| ``ipaPermDefaultAttr: cn``
| ``ipaPermDefaultAttr: l``
| ``ipaPermDefaultAttr: o``
| ``ipaPermExcludedAttr: cn``
| ``ipaPermExcludedAttr: sn``
| ``ipaPermIncludedAttr: cn``
| ``ipaPermIncludedAttr: o``
| ``ipaPermIncludedAttr: sn``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: MANAGED``
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
| ``ipaPermDefaultAttr: cn``
| ``ipaPermDefaultAttr: l``
| ``ipaPermDefaultAttr: o``
| ``ipaPermExcludedAttr: cn``
| ``ipaPermExcludedAttr: sn``
| ``ipaPermIncludedAttr: cn``
| ``ipaPermIncludedAttr: o``
| ``ipaPermIncludedAttr: sn``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: MANAGED``
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
| ``ipaPermDefaultAttr: cn``
| ``ipaPermDefaultAttr: l``
| ``ipaPermDefaultAttr: o``
| ``ipaPermExcludedAttr: cn``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: MANAGED``
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
| ``ipaPermDefaultAttr: cn``
| ``ipaPermDefaultAttr: l``
| ``ipaPermDefaultAttr: o``
| ``ipaPermExcludedAttr: cn``
| ``ipaPermIncludedAttr: sn``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: MANAGED``
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
| ``ipaPermDefaultAttr: cn``
| ``ipaPermDefaultAttr: l``
| ``ipaPermDefaultAttr: o``
| ``ipaPermIncludedAttr: sn``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: MANAGED``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``



Cleanup

| ``ipa permission_del testperm --force``
| ``ipa permission_del testperm2 --force``