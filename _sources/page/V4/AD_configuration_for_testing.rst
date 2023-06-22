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



On forest root AD
----------------------------------------------------------------------------------------------

-  A test group

| ``name: testgroup``
| ``scope: global``
| ``attributes:``
| ``  gidNumber: 10047``

-  A test group with @ in the name

| ``name: group@group``
| ``scope: global``
| ``attributes:``
| ``  gidNumber: 10048 ``

-  A test user with posix attributes defined

| ``name: testuser``
| ``primary group: testgroup``
| ``first name: Test``
| ``last name: User``
| ``password: Secret123``
| ``password never expires: yes``
| ``attributes:``
| ``  uidNumber: 10042``
| ``  gidNumber: 10047``
| ``  loginShell:``
| ``  homeDirectory: /home/testuser``
| ``  gecos: Test User``

-  A test user without posix attributes defined

| `` name: nonposixuser``
| `` first name: Nonposix``
| `` last name: User``
| `` password never expires: yes``
| `` password: Secret123``

-  A test user with posix attributes which is disabled

| `` name: disabledaduser``
| `` first name: Disabledad``
| `` last name: User``
| `` password: Secret123``
| `` password never expires: yes``
| `` account is disabled: yes``
| `` attributes:``
| ``   uidNumber: 10043``
| ``   gidNumber: 10047``
| ``   loginShell: /bin/sh``
| ``   homeDirectory: /home/disableduser``

-  A UPN suffix

`` suffix: UPNsuffix.com``

-  A user with UPN suffix

| `` name: upnuser``
| `` first name: UPN``
| `` last name: User``
| `` password: Secret123456``
| `` password never expires: yes``
| `` Acount logon name: upnuser@UPNsuffix.com``
| `` attributes:``
| ``   uidNumber: 10048``
| ``   gidNumber: 10047``
| ``   loginShell: /bin/sh``
| ``   homeDirectory: /home/upnuser'; ``
| ``   gecos: UPN User``

-  Group with info attribute and gidnumber defined

| `` name: mytestgroup``
| `` scope: global``
| `` attributes:``
| ``   gidNumber: 10055``
| ``   info: mytestuser``

-  A test user with posix attributes defined with same gidnumber of
   mytestgroup.

| `` Name: mytestuser``
| `` GivenName: Test``
| `` Surname: User``
| `` AccountPassword: Secret123``
| `` PasswordNeverExpires: $true``
| `` Enabled: $true``
| `` OtherAttributes: "@{'uidNumber'='10055'; 'gidNumber'='10055'; 'loginShell'='/bin/sh'; 'homeDirectory'='/home/mytestuser'; 'unixHomeDirectory'='/home/mytestuser'; 'gecos'='Test User'}"``



On child (subdomain) AD
----------------------------------------------------------------------------------------------

-  A user group

| `` name: subdomaintestgroup``
| `` scope: global``
| `` attributes:``
| ``   gidNumber: 10147``

-  A test user with posix attributes defined

| `` name: subdomaintestuser``
| `` first name: Subdomaintest``
| `` last name: User``
| `` password: Secret123``
| `` password never expires: yes``
| `` primary group: subdomaintestgroup``
| `` attributes:``
| ``   uidNumber: 10142``
| ``   gidNumber: 10147``
| ``   loginShell: /bin/sh``
| ``   homeDirectory: /home/subdomaintestuser``
| ``   gecos: Subdomaintest User``

-  A test user with posix attributes which is disabled

| `` name: subdomaindisabledadu``
| `` account logon name: subdomaindisabledaduser@CHILD_DOMAIN_NAME``
| `` password: Secret123``
| `` password never expires: yes``
| `` account is disabled: yes``
| `` attributes:``
| ``   uidNumber: 10143``
| ``   gidNumber: 10147``
| ``   loginShell: /bin/sh``
| ``   homeDirectory: /home/subdomaindisableduser``



On tree root AD
----------------------------------------------------------------------------------------------

-  A user group

| `` name: treetestgroup``
| `` scope: global``
| `` attributes:``
| ``   gidNumber: 10247``

-  A test user with posix attributes defined

| `` name: treetestuser``
| `` first name: TreeTest``
| `` last name: User``
| `` password: Secret123456``
| `` password never expires: yes``
| `` primary group: treetestgroup``
| `` attributes:``
| ``   uidNumber: 10242``
| ``   gidNumber: 10247``
| ``   loginShell: /bin/sh``
| ``   homeDirectory: /home/treetestuser``
| ``   gecos: TreeTest User``