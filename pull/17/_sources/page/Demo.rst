Demo
====

Installing FreeIPA server on supported platforms is a matter of couple
minutes, especially when following `Quick Start
Guide <Quick_Start_Guide>`__. However, for people eager to just try the
looks and feel of the most recent FreeIPA or just to test their `web
application <Web_App_Authentication>`__ with `LDAP <Directory_Server>`__
or `Kerberos <Kerberos>`__ authentication it may just not be fast
enough. For these users, we have prepared a free public instance of
FreeIPA server!

Infrastructure
==============

The FreeIPA server is running on a `Red Hat's
OpenStack <http://www.redhat.com/openstack/>`__ instance, on the latest
stable `Fedora <http://fedoraproject.org/>`__. The server controls a
`DNS <DNS>`__ domain named ``demo1.freeipa.org`` and the correspondiong
`Kerberos <Kerberos>`__ realm ``DEMO1.FREEIPA.ORG``. the server itself
is named

`ipa.demo1.freeipa.org <https://ipa.demo1.freeipa.org>`__

Sandbox
-------

The FreeIPA demo server is just a sandbox and is **wiped clean every
day** at 05:00 UTC. In case you had a testing FreeIPA client enrolled,
the easiest recovery is to uninstall your client
(``ipa-client-install --uninstall`` and `install it
again <Demo#2._Configure_FreeIPA_client>`__). In case you are interested
in a more persistent testing environment, try
`downloading <Downloads>`__ a FreeIPA server on a personal virtual
machine.

Users
-----

The FreeIPA domain is configured with the following users (the password
is **Secret123** for all of them):

-  **admin**: The FreeIPA main administrator account, has all the
   privileges
-  **helpdesk**: A regular user with the *helpdesk* role allowing it to
   modify other users or change their group membership
-  **employee**: A regular user with no special privileges
-  **manager**: A regular user, set as *manager* of the *employee* user

Groups
------

To allow testing group-based authentication we created additional groups
in addition to the default FreeIPA ones:

-  **employees**: contains users *employee* and *manager*
-  **managers**: contains user *manager*

Services
--------

Besides core FreeIPA services (`Directory Server <Directory_Server>`__,
`Kerberos <Kerberos>`__, `PKI <PKI>`__), the server is also configured
with a `DNS <DNS>`__ service and a publicly accessible domain
``demo1.freeipa.org`` to allow both testing the DNS management interface
and DNS dynamic updates.



Quick Start
===========

To test the Web UI, simply go to the hostname link above, log in and
click around! To test integration with a personal system, follow a few
easy steps:



1. Install required packages
----------------------------

::

   # yum install freeipa-client



2. Configure FreeIPA client
---------------------------

Given that the server owns a publicly accessible DNS domain, client can
autodiscover all required information by itself if it knows the right
domain:

::

   # ipa-client-install --domain demo1.freeipa.org -p admin -w Secret123
   Discovery was successful!
   Hostname: mytestclient.demo1.freeipa.org
   Realm: DEMO1.FREEIPA.ORG
   DNS Domain: demo1.freeipa.org
   IPA Server: ipa.demo1.freeipa.org
   BaseDN: dc=demo1,dc=freeipa,dc=org

   Continue to configure the system with these values? [no]: y
   Synchronizing time with KDC...
   Successfully retrieved CA cert
       Subject:     CN=Certificate Authority,O=DEMO1.FREEIPA.ORG
       Issuer:      CN=Certificate Authority,O=DEMO1.FREEIPA.ORG
       Valid From:  Tue Apr 22 06:42:34 2014 UTC
       Valid Until: Sat Apr 22 06:42:34 2034 UTC

   Enrolled in IPA realm DEMO1.FREEIPA.ORG
   Created /etc/ipa/default.conf
   New SSSD config will be created
   Configured /etc/sssd/sssd.conf
   Configured /etc/krb5.conf for IPA realm DEMO1.FREEIPA.ORG
   trying https://ipa.demo1.freeipa.org/ipa/xml
   Forwarding 'ping' to server 'https://ipa.demo1.freeipa.org/ipa/xml'
   Forwarding 'env' to server 'https://ipa.demo1.freeipa.org/ipa/xml'
   Hostname (mytestclient.demo1.freeipa.org) not found in DNS
   DNS server record set to: mytestclient.demo1.freeipa.org -> 1.2.3.4
   Adding SSH public key from /etc/ssh/ssh_host_rsa_key.pub
   Adding SSH public key from /etc/ssh/ssh_host_ecdsa_key.pub
   Forwarding 'host_mod' to server 'https://ipa.demo1.freeipa.org/ipa/xml'
   SSSD enabled
   Configured /etc/openldap/ldap.conf
   NTP enabled
   Configured /etc/ssh/ssh_config
   Configured /etc/ssh/sshd_config
   Client configuration complete.

`Client <Client>`__ is now enrolled in FreeIPA realm and can "see" all
it's users:

::

   $ getent passwd employee
   employee:*:1120000003:1120000003:Test Employee:/home/employee:/bin/sh
   $ getent group employees
   employees:*:1120000005:employee,manager



3. Try to log in as FreeIPA user
--------------------------------

::

   $ host mytestclient.demo1.freeipa.org
   mytestclient.demo1.freeipa.org has address 1.2.3.4

   $ ssh employee@mytestclient.demo1.freeipa.org
   employee@mytestclient.demo1.freeipa.org's password: 
   -sh-4.2$ klist
   Ticket cache: KEYRING:persistent:1658800007:krb_ccache_keVNyW5
   Default principal: employee@DEMO1.FREEIPA.ORG

   Valid starting       Expires              Service principal
   06/04/2014 04:33:25  06/05/2014 04:33:25  krbtgt/DEMO1.FREEIPA.ORG@DEMO1.FREEIPA.ORG



4. Try other features
---------------------

Knock yourself out! Try SUDO, automount, SELinux user role integration,
`certificates <Certmonger>`__ or any other `client <client>`__ features.

FreeIPA team also recommends testing advanced `integration of your web
application <Web_App_Authentication>`__ with identity management system
like FreeIPA and thus having web application with central user
management, Kerberos and authorization, either group based or HBAC
based.

Feedback
========

In case the demo instance is out of order or you would like to ask for
an enhancement, please `contact the FreeIPA
team <Contribute#Communication>`__.



Other Options
=============

We would also like to invite you test our `Docker <Docker>`__ FreeIPA
server images, they should be easy to set up and run on your host
without a need to configure all the virtual machine machinery.