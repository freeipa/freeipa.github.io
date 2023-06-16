.. _draft_version___work_in_progress_to_be_completed:

\**\* DRAFT VERSION - WORK IN PROGRESS TO BE COMPLETED \**\*
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Following on from the previous documentation for an IPA backend to
protect Apache it's not unusual to require only a subset of users to
have access to a system rather than all valid users.

There's two approaches that will be taken in this document - the first
will be PAM based authentication and the second will be extended the
previous kerberos configuration with LDAP groups. There are pros and
cons to each direction and each are suitable for different scenarios
dependant on the use case.

Using PAM authentication

With the EPEL repository enabled installing the mod_auth_pam package
will install teh module, add a configuration file to httpd to enable the
module and place an httpd file in /etc/pam.d for the PAM configuration.

The default is to use whatever is in /etc/pam.d/password-auth but this
may be overkill. To just use SSSD all that is needed is:

| `` auth        required      pam_env.so``
| `` auth        sufficient    pam_sss.so ``
| `` auth        required      pam_deny.so``
| `` account     sufficient    pam_sss.so``
| `` account     required      pam_deny.so``

To allow httpd to talk to PAM with selinux enabled a boolean needs to be
set:

`` setsebool -P allow_httpd_mod_auth_pam on``

With that in place it's simple enough to add user/group behaviour via
adding the appropriate entries for the directory or location concerned
in the conf files or .htaccess:

| `` AuthPAM_Enabled on``
| `` AuthType Basic``
| `` AuthName "Group Restricted  Server"``
| `` require valid-user``

Pros

This will work with the HBAC options/pages in IPA (add httpd as a
service) so allow for testing of rules. A user won't be considered
'valid' unless it can pass the HBAC tests. This way controls can be
centralized with a standardized httpd config or standard users have very
simple copy/paste to go into .htaccess over multiple servers very
easily. This also does not require any sort of ldap service user to bind
as (and consequently any plain text passwords anywhere) since it is the
system carrying out the check via the secure kerberos/ldap mechanism
through SSSD. Optionally rather than require valid-user require group
can be used - with the downside this wouldn't be visible in the IPA UI
and HBAC would still need to be completed so it could confuse in
troubleshooting.

Cons

Depending on httpd activity this has the potential for being fairly
heavy on load - principally due to HBAC having a fairly short cache
time. This may or may not be a problem depending on the use case of the
httpd server. This only offers basic auth so no single sign on would be
available - if that's a desirable feature.

Using LDAP groups alongside Kerberos Authentication

`With Kerberos already working <Apache_SNI_With_Kerberos>`__ we can add
LDAP groups via mod_authnz_ldap which is included in the base httpd
install. The important lines to look at for basic lockdown via groups
are:

| `` AuthLDAPURL   ``
| `` AuthLDAPBindDN    ``
| `` AuthLDAPBindPassword  ``
| `` require ldap-group``

The bind user and password are only required if anonymous lookups are
disabled (which is preferred in general).

First a dedicated system user should be created to use to bind with. To
do this create a file (eg httpbind.ldif) containing the following:

| `` dn: uid=httpbind,cn=sysaccounts,cn=etc,dc=example,dc=com``
| `` changetype: add``
| `` objectclass: account``
| `` objectclass: simplesecurityobject``
| `` uid: httpbind``
| `` userPassword: ohaimakethissimethingtoughtobreak``
| `` passwordExpirationTime: 20380119031407Z``
| `` nsIdleTimeout: 0``

This can then be imported by:

`` ldapmodify -h ipa01.example.com -p 389 -x -D "cn=Directory  Manager" -w ``\ `` -f httpbind.ldif``

Now the configuration can be carried out. Full details can be found
`here <http://httpd.apache.org/docs/2.2/mod/mod_authnz_ldap.html>`__ but
in brief...

| `` AuthLDAPURL   ``\ ```ldaps://ipa01.example.com:636/dc=example,dc=com`` <ldaps://ipa01.example.com:636/dc=example,dc=com>`__
| `` AuthLDAPBindDN    uid=httpbind,cn=sysaccounts,cn=etc,dc=example,dc=com``
| `` AuthLDAPBindPassword  ohaimakethissimethingtoughtobreak``
| `` require ldap-group``
| `` LDAPTrustedGlobalCert CA_BASE64 /etc/httpd/certs/ca.crt``
| `` AuthLDAPGroupAttributeIsDN off``
| `` ``

Make sure that /etc/ipa/ca.crt has been copied to a location that httpd
can read (such as /etc/httpd/certs/ca.crt if following the previous
guide).

`Category:ImproveOrRemove <Category:ImproveOrRemove>`__
`Category:NoLink <Category:NoLink>`__
