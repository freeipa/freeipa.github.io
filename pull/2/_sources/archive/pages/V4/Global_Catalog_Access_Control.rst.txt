Overview
--------

As outlined in the `High Level description of the Global Catalog
service <V4/Global_Catalog_HLD>`__, FreeIPA Global Catalog service
access is read-only. However, the access is authenticated to limit the
exposure of the information similar to how Global Catalog in Active
Directory by default limits its access to Authenticated Users.

.. _supported_authentication_schemes:

Supported authentication schemes
--------------------------------

Since FreeIPA Global Catalog service does not store information about
credentials associated with the LDAP objects, it will have to proxy
authentication requests to another, primary, sources. As result,
following three authentication methods will be supported:

-  LDAP simple bind authentication
-  SASL GSSAPI authentication
-  SASL GSS-SPNEGO authentication

In all cases the resulted authenticated LDAP identity will be mapped to
the pre-defined LDAP object which will have read-only access rights
associated with the Global Catalog subtree.

.. _ldap_simple_bind_authentication:

LDAP simple bind authentication
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A request for LDAP simple bind authentication will be transformed into a
PAM authentication request. A special PAM service will be provided to
make sure FreeIPA HBAC rules can be applied to it to allow selective
authentication for administrators wishing to disable access to Global
Catalog with the use of LDAP simple bind. By default the HBAC service
*global-catalog* will allow any valid user to authenticate against the
Global Catalog service.

PAM authentication service relies on SSSD service running on the FreeIPA
master. In result, any valid user recognized on the FreeIPA master will
be able to authenticate using the *global-catalog* service, including
valid accounts from the trusted Active Directory forests.

.. _sasl_gssapi_authentication:

SASL GSSAPI authentication
~~~~~~~~~~~~~~~~~~~~~~~~~~

For Kerberos-based authentication SASL GSSAPI method is provided. A
special mapping rule is defined in the *cn=config* of the Global Catalog
directory server instance will map any valid Kerberos principal to the
pre-defined LDAP object with read-only access rights associated with the
Global Catalog subtree.

.. _sasl_gss_spnego_authentication:

SASL GSS-SPNEGO authentication
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Windows client machines utilize SASL GSS-SPNEGO for the authentication,
using their machine accounts. This mechanism is currently not supported
by the CyrusSASL library. CyrusSASL library will need to be extended to
allow SASL SPNEGO authentication to work (a patch exists to prove it is
possible), but only Kerberos credentials will be accepted. An
NTLMSSP-based authentication will not be supported in the initial
implementation.

`MS-ADTS specification chapter
7 <https://msdn.microsoft.com/en-us/library/dd763857.aspx>`__ explains
the authentication methods supported by Microsoft Windows clients when
accessing Active Directory, including Global Catalog service.
