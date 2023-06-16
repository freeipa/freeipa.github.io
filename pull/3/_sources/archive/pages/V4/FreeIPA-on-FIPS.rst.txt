Overview
========

FreeIPA is using components that are capable to be run in FIPS mode but
is itself unable to do so. FreeIPA should use the components'
capabilities and not block users who want to have their system running
FIPS-enabled.

**FreeIPA only supports fresh FIPS installs, current installations
cannot be upgraded to FIPS mode.**



Use Cases
=========

-  As an administrator, I want to configure a FreeIPA server/client on a
   new FIPS-enabled system so that the system complies with my
   organization regulations.

Design
======

To be able to use FreeIPA in an FIPS-enabled system (FIPS refers to
`FIPS-140-2 <http://csrc.nist.gov/publications/fips/fips140-2/fips1402.pdf>`__
here and henceforth) we have to make sure all the cryptographic
functions used in the whole system meet the FIPS requirements. This also
means forcing any third party libraries to use the standardized
cryptographic functions. If cryptographic functions are not necessary,
replace these with some other means fulfilling the same purpose so that
we don't have to deal with this in the future should the standard be
updated (e.g. using hash of random bytes to get a readable version of a
generated password can be done using simple base64 encoding).

Another part of this effort is checking the usability of our current
components. Some of these will force different workflow which may not
suit our needs anymore as user experience usually suffers when system
security standards are raised. This may affect FreeIPA implementation
not only for FIPS mode but also on normal systems for the sake of
unification.

FIPS-enabled server and clients support is planned **only for new
FreeIPA deployments** in RHEL for new versions of FreeIPA, which means
older versions of FreeIPA servers/client and Domain Level 0 are **not
supported**.

.. _multiple_servers_in_topology:

Multiple servers in topology
----------------------------

All the server in the topology have to be either FIPS-enabled or in
non-FIPS mode. Any other configuration is not supported. Disabling or
enabling FIPS mode after a server has been installed is neither
supported, nor recommended. If you aim for a FIPS-enabled topology, you
should start with a fresh FreeIPA installation on a FIPS-enabled system
from the start.

To prevent *accidentally* joining a non-FIPS server into a FIPS-enabled
topology or vice versa, a basic check is performed. Installation of a
replica checks whether FIPS is enabled on the master. If so, the
installation requires the replica to also be FIPS-enabled. If the master
server is detected to be in non-FIPS mode, the replica can only join if
it does not have FIPS enabled. This solution is not designed to prevent
any non-FIPS machine operating as a part of FIPS-enabled topology.

Implementation
==============

As `Design Chapter <#Design>`__ suggests, any cryptographic functions
not conforming to FIPS-140-2 will either have to be replaced or their
behavior will have to differ in FIPS and non-FIPS systems should their
removal cause problems in non-FIPS FreeIPA.

Probably the biggest change is the switch from NSS to OpenSSL as a
backend for SSL connections in HTTPS requests. NSS implementation of SSL
in FIPS requires any user to know the password to the NSS database in
*/etc/ipa/nssdb*, which is unacceptable. See the design for this change:
http://www.freeipa.org/page/V4/Replace_NSS_with_OpenSSL

-  **Dependencies**: No new dependencies should be introduced.
-  **Backup and Restore**: There should be nothing new to back up.

.. _multiple_servers_in_topology_1:

Multiple servers in topology
----------------------------

A new environment variable ``env.fips_mode`` is introduced. It is a
boolean value initialized from */proc/sys/crypto/fips_enabled*. It
indicates whether FIPS mode is enabled on the server.

During the replica installation check, once a remote API connection is
established and server environment data are loaded, FIPS status of the
master is checked. Then, FIPS status of the replica is checked. The
installation will proceed if and only if the status of FIPS is the same
on the master and replica. Otherwise, an error message is displayed, and
the installation is aborted.



Feature Management
==================

UI
--

-  SHA256 fingerprints are displayed instead of MD5.

CLI
---

Displaying server's FIPS mode status:

::

   ipa env fips_mode

Configuration
-------------

FreeIPA will detect whether FIPS is enabled from
*/proc/sys/crypto/fips_enabled*. The steps to configure a RHEL system to
go into FIPS mode can be found in the `official RHEL Security
Guide <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/chap-Federal_Standards_and_Regulations.html#sec-Enabling-FIPS-Mode>`__.



How to Use
==========

Set up your system to be in FIPS-enabled mode and install FreeIPA as you
would normally do. Enabling FIPS after installation of IPA on the
machine is not supported - some operations will not work.



Test Plan
=========

Test scenarios that will be transformed to test cases for FreeIPA
`Continuous Integration <V3/Integration_testing>`__ during
implementation or review phase. This can be also link to `source in
cgit <https://git.fedorahosted.org/cgit/freeipa.git/>`__ with the test,
if appropriate.
