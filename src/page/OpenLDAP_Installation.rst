OpenLDAP_Installation
=====================

Prerequisites
=============

.. code-block:: text

   % yum install cvs cyrus-sasl-devel db4-devel libtool-ltdl-devel



Building OpenLDAP
=================

.. code-block:: text

   % export CVSROOT=:pserver:anonymous@cvs.openldap.org:/repo/OpenLDAP
   % cvs login
   Password: OpenLDAP
   % cvs -z3 checkout -P -r OPENLDAP_REL_ENG_2_4_19 ldap

.. code-block:: text

   % cd ldap
   % export CPPFLAGS=-D_GNU_SOURCE
   % ./configure --enable-spasswd --enable-overlays
   % make depend
   % make
   % make install

Alternative configuration:

.. code-block:: text

   % export CFLAGS="-fno-omit-frame-pointer"
   % ./configure --enable-overlays --enable-modules --enable-dynamic



Building JLDAP
==============

.. code-block:: text

   % cvs -z3 checkout -P -r Oct_ndk_2007 jldap
   % cd jldap
   % ant

`Category:Obsolete <Category:Obsolete>`__