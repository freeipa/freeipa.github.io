Prerequisites
=============

::

   % yum install python-configobj python-ldap certmonger krb5-workstation \
   pam_krb5 sssd pyOpenSSL python-kerberos python-lxml python-nss \
   krb5-server krb5-server-ldap mod_auth_kerb mod_nss mod_python mod_wsgi \
   openldap-clients pki-ca pki-silent python-pyasn1 slapi-nis nss-pam-ldapd \
   python-netaddr krb5-pkinit-openssl bind bind-dyndb-ldap

Obsolete:

::

   % yum install python-kerberos krb5-server krb5-server-ldap \
   mod_auth_kerb mod_nss mod_python freeradius openldap-clients \
   slapi-nis python-tgexpandingformwidget python-cherrypy pyOpenSSL \
   python-nss python-assets python-wehjit python-lxml

.. _hosts_file:

Hosts File
==========

Edit /etc/hosts:

::

   127.0.0.1     localhost.localdomain localhost
   ::1           localhost6.localdomain6 localhost6
   192.168.1.100 ipa1.example.com ipa1

.. _installing_released_binaries:

Installing Released Binaries
============================

::

   % yum install ipa-server

.. _installing_daily_build:

Installing Daily Build
======================

Create a .repo file in /etc/yum.repos.d:

::

   [ipa-daily]
   name=ipa-daily
   baseurl=http://nalin.fedorapeople.org/freeipa-devel/12/i386/
   gpgcheck=0

.. _building_from_source_code:

Building from Source Code
=========================

::

   % yum install git autoconf automake libtool \
   openldap-devel libcurl-devel xmlrpc-c-devel \
   nspr-devel nss-devel 389-ds-base-devel svrcore-devel mozldap-devel \
   gettext rpm-build python-paste \
   openssl-devel e2fsprogs-devel libcap-devel libuuid-devel \
   python-devel popt-devel python-setuptools python-krbV

Obsolete:

::

   % yum install krb5-devel \
   cyrus-sasl-devel \
   python-ldap \
   python-pyasn1 TurboGears openldap-devel 

::

   % git clone git://git.fedorahosted.org/git/freeipa.git
   % cd freeipa

IPA 1.2:

::

   % git checkout -b ipa-1-2 origin/ipa-1-2
   % make local-dist

IPA 2.0:

::

   % git checkout -b ipa-2-0 origin/HEAD
   % make rpms

The RPMs will be created in dist/rpms directory.

`Category:Obsolete <Category:Obsolete>`__
