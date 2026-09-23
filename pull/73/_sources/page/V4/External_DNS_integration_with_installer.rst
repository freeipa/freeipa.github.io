External_DNS_integration_with_installer
=======================================

Overview
--------

FreeIPA should have ability to manage DNS records required for FreeIPA
infrastructure even if DNS is managed by some non-FreeIPA DNS server.
FreeIPA should update its DNS records in external DNS during
installation and replica management.



User stories
------------

-  As an FreeIPA administrator, I want to install FreeIPA replica and
   get its DNS records automatically populated as needed. My company is
   using DNS service outside of FreeIPA and is not willing to migrate to
   FreeIPA DNS.

   -  E.g. FreeIPA server installation is done in AD dominated
      environment where AD admins do not want to delegate DNS domain to
      FreeIPA admins (i.e. political issues).
   -  For now **only with FreeIPA installers and replica management
      tools will be integrated with external DNS**.



Future extensions
----------------------------------------------------------------------------------------------

-  As FreeIPA administrator, I want to have ability to use DNS locations
   for location-awareness even though FreeIPA DNS is not used by my
   organization.

-  As an lower-level admin, I need to add/delete DNS records when
   adding/removing hosts and services managed by FreeIPA. I want to use
   IPA commands and web UI to do this even though DNS is external to
   FreeIPA.

   -  Should it work for users or only for automated DNS changes from
      IPA installers?

-  As an lower-level admin, I'm enrolling hosts to FreeIPA realm. I want
   the hosts to register themserves into external DNS.

   -  Potential limitation: Should DNS updates from FreeIPA clients and
      from FreeIPA users be `explicitly out of
      scope <https://www.redhat.com/archives/freeipa-devel/2014-December/msg00044.html>`__
      of this proposal.



External DNS services to consider
----------------------------------------------------------------------------------------------

+------------------------------------+--------------------------------+
| **Service**                        | **Expected difficulty**        |
+------------------------------------+--------------------------------+
| no locations                       | manual locations\ `1 <#fn1>`__ |
+------------------------------------+--------------------------------+
| `Microsoft DNS                     | ?                              |
| server                             |                                |
| <http://www.microsoft.com/dns>`__: |                                |
| standalone, no Active Directory    |                                |
+------------------------------------+--------------------------------+
| Microsoft DNS server: integrated   | ?                              |
| with Active Directory without      |                                |
| trust                              |                                |
+------------------------------------+--------------------------------+
| Microsoft DNS server: integrated   | medium\ `2 <#fn2>`__           |
| with Active Directory with two-way |                                |
| trust                              |                                |
+------------------------------------+--------------------------------+
| Microsoft DNS server integrated:   | ?                              |
| with Active Directory with one-way |                                |
| trust                              |                                |
+------------------------------------+--------------------------------+
| `InfoBlox <https://w               | ?                              |
| ww.infoblox.com/products/ddi/>`__: |                                |
| standalone                         |                                |
+------------------------------------+--------------------------------+
| InfoBlox: with keytab from FreeIPA | easy                           |
| realm                              |                                |
+------------------------------------+--------------------------------+
| InfoBlox: with keytab from AD      | ?                              |
| realm, no trust to FreeIPA realm   |                                |
+------------------------------------+--------------------------------+
| InfoBlox: with keytab from AD      | easy                           |
| realm, two-way trust to FreeIPA    |                                |
| realm                              |                                |
+------------------------------------+--------------------------------+
| InfoBlox: with keytab from AD      | ?                              |
| realm, one-way trust to FreeIPA    |                                |
| realm                              |                                |
+------------------------------------+--------------------------------+
| `AWS Route 53                      | ?                              |
| service <h                         |                                |
| ttps://aws.amazon.com/route53/>`__ |                                |
+------------------------------------+--------------------------------+
| Azure                              | ?                              |
| `internal <https://docs.           |                                |
| microsoft.com/en-us/azure/virtual- |                                |
| network/virtual-networks-name-reso |                                |
| lution-for-vms-and-role-instances# |                                |
| azure-provided-name-resolution>`__ |                                |
| DNS                                |                                |
+------------------------------------+--------------------------------+
| Azure                              | ?                              |
| `external <https://azure.mic       |                                |
| rosoft.com/en-us/services/dns/>`__ |                                |
| DNS                                |                                |
+------------------------------------+--------------------------------+
| `Google Cloud                      | ?                              |
| DNS                                |                                |
| <https://cloud.google.com/dns/>`__ |                                |
+------------------------------------+--------------------------------+

Design
------

-  Add new DNS zone type 'external' and transform dns\* commands from
   FreeIPA framework to equivalent "nsupdate" commands and send DNS
   updates to existing DNS servers.

-  Use DNSSEC key distribution mechanism for TSIG key distribution to
   FreeIPA servers.
-  Store Kerberos keytabs for GSS-API as normal keytabs in LDAP.

   -  use ``ipa-getkeytab`` for key distribution to FreeIPA servers

-  Store keys on server's disk in location accessible only to root user

The proposed solution. This may include but is not limited to:

-  High Level schema (`Example 1 <V4/OTP>`__, `Example
   2 <V4/Migrating_existing_environments_to_Trust>`__)
-  Information or update workflow
-  Access control (may include `new permissions <V4/Permissions_V2>`__)

For other hints what to consider see `general
considerations <General_considerations>`__ page.

Implementation
--------------



Microsoft DNS without Active Directory
----------------------------------------------------------------------------------------------

As far as I can tell, Windows DNS 2016 without AD does not support
GSS-TSIG, I did not find any way to set plain TSIG, and it does not
allow any way of ACL specification on the zone.

This means that any client can update everything. For this case sending
unsigned DNS update is an easy thing to do. It is up to the reader to
decide if it is wise to run infrastructure this way or not, but it is
not FreeIPA's bussiness.



Microsoft Active Directory
----------------------------------------------------------------------------------------------

Windows DNS server integrated with AD supports
`GSS-TSIG <https://technet.microsoft.com/en-us/library/cc961412.aspx>`__.
In case of two-way trust, this allows us to use existing IPA keytab and
set ACL on Windows side to allow updates from clients. If we decide to
go with DNS proxy route,
`https://technet.microsoft.com/cs-cz/library/dd334715(v=ws.10).aspx
DnsUpdateProxy
Group <https://technet.microsoft.com/cs-cz/library/dd334715(v=ws.10).aspx_DnsUpdateProxy_Group>`__
could get handy.

Any implementation details you would like to spell out. Describe any
technical details here. Make sure you cover

-  **Dependencies**: any new dependencies that FreeIPA project or it's
   part would gain and that needs to be packaged in distros?
-  **Backup and Restore**: any new file to back up or change required in
   `Backup and Restore <V3/Backup_and_Restore>`__?

If this section is not trivial, move it to ``/Implementation`` sub page
and only include link.



Feature Management
------------------

UI

How the feature will be managed via the Web UI.

CLI

Overview of the CLI commands. Example:

+------------+--------------------------------------------------------+
| Command    | Options                                                |
+============+========================================================+
| config-mod | --user-auth-type=password/otp/radius                   |
+------------+--------------------------------------------------------+
| user-mod   | --user-auth-type=password/otp/radius --radius=STR      |
|            | --radius-username=STR                                  |
+------------+--------------------------------------------------------+

Configuration
----------------------------------------------------------------------------------------------

Any configuration options? Any commands to enable/disable the feature or
turn on/off its parts?

Upgrade
-------

Any impact on upgrades? Remove this section if not applicable.



How to Test
-----------

Easy to follow instructions how to test the new feature. FreeIPA user
needs to be able to follow the steps and demonstrate the new features.

The chapter may be divided in sub-sections per `Use
Case <#Use_Cases>`__.



Test Plan
---------

Test scenarios that will be transformed to test cases for FreeIPA
`Continuous Integration <V3/Integration_testing>`__ during
implementation or review phase. This can be also link to `source in
cgit <https://git.fedorahosted.org/cgit/freeipa.git/>`__ with the test,
if appropriate.

Author
------

`Petr Špaček <User:pspacek>`__

--------------

#. Support for DNS locations require manual configuration of particular
   DNS views and keys on each DNS server.\ `↩︎ <#fnref1>`__
#. Keep in mind that DNS is required for establishing of the trust so
   Trust credentials cannot be used before establishing the trust
   ...\ `↩︎ <#fnref2>`__