Drop_CSV
========

\__NOTOC_\_

Overview
========

Ticket `3352 <https://fedorahosted.org/freeipa/ticket/3352>`__:

We support CSV in some arguments so one can do things like this:

``   ipa permission-add some-perm --permissions=read,write --attrs=sn,cn``

This introduces great difficulty when trying to include a value with an
embedded space in it. Trying to escape the space in a way that we can
parse and make bash happy can be very difficult.

The level of effort in working around the CSV problems has come to the
point where the benefits of it are outweighed by the problems.

There are several ways to workaround lack of CSV:

Provide an agugment multiple times on the command-line:

``   ipa permission-add some-perm --permissions=read --permissions=write --attrs=sn --attrs=cn``

Let bash do the expansion for you:

``   ipa permission-add some-perm --permissions={read,write} --attrs={sn,cn}``

For convenience, a warning and instructions will be provided if it
appears the user is using the old syntax.



Use Cases
=========

Users of the CLI will no longer be able to use comma-separated lists to
mean multiple values. Instead they must use multiple options, or the
Bash "curly brace" shortcut (or equivalent in other shells). See
Overview for command examples, and Affected Arguments below for all
affected arguments.



Convenience warning/instructions
--------------------------------

This feature depends on JSON-RPC, see Implementation below.

If an invalid formerly-CSV argument is passed to CLI, and it contains a
comma, a message will be shown that explains this change:

| ``   $ ipa permission-add some-perm --permissions=read,write --attrs=sn,cn``
| ``   ipa: WARNING: Comma-separated value lists are no longer supported.``
| ``       Instead of:``
| ``         --permissions=abc,xyz``
| ``       use:``
| ``         --permissions=abc --permissions=xyz``
| ``       or (in Bash):``
| ``         --permissions={abc,xyz}``
| ``   ipa: ERROR: invalid 'permissions': "read,write" is not a valid permission``

| ``   $ ipa permission-add some-perm --permissions={read,write} --attrs={sn,cn}``
| ``   ----------------------------``
| ``   Added permission "some-perm"``
| ``   ----------------------------``
| ``     Permission name: some-perm``
| ``     Permissions: read, write``
| ``     Attributes: sn, cn``

As the change only affects the IPA client, this warning will appear only
there, not in the Web UI or in the API.

Design
======

Remove CSV parsing from client. Remove the "csv_separator" and
"csv_skipspace" Param arguments. The "csv" argument will stay, but will
now mean "this param used CSV in the past".

Modify the JSON transport to send error details ("kw") through the wire.

On the client, if a ValidationError is received on a formerly-CSV
argument whose value contains a comma, show warning & instructions as
above.

Implementation
==============

To know when to display the warning we need extra error information that
we can only get through JSON-RPC (`RFE here <V3/JSON-RPC>`__). If that
is not implemented yet, or if XML-RPC is selected (e.g. via the -e
option to ``ipa``), no warning will be shown. The error message will
stay the same.



Feature Managment
=================

UI

N/A

CLI

See Overview



Major configuration options and enablement
==========================================

N/A

Replication
===========

N/A



Updates and Upgrades
====================

N/A

Dependencies
============

N/A



External Impact
===============

QA: Need to update tests

Docs: Need to update documentation



Affected Arguments
==================

| ``   automember_add_condition: automemberexclusiveregex``
| ``   automember_add_condition: automemberinclusiveregex``
| ``   automember_remove_condition: automemberexclusiveregex``
| ``   automember_remove_condition: automemberinclusiveregex``
| ``   config_mod: ipaconfigstring``
| ``   config_mod: ipagroupobjectclasses``
| ``   config_mod: ipakrbauthzdata``
| ``   config_mod: ipauserobjectclasses``
| ``   dnsconfig_mod: idnsforwarders``
| ``   dnsrecord_add: a6record``
| ``   dnsrecord_add: aaaarecord``
| ``   dnsrecord_add: afsdbrecord``
| ``   dnsrecord_add: aplrecord``
| ``   dnsrecord_add: arecord``
| ``   dnsrecord_add: certrecord``
| ``   dnsrecord_add: cnamerecord``
| ``   dnsrecord_add: dhcidrecord``
| ``   dnsrecord_add: dlvrecord``
| ``   dnsrecord_add: dnamerecord``
| ``   dnsrecord_add: dnskeyrecord``
| ``   dnsrecord_add: dsrecord``
| ``   dnsrecord_add: hiprecord``
| ``   dnsrecord_add: ipseckeyrecord``
| ``   dnsrecord_add: keyrecord``
| ``   dnsrecord_add: kxrecord``
| ``   dnsrecord_add: locrecord``
| ``   dnsrecord_add: mxrecord``
| ``   dnsrecord_add: naptrrecord``
| ``   dnsrecord_add: nsec3paramrecord``
| ``   dnsrecord_add: nsec3record``
| ``   dnsrecord_add: nsec_part_types``
| ``   dnsrecord_add: nsecrecord``
| ``   dnsrecord_add: nsrecord``
| ``   dnsrecord_add: ptrrecord``
| ``   dnsrecord_add: rprecord``
| ``   dnsrecord_add: rrsigrecord``
| ``   dnsrecord_add: sigrecord``
| ``   dnsrecord_add: spfrecord``
| ``   dnsrecord_add: srvrecord``
| ``   dnsrecord_add: sshfprecord``
| ``   dnsrecord_add: tarecord``
| ``   dnsrecord_add: tkeyrecord``
| ``   dnsrecord_add: tsigrecord``
| ``   dnsrecord_add: txtrecord``
| ``   dnsrecord_del: a6record``
| ``   dnsrecord_del: aaaarecord``
| ``   dnsrecord_del: afsdbrecord``
| ``   dnsrecord_del: aplrecord``
| ``   dnsrecord_del: arecord``
| ``   dnsrecord_del: certrecord``
| ``   dnsrecord_del: cnamerecord``
| ``   dnsrecord_del: dhcidrecord``
| ``   dnsrecord_del: dlvrecord``
| ``   dnsrecord_del: dnamerecord``
| ``   dnsrecord_del: dnskeyrecord``
| ``   dnsrecord_del: dsrecord``
| ``   dnsrecord_del: hiprecord``
| ``   dnsrecord_del: ipseckeyrecord``
| ``   dnsrecord_del: keyrecord``
| ``   dnsrecord_del: kxrecord``
| ``   dnsrecord_del: locrecord``
| ``   dnsrecord_del: mxrecord``
| ``   dnsrecord_del: naptrrecord``
| ``   dnsrecord_del: nsec3paramrecord``
| ``   dnsrecord_del: nsec3record``
| ``   dnsrecord_del: nsecrecord``
| ``   dnsrecord_del: nsrecord``
| ``   dnsrecord_del: ptrrecord``
| ``   dnsrecord_del: rprecord``
| ``   dnsrecord_del: rrsigrecord``
| ``   dnsrecord_del: sigrecord``
| ``   dnsrecord_del: spfrecord``
| ``   dnsrecord_del: srvrecord``
| ``   dnsrecord_del: sshfprecord``
| ``   dnsrecord_del: tarecord``
| ``   dnsrecord_del: tkeyrecord``
| ``   dnsrecord_del: tsigrecord``
| ``   dnsrecord_del: txtrecord``
| ``   dnsrecord_find: a6record``
| ``   dnsrecord_find: aaaarecord``
| ``   dnsrecord_find: afsdbrecord``
| ``   dnsrecord_find: aplrecord``
| ``   dnsrecord_find: arecord``
| ``   dnsrecord_find: certrecord``
| ``   dnsrecord_find: cnamerecord``
| ``   dnsrecord_find: dhcidrecord``
| ``   dnsrecord_find: dlvrecord``
| ``   dnsrecord_find: dnamerecord``
| ``   dnsrecord_find: dnskeyrecord``
| ``   dnsrecord_find: dsrecord``
| ``   dnsrecord_find: hiprecord``
| ``   dnsrecord_find: ipseckeyrecord``
| ``   dnsrecord_find: keyrecord``
| ``   dnsrecord_find: kxrecord``
| ``   dnsrecord_find: locrecord``
| ``   dnsrecord_find: mxrecord``
| ``   dnsrecord_find: naptrrecord``
| ``   dnsrecord_find: nsec3paramrecord``
| ``   dnsrecord_find: nsec3record``
| ``   dnsrecord_find: nsecrecord``
| ``   dnsrecord_find: nsrecord``
| ``   dnsrecord_find: ptrrecord``
| ``   dnsrecord_find: rprecord``
| ``   dnsrecord_find: rrsigrecord``
| ``   dnsrecord_find: sigrecord``
| ``   dnsrecord_find: spfrecord``
| ``   dnsrecord_find: srvrecord``
| ``   dnsrecord_find: sshfprecord``
| ``   dnsrecord_find: tarecord``
| ``   dnsrecord_find: tkeyrecord``
| ``   dnsrecord_find: tsigrecord``
| ``   dnsrecord_find: txtrecord``
| ``   dnsrecord_mod: a6record``
| ``   dnsrecord_mod: aaaarecord``
| ``   dnsrecord_mod: afsdbrecord``
| ``   dnsrecord_mod: aplrecord``
| ``   dnsrecord_mod: arecord``
| ``   dnsrecord_mod: certrecord``
| ``   dnsrecord_mod: cnamerecord``
| ``   dnsrecord_mod: dhcidrecord``
| ``   dnsrecord_mod: dlvrecord``
| ``   dnsrecord_mod: dnamerecord``
| ``   dnsrecord_mod: dnskeyrecord``
| ``   dnsrecord_mod: dsrecord``
| ``   dnsrecord_mod: hiprecord``
| ``   dnsrecord_mod: ipseckeyrecord``
| ``   dnsrecord_mod: keyrecord``
| ``   dnsrecord_mod: kxrecord``
| ``   dnsrecord_mod: locrecord``
| ``   dnsrecord_mod: mxrecord``
| ``   dnsrecord_mod: naptrrecord``
| ``   dnsrecord_mod: nsec3paramrecord``
| ``   dnsrecord_mod: nsec3record``
| ``   dnsrecord_mod: nsec_part_types``
| ``   dnsrecord_mod: nsecrecord``
| ``   dnsrecord_mod: nsrecord``
| ``   dnsrecord_mod: ptrrecord``
| ``   dnsrecord_mod: rprecord``
| ``   dnsrecord_mod: rrsigrecord``
| ``   dnsrecord_mod: sigrecord``
| ``   dnsrecord_mod: spfrecord``
| ``   dnsrecord_mod: srvrecord``
| ``   dnsrecord_mod: sshfprecord``
| ``   dnsrecord_mod: tarecord``
| ``   dnsrecord_mod: tkeyrecord``
| ``   dnsrecord_mod: tsigrecord``
| ``   dnsrecord_mod: txtrecord``
| ``   dnszone_add: idnsforwarders``
| ``   dnszone_find: idnsforwarders``
| ``   dnszone_mod: idnsforwarders``
| ``   group_add_member: group``
| ``   group_add_member: ipaexternalmember``
| ``   group_add_member: user``
| ``   group_find: group``
| ``   group_find: in_group``
| ``   group_find: in_hbacrule``
| ``   group_find: in_netgroup``
| ``   group_find: in_role``
| ``   group_find: in_sudorule``
| ``   group_find: no_group``
| ``   group_find: no_user``
| ``   group_find: not_in_group``
| ``   group_find: not_in_hbacrule``
| ``   group_find: not_in_netgroup``
| ``   group_find: not_in_role``
| ``   group_find: not_in_sudorule``
| ``   group_find: user``
| ``   group_remove_member: group``
| ``   group_remove_member: ipaexternalmember``
| ``   group_remove_member: user``
| ``   hbacrule_add_host: host``
| ``   hbacrule_add_host: hostgroup``
| ``   hbacrule_add_service: hbacsvc``
| ``   hbacrule_add_service: hbacsvcgroup``
| ``   hbacrule_add_sourcehost: host``
| ``   hbacrule_add_sourcehost: hostgroup``
| ``   hbacrule_add_user: group``
| ``   hbacrule_add_user: user``
| ``   hbacrule_remove_host: host``
| ``   hbacrule_remove_host: hostgroup``
| ``   hbacrule_remove_service: hbacsvc``
| ``   hbacrule_remove_service: hbacsvcgroup``
| ``   hbacrule_remove_sourcehost: host``
| ``   hbacrule_remove_sourcehost: hostgroup``
| ``   hbacrule_remove_user: group``
| ``   hbacrule_remove_user: user``
| ``   hbacsvcgroup_add_member: hbacsvc``
| ``   hbacsvcgroup_remove_member: hbacsvc``
| ``   hbactest: rules``
| ``   host_add: ipasshpubkey``
| ``   host_add: macaddress``
| ``   host_add_managedby: host``
| ``   host_find: enroll_by_user``
| ``   host_find: in_hbacrule``
| ``   host_find: in_hostgroup``
| ``   host_find: in_netgroup``
| ``   host_find: in_role``
| ``   host_find: in_sudorule``
| ``   host_find: macaddress``
| ``   host_find: man_by_host``
| ``   host_find: man_host``
| ``   host_find: not_enroll_by_user``
| ``   host_find: not_in_hbacrule``
| ``   host_find: not_in_hostgroup``
| ``   host_find: not_in_netgroup``
| ``   host_find: not_in_role``
| ``   host_find: not_in_sudorule``
| ``   host_find: not_man_by_host``
| ``   host_find: not_man_host``
| ``   host_mod: ipasshpubkey``
| ``   host_mod: macaddress``
| ``   host_remove_managedby: host``
| ``   hostgroup_add_member: host``
| ``   hostgroup_add_member: hostgroup``
| ``   hostgroup_find: host``
| ``   hostgroup_find: hostgroup``
| ``   hostgroup_find: in_hbacrule``
| ``   hostgroup_find: in_hostgroup``
| ``   hostgroup_find: in_netgroup``
| ``   hostgroup_find: in_sudorule``
| ``   hostgroup_find: no_host``
| ``   hostgroup_find: no_hostgroup``
| ``   hostgroup_find: not_in_hbacrule``
| ``   hostgroup_find: not_in_hostgroup``
| ``   hostgroup_find: not_in_netgroup``
| ``   hostgroup_find: not_in_sudorule``
| ``   hostgroup_remove_member: host``
| ``   hostgroup_remove_member: hostgroup``
| ``   migrate_ds: exclude_groups``
| ``   migrate_ds: exclude_users``
| ``   migrate_ds: groupignoreattribute``
| ``   migrate_ds: groupignoreobjectclass``
| ``   migrate_ds: groupobjectclass``
| ``   migrate_ds: userignoreattribute``
| ``   migrate_ds: userignoreobjectclass``
| ``   migrate_ds: userobjectclass``
| ``   netgroup_add_member: group``
| ``   netgroup_add_member: host``
| ``   netgroup_add_member: hostgroup``
| ``   netgroup_add_member: netgroup``
| ``   netgroup_add_member: user``
| ``   netgroup_find: group``
| ``   netgroup_find: host``
| ``   netgroup_find: hostgroup``
| ``   netgroup_find: in_netgroup``
| ``   netgroup_find: netgroup``
| ``   netgroup_find: no_group``
| ``   netgroup_find: no_host``
| ``   netgroup_find: no_hostgroup``
| ``   netgroup_find: no_netgroup``
| ``   netgroup_find: no_user``
| ``   netgroup_find: not_in_netgroup``
| ``   netgroup_find: user``
| ``   netgroup_remove_member: group``
| ``   netgroup_remove_member: host``
| ``   netgroup_remove_member: hostgroup``
| ``   netgroup_remove_member: netgroup``
| ``   netgroup_remove_member: user``
| ``   permission_add: attrs``
| ``   permission_add: permissions``
| ``   permission_add_member: privilege``
| ``   permission_find: attrs``
| ``   permission_find: permissions``
| ``   permission_mod: attrs``
| ``   permission_mod: permissions``
| ``   permission_remove_member: privilege``
| ``   privilege_add_member: role``
| ``   privilege_add_permission: permission``
| ``   privilege_remove_member: role``
| ``   privilege_remove_permission: permission``
| ``   role_add_member: group``
| ``   role_add_member: host``
| ``   role_add_member: hostgroup``
| ``   role_add_member: user``
| ``   role_add_privilege: privilege``
| ``   role_remove_member: group``
| ``   role_remove_member: host``
| ``   role_remove_member: hostgroup``
| ``   role_remove_member: user``
| ``   role_remove_privilege: privilege``
| ``   selinuxusermap_add_host: host``
| ``   selinuxusermap_add_host: hostgroup``
| ``   selinuxusermap_add_user: group``
| ``   selinuxusermap_add_user: user``
| ``   selinuxusermap_remove_host: host``
| ``   selinuxusermap_remove_host: hostgroup``
| ``   selinuxusermap_remove_user: group``
| ``   selinuxusermap_remove_user: user``
| ``   service_add_host: host``
| ``   service_find: man_by_host``
| ``   service_find: not_man_by_host``
| ``   service_remove_host: host``
| ``   sudocmdgroup_add_member: sudocmd``
| ``   sudocmdgroup_remove_member: sudocmd``
| ``   sudorule_add_allow_command: sudocmd``
| ``   sudorule_add_allow_command: sudocmdgroup``
| ``   sudorule_add_deny_command: sudocmd``
| ``   sudorule_add_deny_command: sudocmdgroup``
| ``   sudorule_add_host: host``
| ``   sudorule_add_host: hostgroup``
| ``   sudorule_add_runasgroup: group``
| ``   sudorule_add_runasuser: group``
| ``   sudorule_add_runasuser: user``
| ``   sudorule_add_user: group``
| ``   sudorule_add_user: user``
| ``   sudorule_remove_allow_command: sudocmd``
| ``   sudorule_remove_allow_command: sudocmdgroup``
| ``   sudorule_remove_deny_command: sudocmd``
| ``   sudorule_remove_deny_command: sudocmdgroup``
| ``   sudorule_remove_host: host``
| ``   sudorule_remove_host: hostgroup``
| ``   sudorule_remove_runasgroup: group``
| ``   sudorule_remove_runasuser: group``
| ``   sudorule_remove_runasuser: user``
| ``   sudorule_remove_user: group``
| ``   sudorule_remove_user: user``
| ``   trust_find: ipantsidblacklistincoming``
| ``   trust_find: ipantsidblacklistoutgoing``
| ``   trust_mod: ipantsidblacklistincoming``
| ``   trust_mod: ipantsidblacklistoutgoing``
| ``   user_add: ipasshpubkey``
| ``   user_find: in_group``
| ``   user_find: in_hbacrule``
| ``   user_find: in_netgroup``
| ``   user_find: in_role``
| ``   user_find: in_sudorule``
| ``   user_find: not_in_group``
| ``   user_find: not_in_hbacrule``
| ``   user_find: not_in_netgroup``
| ``   user_find: not_in_role``
| ``   user_find: not_in_sudorule``
| ``   user_mod: ipasshpubkey``



RFE author
==========

`Pviktorin <User:Pviktorin>`__; ticket/overview by
`Rcritten <User:Rcritten>`__