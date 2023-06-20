Squid_Integration_with_FreeIPA_using_Single_Sign_On
===================================================

**HOWTO: Configure Squid to authenticate IPA users using Kerberos with
Single Sign On.**

**Provided by Dale Macartney**

Before I start, I would like to give a great deal of thanks to Mallapadi
Niranjan at Red Hat for assisting me with the troubleshooting of this
setup. Without his help, I would not have been able to provide this
level of detail in this document.

The below details will walk you through how to add a Red Hat Enterprise
Linux 6.2 system to an IPA domain, and then configure Squid to allow
single sign on web caching.

Details of this example are as follows

#. Domain name: example.com
#. IPA Server: ds01.example.com
#. Squid Server: squid01.example.com
#. IPA Client: workstation01.example.com
#. IPA User: user1



Add system to IPA Domain (ensure DNS is working correctly otherwise this step will fail)
----------------------------------------------------------------------------------------

``# ipa-client-install -U -p admin -w mysecretpassword``



Install Squid and set service to start on boot
----------------------------------------------

| ``# yum install squid``
| ``# chkconfig squid on``



Edit /etc/squid/squid.conf to configure kerberos authentication
---------------------------------------------------------------

add the below lines to the top of /etc/squid/squid.conf

| ``auth_param negotiate program /usr/lib64/squid/squid_kerb_auth -d -s HTTP/squid01.example.com``
| ``auth_param negotiate children 10``
| ``auth_param negotiate keep_alive on``
| ``acl auth proxy_auth REQUIRED``

Change the below text:

| ``http_access allow localnet``
| ``http_access allow localhost``

And finally deny all other access to this proxy

``http_access deny all``

to look like this:

| ``http_access allow localnet``
| ``http_access allow localhost``
| ``http_access deny !auth``
| ``http_access allow auth``

And finally deny all other access to this proxy

``http_access deny all``



Generate a kerberos keytab for Squid HTTP access
------------------------------------------------

On the IPA server run:

| ``# kinit admin``
| ``Password for admin@EXAMPLE.COM:``
| ``# ipa service-add HTTP/squid01.example.com``

If successful, you will see the below output

| ``----------------------------------------------------``
| ``Added service "HTTP/squid01.example.com@EXAMPLE.COM"``
| ``----------------------------------------------------``
| ``  Principal: HTTP/squid01.example.com@EXAMPLE.COM``
| ``  Managed by: squid01.example.com``

On the Squid server run:

``# ipa-getkeytab -s ds01.example.com -p HTTP/squid01.example.com -k /etc/squid/krb5.keytab``

if successful, you will see the below output:

``Keytab successfully retrieved and stored in: /etc/squid/krb5.keytab``



Change the permissions of the keytab to allow Squid to read the file (Note, this should be kept secure, so only grant enough privileges as absolutely necessary.)
-----------------------------------------------------------------------------------------------------------------------------------------------------------------

| ``# chown root:squid /etc/squid/krb5.keytab``
| ``# chmod 640 /etc/squid/krb5.keytab``



Add keytab location to /etc/sysconfig/squid
-------------------------------------------

Add the below lines at the end of the file /etc/sysconfig/squid

| ``KRB5_KTNAME=/etc/squid/krb5.keytab``
| ``export KRB5_KTNAME``



Restart squid
-------------

| ``# service squid restart``
| ``Stopping squid: ................                           [  OK  ]``
| ``Starting squid: .                                          [  OK  ]``



Open Port 3128 in IPTables
--------------------------

| ``# iptables -I INPUT -p tcp --dport 3128 -j ACCEPT``
| ``# service iptables save``



Configure browser on IPA Client
-------------------------------

#. Log into a Desktop Environment on your IPA client with an IPA user
   account. (My tests involved using RHEL with Gnome Desktop.)
#. Launch Firefox, and open the Firefox preferences.
#. Select Advanced and click the Network tab
#. Click Settings
#. Select the "Manual proxy configuration" radio button
#. In the HTTP Proxy: field, enter squid01.example.com, and enter 3128
   in the Port field.
#. Check the tickbox that says "Use this proxy server for all protocols"
#. Click Ok, then click Close.



Verify your configuration
-------------------------

On the Squid server, tail your squid access logs

``# tail -f /var/log/squid/access.log``

On the IPA Client, browse to a website, (I went to www.redhat.com)

Watch the logs on your server appear as the web request is made.

If everything is working as expected, you will see messages similar to
the below.

| ``1328722977.370     31 192.168.122.91 TCP_MISS/200 34444 GET ``\ ```http://www.redhat.com/rhecm/rest-rhecm/jcr/repository/collaboration/jcr:system/jcr:versionStorage/5337fdf20a0526027ecb0b4331b2b334/2/jcr:frozenNode/rh:homepageBground`` <http://www.redhat.com/rhecm/rest-rhecm/jcr/repository/collaboration/jcr:system/jcr:versionStorage/5337fdf20a0526027ecb0b4331b2b334/2/jcr:frozenNode/rh:homepageBground>`__\ `` user1@EXAMPLE.COM DIRECT/2.19.215.214 image/png``
| ``1328722979.315      7 192.168.122.91 TCP_REFRESH_UNMODIFIED/304 546 GET ``\ ```http://www.redhat.com/rh-resources/skin/RedhatStyle/Redhat/images/ui/whitedot.png`` <http://www.redhat.com/rh-resources/skin/RedhatStyle/Redhat/images/ui/whitedot.png>`__\ `` user1@EXAMPLE.COM DIRECT/2.19.215.214 image/png``
| ``1328722984.326     18 192.168.122.91 TCP_MISS/200 34444 GET ``\ ```http://www.redhat.com/rhecm/rest-rhecm/jcr/repository/collaboration/jcr:system/jcr:versionStorage/5337fdf20a0526027ecb0b4331b2b334/2/jcr:frozenNode/rh:homepageBground`` <http://www.redhat.com/rhecm/rest-rhecm/jcr/repository/collaboration/jcr:system/jcr:versionStorage/5337fdf20a0526027ecb0b4331b2b334/2/jcr:frozenNode/rh:homepageBground>`__\ `` user1@EXAMPLE.COM DIRECT/2.19.215.214 image/png``

Note that the requests will be showing up in the logs as
user1@EXAMPLE.COM (my IPA test user).