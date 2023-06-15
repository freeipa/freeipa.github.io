Overview
--------

A LDAP entry contains a set of attributes and values. RFCs related to
LDAP give the possibility to represent an object with different set of
attributes. For example `RFC
4519 <https://www.ietf.org/rfc/rfc4519.txt>`__, allows a group to
contain 'member' or 'uniquemember'. Many applications, like Vsphere,
expect specific attribute (e.g. 'uniquemember') where the LDAP entry
contains an other one (e.g. 'member').

For authorisation purpose, the need of those applications is to retrieve
the groups that a given entry is **uniquemember** of. The groups,
containing *member*, are not retrieved and the application fails to
grant authorisation.

This document describes how the application can retrieve those groups

.. _use_cases7:

Use Cases
---------

An application, like vsphere, uses freeipa (and 389-ds) as its identity
repository. A given user and the groups it belongs to are stored in
389-ds LDAP server. When a user logs into vsphere, vsphere authenticates
and grant authorizations by sending LDAP requests. To retrieve the
groups the user is member of, it sends the requests

::

   [13/Nov/2018:17:29:24.347094829 -0500] conn=814140 op=0 BIND dn="uid=<USER_UID>,cn=users,cn=accounts,<SUFFIX>" method=128 version=3
   [13/Nov/2018:17:29:24.348379396 -0500] conn=814140 op=0 RESULT err=0 tag=97 nentries=0 etime=0 dn="uid=<USER_UID>,cn=users,cn=accounts,<SUFFIX>"
   [13/Nov/2018L17:29:24.348958886 -0500] conn=814140 op=1 UNBIND                                                                                    
   ...
   [13/Nov/2018:17:29:24.382266266 -0500] conn=814141 op=0 BIND dn="uid=ldapsearch,cn=users,cn=accounts,<SUFFIX>" method=128
   version=3
   [13/Nov/2018:17:29:24.383197487 -0500] conn=814141 op=0 RESULT err=0 tag=97 nentries=0 etime=0 dn="uid=ldapsearch,cn=users,cn=accounts,<SUFFIX>"
   [13/Nov/2018:17:29:24.383647394 -0500] conn=814141 op=1 SRCHbase="cn=users,cn=accounts,<SUFFIX>" scope=2 filter="(&(objectClass=inetOrgPerson)(uid=<USER_UID>))" attrs="sn givenName uid entryuuid"
   [13/Nov/2018:17:29:24.384418722 -0500] conn=814141 op=1 RESULT err=0 tag=101 nentries=1 etime=0
   [13/Nov/2018:17:29:24.386156624 -0500] conn=814141 op=2 SRCH base="cn=groups,cn=accounts,<SUFFIX>" scope=2 filter="(&(objectClass=groupOfUniqueNames) (uniqueMember=uid=<USER_UID>,cn=users,cn=accounts,<SUFFIX>))" attrs="cn entryuuid"
   [13/Nov/2018:17:29:24.386799084 -0500] conn=814141 op=2 RESULT err=0 tag=101 nentries=0 etime=0 notes=P pr_idx=0 pr_cookie=-1

So there is a SRCH (conn=814141 op=2) of the groups, assuming the groups
are **groupOfUniqueNames**, that contain the as **uniqueMember**. If the
group uses **member** rather than **uniqueMember** then the SRCH does
not return any groups.

Note that the SRCH that lookup the groups (that contain ) request only
**cn** and **entryuuid**. So the application **does not check** that the
is **uniqueMember** or the returned groups.

In conclusion: the data transformation **only** require a filter
transformation.

.. _how_to_use7:

How to use
----------

Assuming a group does contain ' *member* ' but not ' *uniquemember* ',
the admin should configure filter transformation
`plugin <#plugin_configuration>`__

Then with those data

| ``   # Vsphere group containing 'foo' user``
| ``   dn: cn=vsphere_group,cn=groups,cn=accounts,``
| ``   objectClass: groupofnames``
| ``   member: uid=foo,cn=users,cn=accounts,``
| ``   ``
| ``   # 'foo' user                                                                                                                    ``
| ``   dn: uid=foo,cn=users,cn=accounts,``
| ``   memberOf: cn=ipausers,cn=groups,cn=accounts,``
| ``   memberOf: cn=vsphere_group,cn=groups,cn=accounts,``
| ``   cn: f oo``
| ``   ...``

The following search request will retrieve *vsphere_group* as
*groupOfUniqueNames* having *foo* as *uniqueMember*

| ``   ldapsearch LLL -D "uid=ldapsearch,cn=users,cn=accounts,``\ ``" -W -b "cn=groups,cn=accounts,``\ ``" "(&(objectclass=groupofnames)(member=uid=foo,cn=users,cn=accounts,``\ ``))" cn entryuuid``
| ``   ``
| ``   dn: cn=vsphere_group,cn=groups,cn=accounts,``
| ``   cn: vsphere_group``

Design
------

The following paragraphs detail the evaluated solutions. Among them the
one that is implemented: `transformation of
filter <#transformation_of_filter>`__

.. _aliasing_uniquemember_and_member:

Aliasing uniquemember and member
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The schema contains the definition of the attribute (name, matching
rules, single/multi value, origin...). The schema allows **aliasing**.
That means that if a search filter component contains an attribute name
that is an alias, the test of the attribute present in the evaluated
entry supports any alias value. That also means that if the requested
attribute is an alias, the returned attribute conform the requested
attribute name.

A constraint is all aliased attribute names are sharing the same
matching rules. So if aliasing standardized attribute names they must
have same matching rules and have the same number of allowed value.

An easy way to obtain target attribute in an entry is to alias the
source and the target attribute. For example in 00core.ldif
::

   ``attributeTypes: ( 2.5.4.31 NAME ( 'member' 'uniqueMember' )                                                                                       ``
   ``  SUP distinguishedName``
   ``  X-ORIGIN 'RFC 4519' )``

The benefits are:

-  The set of modifications is very small
-  it is possible to use *uniqueMember* in the filter

The limitations and drawbacks are

-  In case of aliasing *uniqueMember* and *member* it changes a
   standardized attribute definition
-  A search request that does not specify to retrieve *uniqueMember*
   will retrieve *member*. `First <#Search_requesting_all_attributes>`__
   search fails because ''uniqueMember is not returned.
-  A search request that specifies *uniqueMember* will retrieve
   *uniqueMember* but not *member*. This is one attribute or the other
   (replace), there is no option to retrieve both of them.
-  `comparison <#LDAP_compare_the_target_attribute>`__ comparison
   request fails to compare *uniqueMember* (but it could be possible to
   fix this)
-  *uniqueMember* and *member* have different syntax, DS will process
   *uniqueMember* with *member* syntax (and matching rules).

.. _groups_containing_real_attribute_uniquemember:

groups containing real attribute uniquemember
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In order to
`integrate <https://www.howtovmlinux.com/articles/vmware/vcenter/integrate-freeipa-idm-with-vcsa-vcenter-server-for-user-authentications.html>`__
FreeIPA into VSphere, the application has to check for a given user
(e.g. foo) which groups it is member of.

Administrator has to update the LDAP group with *'uniquemember:* ' (*ipa
group-mod --addattr='uniquemember='*), although it already exists '
*member:* '.

It is more complex for the admin, may impact performance as the group
size will double and risky as 'member' and 'uniquemember' must be
updated in sync.

.. _groups_containing_virtual_attribute_uniquemember:

groups containing virtual attribute uniquemember
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The transformation of attribute name could be achieved with MEP plugin
and COS plugin. The MEP plugin is a POST update plugin that allows a
transformation of attribute name into a dedicated placeholder entry
(managed entry).

It requires a change in the UPG config, so that it adds ' *objectclass:
groupofUniquenames* ' to the UPG. Indeed the UPG will eventually contain
as ' *uniquemember* ' the managing entry DN (user).

It requires a new Group Private Group (GPG) config, that the only
purpose is to add the ' *objectclass: mepOriginEntry* ' to the group
where we want to retrieve ' *uniquemember* '.

.. figure:: data_trans_mep_config.png
   :alt: data_trans_mep_config.png

   data_trans_mep_config.png

It also requires a couple of cos definitions in "cascading" definitions.
The first one adds, in the target group, for **each** ' *member* ' user
in the target group, a ' *mepManagedEntry* ' that refers to the user
UPG. The the second cos definition adds, in the target group, for
**each** ' *member* ' user in the target group the ' *uniquemember* '
attribute that is in the user UPG. The value of the ' *uniquemember* '
is the user DN.

.. figure:: Data_trans_cos_config.png
   :alt: Data_trans_cos_config.png

   Data_trans_cos_config.png

The cos apply on groups and generate multivalue attribute. To the
computed values must override any previously existing value. The target
group has a private group (GPG) so it contains ' *mepManagedEntry* '
referring to it. So the cos will override this value. A plugin (e.g. MEP
plugin) that needs to retrieve the original value must flag its search
to ignore virtual attributes.

The solution above works but with limitation

-  It does not work for nested groups.
-  It works for newly created groups and users. Already existing group
   requires to create its GPG. Already existing user requires to update
   its UPG (groupofUniqueName, uniquemember).
-  It requires a change in mep plugins so that when it lookup '
   *mepManagedEntry* ' it should ignore virtual attribute values
   (computed by COS).

The drawbacks is:

-  it is complex, fragile and limited. It involves several plugins with
   their own configuration. Cascading COS is something looking fragile
   as well as hidden attributes (cos hides local ' *memManagedEntry* '
   that is used by MEP).
-  Its performance are poor. It reduces by 10 the response time and by 3
   the throughput.
-  for legacy deployment it requires some changes in UPG and groups.

The advantage is:

-  Require few changes

.. _implement_a_new_ldap_control:

Implement a new LDAP control
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

LDAP V3 allows control. We could implement a 389-ds specific control

| ``   controlValue ::= SEQUENCE OF transformationDesc``
| ``   ``
| ``   transformationDesc ::= SEQUENCE OF {``
| ``   replace          Boolean``
| ``   sourceAttr       attributeDescription,``
| ``   targetAttr       attributeDescription``
| ``   }``

A *transformationDesc* describes the returned attributes of the returned
entries. If a returned entries contains values for *sourceAttr* then it
returns the values with that attribute name *targetAttr*. If *replace*
is True, it does not return *sourceAttr* values but only *targetAttr*
values. If *replace* is False, it returns the values with both
*sourceAttr* and *targetAttr* attribute names.

If *sourceAttr* does not exist then the *transformationDesc* is ignored.

*sourceAttr* can be real, virtual or operation attributes.

The drawback are:

-  It does not addess the `use case <#Use_Cases>`__ where this is the
   filter that needs to be transformed to find the groups whose given
   user is **uniquemember**
-  It requires to publish a new control
-  It requires application code change

Advantages are:

-  It is quite limited change (decoding a control and applying it when
   returning entries)

.. _transformation_of_filter:

transformation of filter
~~~~~~~~~~~~~~~~~~~~~~~~

The use case requires a transformation of the filter component so that

-  the attributename **uniquemember** is replaced with **member**
-  the ava **(objectclass=groupOfUniqueNames)** is replaced with
   **(objectclass=groupOfNames)**.

A new plugin can transform a filter
(*slapi_compute_add_search_rewriter*) with a dedicated callback called
after search preops.

Here is an example of the plugin configuration

| ``   dn: cn=filter transformation,cn=plugins,cn=config``
| ``   objectClass: top``
| ``   objectClass: nsSlapdPlugin``
| ``   objectClass: extensibleObject``
| ``   cn: filter transformation``
| ``   nsslapd-pluginPath: libfiltertransformation-plugin                                                                                                               ``
| ``   nsslapd-pluginInitfunc: fitler_transformation_init``
| ``   nsslapd-pluginType: object``
| ``   nsslapd-pluginEnabled: on``
| ``   nsslapd-plugin-depends-on-type: database``
| ``   nsslapd-plugin-depends-on-named: State Change Plugin``
| ``   nsslapd-pluginId: filterTransformation``
| ``   nsslapd-pluginConfigArea: cn=filterTransformation,cn=etc,SUFFIX``
| ``   nsslapd-pluginDescription: virtual directory information tree views plugin``
| ``   ``
| ``   dn: cn=filterTransformation,cn=etc,``
| ``   objectClass: top``
| ``   objectClass: nsContainer``
| ``   cn: filterTransformation``
| ``   dn: cn=vsphere_uniquemember,cn=filterTransformation,cn=etc,``
| ``   objectClass: top``
| ``   objectClass: filterTransformationDefinition``
| ``   filterTransformationAvaFrom: (uniquemember=*)``
| ``   filterTransformationAvaTo: (member=*)``
| ``   filterTransformationCondScope: subtree``
| ``   filterTransformationCondBase: cn=groups,cn=accounts,``
| ``   filterTransformationCondAttr: cn``
| ``   filterTransformationCondAttr: entryuuid``
| ``   filterTransformationCondBindDn: uid=ldapsearch,cn=users,cn=accounts,``
| ``   cn: vsphere_uniquemember``
| ``   dn: cn=vsphere_objectclass,cn=filterTransformation,cn=etc,``
| ``   objectClass: top``
| ``   objectClass: filterTransformationDefinition``
| ``   filterTransformationAvaFrom: (objectclass=groupOfUniqueNames)``
| ``   filterTransformationAvaTo: (objectclass=groupOfNames)``
| ``   filterTransformationCondScope: subtree``
| ``   filterTransformationCondBase: cn=groups,cn=accounts,``
| ``   filterTransformationCondAttr: cn``
| ``   filterTransformationCondAttr: entryuuid``
| ``   filterTransformationCondBindDn: uid=ldapsearch,cn=users,cn=accounts,``
| ``   cn: vsphere_objectclass``

Definition attributes *filterTransformationCond* are used to restrict
the transformation to specific searches. Indeed some applications,
others than vsphere, may not want those transformation. We can restrict
the transformation to searches with scope
*filterTransformationCondScope*, base search
*filterTransformationCondBase*, requested attributes
*filterTransformationCondAttr* and bound as
*filterTransformationCondBindDn*.

The drawback are:

-  requires to create/deliver/configure a new plugin, but it is not a
   large one
-  It transforms the filter and will return entries that may **not**
   match the original filter. So it is convenient for application that
   does not rely on attributes/values present in the original filter.

The advantages are:

-  it is robust and address the use cases
