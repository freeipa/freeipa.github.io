Authenticating via Apache will set a number of environment variables,
depending on the configuration and the authentication method. I'm
skipping digest authentication because that is not commonly used.

.. _common_variables:

Common variables
----------------

A number of useful variables are set by Apache itself, they include:

+------------------+--------------------------------------------------+
| DOCUMENT_ROOT    | The directory the current file/script is         |
|                  | executing                                        |
+------------------+--------------------------------------------------+
| HTTP_ACCEPT      | Contents of Accept: header                       |
+------------------+--------------------------------------------------+
| HTTP_HOST        | Contents of Host: header                         |
+------------------+--------------------------------------------------+
| HTTP_USER_AGENT  | Contents of User-Agent: header                   |
+------------------+--------------------------------------------------+
| QUERY_STRING     | The query string, if any                         |
+------------------+--------------------------------------------------+
| REMOTE_ADDR      | The IP address of the client                     |
+------------------+--------------------------------------------------+
| REMOTE_PORT      | The port of the client machine                   |
+------------------+--------------------------------------------------+
| REQUEST_METHOD   | The HTTP request method                          |
+------------------+--------------------------------------------------+
| REQUEST_SCHEME   | The scheme of the reqeust (e.g. http)            |
+------------------+--------------------------------------------------+
| REQUEST_URI      | The requested URI                                |
+------------------+--------------------------------------------------+
| SERVER_ADDR      | The IP address of the server                     |
+------------------+--------------------------------------------------+
| SERVER_NAME      | The hostname of the server or virtual host       |
+------------------+--------------------------------------------------+
| SERVER_PORT      | The port on the server                           |
+------------------+--------------------------------------------------+
| SERVER_PROTOCOL  | Name and version of the request protocol         |
+------------------+--------------------------------------------------+
| SERVER_SIGNATURE | Server version and virtual host name added to    |
|                  | server-generated pages                           |
+------------------+--------------------------------------------------+
| SERVER_SOFTWARE  | Server identification string                     |
+------------------+--------------------------------------------------+
| UNIQUE_ID        | ID set to be unique across requests.             |
|                  | mod_unique_id                                    |
+------------------+--------------------------------------------------+

.. _basic_authentication:

Basic Authentication
--------------------

Basic authentication is managed by Apache.

-  AUTH_TYPE=Basic
-  REMOTE_USER=username

The configuration may look something like:

| `` AuthType Basic``
| `` AuthName "Restricted Files"``
| `` AuthBasicProvider file``
| `` AuthUserFile /etc/httpd/conf/passwords``
| `` Require valid-user``

The user database would be created with this:

``htpasswd -c /etc/httpd/conf/passwords testuser``

.. _kerberos_authentication:

Kerberos Authentication
-----------------------

Kerberos authentication is managed by mod_auth_gssapi or mod_auth_kerb.

-  AUTH_TYPE=Negotiate
-  REMOTE_USER=admin@EXAMPLE.COM

If delegation is enabled in the client and the server configuration
includes KrbSaveCredentials on, then KRB5CCNAME will be set pointing to
the user's keytab.

The configuration may look something like

| `` AuthType GSSAPI``
| `` AuthName "Kerberos Login"``
| `` GssapiCredStore keytab:/etc/http.keytab``

for mod_auth_gssapi or for mod_auth_kerb:

| `` AuthType Kerberos``
| `` AuthName "Kerberos Login"``
| `` KrbMethodNegotiate on``
| `` KrbMethodK5Passwd off``
| `` KrbServiceName HTTP``
| `` KrbAuthRealms EXAMPLE.COM``
| `` Krb5KeyTab /etc/httpd/conf/ipa.keytab``
| `` KrbSaveCredentials on``

.. _x.509_authentication:

X.509 Authentication
--------------------

X.509 authentication is managed by either mod_nss or mod_ssl (or
mod_gnutls about which I know very little).

No specific AUTH_TYPE is set, see
https://issues.apache.org/bugzilla/show_bug.cgi?id=45058

The value of REMOTE_USER is dependent upon the configuration. If
SSLUserName or NSSUserName is set then that component of the client
certificate DN is set. The exception is when FakeBasicAuth is set, in
which case the full DN is set.

By default only the standard CGI environment variables are included,
plus HTTPS.

A number of SSL-specific variables are set if ExportCertData is enabled
in SSLOptions or NSSOptions.

There may be some slight differences in the variables available in
mod_ssl and mod_nss. For example, SSL_TLS_SNI is not available in
mod_nss.

| ``<Directory "/var/www/secure">``
| ``    NSSOptions +StdEnvVars``
| ``    NSSVerifyClient Require``

The mod_ssl configuration is similar. Replace NSS with SSL.

The variables are named the same between mod_ssl and mod_nss though the
contents may differ slightly. The set of variables available in httpd
2.4 are.

+-------------------------+-------------------------------------------+
| HTTPS                   | HTTPS is being used.                      |
+-------------------------+-------------------------------------------+
| SSL_PROTOCOL            | The SSL protocol version (SSLv2, SSLv3,   |
|                         | TLSv1, TLSv1.1, TLSv1.2)                  |
+-------------------------+-------------------------------------------+
| SSL_SESSION_ID          | The hex-encoded SSL session id            |
+-------------------------+-------------------------------------------+
| SSL_CIPHER              | The cipher specification name             |
+-------------------------+-------------------------------------------+
| SSL_CIPHER_EXPORT       | true if cipher is an export cipher        |
+-------------------------+-------------------------------------------+
| SSL_CIPHER_USEKEYSIZE   | Number of cipher bits (actually used)     |
+-------------------------+-------------------------------------------+
| SSL_CIPHER_ALGKEYSIZE   | Number of cipher bits (possible)          |
+-------------------------+-------------------------------------------+
| SSL_COMPRESS_METHOD     | SSL compression method negotiated         |
+-------------------------+-------------------------------------------+
| SSL_VERSION_INTERFACE   | The mod_ssl program version               |
+-------------------------+-------------------------------------------+
| SSL_VERSION_LIBRARY     | The OpenSSL program version               |
+-------------------------+-------------------------------------------+
| SSL_CLIENT_M_VERSION    | The version of the client certificate     |
+-------------------------+-------------------------------------------+
| SSL_CLIENT_M_SERIAL     | The serial of the client certificate      |
+-------------------------+-------------------------------------------+
| SSL_CLIENT_S_DN         | Subject DN in client's certificate        |
+-------------------------+-------------------------------------------+
| SSL_CLIENT_S_DN_x509    | Component of client's Subject DN          |
+-------------------------+-------------------------------------------+
| SSL_CLIENT_I_DN         | Issuer DN of client's certificate         |
+-------------------------+-------------------------------------------+
| SSL_CLIENT_I_DN_x509    | Component of client's Issuer DN           |
+-------------------------+-------------------------------------------+
| SSL_CLIENT_V_START      | Validity of client's certificate (start   |
|                         | time)                                     |
+-------------------------+-------------------------------------------+
| SSL_CLIENT_V_END        | Validity of client's certificate (end     |
|                         | time)                                     |
+-------------------------+-------------------------------------------+
| SSL_CLIENT_V_REMAIN     | Number of days until client's certificate |
|                         | expires                                   |
+-------------------------+-------------------------------------------+
| SSL_CLIENT_A_SIG        | Algorithm used for the signature of       |
|                         | client's certificate                      |
+-------------------------+-------------------------------------------+
| SSL_CLIENT_A_KEY        | Algorithm used for the public key of      |
|                         | client's certificate                      |
+-------------------------+-------------------------------------------+
| SSL_CLIENT_CERT         | PEM-encoded client certificate            |
+-------------------------+-------------------------------------------+
| SSL_CLIENT_CERT_CHAIN_n | PEM-encoded certificates in client        |
|                         | certificate chain                         |
+-------------------------+-------------------------------------------+
| SSL_CLIENT_VERIFY       | NONE, SUCCESS, GENEROUS or FAILED:reason  |
+-------------------------+-------------------------------------------+
| SSL_SERVER_M_VERSION    | The version of the server certificate     |
+-------------------------+-------------------------------------------+
| SSL_SERVER_M_SERIAL     | The serial of the server certificate      |
+-------------------------+-------------------------------------------+
| SSL_SERVER_S_DN         | Subject DN in server's certificate        |
+-------------------------+-------------------------------------------+
| SSL_SERVER_S_DN_x509    | Component of server's Subject DN          |
+-------------------------+-------------------------------------------+
| SSL_SERVER_I_DN         | Issuer DN of server's certificate         |
+-------------------------+-------------------------------------------+
| SSL_SERVER_I_DN_x509    | Component of server's Issuer DN           |
+-------------------------+-------------------------------------------+
| SSL_SERVER_V_START      | Validity of server's certificate (start   |
|                         | time)                                     |
+-------------------------+-------------------------------------------+
| SSL_SERVER_V_END        | Validity of server's certificate (end     |
|                         | time)                                     |
+-------------------------+-------------------------------------------+
| SSL_SERVER_A_SIG        | Algorithm used for the signature of       |
|                         | server's certificate                      |
+-------------------------+-------------------------------------------+
| SSL_SERVER_A_KEY        | Algorithm used for the public key of      |
|                         | server's certificate                      |
+-------------------------+-------------------------------------------+
| SSL_SERVER_CERT         | PEM-encoded server certificate            |
+-------------------------+-------------------------------------------+
| SSL_TLS_SNI             | Contents of the SNI TLS extension (if     |
|                         | supplied with ClientHello)                |
+-------------------------+-------------------------------------------+

.. _ldap_authentication:

LDAP authentication
-------------------

Apache provides the module mod_authnz_ldap to perform authentication and
authorization over LDAP.

A simple configuration looks like:

| ``   AuthType Basic``
| ``   AuthName "LDAP Protected"``
| ``   AuthBasicProvider ldap``
| ``   AuthLDAPURL ``\ ```ldap://127.0.0.1/dc=example,dc=com?uid?one`` <ldap://127.0.0.1/dc=example,dc=com?uid?one>`__
| ``   Require valid-user``

Authorization can be done by specifying the allowed users, groups,
attribute with in an entry or even a filter.

Attributes can be specified in the AuthLDAPURL value such that those
values are set as environment variables of the form "AUTHENTICATE_", so
any arbitrary list of values may be provided.

.. _proposed_additional_variables:

Proposed Additional Variables
-----------------------------

When Apache module is used for authentication, the authentication result
is passed to the application typically in the form of environment
variable REMOTE_USER. Current web applications however want and need to
create the user record in their internal databases so that foreign keys
validate, and applications also want to do access control checks
(authorizations) -- applications typically don't rely on Apache modules
for authorization.

We are in need of a way for Apache modules to pass information about the
authenticated user beyond the login name (in REMOTE_USER) to the
application. That way the applications do not need to implement all
possible authentication mechanisms (Kerberos, SAML, LDAP, ...) and can
depend on specialized mod_auth_\* modules to do it, while being able to
know what user to populate and maintain in their internal user database.

We propose Apache modules that wish to pass information about users to
applications adopt the following environment variable names:

+----------------+----------------+----------------+----------------+
| Variable name  | Semantics      | Possible       | Example        |
|                |                | source         | `mod_l         |
|                |                |                | ookup_identity |
|                |                |                |  <http://www.a |
|                |                |                | delton.com/apa |
|                |                |                | che/mod_lookup |
|                |                |                | _identity/>`__ |
|                |                |                | configuration  |
+================+================+================+================+
| REMO           | c              | POSIX call     | Look           |
| TE_USER_GROUPS | olon-separated | getgrouplist;  | upOutputGroups |
|                | list of group  | sssd dbus call | REMO           |
|                | names the user | o              | TE_USER_GROUPS |
|                | is in          | rg.freedesktop | :              |
|                |                | .sssd.infopipe |                |
|                |                | .GetUserGroups |                |
+----------------+----------------+----------------+----------------+
| REMOTE         | number of user | alternate way  | Lookup         |
| _USER_GROUP_N, | groups and     | to get the     | UserGroupsIter |
| REMOTE         | individual     | list of        | REM            |
| _USER_GROUP_1, | group names    | groups,        | OTE_USER_GROUP |
| REMOTE         |                | avoiding the   |                |
| _USER_GROUP_2, |                | split needed   |                |
| ...            |                | with           |                |
|                |                | REMO           |                |
|                |                | TE_USER_GROUPS |                |
+----------------+----------------+----------------+----------------+
| REM            | Equivalent of  | pw_gecos field | L              |
| OTE_USER_GECOS | the GECOS      | of result of   | ookupUserGECOS |
|                | value from the | POSIX call     | REM            |
|                | password file, | getpwname; IPA | OTE_USER_GECOS |
|                | could be full  | attribute      | or             |
|                | name.          | gecos, sssd    | LookupUserAttr |
|                |                | dbus call      | gecos          |
|                |                | org.freedeskt  | REM            |
|                |                | op.sssd.infopi | OTE_USER_GECOS |
|                |                | pe.GetUserAttr |                |
|                |                | gecos          |                |
+----------------+----------------+----------------+----------------+
| REMO           | domain the     |                |                |
| TE_USER_DOMAIN | user was       |                |                |
|                | authenticated  |                |                |
|                | in (could be   |                |                |
|                | the domain in  |                |                |
|                | sssd, nss,     |                |                |
|                | LDAP, etc.)    |                |                |
+----------------+----------------+----------------+----------------+
| REM            | user's email   | IPA attribute  | LookupUserAttr |
| OTE_USER_EMAIL | address        | mail,          | mail           |
|                |                | sssd-dbus      | REM            |
|                |                | attribute mail | OTE_USER_EMAIL |
+----------------+----------------+----------------+----------------+
| REMOTE_US      | list of groups |                |                |
| ER_GROUPS_JSON | the user is    |                |                |
|                | in, formatted  |                |                |
|                | as JSON string |                |                |
+----------------+----------------+----------------+----------------+
| REMOTE_        | user's first   | IPA attribute  | LookupUserAttr |
| USER_FIRSTNAME | name           | givenname,     | givenname      |
|                |                | sssd-dbus      | REMOTE_        |
|                |                | attribute      | USER_FIRSTNAME |
|                |                | givenname      |                |
+----------------+----------------+----------------+----------------+
| REMOTE_U       | user's middle  |                |                |
| SER_MIDDLENAME | name           |                |                |
+----------------+----------------+----------------+----------------+
| REMOTE         | user's last    | IPA attribute  | LookupUserAttr |
| _USER_LASTNAME | name           | sn, sssd-dbus  | sn             |
|                |                | attribute sn   | REMOTE         |
|                |                |                | _USER_LASTNAME |
+----------------+----------------+----------------+----------------+
| REMOTE         | user's full    | IPA attribute  | LookupUserAttr |
| _USER_FULLNAME | name formatted | cn or          | cn             |
|                | as one string  | displayname,   | REMOTE         |
|                | (similar to    | sssd-dbus      | _USER_FULLNAME |
|                | and possibly   | attribute cn   | or             |
|                | the same as    | or displayname | LookupUserAttr |
|                | REMO           |                | displayname    |
|                | TE_USER_GECOS) |                | REMOTE         |
|                |                |                | _USER_FULLNAME |
+----------------+----------------+----------------+----------------+
| REMOT          | organizational | IPA attribute  | LookupUserAttr |
| E_USER_ORGUNIT | unit to which  | ou, sssd-dbus  | ou             |
|                | the user       | attribute ou   | REMOT          |
|                | belongs        |                | E_USER_ORGUNIT |
+----------------+----------------+----------------+----------------+
| REMOTE_US      | SID, GUID, or  | IPA attribute  | LookupUserAttr |
| ER_EXTERNAL_ID | other unique   | ipaUniqueId,   | ipaUniqueId    |
|                | identifier     | 389 DS         | REMOTE_US      |
|                | from the       | attribute      | ER_EXTERNAL_ID |
|                | external       | nsUniqueID, AD |                |
|                | identity       | attribute      |                |
|                | provider; used | objectSid      |                |
|                | to reconcile   |                |                |
|                | account after  |                |                |
|                | login change   |                |                |
+----------------+----------------+----------------+----------------+
| EXTER          | when external  |                |                |
| NAL_AUTH_ERROR | authentication |                |                |
|                | fails (and     |                |                |
|                | REMOTE_USER is |                |                |
|                | not set), this |                |                |
|                | variable can   |                |                |
|                | contain error  |                |                |
|                | describing the |                |                |
|                | reason         |                |                |
+----------------+----------------+----------------+----------------+

The character set for values should be UTF-8.

The list above is not exhaustive, authentication and identity modules
can provide additional variables with other values and meanings and
applications are welcome to use them.

Module mod_lookup_identity
(`documentation <http://www.adelton.com/apache/mod_lookup_identity/>`__,
`git
repo <http://fedorapeople.org/cgit/adelton/public_git/mod_lookup_identity.git/>`__)
has been created as a proof of concept for this way of information
passing. The full functionality depends on the sssd-dbus package (not
yet released, in testing).

Module mod_intercept_form_submit
(`documentation <http://www.adelton.com/apache/mod_intercept_form_submit/>`__,
`git
repo <http://fedorapeople.org/cgit/adelton/public_git/mod_intercept_form_submit.git/>`__)
has been created as a proof of concept for PAM authentication based on
form submission and it supports the REMOTE_USER and EXTERNAL_AUTH_ERROR
outputs, plus mod_lookup_identity can work based on the
mod_intercept_form_submit authentication result (latest versions of both
modules required).
