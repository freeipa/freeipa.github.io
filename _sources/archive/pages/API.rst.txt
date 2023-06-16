.. _ipa_xml_rpc_api:

IPA XML-RPC API
===============

The XML-RPC can query the remote interface to retrieve the function and
argument list, including any provided help.

-  mark_group_active(cn)

      **Description:** Mark a group as active

-  get_user_by_uid(uid, sattrs)

      **Description:** Get a specific user's entry.
      Return as a dict of values. Multi-valued fields are represented as
      lists.

-  add_service_principal(name)

      **Description:** Given a name of the form: service/FQDN create a
      service principal for it in the default realm.

-  get_all_users()

      **Description:** Return a list containing a User object for each
      existing user.

-  add_member_to_group(member_dn, group_dn)

      **Description:** Add a member to an existing group.

-  get_radius_profile_by_uid(uid, user_profile=True, sattrs=None)

-  multiCall(calls)

      **Description:** Execute a multicall. Execute each method call in
      the calls list, collecting results and errors, and return those as
      a list.

-  delete_user(uid)

      **Description:** Delete a user. Not to be confused with
      inactivate_user. This makes the entry go away completely.
      uid is the uid of the user to delete
      The memberOf plugin handles removing the user from any other
      groups.

-  get_user_by_principal(principal, sattrs)

      **Description:** Get a user entry searching by Kerberos Principal
      Name.
      Return as a dict of values. Multi-valued fields are represented as
      lists.

-  find_radius_clients(ip_attrs, container=None, sattrs=None,
   searchlimit=0, timelimit=-1)

-  get_users_by_manager(manager_dn, sattrs)

      **Description:** Gets the users that report to a particular
      manager.

-  update_radius_client(oldentry, newentry)

-  find_users(criteria, sattrs, searchlimit=-1, timelimit=-1)

      **Description:** Returns a list: counter followed by the results.
      If the results are truncated, counter will be set to -1.

-  system.methodHelp(method)

-  \_listapi()

-  remove_members_from_group(member_dns, group_dn)

      **Description:** Given a list of member dn's remove them from the
      group. Returns a list of the members not removed from the group.

-  get_custom_fields()

      **Description:** Get the list of custom user fields.
      A schema is a list of dict's of the form:
      label: The label dispayed to the user
      field: the attribute name
      required: true/false
      It is displayed to the user in the order of the list.

-  get_aci_entry(sattrs)

      **Description:** Returns the entry containing access control ACIs.

-  add_user_to_group(user_uid, group_dn)

      **Description:** Add a user to an existing group.

-  modifyPassword(principal, oldpass, newpass)

      **Description:** Set/Reset a user's password
      uid tells us who's password to change
      oldpass is the old password (if available)
      newpass is the new password

-  add_radius_profile(profile, user_profile=True)

-  get_groups_by_member(member_dn, sattrs)

      **Description:** Get a specific group's entry. Return as a dict of
      values.
      Multi-valued fields are represented as lists.

-  get_ipa_config()

      **Description:** Retrieve the IPA configuration

-  delete_group(group_dn)

      **Description:** Delete a group
      group_dn is the DN of the group to delete
      The memberOf plugin handles removing the group from any other
      groups.

-  update_ipa_config(oldconfig, newconfig)

      **Description:** Update the IPA configuration.
      oldconfig and newconfig are XML-RPC structs.
      If oldconfig is not empty then it is used when determine what has
      changed.
      If oldconfig is empty then the value of newconfig is compared to
      the current value of oldconfig.

-  mark_user_inactive(uid)

      **Description:** Mark a user as inactive

-  delete_service_principal(principal)

      **Description:** Delete a service principal.
      principal is the full DN of the entry to delete.
      This should be called with much care.

-  attrs_to_labels(attr_list)

      **Description:** Take a list of LDAP attributes and convert them
      to more friendly labels.

-  add_radius_client(client, container=None)

-  get_user_by_email(email, sattrs)

      **Description:** Get a specific user's entry. Return as a dict of
      values.
      Multi-valued fields are represented as lists.

-  get_all_attrs()

      **Description:** We have a list of hardcoded attributes ->
      readable labels. Return that complete list if someone wants it.

-  get_entry_by_cn(cn, sattrs)

      **Description:** Get a specific entry by cn. Return as a dict of
      values.
      Multi-valued fields are represented as lists.

-  find_groups(criteria, sattrs, searchlimit=-1, timelimit=-1)

      **Description:** Return a list containing a User object for each
      existing group that matches the criteria.

-  remove_member_from_group(member_dn, group_dn)

      **Description:** Remove a member_dn from an existing group.

-  add_groups_to_user(group_dns, user_dn)

      **Description:** Given a list of group dn's add them to the user.
      Returns a list of the group dns that were not added.

-  update_user(oldentry, newentry)

      **Description:** Wrapper around update_entry with user-specific
      handling.
      oldentry and newentry are XML-RPC structs.
      If oldentry is not empty then it is used when determine what has
      changed.
      If oldentry is empty then the value of newentry is compared to the
      current value of oldentry.
      If you want to change the RDN of a user you must use this
      function. update_entry will fail.

-  remove_groups_from_user(group_dns, user_dn)

      **Description:** Given a list of group dn's remove them from the
      user.
      Returns a list of the group dns that were not removed.

-  add_group_to_group(group, tgroup)

      **Description:** Add a user to an existing group.
      group is a DN of the group to add
      tgroup is the DN of the target group to be added to

-  find_radius_profiles(uids, user_profile=True, sattrs=None,
   searchlimit=0, timelimit=-1)

-  find_service_principal(criteria, sattrs, searchlimit=-1,
   timelimit=-1)

      **Description:** Returns a list: counter followed by the results.
      If the results are truncated, counter will be set to -1.

-  mark_user_active(uid)

      **Description:** Mark a user as active

-  add_members_to_group(member_dns, group_dn)

      **Description:** Given a list of dn's, add them to the group cn
      denoted by group
      Returns a list of the member_dns that were not added to the group.

-  add_users_to_group(user_uids, group_dn)

      **Description:** Given a list of user uid's add them to the group
      cn denoted by group
      Returns a list of the users were not added to the group.

-  update_radius_profile(oldentry, newentry)

-  group_members(groupdn, attr_list)

      **Description:** Do a memberOf search of groupdn and return the
      attributes in attr_list (an empty list returns everything).

-  mark_group_inactive(cn)

      **Description:** Mark a group as inactive

-  remove_users_from_group(user_uids, group_dn)

      **Description:** Given a list of user uid's remove them from the
      group
      Returns a list of the user uids not removed from the group.

-  add_user(user, user_container)

      **Description:** Add a user in LDAP. Takes as input a dict where
      the key is the attribute name and the value is either a string or
      in the case of a multi-valued field a list of values.
      user_container sets where in the tree the user is placed.

-  delete_radius_client(ip_addr, container=None)

-  get_radius_client_by_ip_addr(ip_addr, container=None, sattrs=None)

-  system.listMethods()

-  system.methodSignature(method)

-  update_group(oldentry, newentry)

      **Description:** Wrapper around update_entry with group-specific
      handling.
      oldentry and newentry are XML-RPC structs.
      If oldentry is not empty then it is used when determine what has
      changed.
      If oldentry is empty then the value of newentry is compared to the
      current value of oldentry.
      If you want to change the RDN of a group you must use this
      function. update_entry will fail.

-  update_entry(oldentry, newentry)

      **Description:** Update an entry in LDAP
      oldentry and newentry are XML-RPC structs.
      If oldentry is not empty then it is used when determine what has
      changed.
      If oldentry is empty then the value of newentry is compared to the
      current value of oldentry.

-  delete_radius_profile(uid, user_profile)

-  get_entry_by_dn(dn, sattrs)

      **Description:** Get a specific entry. Return as a dict of values.
      Multi-valued fields are represented as lists.

-  update_password_policy(oldpolicy, newpolicy)

      **Description:** Update the IPA configuration
      oldpolicy and newpolicy are XML-RPC structs.
      If oldpolicy is not empty then it is used when determine what has
      changed.
      If oldpolicy is empty then the value of newpolicy is compared to
      the current value of oldpolicy.

-  remove_user_from_group(user_uid, group_dn)

      **Description:** Remove a user from an existing group.

-  add_group(group, group_container)

      **Description:** Add a group in LDAP. Takes as input a dict where
      the key is the attribute name and the value is either a string or
      in the case of a multi-valued field a list of values.
      group_container sets where in the tree the group is placed.

-  set_custom_fields(schema)

      **Description:** Set the list of custom user fields.
      A schema is a list of dict's of the form:
      label: The label dispayed to the user
      field: the attribute name
      required: true/false

      It is displayed to the user in the order of the list.

-  get_password_policy()

      **Description:** Retrieve the IPA password policy

`Category:FreeIPA v1 <Category:FreeIPA_v1>`__
