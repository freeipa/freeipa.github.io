IPAv3_Architecture
==================



IPAv3 Goals
===========

The IPA v3 goal is to be able to set up trust relationships with AD
Forests.

This will allow AD Admins to see IPA as a Resource Domain where all
Linux machines are handled by the Linux Admins.

Ad users will be able to freely access Linux (Unix) machines with their
AD credentials.



Required Features to reach the Goal
-----------------------------------

In order to reach the stated goal, IPA needs to be able to interoperate
with Windows domain controllers of a Trusted Forest. For most operations
all is needed is access to the KDC. But to setup the actual trusts,
manage them and do some name resolution it is necessary to provide
Windows domain controllers with at least access to MS-RPC services. The
MS-RPC services needed are named NETLOGON, LSARPC and SAMR. These
services can be reached over the SMB (or SMB2) transport (port 445) and
directly over TCP/IP through the help of the EPM (End Point Mapper) RPC
service (port 135) that allows a remote server to discover on which
ports the services are listening. The LDAP port MUST be filtered, as our
LDAP schema and DIT are not compatible with what AD clients expect and
may confuse them.



Required Components
-------------------

MS-RPC services are implemented by Samba, as well as the EPM and SMBD
daemons. These services need to be made available both via the SMB/SMB2
protocols, that are handled via SMBD) and via TCP/IP by running
additional daemon(s) that can listen to ports above 1024. These services
need to be able to access the IPA database in order to access and
translate information in the format Windows Servers find suitable.

The KDC also needs to be able to handle some of the information Windows
clients/servers need, in form of an Authorization structure called
MS-PAC (Privileged Access Certificate) embedded in Kerberos Tickets.



IPAv3 Layout
============

The following image shows a high level view of the principal components
we need to integrate/develop in order to achieve the goal.

.. figure:: IPAv3-Layout.png
   :alt: IPAv3-Layout.png

   IPAv3-Layout.png

As you can see in this drawing there are 2 new components compared to a
v2 server. Samba components (in blue) are quite evident. But there is
also a new KDC related component called IPA-KDB. This is an IPA specific
DAL backend, and replaces the original MIT kldap DAL backend used in v2.



Setting up trust relationships
==============================

As mentioned, MS-RPC services are necessary in order to setup a trust
relationship with an AD Forest. The MS-RPC services are also used to
perform some sid2name name2sid lookups and Netlogon related functions
for working trusts.

.. figure:: IPAv3-LDAP-SAMBA.png
   :alt: IPAv3-LDAP-SAMBA.png

   IPAv3-LDAP-SAMBA.png

Interactions
------------

When a AD DC server needs to perform a MS-RPC operation against IPA, two
possible routes can be taken. It can try a direct TCP/IP connection (1)
against the service, possibly contacting the EPM Service (1.b) in
advance to know which ports the service is listening (the service
pre-registers (1.a) to the EPM so that the information is available
there). Alternatively the server can try to use SMB/SMB2 (2) to connect
to the server and open a named pipe (2.a) with the name of the service.

The authentication is performed using the DCE-Style GSSAPI-Krb5 method
or, as a fallback, NTLMSSP. The credentials used will depend on the
operation being performed. During the trust setup operation,
administrative credentials are used in order to establish a trust
secret. Generally for normal operations the trust secret is used. In
case Kerberos authentication is used, the PAC will contain the
credentials and is decoded by libndr_krb5 (4), otherwise the user will
be searched through IPASAM (3) on the LDAP server.

Once connected, the service will either request data or request
modification to the data. In all cases this will be performed through
the IPASAM backend (3) which contains the code used to properly
format/fetch (3.a) data from the LDAP server. The RPC Service is
responsible for performing authorization checks and permit/deny the
requested operation.

Components
----------

In the picture we can see 5 major components of Samba we are currently
interested in:

-  SMBD
-  EPMD
-  The Netlogon/LSA/SAMR daemon(s)
-  IPASAM
-  libndr_krb5

SMBD
----------------------------------------------------------------------------------------------

The SMBD daemon is useful basically only as a transport for MS-RPC over
named pipes. The default configuration should be limited to exposing
named pipes only, and possibly it should disable all named pipes that
are not necessary for setting up the trust relationships and serving the
data necessary to keep them working.

EPMD
----------------------------------------------------------------------------------------------

The EPM Daemon is a very tiny and simple yet fundamental daemon that
allow Windows machines to connect directly to MS-RPC Services over
TCP/IP. This will be either a separate binary or a process forked out by
SMBD at startup.

Netlogon/LSA/SAMR
----------------------------------------------------------------------------------------------

This is the core component of Samba that allows IPA to be configured as
a trusted Forest by Windows. The implementation may comprise separate
daemons or a set of process forked by SMBD at startup. These services
will work in concert to provide access to information and set
information in the LDAP directory through the IPASAM interface. The LDAP
directory is responsible for storing all the long term data. ... These 3
services provide the following functionality:

Netlogon
^^^^^^^^

The Netlogon service deals with Secure Channel setup, pass-through
authentication and domain trusts related functions among other things.

This service is use both at trust set up time and during normal usage to
handle authentication related operations.

LSA
^^^

This is the most important interface for setting up trusts and managing
trusts information. At the same time it also implements methods to
perform name translation services like name2sid, sid2name and others. So
it will be used both for setup operations and as day to day translation
of SIDs and names.

SAMR
^^^^

The SAMR (System Account Management RPC) service handles user group and
domain data (query, add, remove, modify of accounts). This interface is
generally exposed together with the LSA interface and complements it in
some places.

libndr_krb5
----------------------------------------------------------------------------------------------

... Finally the libndr_krb5 library provides the means for packing and
unpacking authorization structures used by Windows, including the MS-PAC
structure embedded in Kerberos Tickets.



IPA - AD trust relationships at work
====================================

During normal operations the most important piece, in order to allow
authentication and SSO is the Kerberos infrastructure. It is especially
important for trust relationships as kerberos is used not only to
perform authentication but also to convey authorization data via the
MS-PAC.

.. figure:: IPAv3-KDC-AD-trusts.png
   :alt: IPAv3-KDC-AD-trusts.png

   IPAv3-KDC-AD-trusts.png

The picture summarizes the set of operations involving the IPA and AD
KDCs from the perspective of both Windows and IPA clients and servers.
It is assumed that a trust relationship is already in place. It is also
assumed the clients already have a valid TGT.



AD client needs services from IPA server
----------------------------------------

The AD client performs a TGS Request for the service to the AD KDC
(a.1), the KDC recognizes that the service belongs to a trusted domain
and send sback to the client a cross-realm TGT and a referral to go ask
the trusted KDC.

The AD client uses the cross-realm TGT to request a ticket to the IPA
KDC (a.2).

At this point the IPA KDC needs to validate the MS-PAC being transmitted
with the cross-realm TGT. The IPA-KDB may, optionally, check the LDAP
directory (c.1) to see if foreign principals are allowed to get tickets
for the requested service. The IPA-KDB plugin then decodes the MS-PAC
using the libndr_krb5 library (c.2) and verify and eventually filters
the data. It perform lookups (c.1) in the LDAP server to check if it
needs to augment the MS-PAC with additional information (local groups
for example), then uses the libndr_krb5 library (c.2) to encoded the PAC
again, sign it and send it back attached to the service ticket.

The AD client can now contact the IPA service (a.3).



IPA client needs services from AD server
----------------------------------------

The IPA client performs a TGT Request for the service it wants to
contact to the IPA KDC (b.1). The KDC recognizes the service belongs to
another realm, checks the realm is known and trusted, and, eventually,
that the client is allowed to request services from foreign realms.

The KDC checks if the client's TGT has a MS-PAC attached to it. If it
doesn't (or it contains a PAD instead) the KDC does a lookup in the
directory (c.1) to get the principal data. With this data (or using the
data from the PAD) it creates a MS-PAC and encodes it using libndr_krb5
(c.2). Then the KDC sends back a cross-realm TGT to the IPA client.

The IPA client contacts the AD KDC (b.2) to request a ticket for the AD
service, presenting the cross-realm TGT containing the MS-PAC provided
by the IPA KDC.

The AD server validates and filters the PAC and returns a ticket for the
AD server.

The IPA client can now contact the Ad service (b.3).



IPA managed server and MS-PAC
=============================

In a domain with AD trusts an IPA managed servers need to handle
identity/authorization data conveyed in the form of a MS-PAC structure.

.. figure:: IPAv3-MS-PAC-Login.png
   :alt: IPAv3-MS-PAC-Login.png

   IPAv3-MS-PAC-Login.png

When a client connects (1) to the server and uses GSSAPI-Krb5 to
authenticate it can provide a MS-PAC structure with the service ticket
it presents to the login application. This application is linked (2)
against the libgssapi library which can extract the MS-PAC data and
pass it (3) to SSSD through a local Unix socket or equivalent mechanism.
The SSSD validates the MS-PAC data by checking signatures and then
use libndr_krb5 (4) to decode the MS-PAC. Once the MS-PAC is decoded,
SSSD will update the cache with the information contained so that
following getent requests can be properly fulfilled.

If the user space application requires more information than is
available in the PAC (for example various group names) then SSSD may
contact (5) the IPA Identity Server to get the information it
needs. The IPA server may need eventually to contact the AD Domain
to resolve Names to SIDs or SIDs to Name to reply to the client's
request. IPA will use a LSARPC call, eventually on a Secure Channel, to
contact (6) the AD domain controller and perform queries.

NOTE: In many cases the IPA KDC will have filtered all foreign groups
from the MS-PAC and augmented it with local groups, so that this last
step is rarely necessary.

The method to be used is not completely finalised yet. One option
assumes libgssapi will be modified to use a mechglue-proxy so that SSSD
does the actual acceptor exchange and gives back the application only
the session keys. Another option assumes that we have to trust all
applications that have access to kerberos keys and the only thing being
passed to SSSD is the actual MS-PAC. A third option is about not
trusting applications but still only getting the MS-PAC blob, this means
SSSD will need to validate the MS-PAC by asking one of the IPA KDCs to
verify the KDC signature.

An Ms-PAC contains only SIDs to represent group memberships, SSSD
will be able to translate SIDs directly into GIDs, but will not have
direct access to the group names (unless these groups have been
previously cached). In this case only the initgroups() call can be
successfully replied to w/o additional name resolution work.

The protocol that will be used to resolve "foreign" users and
groups from SSSD is not yet defined. It may involve using LSARPC calls
against the IPA's Samba instance, or perhaps a special LDAP extended
operation. This protocol will be better defined later on and this page
will be corrected to reflect the decision.



IPA managed server and Password based Login
===========================================

In a domain with AD trusts an IPA managed servers need to handle
password based authentication too.

.. figure:: IPAv3-Password-Login.png
   :alt: IPAv3-Password-Login.png

   IPAv3-Password-Login.png

In this case a client wishes to connect (1) but the protocol being used
or other reasons (no kerberos support on client, etc..) requires the
Login application to accept a user/password pair. In this case during
(1) User names will have to be fully qualified. If the AD domain name is
ad.example.com with a short name of AD, Ideally we will accept at least
the 2 following forms for a username:

-  AD\username
-  username@ad.example.com

These 2 forms allow SSSD to understand that we are trying to log into
the system as a user from a specific domain (as opposed to the default
which is IPA's). SSSD will query IPA (3) or used cached knowledge to
check if this domain is known and trusted and to get back indication on
how to reach the other domain KDC. Then SSSD proceeds (4) to contact the
AD KDC to ask for a TGT for the user using the user's password as the
shared secret.

AD will reply back with a TGT containing the MS-PAC. At this point SSSD
will perform validation by first asking the AD server (5) for a
cross-realm TGT for the IPA domain and then using this TGT to get a
host/ ticket (6) against itself. At this point he IPA KDC will perform
the usual filtering and signing of the MS-PAC (*) and attach it to the
service ticket for the host.

Once the service ticket is obtained SSSD can validate that the user's
TGT is correct, and can check the signatures on the MS-PAC sent back by
the IPA KDC, and can decode (7) it. The resulting structure is used to
populate SSSD caches and authentication and operations proceed in the
same way as in the previous scenario.

(*) In a not too distant future, the IPA KDC may even decide to
translate the MS-PAC into a PAD (Principal Authorization Data) which is
similar but contains information in a way that is more complete for
Posix machines. We are currently working on a draft proposal within IETF
to have the PAD standardized so that we can soon start to use it in IPA.



Finding a name for a SID
========================

For groups memberships the PAC only contains SIDs and no groups names.
In order to use group name for access control or other kind of
permission checking the SIDs have to be resolved to groups names. This
can either be a name of a group of the IPA domain which has a mapping to
a SID or the name of a group in the AD domain.

.. figure:: IPAv3-Sid-2-Name.png
   :alt: IPAv3-Sid-2-Name.png

   IPAv3-Sid-2-Name.png

Once the Kerberos ticket is received, e.g. via a GSSAPI login (1), the
PAC is extracted (2) and send to SSSD (3). SSSD splits the PAC into its
components (4). If SSSD cannot find the name of a group related to a SID
in its local cache it uses an LDAP extended operation (5) to ask the IPA
server to return the names of group objects given by a list of SIDs.

For every SID in the list the IPA server will first check if a mapping
to a local group is available (6) or if the SID can be found in a cache
(7). If there are still unresolved SIDs the IPA server will open a RPC
connection to a domain controller of the AD domain with the help of the
trust credentials and sends a request to resolve the SIDs to names (8).
This RPC call is preferably done directly from the extended operation
plugin of the directory server. But if it is easier an external program
like rpcclient or winbind can be used for a first step. The returned
names are stored together with the corresponding SID in the cache and
returned to the client.