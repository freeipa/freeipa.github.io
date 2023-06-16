The FreeIPA Directory Service is built on the `389
DS <http://directory.fedoraproject.org>`__ LDAP server. It is the base
stone of the whole Identity Management solution. It serves as a data
backend for all identity, authentication (`Kerberos <Kerberos>`__) and
authorization services and other policies.

The used technology allows FreeIPA to offer a multi-master environment,
where administrator can deploy a number of replicating FreeIPA servers,
thus distributing the load on Identity Management servers and providing
higher redundancy at the same point.

.. _data_storage:

Data Storage
------------

All FreeIPA identity, policy, configuration or certificates are stored
in the Directory Server. FreeIPA objects are stored in one suffix
calculated from realm name (e.g. ``dc=example,dc=com`` for a realm
EXAMPLE.COM), certificates are stored in a second suffix, ``o=ipaca``.

.. _accessing_data:

Accessing Data
~~~~~~~~~~~~~~

Access to different parts of the Directory Server tree is protected by
DS ACI configuration. Some parts of the tree (like users) can be open
anonymously to everyone, others may be open only to authenticated users
(like sudo) and other's only to privileged users (like DNS tree). Write
access is even more strict, but it can be allowed to FreeIPA users using
the permission plugins (run ``ipa help permission`` command for more
information) which can add DS ACIs allowing that operation in a
user'convenient way.

.. _modifying_data:

Modifying Data
~~~~~~~~~~~~~~

As Directory Server is communicating with standard
`LDAPv3 <http://www.ietf.org/rfc/rfc2251.txt>`__ protocol, standard LDAP
`clients <Client>`__ can be used to read all the identity and policy
objects. However, adding or modification of directory entries by custom
LDAP clients is not recommended as it could lead to incomplete or
inconsistent entries in the tree where some expected attribute is in an
invalid format or missing at all.

To make the manipulation of the entries easier, the provided CLI and Web
UI interfaces provide a supported way of performing the modifications.
Run ``ipa help topics`` to see the list of supported commands.

.. _server_plugins:

Server Plugins
--------------

Just a plain data storage is not sufficient for an efficient Identity
Management solution. It also needs to provide more advanced functions
supporting it's clients, for instance validation for selected
attributes, authentication hooks to prevent brute forcing of a user's
password, extended operations etc.

FreeIPA configures primarily the following set of plugins (to see all of
them, traverse ``cn=plugins,cn=config`` in the tree):

-  **ipa_pwd_extop**: Handles password changes, enforces the FreeIPA
   password policy (``ipa help pwpolicy``) for new or changed passwords
-  **IPA Lockout**: hooks into authentication to the Directory Server
   (i.e. LDAP BIND operation) and makes sure nobody is brute forcing the
   user's password by running too many passwords attempt. If it does, it
   locks out the user for configured amount of time.
-  **ipa_enrollment_extop**: provides extended operation for enrollment
   of new `clients <Client>`__ and creating new client host entry
-  **Schema Compatibility**: publishes an alternate trees containing a
   computed different view on objects in the DS. For instance, as
   FreeIPA stores users using RFC 2307bis schema, it publishes alternate
   tree ``cn=users,cn=compat,dc=example,dc=com`` with users in a RFC
   2307 schema. It is also used by `Trusts <Trusts>`__ feature to allow
   Active Directory users access legacy system without a recent SSSD
   version.
-  **ipa-winsync**: enables user synchronization with Active Directory.
   Note that winsync synchronization is rather obsoleted and
   `Trusts <Trusts>`__ are a preferred way of FreeIPA - Active Directory
   interoperation
-  **IPA DNS**: `DNS <DNS>`__ zones stored in LDAP do not replicate SOA
   serial number among replicas, which effectivelly means that the SOA
   serial is localy-significant. This plugin makes sure that new
   replicated zones get SOA serial set to initial value 1, so the
   mandatory attribute ``idnsSOAserial`` is present.



Backup and Restore
------------------

See a separate `Backup and Restore <Backup_and_Restore>`__ article to
read more about backup and restore scenarios and how to approach them.

Documentation
-------------

.. _project_documentation:

Project documentation
~~~~~~~~~~~~~~~~~~~~~

-  `389 Directory Server project
   pages <http://directory.fedoraproject.org/>`__
-  `Red Hat Directory Server extensive
   documentation <https://access.redhat.com/documentation/en-US/Red_Hat_Directory_Server/>`__

.. _technical_articles:

Technical articles
~~~~~~~~~~~~~~~~~~

-  `SyncRepl - LDAP synchronization
   protocol <http://www.port389.org/docs/389ds/design/content-synchronization-plugin.html>`__
