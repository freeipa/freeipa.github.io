NSS_Impersonation
=================



Using mod_nss's NSSVerifyClient require + LookupUserByCertificate + GssapiImpersonate
=====================================================================================

TL;DR: mod_nss's NSSVerifyClient require + LookupUserByCertificate On +
GssapiImpersonate On work for generic Apache setup but it is fragile and
updates are likely needed to mod_lookup_identity and mod_nss.

This was an email summary, that's why it's written in an Ich form.

See also `setup of this feature for FreeIPA
WebUI <V4/External_Authentication/Setup>`__.

--------------



Setup and test steps
--------------------

I have used RHEL 7.2, IPA-enrolled to RHEL 7.2 IPA. I've installed

-  mod_nss-1.0.11-6.el7.x86_64
-  mod_lookup_identity-0.9.3-1.el7.x86_64
-  sssd-dbus-1.13.0-40.el7.x86_64

and built mod_auth_gssapi from master
(3e0f4e980b4bea2f4f347fc39ea3deddf95fe71e) with make -j && make install.

I've configured SSSD:

::

   --- /etc/sssd/sssd.conf.orig    2016-06-16 04:55:01.056705019 -0400
   +++ /etc/sssd/sssd.conf 2016-06-16 07:26:58.078165651 -0400
   @@ -12,7 +12,7 @@
    ipa_server = _srv_, ipa.example.com
    dns_discovery_domain = example.com
    [sssd]
   -services = nss, sudo, pam, ssh
   +services = nss, sudo, pam, ssh, ifp
    config_file_version = 2

    domains = example.com

and run

| ``       setsebool -P httpd_dbus_sssd on``
| ``       systemctl restart sssd``

I have created HTTP/ service for the machine and got its keytab:

| ``       echo Secret123 | kinit admin``
| ``       ipa service-add HTTP/$(hostname)``
| ``       IPA_SERVER=$(awk '/^server =/ {print $3}' /etc/ipa/default.conf)``
| ``       ipa-getkeytab -s $IPA_SERVER -k /etc/http.keytab -p HTTP/$(hostname)``
| ``       chown apache /etc/http.keytab``
| ``       chmod 600 /etc/http.keytab``

On the IPA server, I've run

| ``       [root@ipa ~]# kadmin.local``
| ``       Authenticating as principal admin/admin@EXAMPLE.COM with password.``
| ``       kadmin.local:  modprinc +ok_to_auth_as_delegate HTTP/client.example.com``
| ``       Principal "HTTP/client.example.com@EXAMPLE.COM" modified.``
| ``       kadmin.local:  quit``

I have created directory for the delegated credentials:

| ``       mkdir /var/run/httpd/gssapi_deleg``
| ``       chown apache /var/run/httpd/gssapi_deleg``

I have created user and associated client certificate (the test
certificate alpha from mod_nss's default /etc/httpd/alias) with it:

| ``       ipa user-add --first Robert --last Chase --random bob``
| ``       ipa user-add-cert --certificate="$( certutil -L -d /etc/httpd/alias -a -n alpha | grep -v '.---' )" bob``

I had to change bob's password or I was getting CLIENT KEY EXPIRED later
during gss_acquire_cred_impersonate_name:

``       kpasswd bob``

I have created test content:

::

   # cat > /var/www/cgi-bin/set.cgi <<EOF
   #!/bin/bash
   echo Content-Type: text/plain
   echo Pragma: no-cache
   echo
   set
   EOF

   chmod a+x /var/www/cgi-bin/set.cgi

Started Apache and tested I can talk to it via HTTPS:

``       SSL_DIR=/etc/httpd/alias curl -v -Lsi ``\ ```https://$(hostname):8443/cgi-bin/set.cgi`` <https://$(hostname):8443/cgi-bin/set.cgi>`__

I've configured the modules:

::

   --- /etc/httpd/conf.d/nss.conf.orig     2015-09-22 16:51:00.000000000 -0400
   +++ /etc/httpd/conf.d/nss.conf  2016-06-16 07:37:00.591724042 -0400
   @@ -70,11 +70,13 @@
    #
    # Only renegotiate if the peer's hello bears the TLS renegotiation_info
    # extension. Default off.
   -NSSRenegotiation off
   +# NSSRenegotiation off
   +NSSRenegotiation on

    # Peer must send Signaling Cipher Suite Value (SCSV) or
    # Renegotiation Info (RI) extension in ALL handshakes.  Default: off
   -NSSRequireSafeNegotiation off
   +# NSSRequireSafeNegotiation off
   +NSSRequireSafeNegotiation on

    ##
    ## SSL Virtual Host Context
   @@ -146,6 +148,16 @@
    #   Client certificate verification type.  Types are none, optional and
    #   require.
    #NSSVerifyClient none
   +<Location /cgi-bin/set.cgi>
   +NSSVerifyClient require
   +NSSUserName SSL_CLIENT_CERT
   +LookupUserByCertificate On
   +
   +GssapiImpersonate On
   +GssapiDelegCcacheDir /var/run/httpd/gssapi_deleg
   +GssapiCredStore keytab:/etc/http.keytab
   +GssapiCredStore client_keytab:/etc/http.keytab
   +</Location>

    #
    #   Online Certificate Status Protocol (OCSP).

uncommented LoadModule in

``       /etc/httpd/conf.modules.d/55-lookup_identity.conf``

and run

| ``       echo LoadModule auth_gssapi_module modules/mod_auth_gssapi.so > /etc/httpd/conf.modules.d/09-gssapi.conf``
| ``       systemctl restart httpd``

I've now run

``       SSL_DIR=/etc/httpd/alias curl -Lsi --cert alpha ``\ ```https://$(hostname):8443/cgi-bin/set.cgi`` <https://$(hostname):8443/cgi-bin/set.cgi>`__

and in the log I saw

::

   ==> /var/log/httpd/error_log <==
   [Thu Jun 16 08:22:28.070370 2016] [:notice] [pid 18961] lookup_user_by_certificate found [bob]

   ==> /var/log/httpd/access_log <==
   2620:52:0:1322:221:5eff:fe20:2f4e - -----BEGIN CERTIFICATE-----\nMIICeDCCAeGgAwIBAgIBAjANBgkqhkiG9w0BAQsFADA/MQswCQYDVQQGEwJVUzEU\nMBIGA1UEChMLZXhhbXBsZS5jb20xGjAYBgNVBAMTE
   UNlcnRpZmljYXRlIFNoYWNr\nMB4XDTE2MDYxNjA4NTM0MVoXDTIwMDYxNjA4NTM0MVowgaAxCzAJBgNVBAYTAlVT\nMRQwEgYDVQQKEwtleGFtcGxlLmNvbTEPMA0GA1UECxMGUGVvcGxlMRUwEwYKCZIm\niZPyLGQBARMFYWxwaGExFDA
   SBgNVBAMTC0ZyYW5rIEFscGhhMT0wOwYJKoZIhvcN\nAQkBFi5hbHBoYUBxZS1ibGFkZS0xMC5pZG1xZS5sYWIuZW5nLmJvcy5yZWRoYXQu\nY29tMIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC7XmqZ98Ohbom0YHr8yr5M\nvMeuE
   ju+uVmv2vNEjAzrK3bdKcvqVHcx9sGJz376X6PrJxOthFiItxKpEWxshadK\nDwxrz0JPiDyZQW5FPYIuFx/vH8hnPE5LetTw7rf1ukUU4CpfnonLuH7LBwGmpUIl\neRV4ATUb0GYIF/P8gdtOZwIDAQABoyIwIDARBglghkgBhvhCAQEEB
   AMCB4AwCwYD\nVR0PBAQDAgWgMA0GCSqGSIb3DQEBCwUAA4GBAGVMJU24Cjz9CPBmiW61l4B+ufI7\nLvyxCQirRq4rkus0fmkYFHd3+zB40dUcnM/o1Vv5dV3uCxPOjiZz72Ur/bVG3Igw\nI02zZc86+jV9mO5FSfu10myoUNExnsR3uKY
   WZUW/5rl4GRMtFa8Gruk4cFa0+DJx\nL/dRR/x2uOqDY0Rb\n-----END CERTIFICATE-----\n [16/Jun/2016:08:22:28 -0400] "GET /cgi-bin/set.cgi HTTP/1.1" 200 4196

and /var/run/httpd/gssapi_deleg/bob@EXAMPLE.COM got created.

Issues
------

mod_lookup_identity
----------------------------------------------------------------------------------------------

The correct functionality depends on the order in which mod_nss and
mod_lookup_identity are loaded. By default, on RHEL 7.2, mod_nss uses

``       /etc/httpd/conf.modules.d/10-nss.conf``

and mod_lookup_identity uses

``       /etc/httpd/conf.modules.d/55-lookup_identity.conf``

(55-lookup_identity.conf has the LoadModule commented out), so that
order works. But it would be good to add mod_nss to

``       ``\ ```https://github.com/adelton/mod_lookup_identity/blob/master/mod_lookup_identity.c#L749`` <https://github.com/adelton/mod_lookup_identity/blob/master/mod_lookup_identity.c#L749>`__

to force mod_lookup_identity to be run after mod_nss.

New release mod_lookup_identity-0.9.7 was done to address this issue.

mod_nss
----------------------------------------------------------------------------------------------

Second issue is the fact that as shown by the access_log above, the
r->user and REMOTE_USER are set back to the SSL_CLIENT_CERT value in the
fixup phase, even if we've set it to bob and mod_auth_gssapi found bob
there. It's because the r->user is set both at

``       ``\ ```https://git.fedorahosted.org/cgit/mod_nss.git/tree/nss_engine_kernel.c#n627`` <https://git.fedorahosted.org/cgit/mod_nss.git/tree/nss_engine_kernel.c#n627>`__

where we find it, but also in

``       ``\ ```https://git.fedorahosted.org/cgit/mod_nss.git/tree/nss_engine_kernel.c#n962`` <https://git.fedorahosted.org/cgit/mod_nss.git/tree/nss_engine_kernel.c#n962>`__

That second operation should likely be only run when

``       (dc->nOptions & SSL_OPT_FAKEBASICAUTH)``

When I patch mod_nss that way, the curl will show

::

   ==> /var/log/httpd/error_log <==
   [Thu Jun 16 08:40:41.175368 2016] [:notice] [pid 22993] lookup_user_by_certificate found [bob]

   ==> /var/log/httpd/access_log <==
   2620:52:0:1322:221:5eff:fe20:2f4e - bob@EXAMPLE.TEST [16/Jun/2016:08:40:41 -0400] "GET /cgi-bin/set.cgi HTTP/1.1" 200 3310

which likely is exactly what we want.

I have filed https://bugzilla.redhat.com/show_bug.cgi?id=1347298 for
this.