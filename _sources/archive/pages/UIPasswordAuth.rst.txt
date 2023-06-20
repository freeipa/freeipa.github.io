UIPasswordAuth
==============

By default, the IPA Web UI uses Kerberos Negotiate to perform a single
sign-on login. This is handled automatically in Firefox if it is
properly configured and you have a TGT.

There are some cases where this is not possible, such as an unsupported
browser or operating system (Windows for example).

One alternative is to have the Web UI fall back to a Basic
Authentication-type dialog box prompting for username/password. To do
this edit ``/etc/httpd/conf.d/ipa.conf``

Set the value of ``KrbMethodK5Passwd`` to ``on`` and restart the httpd
service (``/sbin/service httpd restart``)

The web server will first attempt to use Kerberos Negotiate to log the
user in. If that fails then the user will be presented with a login
prompt.