The `Web App
Authentication <http://www.freeipa.org/page/Web_App_Authentication>`__
using sssd can be used with great benefits in environments where the
users on the operating system level and users in the web applications
come from the same identity providers. Ideally the providers are then
able to control access to OS vs. to the applications (for example with
FreeIPA, using host-based access control (HBAC) rules), and users can
enjoy single sign-on to both the OS and to the applications, based on
their access privileges. We can use mod_gssapi's

::

     GssapiLocalName on

or mod_auth_kerb's

::

     KrbLocalUserMapping On

option to enable stripping of the @REALM part from user's username, so
existing web application which already had users populated and managed
manually can start to authenticate the same users against organization's
Kerberos server, once mod_auth_gssapi/mod_auth_kerb is enabled.

.. _the_realm:

The REALM
---------

In some cases, mixing of internal application users and externally
authenticated users might not be desirable. If web application populates
its internal user database with records of externally-authenticated
users, the admin might want to explicitly separate those managed
internally from those managed by central identity and authentication
server.

One possible solution is to create all internal users without any @REALM
or @domain parts, and have all externally-authenticated users have
username in the form of 'login@REALM'.

For Kerberos ticket-based SSO using mod_auth_gssapi/mod_auth_kerb, the
default value of

::

     GssapiLocalName Off

or

::

     KrbLocalUserMapping Off

will pass the full name of the user including the @REALM part to the
application.

For PAM authentication using mod_intercept_form_submit, it might be
userful to force the @REALM part to the username even if the user does
not specify it in the login field of the logon form. Version 0.9.6 of
mod_intercept_form_submit added directive ``InterceptFormLoginRealms``.
Whenever the login name used for PAM authentication does not include the
'@' part, the realm specified is appended and authentication attempted
with the full user name (or if multiple realms are specified, they are
tried in cycle). For example, if mod_intercept_form_submit is configured
as

::

     InterceptFormPAMService wikiapp
     InterceptFormLogin login_fld
     InterceptFormPassword passwd_fld
     InterceptFormLoginRealms EXAMPLE.COM

and user attempts to log in with login 'bob', PAM authentication of
'bob@EXAMPLE.COM' is attempted against the PAM service wikiapp.

Upon successful authentication, this full user name is set to Apache's
internal structure r->user and to the environment variable REMOTE_USER,
so application will see the full user name 'bob@EXAMPLE.COM'.

This way, user names produced by Kerberos SSO and PAM authentication via
mod_intercept_form_submit will have the same format and the same user
records will be used by the applications.

.. _multiple_freeipa_servers:

Multiple FreeIPA servers
------------------------

For web application development, using single identity provider might
not be ideal. Web application developers might want to install their
testing FreeIPA or other identity and authentication provider and use it
solely for the development and testing of the web application, having it
completely under their control, including the ability to add and modify
users or get keytabs. At the same time they may want to control the
access to the operating system using the organization's identity and
authentication provider.

To use second FreeIPA server for the web application development or
testing on already IPA-enrolled machine, we install and configure the
server on separate host using

::

   ipa-server-install --no_hbac_allow

It is important to choose different realm than the one used by the
primary identity provider to maintain proper namespace separation.

In the example above, we have used the ``--no_hbac_allow`` option.
Alternatively you might want to `disable the default allow_all HBAC
rule <http://www.freeipa.org/page/Howto/HBAC_and_allow_all>`__. Since
the testing users we will create in our second FreeIPA server will be
seen by sssd on our web server, incorrectly configured access control
can grant ssh access using these testing users.

We will want to manually create host and service records for our web
application machine. Assuming it is wikiapp.example.com, we run

::

   kinit admin
   ipa host-add wikiapp.example.com
   ipa service-add HTTP/wikiapp.example.com

on the second FreeIPA server. We can also use ``ipa user-add`` to add
some testing users. Make sure HBAC rules are properly set.

On the IPA-enrolled machine, we cannot run ``ipa-client-install`` -- it
will complain that the system is already IPA-enrolled. Therefore, we
will configure access to the second FreeIPA server manually. In the
following examples we will assume that the realm of the second FreeIPA
server is EXAMPLE.COM and its hostname is ipa2.devel.company.net.

To **/etc/krb5.conf**, we add

::

   [realms]
           EXAMPLE.COM = {
                   kdc = ipa2.devel.company.net:88
                   master_kdc = ipa2.devel.company.net:88
                   admin_server = ipa2.devel.company.net:749
           }

   [domain_realm]
           ipa2.devel.company.net = EXAMPLE.COM

We should now be able to

::

   kinit admin@EXAMPLE.COM

If it succeeds, we can retrieve the keytab for the host and for the HTTP
service. The keytab for the host is used by sssd when connecting to our
second FreeIPA server to get for example user attributes. The keytab for
the HTTP service is used by mod_auth_gssapi/mod_auth_kerb to facilitate
Negotiate authentication.

::

   ipa-getkeytab -s ipa2.devel.company.net -k /etc/krb5.keytab -p host/wikiapp.example.com@EXAMPLE.COM
   ipa-getkeytab -s ipa2.devel.company.net -k /etc/http.keytab -p HTTP/wikiapp.example.com@EXAMPLE.COM

This is the only time the

::

   [domain_realm]
           ipa2.devel.company.net = EXAMPLE.COM

part in krb5.conf is needed. We could have omitted it and could have run
the ipa-getkeytab commands directly on the second FreeIPA server and
copied the keytabs to our machine manually. That approach however can
fail if our machine and the FreeIPA server have different OS versions --
the resulting keytabs may not be usable. Besides, it is good to validate
that the authentication works, for admin anyway.

Note that we have stored the host keytab in /etc/krb5.keytab which has
already been created and populated by ``ipa-client-install`` with our
machine's primary keytab, from the primary FreeIPA. The same keytab file
can be used as long as the realms are different.

The next part to configure is sssd, in **/etc/sssd/sssd.conf**:

::

   # in [sssd] section, append EXAMPLE.COM to domains
   [sssd]
   services = nss, pam, ssh, ifp
   config_file_version = 2
   domains = company.net, EXAMPLE.COM
   # add new section [domain/EXAMPLE.COM]
   [domain/EXAMPLE.COM]
   id_provider = ipa
   auth_provider = ipa
   access_provider = ipa
   ipa_server = ipa2.devel.company.net
   ldap_user_extra_attrs = mail, givenname, sn
   use_fully_qualified_names = True

The ``ldap_user_extra_attrs`` should be set to whatever our application
and mod_lookup_identity configuration will need for proper operation.
The ``use_fully_qualified_names`` option will enforce full user names to
be used. It is easy to achieve even for mod_intercept_form_submit with
``InterceptFormLoginRealms`` option described above and it will ensure
that mod_lookup_identity correctly makes lookup calls to the correct
FreeIPA server, should there be for example user 'bob' both in the
central organization's server and in ipa2.devel.company.net.

Restarting sssd

::

   service sssd restart

and possibly httpd (if we've done some changes) should make the machine
ready for use.

Assuming the basic configuration of Apache, Apache modules and web
application described at
`Web_App_Authentication <Web_App_Authentication>`__ or in
`Web_App_Authentication/Example_setup
example <Web_App_Authentication/Example_setup_example>`__ is correct,
this setup should allow the web application developer to obtain ticket
from ipa2.devel.company.net for testing user in the EXAMPLE.COM domain
and authenticate against the web application, and so should the PAM
authentication via mod_intercept_form_submit work.
