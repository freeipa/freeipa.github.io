Samba_4_Installation
====================

Prerequisites
=============

.. code-block:: text

   % yum install gcc git autoconf make python-devel perl-Mozilla-LDAP \
   perl-LDAP phpldapadmin openldap-devel bind ctags-etags



Getting the Source Code
=======================

.. code-block:: text

   % git clone git://git.samba.org/samba.git
   % cd samba

If you need to get a specific version (e.g. release-4-0-0alpha10):

.. code-block:: text

   % git checkout -b devel <tag>

To view the list of tags:

.. code-block:: text

   % git tag

Getting development branch:

.. code-block:: text

   % git remote add github git://github.com/endisd/samba.git
   % git fetch github
   % git checkout --track -b development github/development



Building Samba
==============

.. code-block:: text

   % cd samba/source4
   % ./autogen.sh
   % ./configure.developer --disable-external-libtalloc \
   --disable-external-libtdb --disable-external-libtevent \
   --disable-external-libldb
   % make etags idl_full clean all
   % make install



Testing Samba
=============

To test Samba with standard backend:

.. code-block:: text

   % cd samba/source4
   % make quicktest

To test Samba with OpenLDAP backend:

.. code-block:: text

   % export OPENLDAP_SLAPD=/usr/local/libexec/slapd
   % export TEST_LDAP=yes
   % export TEST_OPTIONS=--ldap=openldap
   % make quicktest

To test Samba with DS backend:

.. code-block:: text

   % export FEDORA_DS_ROOT=/usr
   % export TEST_LDAP=yes
   % export TEST_OPTIONS=--ldap=fedora-ds
   % make quicktest

References
==========

-  `Samba 4 <http://wiki.samba.org/index.php/Samba4>`__
-  `Samba 4/HOWTO <http://wiki.samba.org/index.php/Samba4/HOWTO>`__

`Category:Obsolete <Category:Obsolete>`__