OpenShift_Broker_Apache_+_mod_auth_kerb_for_IdM
===============================================

**DRAFT**: Apache + ``mod_auth_kerb`` configuration on Broker for IPA
and OpenShift

Notes:

-  The service, ``HTTP/$BROKER_FQDN`` is enrolled in IPA
   (``ipa service-add`` on IPA Server or Broker machine that has
   ipa-admintools)
-  An IPA-issued keytab is on the same machine as Apache
   (``ipa-getkeytab`` on Broker machine)
-  Apache owns the keytab
-  ``mod_auth_kerb`` is installed (simply
   ``yum install -y mod_auth_kerb`` if not found in
   ``/etc/httpd/modules/``)
-  There are two apache configurations, one "Broker" and one "Console" -
   they share the same keytab



Apache configuration:

**Broker**: In
``/var/www/openshift/broker/httpd/conf.d/openshift-origin-remote-user.conf``:

::

   LoadModule auth_basic_module modules/mod_auth_basic.so
   LoadModule authz_user_module modules/mod_authz_user.so
   LoadModule auth_kerb_module modules/mod_auth_kerb.so

   &lt;Location /broker&gt;
       AuthName "OpenShift"
       AuthType Kerberos
       KrbMethodNegotiate On
       KrbMethodK5Passwd On
       KrbServiceName HTTP/$BROKER_FQDN
       KrbAuthRealms $KERBEROS_REALM
       Krb5KeyTab /path/to/http.keytab
       Require valid-user

       # The node-&gt;broker auth is handled in the Ruby code
       BrowserMatch Openshift passthrough
       Allow from env=passthrough

       Order Deny,Allow
       Deny from all
       Satisfy any
   &lt;/Location&gt;

   # The following APIs do not require auth:
   &lt;Location /broker/rest/application_templates*&gt;
       Allow from all
   &lt;/Location&gt;

   &lt;Location /broker/rest/cartridges*&gt;
       Allow from all
   &lt;/Location&gt;

   &lt;Location /broker/rest/api*&gt;
       Allow from all
   &lt;/Location&gt;

**Console**: In
``/var/www/openshift/console/httpd/conf.d/openshift-origin-remote-user.conf``:

::

   LoadModule auth_basic_module modules/mod_auth_basic.so
   LoadModule authz_user_module modules/mod_authz_user.so
   LoadModule auth_kerb_module modules/mod_auth_kerb.so

   # Turn the authenticated remote-user into an Apache environment variable for the console security controller
   RewriteEngine On
   RewriteCond %{LA-U:REMOTE_USER} (.+)
   RewriteRule . - [E=RU:%1]
   RequestHeader set X-Remote-User "%{RU}e" env=RU

   &lt;Location /console&gt;
       AuthName "OpenShift Developer Console"
       AuthType Kerberos
       KrbMethodNegotiate On
       KrbMethodK5Passwd On
       KrbServiceName HTTP/$BROKER_FQDN
       KrbAuthRealms $KERBEROS_REALM
       Krb5KeyTab /path/to/http.keytab
       require valid-user

       # The node-&gt;broker auth is handled in the Ruby code
       BrowserMatch OpenShift passthrough
       Allow from env=passthrough

       Order Deny,Allow
       Deny from all
       Satisfy any
   &lt;/Location&gt;

Restart services:

::

   $ service openshift-broker restart
   $ service openshift-console restart



Test

::

   $ kinit USER
   # for the Broker - note trailing "/"
   $ curl -Ik --negotiate -u : https://$BROKER_FQDN/broker/rest/api/
   # for the console
   $ curl -Ik --negotiate -u : https://$BROKER_FQDN/console

As well, test the console link in a browser that is configured for
SPNego/Negotiate protocol.

If there are issues, add the ``-v`` flag with curl, and check logs at:

-  Broker:

   -  Fedora: ``/var/log/openshift/broker/httpd/`` both ``error_log``
      and ``access_log`` can be helpful
   -  RHEL: ``/var/www/openshift/broker/logs/httpd/`` both ``error_log``
      and ``access_log`` can be helpful

-  Console:

   -  Fedora: ``/var/log/openshift/console/httpd/`` both ``error_log``
      and ``access_log`` can be helpful
   -  RHEL: ``/var/www/openshift/console/logs/httpd/`` both
      ``error_log`` and ``access_log`` can be helpful

-  IPA Server: ``/var/log/krb5kdc.log``