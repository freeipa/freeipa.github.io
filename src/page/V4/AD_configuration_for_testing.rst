AD_configuration_for_testing
============================



Windows Server preparation
--------------------------

For the AD-related tests to execute successfully the following
preparations must be done on Windows Server machine:

-  install Active Directory feature and promote machine to Domain
   Controller
-  install Certification Authority feature and setup Certification
   Authority
-  install Cygwin and OpenSSH, configure ssshd server



Object existing in Active Directory
-----------------------------------

On Active Directory side, objects in following description should exist.

The authoritative definitions for PR CI are in `freeipa-pr-ci
<https://github.com/freeipa/freeipa-pr-ci/tree/master/ansible/roles/windows/ipa-ad-data/tasks>`__
(``ad-root.yml`` for the forest root domain, ``ad-child.yml`` for the
child subdomain, ``ad-tree.yml`` for the tree root domain).



On forest root AD
----------------------------------------------------------------------------------------------

-  A test group

::

    name: testgroup
    scope: global
    attributes:
      gidNumber: 10047

-  Group with info attribute and gidNumber defined

::

    name: mytestgroup
    scope: global
    attributes:
      gidNumber: 10055
      info: mytestuser

-  A test group with @ in the name

::

    name: group@group
    scope: global
    attributes:
      gidNumber: 10048

-  A second test group (no gidNumber), used as non-default primary group

::

    name: testgroup1
    scope: global

-  A test user with posix attributes defined

::

    name: testuser
    primary group: testgroup
    first name: Test
    last name: User
    password: Secret123
    password never expires: yes
    attributes:
      uidNumber: 10042
      gidNumber: 10047
      loginShell: /bin/sh
      homeDirectory: /home/testuser
      unixHomeDirectory: /home/testuser
      gecos: Test User

-  A test user with only posix attribute uidNumber defined (no gidNumber
   in OtherAttributes)

::

    name: testuser1
    first name: Test1
    last name: User1
    password: Secret123
    password never expires: yes
    attributes:
      uidNumber: 10050
      loginShell: /bin/sh
      homeDirectory: /home/testuser1
      unixHomeDirectory: /home/testuser1
      gecos: Test User1

-  A test user with gidNumber but no corresponding group in AD

::

    name: testuser2
    first name: Test2
    last name: User2
    password: Secret123
    password never expires: yes
    attributes:
      uidNumber: 10060
      gidNumber: 10049
      loginShell: /bin/sh
      homeDirectory: /home/testuser2
      unixHomeDirectory: /home/testuser2
      gecos: Test User2

-  A test user without posix attributes defined

::

    name: nonposixuser
    first name: Nonposix
    last name: User
    password never expires: yes
    password: Secret123

-  A test user without posix attributes defined and non-default primary
   group (testgroup1)

::

    name: nonposixuser1
    first name: Nonposix1
    last name: User1
    password: Secret123
    password never expires: yes
    primary group: testgroup1

-  A test user with posix attributes which is disabled

::

    name: disabledaduser
    first name: Disabledad
    last name: User
    password: Secret123
    password never expires: yes
    account is disabled: yes
    attributes:
      uidNumber: 10043
      gidNumber: 10047
      loginShell: /bin/sh
      homeDirectory: /home/disableduser

-  A UPN suffix

``UPNsuffix.com`` (added to the forest via ``Set-ADForest -UPNSuffixes``)

-  A user with UPN suffix

::

    name: upnuser
    first name: UPN
    last name: User
    password: Secret123456
    password never expires: yes
    primary group: testgroup
    account logon name (UPN): upnuser@UPNsuffix.com
    attributes:
      uidNumber: 10048
      gidNumber: 10047
      loginShell: /bin/sh
      homeDirectory: /home/upnuser
      gecos: UPN User

-  A test user with posix attributes defined with same gidNumber as
   mytestgroup

::

    name: mytestuser
    first name: Test
    last name: User
    password: Secret123
    password never expires: yes
    primary group: mytestgroup
    attributes:
      uidNumber: 10055
      gidNumber: 10055
      loginShell: /bin/sh
      homeDirectory: /home/mytestuser
      unixHomeDirectory: /home/mytestuser
      gecos: Test User

-  A test user with expired AD account (accountExpires in the past)

::

    name: expiredaduser
    first name: Expiredad
    last name: User
    password: Secret123
    password never expires: yes
    account expiration: one day in the past (relative to provisioning)
    attributes:
      uidNumber: 10059
      gidNumber: 10047
      loginShell: /bin/sh
      homeDirectory: /home/expiredaduser
      unixHomeDirectory: /home/expiredaduser
      gecos: Expired AD User



On child (subdomain) AD
----------------------------------------------------------------------------------------------

-  A user group

::

    name: subdomaintestgroup
    scope: global
    attributes:
      gidNumber: 10147

-  A test user with posix attributes defined

::

    name: subdomaintestuser
    first name: Subdomaintest
    last name: User
    password: Secret123
    password never expires: yes
    primary group: subdomaintestgroup
    attributes:
      uidNumber: 10142
      gidNumber: 10147
      loginShell: /bin/sh
      homeDirectory: /home/subdomaintestuser
      gecos: Subdomaintest User

-  A second test user with posix attributes defined (same group)

::

    name: subdomaintestuser2
    first name: Subdomaintest2
    last name: User2
    password: Secret123
    password never expires: yes
    attributes:
      uidNumber: 10144
      gidNumber: 10147
      loginShell: /bin/sh
      homeDirectory: /home/subdomaintestuser2
      gecos: Subdomaintest User2

-  A test user with posix attributes which is disabled

::

    name: subdomaindisabledadu
    account logon name: subdomaindisabledaduser@CHILD_DOMAIN_NAME
    password: Secret123
    password never expires: yes
    account is disabled: yes
    attributes:
      uidNumber: 10143
      gidNumber: 10147
      loginShell: /bin/sh
      homeDirectory: /home/subdomaindisableduser

-  A test user without posix attributes defined

::

    name: subnonposixuser1
    first name: subNonposix1
    last name: subUser1
    password: Secret123
    password never expires: yes

-  A second test user without posix attributes defined

::

    name: subnonposixuser2
    first name: subNonposix2
    last name: subUser2
    password: Secret123
    password never expires: yes

-  A test user with expired AD account (accountExpires in the past)

::

    name: subexpiredaduser
    first name: subexpiredaduser
    last name: User
    password: Secret123
    password never expires: yes
    account expiration: one day in the past (relative to provisioning)
    attributes:
      uidNumber: 10149
      gidNumber: 10147
      loginShell: /bin/sh
      homeDirectory: /home/subexpiredaduser
      unixHomeDirectory: /home/subexpiredaduser
      gecos: Expired AD User



On tree root AD
----------------------------------------------------------------------------------------------

Objects below match `ad-tree.yml
<https://github.com/freeipa/freeipa-pr-ci/blob/master/ansible/roles/windows/ipa-ad-data/tasks/ad-tree.yml>`__.

-  A user group

::

    name: treetestgroup
    scope: global
    attributes:
      gidNumber: 10247

-  A test user with posix attributes defined

::

    name: treetestuser
    primary group: treetestgroup
    first name: TreeTest
    last name: User
    password: Secret123456
    password never expires: yes
    attributes:
      uidNumber: 10242
      gidNumber: 10247
      loginShell: /bin/sh
      homeDirectory: /home/treetestuser
      gecos: TreeTest User
