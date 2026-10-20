Global_Catalog_Support
======================

Overview
--------

Global Catalog is a special service that exposes information about
objects in FreeIPA domain in a way expected by the clients enrolled to
Active Directory environments. Active Directory domain controllers and
other systems use Global Catalog to search across the potentially
multi-domain forest for the information about users, groups, machines,
and other types of objects.

A search in the Global Catalog service allows machines enrolled to an
Active Directory environment to efficiently maintain a relationship
between internal access control lists (ACLs) associated with the
resources provided by the machines, and the visual representation of the
access controls for interaction with users, locally and remotely.

The information stored in the Global Catalog represents a subset of the
information available at the specific domain controllers in Active
Directory. As such, it is only accessible for read-only purposes. Any
change to the information shall come through the domain controllers
responsible for a domain in question.



Use Cases
---------

This design document focuses on two primary user stories where access to
Global Catalog in FreeIPA is crucial.

**User Story 1**: As a Windows Administrator I want to add FreeIPA Users
and Groups to the access control lists of resources in Active Directory,
so that I can let FreeIPA user log into to Windows client enrolled with
Windows Server and file shares.

Acceptance Criteria

-  I can enable "Allow logon locally" Global Policy for a special group
   with all FreeIPA users or a selected FreeIPA Group or an FreeIPA User
   to later enable local logon
-  I can add a special group with all FreeIPA users, selected FreeIPA
   Group or FreeIPA User to "Remote Desktop Users" group to later enable
   remote logon
-  I can grant access to a file share located on Windows Servers

**Limitation**: the use cases not explicitly called out above may or may
not work and are not supported in the first implementation.

**User Story 2**: As an FreeIPA User I'm able to login remotely or
locally to Windows client enrolled in Windows Server, so that the user
can use file shares or run a Windows application.

Acceptance Criteria

-  Prerequisite: this workflow requires a bi-directional Trust between
   FreeIPA Server and AD Forest Root and a Workstation enrolled in the
   forest's domain

   -  When allowed, FreeIPA user can login with his user and password to
      local Windows terminal
   -  When allowed, FreeIPA user can login with his user and password to
      remote Windows terminal using RDP client (test with FreeRDP)

-  FreeIPA user can open, modify or delete a file in a folder, when
   allowed by respective Windows folder ACL
-  FreeIPA user can run a Windows application that does not require any
   additional interfaces (like SMB or DCE RPC) available on FreeIPA
   Server

Design
------

FreeIPA Global Catalog service is a read-only LDAP service available
over port 3268. The content of the entries is based on the primary
FreeIPA LDAP instance. LDAP schema for FreeIPA Global Catalog is
compatible with Active Directory LDAP schema but exposes a minimal
subset of the latter as FreeIPA is not an Active Directory by itself.

-  `High Level description of the Global Catalog
   service <V4/Global_Catalog_HLD>`__
-  `Access control to the Global Catalog
   service <V4/Global_Catalog_Access_Control>`__
-  `Data transformation to the Global
   Catalog <V4/Global_Catalog_Data_Transformation>`__