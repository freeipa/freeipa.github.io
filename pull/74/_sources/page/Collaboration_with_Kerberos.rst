Collaboration_with_Kerberos
===========================

Introduction
============

This page is not an RFE. It is intended to be an informative companion
to `External Users in IPA <External_Users_in_IPA>`__ by articulating the
processes by which external users obtain credentials for the local
realm. The use case supported by these mechanisms is described on
`External Collaboration Domains <External_Collaboration_Domains>`__. In
summary, the external collaboration domain is expected to host web
services, console login services, and file sharing services, but has few
or no locally defined users.

Three primary mechanisms are articulated:

#. Kerberos cross realm trusts. (Includes vanilla Kerberos trusts,
   trusts to IPA, and trusts to AD)
#. Kerberos PKINIT operation
#. Leveraging Non-Kerberos identity technologies

The third mechanism introduces the need for a gateway server to interact
with the non-Kerberos technologies, adapt foreign identities into the
Kerberos name@realm pattern, and to provide a means for the user to
obtain Kerberos credentials from the KDC.

`External Users in IPA <External_Users_in_IPA>`__ articulates a need to
create a reference to external users in the local LDAP store in order to
coordinate and serve POSIX attributes for these users to the hosts in
the local realm. For each mechanism by which external users may enter
the local realm, this page identifies how external users are detected.

Mechanisms
==========

Background
----------

This section develops a visual vocabulary to help articulate Kerberos
message flows. The iconography introduced here will be leveraged
throughout the remainder of the page. This starts out very basic, but it
is necessary later on when cross-realm operation is considered.

The first element to introduce is the Kerberos ticket. This is how
clients are authenticated to services and vice versa. As shown below, a
ticket has two parts: a "client" and a "service". These separately share
a secret with the Kerberos server. They do *not* share a secret with
each other (e.g., the service does not know the client's password.) This
property makes Kerberos a "Trusted Third Party" system. The top half of
the illustration shows that the ticket encodes a "name" and a "realm"
for both the client and the service. In the simplest case, such as when
a user is accessing a service in their home realm, the realm for the
client and the server are the same.

.. figure:: KrbTickets.png
   :alt: KrbTickets.png

   KrbTickets.png

In this visual language, displaying the "client" information inside the
ticket may be redundant. If a ticket is shown participating in an
interaction between a client and a server (as in the bottom part of the
above illustration), the client part is generally omitted. Additionally,
if the realm is or should be clear from context, it is also omitted. If
the realm is shown, it applies to the service.

A user begins their interaction with Kerberos services by providing
proof of identity to the Kerberos Authentication Service (AS). If the AS
accepts the user's identity, it returns a ticket good for admission to
the "Ticket Granting Service" (TGS), typically called a
Ticket-Granting-Ticket (TGT). The Kerberos specification reserves names
beginning with ``krbtgt`` for these special ticket granting tickets. If
a user successfully authenticates to the AS and receives a TGT, they
store it in the cache (step 3 below). The AS and the TGS are typically
hosted on the same physical machine and are collectively known as the
Key Distribution Center (KDC).

.. figure:: KrbAS.png
   :alt: KrbAS.png

   KrbAS.png

Once a user has a TGT in their ticket cache, they may use it, as shown
below, to obtain service tickets. This is the basis of Kerberos Single
Sign On (SSO), as the user needs only provide proof of their identity
once, at the beginning of the session. As long as the user's TGT has not
expired, they may obtain tickets for whatever service *within the realm*
they require. This is depicted in the illustration below for the
simplest case, where the user and the service are in the same realm
(CREALM). The realm is displayed on the ticket in this case to stress
the fact that the TGT being used is correctly being applied to the
specific TGS for which it was issued. This may seem pedantic, but cross
realm operation opens the door to more than one KDC, and it becomes
important to keep track.

.. figure:: KrbTGS.png
   :alt: KrbTGS.png

   KrbTGS.png

Note: all applications connected to your control session can share the
same ticket cache. This is generally what you want. Fire up a browser,
and it should be able to log you in to a kerberized website. Ssh to your
machine, it should know who you are. Generally, this sharing does not
lead to substantial savings in network traffic, as you can't ``ssh`` to
a website or point your browser at ``sshd``. However, it does mean that
all your client apps are looking through a common cache for a TGT they
can use, which allows all your apps to take advantage of SSO.

.. figure:: KrbTicketCache.png
   :alt: KrbTicketCache.png

   KrbTicketCache.png



Mechanism 1: Kerberos cross-realm trusts
----------------------------------------

Kerberos SSO may work across realms, if the realm administrators have
cooperated to make it do so. The mechanism by which this is accomplished
is known as a cross-realm ticket granting ticket. In the picture below,
this is the ticket for the principal named ``krbtgt/CREALM`` issued by
``REALM``. The name has a special form: the text after the slash
indicates the realm to which the cross realm TGT was issued. The ticket
is to be used to obtain service tickets for services in REALM (not
CREALM). Both realms must have a principal with this same name in order
to make cross-realm operation work.

This is known as a one-way trust, and it allows users from CREALM to
authenticate to services in REALM. It does not allow users from REALM to
access services in CREALM. If a two-way trust is desired, a second one
way trust (in the opposite direction) must be established. The principal
name for the complementary trust in this case is ``krbtgt/REALM``, and
it would be issued by CREALM.

.. figure:: KrbCrossRealm.png
   :alt: KrbCrossRealm.png

   KrbCrossRealm.png

The above figure illustrates how native Kerberos cross realm operation
works. The client must walk the "authentication path" composed of
cross-realm trusts. Starting in its own realm, CREALM, at step 1, it
obtains a cross-realm TGT for REALM. It then contacts the TGS for REALM
using this cross realm TGT at step 2 and requests a service ticket for
the desired service. In step 3, the client contacts the desired service
using the freshly minted service ticket. The picture above represents
the simplest case of a one-hop authentication path. More hops are
possible.

It is important to note that the client's identity is always
CLIENT@CREALM. This is not a conflict with the identity CLIENT@REALM. No
account is provisioned in the local realm, and the user's password is
not shared between realms. This is not the same thing as creating an
account for CLIENT in the local realm, as a new password would be
demanded and the client's identity would change to CLIENT@REALM upon
login.

Note that the first interaction of a particular external user with the
local IPA realm is with the Kerberos TGS server. For this mechanism, it
is possible to monitor all interaction of external users with the local
domain's services by auditing the presentation of cross realm tickets to
the local TGS, noting the name and realm of the user who presented it,
and recording which service was requested.



Mechanism 2: Kerberos PKINIT cross-realm operation
--------------------------------------------------

Native Kerberos cross-realm operation is most commonly used inside a
single organization, where the Kerberos realm administrators for all
realms involved are either the same people, or are part of the same
administrative team. In addition, clients must have the ability to
connect to all KDCs along the authentication path from where their user
account resides to the realm containing the service they wish to use.
KDCs, being high-value targets, are often protected by organizational
firewalls, and therefore may only be accessible to clients connected to
the organizational intranet. The degree of close coordination and
accessibility required by the native Kerberos method is not typically
found in an inter-organizational setting.

An official shortcut has been devised which relaxes these coordination
and accessibility requirements. "Public Key Cryptography for Initial
Authentication in Kerberos (PKINIT)", or `RFC
4556 <http://tools.ietf.org/html/rfc4556>`__, eliminates the need for
administrators to coordinate cross-realm TGTs as well as need for the
client to "walk the authentication path." The client's first interaction
with the local realm, which contains the service to which the client
wishes to connect, is to directly contact the local AS.

.. figure:: KrbPKINIT.png
   :alt: KrbPKINIT.png

   KrbPKINIT.png

Recall from earlier that the local AS demands that the user provide
acceptable proof of identity. In the case of an external user, the local
AS does not directly share a secret with the external user (i.e., it
does not know the user's password), so an alternative method must be
used. In step one, the client presents a digital certificate with
special parameters defined in "kx509 Kerberized Certificate Issuance
Protocol in Use in 2012" (`RFC
6717 <http://tools.ietf.org/html/rfc6717>`__). This digital certificate
binds the enclosed public key with the user's Kerberos identity, and
also identifies the entity making the assertion. If the AS is configured
to trust whomever certified the user's identity, and the user is able to
prove that they hold the private key corresponding to the certified
public key, a TGT for the local realm is returned to the user (step 2).
This TGT can then be cached by the client (step 3) to obtain service
tickets (step 4) within the local realm. The service tickets may also be
cached for re-use (step 5).

For this mechanism, monitoring the interaction of external users with
the local realm requires auditing any PKINIT-style authentications with
the local AS. Also, because this method is designed to relax the
requirement for cross-realm coordination, the likelihood of locating the
original user attributes in "CREALM" is smaller than for use case 1.



Mechanism 3: Non-Kerberos web Identity Providers
------------------------------------------------

Perhaps the most ubiquitous source of user identities is the collection
of web-oriented authentication technologies. These technologies, such as
OpenID and SAML, allow authentication information to be exchanged easily
over the web. Consumers of identity information are sometimes known as
Relying Parties (RP), and those parties who certify users are known as
Identity Providers (IdPs). Allowing the local IPA domain to consume
identities from these technologies requires a little more work than the
previous use cases for two reasons:

#. An adaptation layer, or gateway, needs to be established between
   Kerberos and the foreign authentication technology.
#. It is virtually certain that identities present in these web-oriented
   technologies lack the user attributes required to support console
   logins and file sharing.

Leveraging foreign technologies to acquire Kerberos credentials is not
widely practiced. More than one method has been proposed, and the
promise of the methods must be evaluated. The weakness of each is the
client side support (i.e., something new needs to be running on the
client's machine). However, each method shares the following
characteristics:

#. The client authenticates to the gateway server using the foreign
   technology.
#. The gateway server maps the foreign credentials to Kerberos
   ``name@realm`` strings.
#. The gateway server attests to the user's identity.
#. The KDC issues a TGT to the external user based on the gateway
   server's attestation.

All methods which involve a gateway server provide two points at which
monitoring could be performed: the gateway itself or the KDC. The
auditing of exchanges with the local AS and TGS, as described in
Mechanisms 1 and 2, would suffice. Auditing the gateway would "catch"
only those identities originating in a foreign technology, and not those
coming in from a trust or from PKINIT. Some have suggested having the
gateway actually create the external user entry in IPA. The location of
auditing and the means of creating user entries are orthogonal concerns
to describing the authentication mechanisms.



Logging in with a SASL/GSSAPI client
----------------------------------------------------------------------------------------------

Much work has been done recently on adapting web IdP authentication to
the Simple Authentication and Security Layer
(`SASL <http://www.ietf.org/rfc/rfc4422.txt>`__) and the Generic
Security Services Application Programming Interface
(`GSS-API <https://tools.ietf.org/html/rfc2743>`__). SASL and GSS-API
mechanisms for SAML (`RFC 6595 <https://tools.ietf.org/html/rfc6595>`__)
and OpenID (`RFC 6616 <http://tools.ietf.org/html/rfc6616>`__) have been
defined. The following figure outlines how these technologies might be
used to obtain Kerberos tickets.

.. figure:: KrbExternalIdPOptionA.png
   :alt: KrbExternalIdPOptionA.png

   KrbExternalIdPOptionA.png

In step 1, the client generates a public/private key pair, and a
certificate signing request (CSR). The client sends this to Ipsilon in
step 2, and authenticates using the procedure specified in either RFC
6595 or RFC 6616 (or by some other means, such as against an LDAP
directory). If successful, Ipsilon causes DogTag to sign the client's
CSR, generating a kx509 certificate. This certificate is returned to the
user in step 5. The client sends the kx509 certificate to the AS server
in step 6 and receives a TGT in step 7. From this point on, the client
has access to the services of the realm, and can delete the
public/private key pair as well as the kx509 certificate.



Logging in with a browser
----------------------------------------------------------------------------------------------

Note that this section relies on the ability to transfer a TGT over http
to a browser, which can then store it in the client's local ticket
cache. This functionality does not currently exist.

The following illustration shows an outline of the proposed workflow. In
step one, an un-authenticated CLIENT logs into a gateway server
configured to authenticate against web IdPs. The proof of identity in
this case is that the web-based IdP authenticates the user. The gateway,
named "Ipsilon" in the diagram, after the RedHat IdP gateway server,
represents the adaptation layer between Kerberos and the web-oriented
authentication technology. If Ipsilon is satisfied that the user has
been authenticated by a trustworthy web IdP, it requests a TGT from the
local realm's AS on the user's behalf using the PKINIT mechanism
described in Use Case 2. (Step 2) The AS, having been configured to
trust the gateway server's judgement, issues a TGT for the user and
returns it. The gateway server then relays the TGT to the client's
browser (step 3). The browser deposits the TGT into the active session's
ticket cache (step 4), making it available for other programs to use
(steps 5 and 6).

.. figure:: KrbExternalIdP.png
   :alt: KrbExternalIdP.png

   KrbExternalIdP.png

In order to use PKINIT on the Kerberos side, the gateway server needs to
take care of some details. These details are given in the figure below.

.. figure:: KrbExternalIdPDetail.png
   :alt: KrbExternalIdPDetail.png

   KrbExternalIdPDetail.png

The gateway server could also permit a browser-based username/password
bind against an LDAP directory as step 1 of this workflow. The remainder
of the workflow would remain the same.

Design
======



Auditing cross realm activity
-----------------------------

The common thread throughout all three use cases is that an actionable
audit stream for cross-realm activities must be available. For native
Kerberos cross-realm trusts (which includes trusts to AD), the local TGS
server must report all usage of cross-realm TGTs. For Kerberos PKINIT,
the local AS server must report all PKINIT message exchanges.
Implementing support for web-based IdPs as described here uses the
Kerberos PKINIT method, and requires no additional audit capabilities.

The existing IPA KDB plugin audits AS exchanges. This needs to be
expanded as described here to notice whether the cname and crealm of
PKINIT requests already exists in the LDAP store. Likewise, auditing the
TGS exchanges must be implemented.



Essential functionality delegated to gateway server
---------------------------------------------------

The IPA server's cross realm functionality, as proposed in this RFE,
deals exclusively with foreign Kerberos principals of the form
"name@realm" with optional siphoning of user attribute data from donor
realms. This is made possible because non-Kerberos identity technologies
are made to appear Kerberos-like by an external gateway server.
Functions provided by this server:

-  Delivery of the TGT to clients over nonstandard pathways (HTTP)
-  Mapping of non-Kerberos identities to a globally unique, consistent
   Kerberos-like ``cname``/``crealm`` pair.
-  Actually interacting with non-Kerberos authentication providers to
   determine the user's identity.
-  Any configuration or shared metadata required to interact with the
   non-Kerberos authentication providers.

To be clear, this is essential functionality for Use Case 3 only. Use
Cases one and two do not require this.