Overview
--------

User principal name (UPN) in Active Directory is the primary form of
addressing users. UPN has structure of 'user name@suffix' where both
user name and suffix parts may vary. By default the suffix is the same
as the Active Directory domain name but AD administrators may create
additional name suffixes and associate them with specific users. These
additional UPNs for users may then be used for Kerberos authentication
against Active Directory domains.

Alternative UPNs are often used when several companies with Active
Directory deployments merge and want to provide unified logon namespace.

The purpose of this feature is to allow using alternative UPNs
associated with the Active Directory users when accessing resources in
FreeIPA domain.



Use Cases
---------

As an Active Directory user, I want to login using my user@EXAMPLE user
principal name even if my Active Directory domain is named
REGION.EXAMPLE.COM.

Design
------

Support for UPNs is split to three different components:

Client-side
   SSSD already supports logon with UPN by asking a KDC to accept
   enterprise logon names. By default, the use of enterprise principals
   is disabled, therefore, ``krb5_use_enterprise_principal = True``
   needs to be added to sssd.conf to enable it.

KDC
   IPA KDC does understand multiple domains associated with the trusted
   AD forest. However, since no information about name suffixes
   associated with the forest is available, it cannot take them into
   account when processing enteprise logon names to issue referrals to
   the correct realm. Support needs to be added to allow IPA KDC to look
   up name suffixes associated with a trusted forest.

IPA framework
   Changes needed on IPA framework side to fetch from Active Directory a
   list of name suffixes and store them in the trusted domain objects.

Implementation
--------------

For retrieving name suffixes, IPA framework needs to move to use
NETLOGON netr_DsRGetForestTrustInformation function instead of
netr_DsrEnumerateDomainTrusts. This allows to retrieve both domains and
top level names associated with the forest.

As top level names (TLNs) have only a single string as a name suffix,
they cannot be stored as trusted domains (they lack SID and NetBIOS
name). Thus, either IPA KDB driver needs to be extended to understand
trusted domains without SID and NetBIOS name, or TLNs need to be stored
as a property of tree root domains of the forest.



Feature Management
------------------

UI
~~

TLNs are added as a property of tree root domains of the forest,
appropriate panel needs to be extended to display them.

CLI
~~~

When TLNs are added as a property of tree root domains of the forest,
appropriate attribute need to be handled by **trust-show** command.

Configuration
~~~~~~~~~~~~~

No configuration options.

Upgrade
-------

An attribute 'ipaNTAdditionalSuffixes' is added to 'ipaNTTrustedDomain'
object class. Replicas which are not upgraded to the FreeIPA version
which contains the attribute and changed object class definition, will
receive schema changes as part of replication. The IPA framework running
on non-upgraded replica will not be able to fill-in the attribute values
and will not be able to show it until the packages for the FreeIPA
framework are upgraded.

.. _how_to_test38:

How to Test
-----------

In order to test UPN-based logons, create additional name suffixes in
Active Directory and establish trust to it. After trust is established,
the name suffixes should be usable when trying to kinit as enterprise
principal.

See
https://technet.microsoft.com/en-us/library/cc739093%28v=ws.10%29.aspx
for details of AD naming and how to add UPNs.



Test Plan
---------

`Support of UPN for trusted domains V4.4 test
plan <V4/Support_of_UPN_for_trusted_domains/Test_Plan>`__
