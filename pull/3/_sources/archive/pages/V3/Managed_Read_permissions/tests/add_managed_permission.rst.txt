First, create a managed permission entry as follows:

| ``dn: cn=testperm,cn=permissions,cn=pbac,$SUFFIX``
| ``cn: testperm``
| ``ipaPermBindRuleType: permission``
| ``ipaPermDefaultAttr: cn``
| ``ipaPermDefaultAttr: l``
| ``ipaPermDefaultAttr: o``
| ``ipaPermLocation: cn=users,cn=accounts,$SUFFIX``
| ``ipaPermRight: write``
| ``ipaPermTarget: uid=*,cn=users,cn=accounts,$SUFFIX``
| ``ipaPermissionType: SYSTEM``
| ``ipaPermissionType: V2``
| ``ipaPermissionType: MANAGED``
| ``objectClass: groupofnames``
| ``objectClass: ipapermission``
| ``objectClass: ipapermissionv2``
| ``objectClass: top``

This corresponds to
``ipa permission-add permission1 --type=user permissions=write``, with
added ``ipaPermissionType`` of ``MANAGED``, and ``ipaPermDefaultAttr``
of ``cn``, ``l``, and ``o``.

Also create the corresponding ACI at ``cn=users,cn=accounts,$SUFFIX``:

``(targetattr = "cn || l || o")(target = "``\ ```ldap:///uid=`` <ldap:///uid=>`__\ ``*,cn=users,cn=accounts,$SUFFIX")(version 3.0;acl "permission:testperm";allow (write) groupdn = "``\ ```ldap:///cn=testperm,cn=permissions,cn=pbac,$SUFFIX`` <ldap:///cn=testperm,cn=permissions,cn=pbac,$SUFFIX>`__\ ``";)``

The first two tests check that this preparation was successful.
