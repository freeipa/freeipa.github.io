AD
==



Test assumptions on the Active Directory side
---------------------------------------------



DNS has to be setup manually on AD.
----------------------------------------------------------------------------------------------

See `DNS configuration
instructions <Active_Directory_trust_setup#DNS_configuration>`__



Active Directory Posix support
----------------------------------------------------------------------------------------------

Active Directory used for this Integration test should have support for
POSIX attributes installed, as described in:

``       ``\ ```http://technet.microsoft.com/en-us/library/cc731178.aspx`` <http://technet.microsoft.com/en-us/library/cc731178.aspx>`__

For explicit steps how to setup support for POSIX attributes on the
Active Directory, please refer to the steps described in the
`V3/Use_posix_attributes_defined_in_AD <V3/Use_posix_attributes_defined_in_AD>`__
, test plan for "Test case: Leverage POSIX attributes defined in AD".



Object existing in Active Directory
----------------------------------------------------------------------------------------------

On Active Directory side, the following objects should exist:

-  A user group

`` Test Group (testgroup), with GID 10047.``

-  A test user with posix attributes defined

| `` UID: 10042``
| `` Home directory: /home/testuser``
| `` Shell: /bin/sh``
| `` Full name: Test User``
| `` Primary group: Test Group``
| `` Logon: testuser``
| `` Password: Secret123``

-  A test user without posix attributes defined

| `` Full name: Nonposix User``
| `` Logon: nonposixuser``
| `` Password: Secret123``

-  A test user with posix attributes which is disabled

| `` Password: Secret123``
| `` Logon: disabledaduser``



How to extend the Active Directory test suite
---------------------------------------------