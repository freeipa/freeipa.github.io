External_Collaboration_Domains
==============================

Scope
-----

This page describes a use case. It contains only the "what", not the
"how". The page contains a text description, an illustration, and a
handful of basic functional requirements. The "how" is deferred, and is
expected to be handled as an RFE.

Introduction
------------

There is a group of people belonging to different organizations, for
example universities, who launch a project together. Most participants
have one or more identities in IdPs operated by their own home
organization (domains), but nearly all will also have identities in
places like Google and Facebook. The project's resources are deployed in
pre-existing public-facing networks ("collaboration domains") hosted by
one or more of the participating organizations. Jointly all the users
want to be able to access the systems that belong to the project. This
includes login into the systems as well as accessing file shares and web
applications that constitute the joint project. They also want to be
able to SSO as much as possible. Avoiding the creation of additional
identities having a separate password is crucial to success of this
environment.

.. figure:: CollaborationRealm.png
   :alt: External Collaboration Realms

   External Collaboration Realms

The internal "desktop" infrastructure of hosting organizations has a
one-way trust with the collaboration realm belonging to that same
organization. The purpose of this trust is to ensure that all local
users on the company intranet can access the collaboration resources
using their institutional desktop-oriented identities. Trust between
collaboration realms from different organizations is optional.

The goal is to make it possible for resources deployed in these
collaboration realms to understand external identities accessing
resources on the file system level thus POSIX attributes need to be
generated, assigned and managed for those users. The vast majority of
identities will be external to the collaboration domains, being
contributed via a trust from the organization's internal "desktop"
infrastructure, trust from other collaboration domains, PKINIT from a
different kerberos domain, or from an IdP implemented in a non-Kerberos
technology.



Functional Requirements
-----------------------

-  Heterogeneous connection origins

   -  Support "home organization" users connecting from the corporate
      intranet, from a collaborator's organization, or from home.
   -  Support "other" users, connecting from collaborator's organization
      or from home.

-  Reduce as much as possible the need to create accounts.
-  Establishing an external collaboration domain must not require
   exposure of an organization's internal "desktop" infrastructure to
   the public internet. (i.e., firewall stays intact)
-  Coordination between organizations must not be required.
-  Minimize the number of times the user's password is demanded.
-  Users, regardless of origin, must be able to ssh, mount nfs shares,
   and otherwise utilize the systems to which they have access.
-  Users must be able to participate in groups defined within the
   collaboration domain.
-  While not required, it would be desirable for users to be able to
   specify which identities belong to them.