Overview
========

Samba 4 uses the following attribute syntaxes which are not supported by
Fedora DS:

-  Printable String
-  UTC Time
-  DN with String
-  Presentation Address

.. _proposed_solution:

Proposed Solution
=================

To fix the problem, they must be mapped into different syntaxes that are
supported by Fedora DS. This can be done by adding the following lines
into samba/source4/setup/schema-map-fedora-ds-1.0:

::

   #Printable String as IA5 String
   1.3.6.1.4.1.1466.115.121.1.44:1.3.6.1.4.1.1466.115.121.1.26
   #UTC Time as Generalized Time
   1.3.6.1.4.1.1466.115.121.1.53:1.3.6.1.4.1.1466.115.121.1.24
   #DN with String as Directory String
   1.2.840.113556.1.4.904:1.3.6.1.4.1.1466.115.121.1.15
   #Presentation Address as Directory String
   1.3.6.1.4.1.1466.115.121.1.43:1.3.6.1.4.1.1466.115.121.1.15

Patch
=====

The patch has been committed in this revision:

-  `s4:provision Fixes for Fedora DS schema mapping with full AD
   schema <http://gitweb.samba.org/?p=samba.git;a=commit;h=a6c9233a128f21dc883cc9534c70eb176214faa5>`__

`Category:Obsolete <Category:Obsolete>`__
