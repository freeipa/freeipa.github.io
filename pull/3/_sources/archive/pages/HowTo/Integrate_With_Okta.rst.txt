There are 2 steps to getting `OKTA <https://www.okta.com/>`__ and
FreeIPA to talk together.

.. _the_agent:

1.The Agent
-----------

Download the correct agent and install it on your FreeIPA Server. This
is all well documented and supported within OKTA.

.. _attribute_mapping:

2. Attribute Mapping
--------------------

All these steps are done withing Okta itself, see proposed mappings for
*LDAP Configuration*.

.. _ldap_version:

LDAP Version
~~~~~~~~~~~~

This can be any of them, I chose Sun because it had some of the right
attributes, but it doesn't matter.

Objects
~~~~~~~

-  Unique Identifier Attribute: ``ipauniqueid``
-  DN Attribute - ``dn``

User
~~~~

-  Object Class - ``posixaccount``
-  Account Lock Attribute - ``nsaccountlock``
-  Account Lock Value - ``true``
-  Password Attribute - ``userpassword``
-  Password Expiration Attribute - ``krbpasswordexpiration``

.. _extra_user_attributes:

Extra User Attributes
~~~~~~~~~~~~~~~~~~~~~

I didn't fill any of these out.

Group
~~~~~

-  Object Class - ``posixgroup``
-  Member Attribute - ``member``
-  User Attribute - ``memberof``

Role
~~~~

I'm not sure this actually is mapped correctly:

-  Object Class - ``role``
-  Member Attribute - ``member``

.. _search_base:

Search Base
~~~~~~~~~~~

Replace ``dc=example,dc=com`` with your realm.

-  User Search Base - ``cn=users,cn=accounts,dc=example,dc=com``
-  Group Search Base - ``cn=groups,cn=accounts,dc=example,dc=com``
