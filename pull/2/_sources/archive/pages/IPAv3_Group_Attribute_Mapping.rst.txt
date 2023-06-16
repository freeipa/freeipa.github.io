Overview
========

This document describes the group attribute mapping from IPA to Samba
and vice versa in various scenarios.

.. _mapping_ipa_group_to_samba_group:

Mapping IPA Group to Samba Group
================================

.. _ipa_group_doesnt_exist_in_samba:

IPA Group Doesn't Exist in Samba
--------------------------------

If an IPA group doesn't have a corresponding Samba group, a new Samba
group should be created with IPA attributes:

+-----------------+---------------------------------------------------+
| Samba Attribute | Source                                            |
+=================+===================================================+
| dn              | CN=<ipa.cn>,CN=Users,DC=domain1,DC=com If this DN |
|                 | conflicts with another DN belonging to a          |
|                 | different entity, a counter will be appended to   |
|                 | the RDN value until it no longer conflicts.       |
+-----------------+---------------------------------------------------+
| objectClass     | group                                             |
+-----------------+---------------------------------------------------+
| cn              | ipa.cn                                            |
+-----------------+---------------------------------------------------+
| sAMAccountName  | ipa.cn                                            |
+-----------------+---------------------------------------------------+
| description     | ipa.description                                   |
+-----------------+---------------------------------------------------+
| member          | transform IPA members into Samba members          |
+-----------------+---------------------------------------------------+

Once the Samba group is created, the IPA group should be updated with
Samba attributes:

============== ================
IPA Attributes Source
============== ================
objectClass    extensibleObject
objectGUID     samba.objectGUID
objectSid      samba.objectSid
============== ================

.. _ipa_group_exists_in_samba_but_not_linked:

IPA Group Exists in Samba but Not Linked
----------------------------------------

If an IPA group has a corresponding Samba group but they are not linked
yet, the Samba group should be updated with IPA attributes:

================ ========================================
Samba Attributes Source
================ ========================================
cn               ipa.cn
description      ipa.description
member           transform IPA members into Samba members
================ ========================================

Once the Samba group is updated, the IPA group should be updated with
Samba attributes:

============== ================
IPA Attributes Source
============== ================
objectClass    extensibleObject
objectGUID     samba.objectGUID
objectSid      samba.objectSid
============== ================

.. _ipa_group_exists_in_samba_and_linked:

IPA Group Exists in Samba and Linked
------------------------------------

If an IPA group has a corresponding Samba group and they are already
linked, the Samba group should be updated with IPA attributes:

================ ========================================
Samba Attributes Source
================ ========================================
cn               ipa.cn
description      ipa.description
member           transform IPA members into Samba members
objectSid        ipa.objectSid
================ ========================================

There is no need to update IPA group.

.. _mapping_samba_group_to_ipa_group:

Mapping Samba Group to IPA Group
================================

.. _samba_group_doesnt_exist_in_ipa:

Samba Group Doesn't Exist in IPA
--------------------------------

A new group should be generated from Samba attributes and added to IPA:

=========== ===========================================
IPA         Samba
=========== ===========================================
dn          cn=,cn=groups,cn=accounts,dc=domain1,dc=com
objectClass groupOfNames, posixGroup, extensibleObject
cn          sAMAccountName
description description
member      transform Samba members into IPA members
objectGUID  objectGUID
objectSid   objectSid
=========== ===========================================

.. _samba_group_exists_in_ipa:

Samba Group Exists in IPA
-------------------------

The Samba group should be updated with IPA attributes:

=========== ========================================
IPA         Samba
=========== ========================================
objectClass extensibleObject
description description
member      transform Samba members into IPA members
objectGUID  objectGUID
objectSid   objectSid
=========== ========================================

`Category:Obsolete <Category:Obsolete>`__
