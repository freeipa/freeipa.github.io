DS_Installation
===============



Installing Binaries
===================

Install prerequisites:

.. code-block:: text

   % yum install perl-Mozilla-LDAP

To install officially released version:

.. code-block:: text

   % yum install 389-ds-base

To install unreleased version:

.. code-block:: text

   % yum --enablerepo=updates-testing install 389-ds-base



Building from Source Code
=========================

Install prerequisites:

.. code-block:: text

   % yum install git gcc-c++ cyrus-sasl-devel db4-devel krb5-devel libicu-devel \
   mozldap-devel net-snmp-devel nspr-devel nss-devel openssl-devel pam-devel \
   pcre-devel zlib-devel

Get the source code from git repository:

.. code-block:: text

   % git clone git://git.fedorahosted.org/389/ds.git
   % cd ds

Run the following command:

.. code-block:: text

   % rpm --eval '%{configure}'

     CFLAGS="${CFLAGS:--O2 -g}" ; export CFLAGS ;
     CXXFLAGS="${CXXFLAGS:--O2 -g}" ; export CXXFLAGS ;
     FFLAGS="${FFLAGS:--O2 -g}" ; export FFLAGS ;
     ./configure --host=x86_64-unknown-linux-gnu --build=x86_64-unknown-linux-gnu \
           --target=x86_64-redhat-linux \
           --program-prefix= \
           --prefix=/usr \
           --exec-prefix=/usr \
           --bindir=/usr/bin \
           --sbindir=/usr/sbin \
           --sysconfdir=/etc \
           --datadir=/usr/share \
           --includedir=/usr/include \
           --libdir=/usr/lib64 \
           --libexecdir=/usr/libexec \
           --localstatedir=/var \
           --sharedstatedir=/var/lib \
           --mandir=/usr/share/man \
           --infodir=/usr/share/info

Use the output to run the configuration program.

Execute the following command to build & install:

.. code-block:: text

   % make
   % make install



Building RPM
============

Install prerequisites:

.. code-block:: text

   % yum install cvs icu bzip2-devel rpm-build redhat-rpm-config

Prepare build directory:

.. code-block:: text

   % mkdir ~/rpmbuild
   % cd ~/rpmbuild
   % mkdir BUILD BUILDROOT RPMS SOURCES SPECS SRPMS

Create source tarball:

.. code-block:: text

   % cd DS_SRC
   % git archive --prefix=389-ds-base-1.2.3/ master | bzip2 > ~/rpmbuild/SOURCES/389-ds-base-1.2.3.tar.bz2

Download the RPM spec:

.. code-block:: text

   % cvs -d :pserver:anonymous@cvs.fedoraproject.org:/cvs/extras \
   co -r three89-ds-base-1_2_3-1_fc10 389-ds-base

Copy files:

.. code-block:: text

   % cd 389-ds-base/F-10
   % cp 389-ds-base-devel.README ~/rpmbuild/SOURCES
   % cp 389-ds-base-git.sh ~/rpmbuild/SOURCES
   % cp 389-ds-base.spec ~/rpmbuild/SPECS
   % rpmbuild -ba 389-ds-base.spec

References
==========

-  `Building 389 Directory
   Server <http://directory.fedoraproject.org/wiki/Building>`__

`Category:Obsolete <Category:Obsolete>`__