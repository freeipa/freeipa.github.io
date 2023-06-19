In this text we will assume the identity provider used is IPA server and
we will look at the setup and modifications that might be needed in
typical web application to be able to use these central identities. We
will explore how Kerberos authentication can be added, how IPA
server-backed host-based access control (HBAC) can be configured for the
Kerberos authentication on per-service basis, how the user identity (and
its password) in the IPA server can be used for application's login-form
validation while preserving application's standard look and feel, and
how the application can obtain additional information about the
externally-authenticated user.

.. _example_application:

Example application
-------------------

Let us assume we have an existing web application which uses HTTP
cookie-based sessions to preserve authentication results across request.
An example can be found at
`fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/tree/?id=start <http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/tree/?id=start>`__.
It is plain CGI application written in Perl. Assuming you have the
Apache HTTP Server, perl, and the CGI module installed with command like

::

   yum install httpd perl-CGI -y

you can retrieve the example application for testing using curl:

::

   curl -Lo /var/www/app.cgi 'http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/plain/app.cgi?id=start'

Make sure it is executable as CGI script:

::

   chmod a+x /var/www/app.cgi
   yum install /usr/sbin/semanage -y
   semanage fcontext -a -t httpd_sys_script_exec_t '/var/www/app\.cgi'
   restorecon -rvv /var/www/app.cgi

Configure it to "live" in **/application** location (for example,
http://wikiapp.example.com/application) via ScriptAlias directive in
/etc/httpd/conf/httpd.conf or in some configuration file in
/etc/httpd/conf.d/*.conf:

::

   ScriptAlias /application /var/www/app.cgi

Then restart the Apache HTTP daemon:

::

   service httpd restart

The application when deployed at **wikiapp.example.com/application** and
accessed from browser will show a very simple page:

Application

This is a test application; public view, not much to see.

--------------

Log in

You can also use command line tools to test the application:

::

   curl -i http://wikiapp.example.com/application

or locally

::

   curl -i http://$( hostname )/application

The link **Log in** leads to **wikiapp.example.com/application/login**
which will display a login-form:

Log in to application

Login: [ . . . . . . . . . ]

Password: [ \* \* \* \* \* \* \* ]

< Log in >

--------------

Back to application

Login-form submission leads back to the page
**wikiapp.example.com/application/login** and upon success (we just test
that the password is the login name reversed) it will set HTTP cookie
**the-test-cookie** to contain the name of the user (the cookie value
when **bob** logs in in the example bellow will be **ok:bob**) and then
redirect to the main page:

Logged in as bob

You will be redirected to the home page

--------------

Back to application

Application authenticated (bob)

Test application; logged in as user bob. There is much more content for
authenticated users. There is much more content for authenticated users.
There is much more content for authenticated users. There is much more
content for authenticated users. There is much more content for
authenticated users. There is much more content for authenticated users.
There is much more content for authenticated users. There is much more
content for authenticated users. There is much more content for
authenticated users. There is much more content for authenticated users.

--------------

Log out

Real applications will probably do a lookup of the user and password in
the database and create session record in the database and store that
session's key in the cookie. Or at the least they would store the
identity of the user in the cookie together with some cryptographical
signature to prevent user to impersonate someone else. Remember: this is
just an example to show additional setups later.

Clicking **Log out** will just set the cookie value to **xx** meaning no
active user, and redirect back to **wikiapp.example.com/application**.

.. _enrolling_the_web_machine_to_ipa_server:

Enrolling the web machine to IPA server
---------------------------------------

We assume that the machine on which the wikiapp is hosted is enrolled to
an IPA server. The commands

::

   yum install /usr/sbin/ipa-client-install -y
   ipa-client-install

can be used to achieve the enrollment -- ``ipa-client-install`` will
configure sssd and other components of the system to know about the IPA
server. It will ask questions about the name of the IPA server and other
parameters, or you can explore its ``--help`` option and man page.

Kerberos
--------

The IPA server serves as a Kerberos Key Distribution Center, among
others. Users that have access to the Kerberos server for the
**example.com** domain can use

::

   kinit

to obtain their ticket-granting ticket which can then be used by
applications to obtain tickets to authenticate against other services.
Our goal will be to Kerberize **wikiapp.example.com/application**. In
typical setup, the tickets are in a realm matching the domain name,
usually written in upper case, in our case **EXAMPLE.COM**.

We will need Apache module **mod_auth_gssapi** or **mod_auth_kerb**
installed and configured. We will also need to obtain keytab from the
IPA server for our HTTP service. On the IPA server:

::

   kinit admin  # or other user with permissions to create new service for wikiapp.example.com
   ipa service-add HTTP/wikiapp.example.com

On our web application machine:

::

   kinit admin   # or other user with permissions to retrieve the keytab
   ipa-getkeytab -s $( awk '/^server/ { print $3 }' /etc/ipa/default.conf ) -k /etc/http.keytab -p HTTP/wikiapp.example.com
   chown apache /etc/http.keytab
   chmod 600 /etc/http.keytab
   yum install mod_auth_gssapi -y
   # or yum install mod_auth_kerb -y

We then configure **mod_auth_gssapi** or **mod_auth_kerb** to require
Negotiate authentication for **wikiapp.example.com/application/login**:
`wikiapp_kerb.conf for
mod_auth_gssapi <https://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/commit/?h=negotiate-mod_auth_gssapi>`__,
retrieve with curl using

::

   curl -Lo /etc/httpd/conf.d/wikiapp_kerb.conf 'http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/plain/auth_kerb.conf?id=negotiate-mod_auth_gssapi'

or `wikiapp_kerb.conf for
mod_auth_kerb <http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/commit/?id=negotiate>`__,
retrieve with curl using

::

   curl -Lo /etc/httpd/conf.d/wikiapp_kerb.conf 'http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/plain/auth_kerb.conf?id=negotiate'

.. _successful_negotiate:

Successful Negotiate
~~~~~~~~~~~~~~~~~~~~

We can restart the Apache now

::

   service httpd restart

and try to access the login page either from browser or via command line

::

   curl -i --negotiate -u : http://$( hostname )/application/login

When the user has valid Kerberos ticket in the EXAMPLE.COM realm and
clicks the **Log in** link leading to
**wikiapp.example.com/application/login**, mod_auth_gssapi/mod_auth_kerb
will return with status 401 and header ``WWW-Authenticate: Negotiate``.
The browser will try to obtain the ticket for
**HTTP/wikiapp.example.com@EXAMPLE.COM** and resubmit the request with
appropriate ``Authorization`` header. In the
**/var/log/httpd/access_log** will see that we have authenticated
correctly

::

   192.168.89.2 - - [08/Jan/2014:22:20:30 -0500] "GET /application/login HTTP/1.1" 401 127 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:26.0) Gecko/20100101 Firefox/26.0"
   192.168.89.2 - bob@EXAMPLE.COM [08/Jan/2014:22:20:32 -0500] "GET /application/login HTTP/1.1" 200 1980 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:26.0) Gecko/20100101 Firefox/26.0"

However, the application will still show the same login form, rather
than understanding that the user has already authenticated using
Kerberos. To achieve that, we need to change the application to
understand the **REMOTE_USER** environment variable which is set by
Apache authentication modules when their authentication attempt passes:
`trust
REMOTE_USER <http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/commit/?id=trust-REMOTE_USER>`__,
apply with curl using

::

   curl -L 'http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/patch/app.cgi?id=trust-REMOTE_USER' | patch -p1 /var/www/app.cgi

With the above changes in place, application will consult the
**REMOTE_USER** environment variable and it will skip any attempt of
internal validation of login and password and just consider the user
logged-in:

Logged in as bob@EXAMPLE.COM

You will be redirected to the home page

--------------

Back to application

Application authenticated (bob@EXAMPLE.COM)

Test application; logged in as user bob@EXAMPLE.COM. There is much more
content for authenticated users. There is much more content for
authenticated users. [...]

--------------

Log out

No login form will be shown.

.. _failed_negotiate:

Failed Negotiate
~~~~~~~~~~~~~~~~

Note the ErrorDocument client-side redirect to **/application/login2**
-- it is there as a fallback to the login form in case the user has no
valid ticket:

::

   kdestroy -A
   curl -i --negotiate -u : http://$( hostname )/application/login

or click using browser.

With the application now, the **wikiapp.example.com/application/login2**
will display

Application

This is a test application; public view, not much to see.

--------------

Log in

This is not right. Clearly, the application does not know that the
**login2** location is also supposed to display a login page.

For the fallback to work, we need to make sure
**wikiapp.example.com/application/login2** is location for the same
logon-form logic as **wikiapp.example.com/application/login**. In our
case, we just modify the application to consider any path starting with
**login** as login application: `support
login2 <http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/commit/?id=login2>`__,
apply with curl using

::

   curl -L 'http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/patch/app.cgi?id=login2' | patch -p1 /var/www/app.cgi

In real applications, this part can either go to the application code,
application/framework mapping, or to the Apache configuration.

With the change applied, if the browser cannot obtain the ticket, it
will just show the content of the document with we've configured with
the ErrorDocument directive to be a redirect to **/application/login2**.
After a short message

Kerberos authentication did not pass.

the login form will be displayed from
**wikiapp.example.com/application/login2** and the user can log in as
usual, with their login and password.

.. _additional_yum_repo:

Additional yum repo
-------------------

The following sections of this document describe software which is
already part of Fedora and RHEL/CentOS 6. The upstream repositories with
packages for popular OSes (namely RHEL 7) are at
http://copr.fedoraproject.org/coprs/adelton/identity_demo/ -- click on
the **adelton-identity_demo*.repo** link in the line matching your OS
and version and store the .repo file in **/etc/yum.repos.d**.

You can also retrieve the .repo file via curl: for example, for RHEL 7:

::

   curl -Lo /etc/yum.repos.d/identity_demo.repo 'http://copr.fedoraproject.org/coprs/adelton/identity_demo/repo/epel-7/adelton-identity_demo-epel-7.repo'

.. _host_and_service_based_access_control_for_kerberos:

Host (and service) based access control for Kerberos
----------------------------------------------------

The module
`mod_authnz_pam <http://www.adelton.com/apache/mod_authnz_pam/>`__ can
be used to run PAM access check for a particular service. Together with
sssd and IPA server, this allows fine-granular control over access to
various services.

We have the host wikiapp.example.com IPA-enrolled but in the default
setup, the IPA server has just one generic **allow_all** HBAC rule. You
need to `disable that rule and replace it with more granular
configuration, creating for example PAM service wikiapp for our
application <http://www.freeipa.org/page/Howto/HBAC_and_allow_all>`__.
You should see

::

   ipa hbactest --user=bob --host=wikiapp.example.com --service=wikiapp

not matching any rule and after adding the host to the **allow_wikiapp**
HBAC rule, see it match:

::

   ipa hbacrule-add-host allow_wikiapp --hosts=wikiapp.example.com
   ipa hbacrule-add-user allow-wikiapp --user=bob
   ipa hbactest --user=bob --host=wikiapp.example.com --service=wikiapp

Configure PAM service wikiapp. Create **/etc/pam.d/wikiapp** with the
following content:

::

   auth    required   pam_sss.so
   account required   pam_sss.so

Note that the **wikiapp** HBAC service name needs to match the PAM
service name but it's just a string, it does not need to match the
hostname. We could have used **wiki** or **test** instead.

Install the mod_authnz_pam

::

   yum install mod_authnz_pam -y

Our current wikiapp_kerb.conf needs to be amended to load mod_authnz_pam
and ``require pam-account wikiapp``: for mod_auth_gssapi
`mod_authnz_pam-pam-account-mod_auth_gssapi <http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/commit/?id=mod_authnz_pam-pam-account-mod_auth_gssapi>`__,
apply with curl using

::

   curl -L 'http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/patch/auth_kerb.conf?id=mod_authnz_pam-pam-account-mod_auth_gssapi' | patch -p1 /etc/httpd/conf.d/wikiapp_kerb.conf

and for mod_auth_kerb
`mod_authnz_pam-pam-account <http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/commit/?id=mod_authnz_pam-pam-account>`__,
apply with curl using

::

   curl -L 'http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/patch/auth_kerb.conf?id=mod_authnz_pam-pam-account' | patch -p1 /etc/httpd/conf.d/wikiapp_kerb.conf

Enable Apache to use the PAM stack and restart it:

::

   setsebool -P allow_httpd_mod_auth_pam 1
   service httpd restart

After restarting Apache, we can check that the access works and if we
remove either the machine or the user from the HBAC rule

::

   ipa hbacrule-remove-host allow_wikiapp --hosts=wikiapp.example.com
   ipa hbacrule-remove-user allow_wikiapp --users=bob

or perhaps indirectly from host/user-group, the Kerberos authentication
will still fail and logon form will be shown.

.. _access_control_with_user_groups_using_pam_access:

Access control with user groups using pam_access
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The host-based access control (HBAC) in IPA can control access of users
to services running on various hosts. The HBAC rules can use user groups
in IPA. Sometimes, the admin might not want to manage the access
centrally and might prefer to locally set the list of groups that can
access the web application. Eventually, sssd will provide way to set up
access control on per-PAM service basis. For now, with mod_authnz_pam,
we have all the PAM modules at our disposal, including pam_access. For
example, adding line

::

   account required   pam_access.so accessfile=/etc/http-access.conf

to **/etc/pam.d/wikiapp** will enable the access control using file
**/etc/http-access.conf**. If the content of that file is

::

   + : (wiki-group-test) : ALL
   - : ALL : ALL

only users in the wiki-group-test group will be granted access. Both
local groups from **/etc/group** and the IPA-managed groups are
considered for this access control check.

.. _external_identities_for_login_form:

External identities for login form
----------------------------------

With module
`mod_intercept_form_submit <http://www.adelton.com/apache/mod_intercept_form_submit/>`__,
the same PAM service **wikiapp** that we used to run access check for
the Kerberos authentication can be used to silently try authentication
against the IPA server (via PAM and sssd) whenever the user submits the
login form. The module needs to be installed

::

   yum install mod_intercept_form_submit -y

and configured:
`intercept-form-submit <http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/commit/?id=intercept-form-submit>`__,
apply with curl using

::

   curl -Lo /etc/httpd/conf.d/wikiapp_form_submit.conf 'http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/plain/intercept_form_submit.conf?id=intercept-form-submit'
   service httpd restart

With **/etc/pam.d/wikiapp** in place and **mod_intercept_form_submit**,
the application will see **REMOTE_USER** populated whenever the
authentication via the PAM stack succeeds.

If it fails, the application will still have a chance to run local
authentication.

Test with ``ipa hbacrule-remove-user`` and ``ipa hbacrule-add-user``
that the authentication using the **mod_intercept_form_submit**,
observes the access control just like **mod_authnz_pam** does for
Kerberos. In fact, **mod_intercept_form_submit** is calling
**mod_authnz_pam** internally.

Note that we configure the module on **/application/login2** because
with Kerberos on **/application/login**, that is where the for
submission will run. If we omitted the Kerberos configuration, we would
want **mod_intercept_form_submit** configured on **/application/login**.

.. _storing_external_users_in_internal_databases:

Storing external users in internal databases
--------------------------------------------

Our example CGI script does not use any database and it simulates the
users by accepting any reasonable login name and matching password. For
externally authenticated users, it accepts whatever value is set in
REMOTE_USER.

Real application would have users stored in the database and even for
externally authenticated users, it will probably want to store these
external users in its database, albeit with some "external" flag, for
referential integrity to work. So in reality, the change of application
code to process REMOTE_USER would probably create the user in the
database first and then create session for this new user.

.. _additional_attributes:

Additional attributes
~~~~~~~~~~~~~~~~~~~~~

Applications expect not just the login name of a user to be present --
they might need their email address to send them notifications, they
might want to know their full name just to make the user interface less
cryptic. When the user is created and stored by the application, the
application has full control over what fields (attributes) it will
require to be present -- without it the user record will not be allowed.

However, when the user authentication happens against external identity
provider, asking user for their email address and name that they already
have correctly filled in the central server might not be ideal. It would
not only slow user's work down, it could also lead to inconsistencies,
and in some enterprises, only dedicated departments can modify the
personnel information in the central identity store.

With module
`mod_lookup_identity <http://www.adelton.com/apache/mod_lookup_identity/>`__
and sssd-dbus package, sssd can retrieve additional attributes from the
IPA server and make them available to the to the module and thus to the
application during authentication using Apache module.

We start with configuring sssd: install sssd-dbus

::

   yum install sssd-dbus -y

and enable and configure its **ifp** subsystem:

::

   --- /etc/sssd/sssd.conf.orig    2013-12-10 03:09:20.751552952 -0500
   +++ /etc/sssd/sssd.conf    2013-12-12 00:52:30.791240631 -0500
   @@ -11,8 +11,10 @@
    chpass_provider = ipa
    ipa_server = _srv_, ipa.example.com
    dns_discovery_domain = example.com
   +ldap_user_extra_attrs = mail, givenname, sn
   +
    [sssd]
   -services = nss, pam, ssh
   +services = nss, pam, ssh, ifp
    config_file_version = 2

    domains = example.com
   @@ -28,3 +30,7 @@

    [pac]

   +[ifp]
   +allowed_uids = apache, root
   +user_attributes = +mail, +givenname, +sn
   +

Restart sssd and attempt to retrieve some information using
**dbus-send**:

::

   service sssd restart
   dbus-send --print-reply --system --dest=org.freedesktop.sssd.infopipe /org/freedesktop/sssd/infopipe org.freedesktop.sssd.infopipe.GetUserAttr string:bob array:string:gecos,mail
   dbus-send --print-reply --system --dest=org.freedesktop.sssd.infopipe /org/freedesktop/sssd/infopipe org.freedesktop.sssd.infopipe.GetUserGroups string:bob

You may need to set ``setenforce 0`` for the above part to work. For
dbus calls from httpd which we will do below,

::

   setenforce 1
   setsebool -P httpd_dbus_sssd on

should work provided you have recent enough selinux-policy.

If we are able to retrieve information using dbus, we can proceed to
install and configure mod_lookup_identity:

::

   yum install mod_lookup_identity -y

and configure it: `additional-attributes,
lookup_identity.conf <http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/tree/lookup_identity.conf?id=additional-attributes>`__,
use curl to retrieve:

::

   curl -Lo /etc/httpd/conf.d/wikiapp_lookup.conf 'http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/plain/lookup_identity.conf?id=additional-attributes'

When you then restart Apache

::

   service httpd restart

and connect in authenticated manner to it, you will see variables
populated by the module:

::

   kinit bob
   curl -i --negotiate -u : http://$( hostname )/application/login | grep REMOTE_USER

The grep should list something like

::

   REMOTE_USER=bob@EXAMPLE.COM
   REMOTE_USER_EMAIL=bob@example.com
   REMOTE_USER_FIRSTNAME=Robert
   REMOTE_USER_GECOS=Robert Puk
   REMOTE_USER_LASTNAME=Puk

-- the web page actually contains (commented out in HTML) list of all
environment variables that were passed to our application.

The last needed step is to teach the application to use the additional
attributes: `additional-attributes, application
change <http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/diff/app.cgi?id=additional-attributes>`__,
apply with curl using

::

   curl -L 'http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/patch/app.cgi?id=additional-attributes' | patch -p1 /var/www/app.cgi

In our simple application, we just store the values in the cookie to
show later.

Application authenticated (Robert Puk (bob: bob@example.com))

Test application; logged in as user Robert Puk (bob@example.com). There
is much more content for authenticated users. There is much more content
for authenticated users. There is much more content for authenticated
users. There is much more content for authenticated users. There is much
more content for authenticated users. There is much more content for
authenticated users. There is much more content for authenticated users.
There is much more content for authenticated users. There is much more
content for authenticated users. There is much more content for
authenticated users.

--------------

Log out

Please note that the additional attributes are retrieved by Apache
modules and it does not matter if the user has authenticated using
Kerberos (mod_auth_gssapi or mod_auth_kerb) or via
mod_intercept_form_submit -- the mod_lookup_identity module takes the
authentication result and uses it to get the values.

Real applications can use these attributes from central identity
provider to have the same set of information as their internal users,
typically storing the attributes in their internal user databases when
populating the record for the externally authenticated users. Typical
list of attributes that the applications might be interested in is
proposed at
http://www.freeipa.org/page/Environment_Variables#Proposed_Additional_Variables.

.. _application_level_roles:

Application-level roles
~~~~~~~~~~~~~~~~~~~~~~~

Web applications make it very easy to handle and add new users but most
applications will sooner or later need to start distinguishing different
roles for different users. As with user identities themselves,
applications will typically store them in a database, and as with
external authentication, even application-level roles may need to be
partially managed.

In IPA server, the easiest way to assign quality to a user is via
groups. The mod_lookup_identity module makes it easy to retrieve group
membership into REMOTE_USER_GROUPS or similar environment variable. The
group membership can then be expanded into roles that the applications
uses. Admin of the application will need to set the initial mapping of
groups to roles but after then, upon each login, roles can be updated
from the central identity provider.

That means, that if a company hires new sysadmin and they set their
group membership in sysadmin's group correctly in the central IPA
server, system management tools across the company can retrieve this
group memebership on the fly and make the new employee no only able to
log in to the system management tools but also assign correct roles and
permissions to them.

.. _passing_information_to_applications:

Passing information to applications
-----------------------------------

In this example application, we have used CGI script and environment
variables that are populated by Apache modules and then inherited by the
CGI script when it is forked and run.

In real deployments, CGI scripts are rarely used these days, primarily
for performance reasons. Typically, the applications are either loaded
in the context of the Apache server, or they run in daemon fashion in
parallel to Apache and control is handed over to them over sockets or
via other means.

Many of these frameworks will pass environment variables behind the
scenes and applications can consult them just like they would in the CGI
scenario.

In some cases, only the REMOTE_USER information (or the internal value
``r->user`` of the Apache request) is passed by the framework and
additional attributes need to be handled separately. For example, when
mod_proxy_ajp is used to hand over the request from Apache to tomcat,
only REMOTE_USER and then environment variables that start with AJP\_
are passed. So the environment variables populated by
mod_lookup_identity need to be prefixed with AJP_. In Apache
configuration:

::

     LookupUserAttr mail AJP_REMOTE_USER_EMAIL " " 

In application code:

::

          String email = String(((String) request.getAttribute("REMOTE_USER_EMAIL")).getBytes("ISO8859-1"), "UTF-8"); 

In case of deployments that use mod_proxy_balancer where no out-of-band
information passing is available, headers of the HTTP request can be
used. Of course, caution is needed to properly clear any headers of the
same name in the incoming HTTP request to prevent the end user from
breaching the authentication/access control:

::

     RequestHeader unset X-THE-USER
     RequestHeader set X-THE-USER %{REMOTE_USER}e env=REMOTE_USER
     RequestHeader unset X-THE-USER-EMAIL
     RequestHeader set X-THE-USER-EMAIL %{REMOTE_USER_EMAIL}e env=REMOTE_USER_EMAIL

On the application end, in case of CGI script, the incoming HTTP request
headers are presented as environment variables prefixed with HTTP_, with
dashes turned into underscores, so the application code would need to be
along the lines of

::

          if (defined $ENV{HTTP_X_THE_USER}) {
                  $login = $ENV{HTTP_X_THE_USER};
          [...]

See
http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/commit/?id=proxy-setup
for a frontend proxy configuration and application patch to read the
HTTP headers.

Of course, the application or its framework might also have other means
to get values of the HTTP headers of the request more directly.

.. _web_framework_configurations:

Web framework configurations
----------------------------

For tomcat, the Connectors need to be set with
``tomcatAuthentication="false"`` to accept the REMOTE_USER information
from Apache:

::

   <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" URIEncoding="UTF-8" address="127.0.0.1" tomcatAuthentication="false" />

Conclusion
----------

On a simple example CGI application, we have shown how relatively small
changes to the application code can make use of the REMOTE_USER
environment variable and additional variables with extended attributes,
making it possible to use central identity provider like IPA server for
Kerberos and login/password authentication. We have also shown the
configuration of mod_auth_gssapi/mod_auth_kerb and of new modules,
mod_authnz_pam, mod_intercept_form_submit, and mod_lookup_identity can
can together form flexible solution to meet the needs of web
applications deployed in environments with central user management.

In the section below, links to changes that went to real-life projects
to make some of these changes possible are shown.
