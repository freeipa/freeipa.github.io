The typical web applications nowadays use HTTP
`cookie <http://en.wikipedia.org/wiki/HTTP_cookie>`__-based
authentication sessions, usually with login-form to enter login and
password pair which is then validated by the application against some
internal user database. Session record is then created and cookie set,
which the browser will send with each subsequent request to the
application. The application can then show data related to the
authenticated user (shopping cart content, user's posts, stored files)
throughout their work with the application.

In large organizations and enterprise deployments, user identities are
usually managed in some central manner. Many programming languages and
frameworks provide libraries/modules to authenticate for example against
`LDAP <http://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol>`__
sources. However, when the full complexity of enterprise setups is
considered, including
`Kerberos <http://en.wikipedia.org/wiki/Kerberos_%28protocol%29>`__
authentication, `failovers <http://en.wikipedia.org/wiki/Failover>`__,
use of `Active
Directory <http://en.wikipedia.org/wiki/Active_Directory>`__, and
identity federation, it can be useful to offload the authentication
support to solution independent on the application code or framework --
`Apache <http://en.wikipedia.org/wiki/Apache_HTTP_Server>`__ modules,
and where needed, `System Security Services Daemon
(sssd) <https://fedorahosted.org/sssd/>`__.

With fairly minimal changes, web applications can consume the results of
authentication performed by Apache modules, using the standard
REMOTE_USER environment variable/attribute/method. If applications know
how to handle the authentication result coming from the underlying
(front end) web server, it is then just a matter of configuration of the
web server to add access control to Kerberos authentication, federated
authentication via SAML, or use central identity management server like
FreeIPA to authenticate [login, password] values submitted by user to
application's native logon-form.

If the application is then extended to understand `additional proposed
environment
variables <Environment_Variables#Proposed_Additional_Variables>`__, it
can receive not just the authentication and authorization result but
also additional information about the user, for example email address or
full name, typically needed by application to provide the full
functionality and good user experience. With group membership
information available to the application, the application can base its
application-specific roles/access rights/authorization for the user
based on the external user information, again without having to add
support for all of them in the application or framework code directly.

There is an `example setup <Web_App_Authentication/Example_setup>`__
accompanying this page which demonstrates the concepts described below
on a trivial CGI application.

We do not expect applications to drop their existing functionality that
served them well, this is merely an additional possibility.

.. _the_use_of_sssd:

The use of sssd
---------------

The `System Security Services Daemon
(sssd) <https://fedorahosted.org/sssd/>`__ is present as a standard part
of the latest Red Hat Enterprise Linux, Fedora, and related
distributions. It provides access to identity and authentication
services and is primarily aimed at the operating system level. In this
document, we will explore ways to use it for authentication and identity
access of web applications, while preserving the distinction of the
operating system and web application deployed on it.

We will assume that the system on which the web application is deployed
is IPA-enrolled. Using the command

::

   ipa-client-install

the local configuration of a couple of subsystems including sssd can be
set up to point to a FreeIPA server. It also creates a host record on
the server, making it possible to add services and get their Kerberos
keytab.

Services are important because they make it possible to have
fine-grained control over a user's access to the system. We might want
to give a network administrator ssh access to the machine, but not
access to the accounting system running on it. We might want to allow
employees in the finance department to be able to access the accounting
web application, but not ssh access to the underlying host.

Kerberos
--------

All contemporary web browsers support SPNEGO and Kerberos. When
accessing http://server.example.com/ or https://server.example.com/, the
server can propose **Negotiate** authentication method. If the user
currently has ticket granted ticket (obtained for example via ``kinit``
command), the brower will obtain Kerberos tickets for principal
HTTP/server.example.com@EXAMPLE.COM (as opposed to ssh which uses
host/server.example.com@EXAMPLE.COM) and use related GSSAPI data to
authenticate to the server.

To enable this method on typical Apache installation,
**mod_auth_gssapi** or **mod_auth_kerb** module needs to be `installed
and configured <Web_App_Authentication/Example_setup#Kerberos>`__.

.. _application_change_remote_user:

Application change: REMOTE_USER
-------------------------------

The application then needs to be able to retrieve the result of the
authentication, the login (principal) of the authenticated user. The
standard mechanism for CGI deployments is via REMOTE_USER environment
variable, and almost every web framework has a way to get this value
from the web server, either via environment variable, attribute, or
dedicated method call. (It's the same value and mechanism used for the
HTTP Basic Authentication.)

The application not only needs to retrieve the value but its processing
flow needs to be modified to trust this value. Note that this variable
will not be set unless the admin deploying and setting up the
application configured the underlying Apache correctly. For application
developers it might be a step from having complete control over the
internal user database to more uncertainty, having to trust that the
external value is correct.

Furthermore, many applications will need to have record for any
authenticated user in its internal database, even for the externally
(Kerberos) authenticated ones, for foreign keys to work. That is fine --
the first time the application sees REMOTE_USER set to value it does not
have in the database, it can create the user record in its internal
database on the fly. That way, even if it uses object-relational
mapping, the application will still work. (See below if the application
requires that the user record has certain attributes besides login NOT
NULL.)

.. _other_authentication_modules:

Other authentication modules
----------------------------

Besides Kerberos (with mod_auth_gssapi or mod_auth_kerb), there are
other mechanisms that can be configured in Apache to authenticate:

================================ ============================
Authentication Method            Apache Authentication Module
================================ ============================
Pure Application Level           *None*
Kerberos Single Sign-On (ticket) mod_auth_gssapi
mod_auth_kerb                    
SAML-based                       mod_auth_mellon
Certificate-based                mod_nss
mod_ssl                          
\                                
================================ ============================

.. _host_and_service_based_access_control:

Host and service based access control
-------------------------------------

If mod_auth_gssapi/mod_auth_kerb is configured and application extended
to consult and trust the REMOTE_USER value, it may have a potentially
unwanted side effect -- any user who is able to get the Kerberos ticket
for HTTP/server.example.com@EXAMPLE.COM would then be able to log into
the application. If the application is only intended for small set of
users and if any successful external authentication will populate user
record in application's database, that will pollute the database with
users who have no access rights in the application yet manage to come
across its URL and get authenticated.

A possible solution is to apply access control check to the Kerberos
authentication method on the Apache level. That way, even if the user is
able to get the Kerberos ticket the authentication will still fail. For
this to work, we will use Apache module **mod_authnz_pam**, configure
PAM service to use pam_sss.so, hooking it via sssd to the FreeIPA
server, and set up host-based access control (HBAC) rule in FreeIPA to
separate service, to only give access to a particular set or group of
users.

.. _hbac_rules:

HBAC rules
~~~~~~~~~~

We will start from the end -- from the FreeIPA HBAC service. It is just
a string which distinguishes one service from another. Running

::

   ipa hbacsvc-find

will show pre-created services like ssh, kdm, login, or kdm. Their names
are then used to define the respective PAM service on the client -- so
for ssh, the configuration is in /etc/pam.d/ssh. If we are adding
service for a reporting web application in our organization, we can name
it **reporting** or **reporting.example.com** or **reporting-prod** and
**reporting-qa** if we have multiple environments. Please consult help
pages

::

   ipa help hbacsvc
   ipa help hbacrule
   ipa help hbactest

for detailed description of creating HBAC services and rules in FreeIPA.
Please also note that you will probably need to `disable the default
allow_all HBAC rule <Howto/HBAC_and_allow_all>`__ for the mechanism to
work properly.

.. _pam_service:

PAM service
~~~~~~~~~~~

On the IPA-enrolled machine on which the web application is being
configured, we need to define the PAM service to use sssd. We create
file named the same as the HBAC service we've created with
``ipa hbacsvc-add`` and configure ``pam_sss.so`` for both auth and
account. For example, if the HBAC service is **reporting-prod**, we will
need file **/etc/pam.d/reporting-prod** with content

::

   auth    required   pam_sss.so
   account required   pam_sss.so

mod_authnz_pam
~~~~~~~~~~~~~~

The module **mod_authnz_pam** adds access control checks to
authentication phase of HTTP request processing in Apache. The typical
mod_auth_gssapi/mod_auth_kerb configuration will have

::

   require valid-user

in it, saying that any authenticated user should be allowed. When we
change it to ``require pam-account PAM-service``, the user will only be
authenticated by Apache if it matches the ``account`` check in PAM,
which in case of ``pam_sss.so`` and sssd being configured to consult
FreeIPA will lead to HBAC rule check, with the PAM service name used as
the HBAC service. For our **reporting-prod** example, the
``require valid-user`` will change to

::

   require pam-account reporting-prod

We can even used different PAM services for different parts of the
application, provided they can be identified using URLs. If the
application has a special admin section, we can define separate PAM
service (which possibly more strict rules) for this part:

::

   <Location /app>
   require pam-account reporting-prod
   </Location>
   <Location /app/admin>
   require pam-account reporting-prod-admin
   </Location>

Overview
~~~~~~~~

The **mod_authnz_pam** module can be configured with any other module
which uses the ``require`` Apache directive. The deployment matrix then
changes to:

================================ ===============
Authentication Method            Apache Modules
================================ ===============
Authentication                   Access Control
Pure Application Level           *None*
Kerberos Single Sign-On (ticket) mod_auth_gssapi
mod_auth_kerb                    
SAML-based                       mod_auth_mellon
Certificate-based                mod_nss
mod_ssl                          
\                                
================================ ===============

Please consult the `example setup
page <Web_App_Authentication/Example_setup#Host_.28and_service.29_based_access_control_for_Kerberos>`__
for detailed configuration steps.

.. _login_form_using_freeipa:

Login form using FreeIPA
------------------------

In many situations, neither Kerberos nor any other authentication method
which requires some additional setup on client's side (like
certificates) can be used or mandated, for practical reasons. Still, if
the organization has a central user management in the form of FreeIPA,
it can connect its existing applications to the FreeIPA authentication
service, while retaining the application-specific look and feel of its
login form. All that it takes for application is to understand the
REMOTE_USER result of Apache authentication modules.

The central authentication is achieved using the HBAC and PAM service
described above, and Apache module **mod_intercept_form_submit**. The
module can be configured to look at HTTP POST request resulting from
user submitting application's login form, and if login and password are
found in the request, it will run PAM authentication and access control
checks, using service specified with ``InterceptFormPAMService``
directive. The module internally calls mod_authnz_pam. When the service
is properly configured to use ``pam_sss.so`` and sssd is configured to
use FreeIPA, this form submit interception will validate the
login/password pair, plus do the access control check like in the setup
with Kerberos, described above.

The successful result of this authentication is again passed using the
REMOTE_USER mechanism. If application consults this value and trusts it,
it will consider the user authenticated without checking its local user
database.

The failed authentication result is signalled to the application via
environment variable EXTERNAL_AUTH_ERROR and applications are welcome to
use this result indication.

The proposed mix of authentication setups expands to

================================ =========================
Authentication Method            Apache Modules
================================ =========================
Authentication                   Access Control
Pure Application Level           *None*
Kerberos Single Sign-On (ticket) mod_auth_gssapi
mod_auth_kerb                    
SAML-based                       mod_auth_mellon
Certificate-based                mod_nss
mod_ssl                          
Login form-based                 mod_intercept_form_submit
\                                
================================ =========================

The `example setup
page <Web_App_Authentication/Example_setup#External_identities_for_login_form>`__
has more details about the configuration.

.. _additional_user_information:

Additional user information
---------------------------

The FreeIPA server can not only store plain login identities and
passwords for authentication services, it can also hold additional user
attributes like email addresses, phone numbers, or full names of users,
as well as group membership. The sssd is then able to access this
information and make it available to applications via new **sssd-dbus**
package.

Using Apache module **mod_lookup_identity** which can talk to sssd's ifp
service over dbus, any Apache module's authenticated user can have
additional environment variables populated from the central identity
provider like FreeIPA. This can be used if the application requires that
additional attributes are filled before storing the user in its internal
database, or simply if the application makes use of such data. On the
sssd side, the list of LDAP attributes that need to be retrieved and
cached is specified, and then in mod_lookup_identity's configuration,
these attributes are mapped to environment variables.

One type of data that the sssd-dbus calls provides is user's group
membership. This can be used to populate application-specific roles of
the externally-authenticated user. Consider a situation when a newly
hired network administrator is added to group **netadmin** in
organization's FreeIPA server. Module mod_lookup_identity is able to
retrieve this group name and populate environment variable like
REMOTE_USER_GROUP_N, REMOTE_USER_GROUP_1, ..., or REMOTE_USER_GROUPS as
colon-separated list. System management and provisioning application can
hold internal mapping of the external group **netadmin** to its internal
role and access control handling, making such a user automatically have
appropriate privileges.

When using the attributes to populate the database with
externally-authenticated users, it is good to consider the case when
user's details or group membership in the central identity provider
change. It might be useful to not only populate the user record when it
is not found in application's database the first time the user
authenticates, but also compare and update the information every time
the user authenticates, if needed. This is especially important if group
membership is linked to application's role handling.

Populating of additional attributes, mapping of groups to roles, and
update of this information in application's database are therefore
additional changes that the web application developers might consider
adding to their application to make deployment of their application
easier in large enterprise environment, without getting necessarily deep
into the details of each possible identity provider which the
organizations might use. The applications only need to assume that the
environment variables (or whatever is the method of handling this
information in its programming language or framework) might be populated
by the HTTP daemon setup and its modules.

It is also our hope that other modules that might have the additional
user attributes available (like SAML) might populate the `proposed
environment
variables <Environment_Variables#Proposed_Additional_Variables>`__
directly, so even if the web application deployment does not use neither
FreeIPA, sssd, nor any of the Apache modules mentioned on this page,
effort that went into modifications of web applications can still be
used.

The `example setup
page <Web_App_Authentication/Example_setup#Storing_external_users_in_internal_databases>`__
describes the sssd-dbus and mod_lookup_identity setup in more detail.

The whole proposed solution for web application authentication using
sssd:

================================ =========================
Authentication Method            Apache Modules
================================ =========================
Authentication                   Access Control
Pure Application Level           *None*
Kerberos Single Sign-On (ticket) mod_auth_gssapi
mod_auth_kerb                    
SAML-based                       mod_auth_mellon
Certificate-based                mod_nss
mod_ssl                          
Login form-based                 mod_intercept_form_submit
\                                
================================ =========================

Note: sssd call also be configured to use different identity providers
than FreeIPA but such setup is beyond the scope of this overview.

.. _namespace_separation:

Namespace Separation
--------------------

In the above description we have assumed that the admin wants to handle
all their application users with external authentication and that the
set of user identities (locally created/managed and the
externally-authenticated) overlap. Depending on the use case this might
or might not be desirable. Consult `Namespace
separation <Web_App_Authentication/Namespace_separation>`__ for possible
setups with externally-authenticated users marked with @REALM and
multiple IPA server setups.

SAML
----

As mentioned above, when the Web application / framework is amended to
be able to process REMOTE_USER and REMOTE_USER_GROUP_\* environment
variables, it's then just a matter of configuration of the front-end
server to enable a particular mechanism of external authentication. For
example, for SAML (Security Assertion Markup Language),
`mod_auth_mellon <https://github.com/UNINETT/mod_auth_mellon>`__ can be
used and starting with version 0.11.0, it can be configured to populate
environment variables exactly like mod_lookup_identity does:

::

       MellonSetEnvNoPrefix "REMOTE_USER_GROUP" "groups"
       MellonEnvVarsIndexStart 1
       MellonEnvVarsSetCount On

and we can pass other attributes as well:

::

       MellonSetEnvNoPrefix "REMOTE_USER_LASTNAME" "surname"
       MellonSetEnvNoPrefix "REMOTE_USER_FIRSTNAME" "givenname"
       MellonSetEnvNoPrefix "REMOTE_USER_EMAIL" "email"

References
----------

OpenStack
~~~~~~~~~

-  Nathan Kinder's https://blog-nkinder.rhcloud.com/?p=130
-  Adam Young's http://adam.younglogic.com/2015/03/key-fed-lookup-redux/

Spacewalk
~~~~~~~~~

In Spacewalk 2.1, it is possible to use both the Kerberos
authentication, and the form-based PAM authentication, including the
HBAC management from FreeIPA and mapping of group membership from the
identity provider to Spacewalk roles (access rights) -- see
https://fedorahosted.org/spacewalk/wiki/SpacewalkAndIPA for full
documentation of the feature.

Satellite
~~~~~~~~~

Based on Spacewalk upstream, the capability is now also available in
Satellite 5.7 and documented in the `Using Identity Management for
Authentication <https://access.redhat.com/documentation/en-US/Red_Hat_Satellite/5.7/html/Installation_Guide/ch06s02.html>`__
chapter of the Installation Guide.

Foreman
~~~~~~~

In Foreman 1.5, the external authentication is fully implemented as
described on this page -- see the `tracking
issue <http://projects.theforeman.org/issues/5031>`__ with links to
individual issues and pull requests that introduced the feature. It is
now documented in `Foreman
manual <http://theforeman.org/manuals/1.6/index.html#5.7ExternalAuthentication>`__.

.. _satellite_6:

Satellite 6
~~~~~~~~~~~

Based on Foreman upstream, the capability is now also available in
Satellite 6.0:
https://access.redhat.com/documentation/en-US/Red_Hat_Satellite/6.1/html/User_Guide/sect-Using-IdM-for-Authentication.html

ManageIQ
~~~~~~~~

The support for external authentication is now in manageiq master:
https://github.com/ManageIQ/manageiq/commit/e0423c18d48380ff8d490ccb08291d2098fde69f.
The feature is configured by the console:
https://github.com/ManageIQ/manageiq/commit/cc6ea8b103a23bee5af8f9d88eac3024fc26cf18,
or manually:
https://github.com/ManageIQ/guides/blob/master/external_auth.md and
https://github.com/ManageIQ/guides/blob/master/external_auth/configuration.md.

CloudForms
~~~~~~~~~~

Based on the ManageIQ upstream, the capability is now also available in
CFME 5.3 and documented in the
`Configuration <https://access.redhat.com/documentation/en-US/Red_Hat_CloudForms/3.1/html/Management_Engine_5.3_Settings_and_Operations_Guide/chap-Configuration.html>`__
chapter of the Settings and Operations Guide.

OpenDayLight
~~~~~~~~~~~~

`Federated Authentication Utilizing Apache &
SSSD <https://jdennis.fedorapeople.org/doc/sssd_configuration.pdf>`__ by
John Dennis.

Videos
------

.. _using_os_level_identity_authentication_and_access_control_for_web_applications:

Using OS-level identity, authentication, and access control for Web applications
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  `Presentation at DevConf
   2015 <http://www.adelton.com/docs/idm/os-auth-stack-for-web-applications>`__
   https://www.youtube.com/watch?v=Hhy5__C-XFc
   {{#ev:youtube|Hhy5__C-XFc}}
   Sadly, the first five minutes of the video are without sound.

.. _identity_management_in_red_hat_enterprise_linux_web_application_authentication_series:

Identity management in Red Hat Enterprise Linux: Web Application Authentication Series
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Part I: Current standard: logon forms and cookie-based sessions
   https://www.youtube.com/watch?v=Qdv8waOk6UE
   {{#ev:youtube|Qdv8waOk6UE}}
-  Part II: Kerberos single sign-on
   https://www.youtube.com/watch?v=_We4O8OuJAY
   {{#ev:youtube|_We4O8OuJAY}}
-  Part III: Additional services of central identity provider
   https://www.youtube.com/watch?v=uG1rxZ4ydUE
   {{#ev:youtube|uG1rxZ4ydUE}}

Django
~~~~~~

-  External Authentication for Django Projects
   https://www.youtube.com/watch?v=62_jD-8zV4M
   {{#ev:youtube|62_jD-8zV4M}}

.. _foreman_demo:

Foreman demo
~~~~~~~~~~~~

-  External authentication with and installer improvements, presented by
   Marek Hulán
   https://www.youtube.com/watch?v=S-8PESGbOUk#t=475
   {{#ev:youtube|S-8PESGbOUk|||||#t=475}}

Presentations
-------------

.. _external_and_federated_identities_on_the_web:

External and Federated Identities on the Web
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  `ApacheCon Core Europe 2015
   presentation <http://www.adelton.com/docs/idm/external-and-federated-identities>`__
   (also as `PDF
   slides <http://www.adelton.com/docs/idm/external-and-federated-identities.pdf>`__)

.. _using_os_level_identity_authentication_and_access_control_for_web_applications_1:

Using OS-level identity, authentication, and access control for Web applications
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  `Developer Conference 2015
   presentation <http://www.adelton.com/docs/idm/os-auth-stack-for-web-applications>`__
   (also as `PDF
   slides <http://www.adelton.com/docs/idm/os-auth-stack-for-web-applications.pdf>`__)

.. _external_identity_and_authentication_providers_for_apache_http_server:

External Identity and Authentication Providers For Apache HTTP Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  `ApacheCon Europe 2014
   presentation <http://www.adelton.com/docs/idm/apache-external-idm-auth>`__
   (also as `PDF
   slides <http://www.adelton.com/docs/idm/apache-external-idm-auth.pdf>`__)

.. _identity_management_scaling_out_and_up:

Identity Management Scaling Out and Up
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  `LinuxCon Europe 2014
   presentation <http://www.adelton.com/docs/idm/idm-scaling-out-and-up>`__
   (also as `PDF
   slides <http://www.adelton.com/docs/idm/idm-scaling-out-and-up.pdf>`__)

.. _external_authentication_for_django_projects:

External Authentication for Django Projects
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  `EuroPython 2015
   presetnation <http://www.adelton.com/django/external-authentication-for-django-projects>`__
   (also as `PDF
   slides <http://www.adelton.com/django/external-authentication-for-django-projects.pdf>`__)
