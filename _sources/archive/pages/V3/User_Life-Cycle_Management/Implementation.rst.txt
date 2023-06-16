.. _top_level:

Top Level
=========

.. _stageuser_activate:

stageuser-activate
------------------

When activating a *stage* user, we need to copy (if they exists) the
credential from the *stage* entry to the *active* entry. Helpdesk is
granted the access to do that:

| ``dn: cn=stage users,cn=accounts,cn=provisioning,$SUFFIX``
| ``add:aci: '(targetattr="userPassword || krbPrincipalKey")(version 3.0; acl "Search existence of password and kerberos keys"; allow(read, search) userdn = "``\ ```ldap:///uid=admin,cn=users,cn=accounts,$SUFFIX`` <ldap:///uid=admin,cn=users,cn=accounts,$SUFFIX>`__\ ``";)'``

TODO: need the same aci for 'stage user administrator ' TODO: clarify if
*admin* is helpdesk or support engineer

.. _user_del:

user-del
--------

When the option *--preserve* moves the entry from the *Active* container
to the *Delete* container, we need to remove the credential. This
requires the following aci for the *helpdesk*:

| ``dn: cn=deleted users,cn=accounts,cn=provisioning,$SUFFIX``
| ``add:aci: '(targetattr="userPassword || krbPrincipalKey")(version 3.0; acl "Admins allowed to reset password and kerberos keys"; allow(read, search, write) userdn = "``\ ```ldap:///uid=admin,cn=users,cn=accounts,$SUFFIX`` <ldap:///uid=admin,cn=users,cn=accounts,$SUFFIX>`__\ ``";)``

TODO: clarify if *admin* is helpdesk or support engineer

Authentication
--------------

In order to prevent authentication with *Delete* and *Stage* users, the
following pointer cos is implemented at the *provisioning/accounts*
level:

| ``dn: cn=provisioning account lock,cn=accounts,cn=provisioning,SUFFIX``
| ``objectClass: top``
| ``objectClass: cosSuperDefinition``
| ``objectClass: cosPointerDefinition``
| ``objectClass: ldapSubEntry``
| ``costemplatedn: cn=Inactivation cos template,cn=staged users,cn=provisioning,SUFFIX``
| ``cosAttribute: nsaccountlock operational``
| ``cn: provisioning account lock``
| ``dn: cn=provisioning account lock cos template,cn=accounts,cn=provisioning,SUFFIX``
| ``objectClass: top``
| ``objectClass: extensibleObject``
| ``objectClass: cosTemplate``
| ``cosPriority: 1``
| ``cn: provisioning account lock cos template``
| ``nsAccountLock: true``
