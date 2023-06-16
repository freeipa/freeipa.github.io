\__NOTOC_\_

Test permission filters

Implemented in
``ipatests.test_xmlrpc.test_permission_plugin.test_permission_filters``

Test multi-valued filters, type, memberof

Like other tests in the test_xmlrpc suite, these tests should run on a
clean IPA installation, or possibly after other similar tests.

Note: the permission entry will look like this:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermBindRuleType: permission``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTargetFilter: (memberOf=cn=ipausers,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermTargetFilter: (memberof=cn=admins,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermTargetFilter: (objectclass=posixaccount)``
| ``ipaPermTargetFilter: (objectclass=top)``
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
| ``ipaPermLocation: $SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTargetFilter: (memberOf=cn=ipausers,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermTargetFilter: (memberof=cn=admins,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermTargetFilter: (objectclass=ipauser)``
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
| ``ipaPermLocation: $SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTargetFilter: (cn=xyz)``
| ``ipaPermTargetFilter: (objectclass=ipauser)``
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
| ``ipaPermTargetFilter: (memberOf=cn=admins,cn=groups,cn=accounts,$SUFFIX)``
| ``ipaPermTargetFilter: (objectclass=posixaccount)``
| ``ipaPermTargetFilter: (uid=abc)``
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
| ``ipaPermLocation: $SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTargetFilter: (uid=abc)``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

.. _section-1:

Cleanup

``ipa permission_del testperm --force``
