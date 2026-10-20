Mapping_Engine
==============

Overview
========

Mapping Engine is the component that transforms one object into another
using a set of mapping rules.

Mapping
=======

A mapping is a collection of mapping rules. A mapping has the following
properties:

-  Name
-  Mapping rules



Mapping Rule
============

A mapping rule is an instruction for generating an attribute value from
an expression. A mapping rule consist of the following properties:

-  Name
-  Action: add, replace, delete
-  Condition
-  Expression: constant, variable, script



Mapping an Object
=================

One mapping can be used to transform an object. There should be a
mapping for each different scenario. The mapping should include rules
for mapping the DN, object classes, and its attributes.



Mapping a DN
============

The following mapping rule constructs a Samba DN from an IPA attribute:

::

   <rule name="dn">
      <expression>
   return "CN="+ipa.cn+",CN=Users,DC=domain1,DC=com";
      </expression>
   </rule>



Mapping an Object Class
=======================

The object class of an object is usually static, it can be generated
from constant values:

::

   <rule name="objectClass">
       <constant>user</constant>
   </rule>
   <rule name="objectClass" action="add">
       <constant>person</constant>
   </rule>
   <rule name="objectClass" action="add">
       <constant>organizationalPerson</constant>
   </rule>



Mapping an Attribute
====================

Some attributes can be mapped directly from another attribute.

::

   <rule name="sAMAccountName">
       <variable>ipa.uid</variable>
   </rule>

Some attributes require more complex mapping. In the following example
Samba's accountExpires is generated from IPA's krbPasswordExpiration.

::

   <rule name="accountExpires">
       <expression>
   import org.safehaus.penrose.ad.*;
   import org.safehaus.penrose.ipa.*;

   if (ipa.krbPasswordExpiration == void || ipa.krbPasswordExpiration == null) return null;
   return ActiveDirectory.toTimestamp(IPA.toDate(ipa.krbPasswordExpiration));
       </expression>
   </rule>

`Category:Obsolete <Category:Obsolete>`__