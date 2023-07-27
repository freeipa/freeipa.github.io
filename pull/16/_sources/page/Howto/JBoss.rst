JBoss
=====



Using IPA services for JBoss EAP/AS
===================================



The versions used
-----------------

In the following description, we will assume JBoss EAP 6.1.0 Final is
used on either Red Hat Enterprise Linux 6 (RHEL) or Fedora 19+. Fedora
includes GSS-Proxy which can provide some additional functionality. The
JBoss AS 7.\* could also be used even if we've experienced some issues
with it (https://bugzilla.redhat.com/show_bug.cgi?id=1014028). We will
assume JBoss is run from the .zip extracted in some directory. If
rpm-based installation is used, some paths might be different.



IPA-enrolled client
-------------------

To use services from IPA server, the machine needs to be IPA-enrolled.
To do that, package ``ipa-client`` on RHEL and ``freeipa-client`` on
Fedora and its dependencies need to be installed:

``# yum install -y ipa-client``

Then the IPA-related systems will be configured using:

``# ipa-client-install``

In the following examples, we will assume that the realm used is
EXAMPLE.NET, the IPA server is ipa.example.com and the JBoss server runs
on www.example.com.



Starting standalone JBoss
-------------------------

The standalone JBoss server can be started using script provided in the
.zip distribution:

``$ ./bin/standalone.sh &``

Then we can access the web service which by default listens on port
8080:

``$ curl ``\ ```http://localhost:8080/`` <http://localhost:8080/>`__

Since many IPA services rely on correct fully qualified domain name of
the web server, we will make JBoss listen not just on the loopback
interface:

::

   diff --git a/standalone/configuration/standalone.xml b/standalone/configuration/standalone.xml
   --- a/standalone/configuration/standalone.xml
   +++ b/standalone/configuration/standalone.xml
   @@ -279,7 +279,7 @@
                <inet-address value="${jboss.bind.address.management:127.0.0.1}"/>
            </interface>
            <interface name="public">
   -            <inet-address value="${jboss.bind.address:127.0.0.1}"/>
   +            <inet-address value="${jboss.bind.address:0.0.0.0}"/>
            </interface>
            <!-- TODO - only show this if the jacorb subsystem is added  -->
            <interface name="unsecure">

After restarting the ``./bin/standalone.sh``, access on public interface
should work:

``$ curl ``\ ```http://$(hostname):8080/`` <http://$(hostname):8080/>`__

We can stop the JBoss server now.



Service principal
-----------------

To be able to configure Kerberos authentication against our JBoss
server, we need a service principal and a keytab. On the IPA server,
command

``$ ipa service-add HTTP/www.example.com``

will create the service and we can then retrieve the keytab on the JBoss
server:

| ``$ kinit admin``
| ``$ ipa-getkeytab -s ipa.example.com -p HTTP/www.example.com -k /path/to/keytab``



Configure JBoss to use SPNEGO authentication
--------------------------------------------

The examples below are taken from https://github.com/kwart/spnego-demo.

In the ``standalone/configuration/standalone.xml`` configuration file,
we will define the SPNEGO security domain:

::

   diff --git a/standalone/configuration/standalone.xml b/standalone/configuration/standalone.xml
   --- a/standalone/configuration/standalone.xml
   +++ b/standalone/configuration/standalone.xml
   @@ -242,6 +242,32 @@
                            <policy-module code="Delegating" flag="required"/>
                        </authorization>
                    </security-domain>
   +                <security-domain name="host" cache-type="default">
   +                    <authentication>
   +                        <login-module code="Kerberos" flag="required">
   +                            <module-option name="debug" value="true"/>
   +                            <module-option name="storeKey" value="true"/>
   +                            <module-option name="refreshKrb5Config" value="true"/>
   +                            <module-option name="useKeyTab" value="true"/>
   +                            <module-option name="doNotPrompt" value="true"/>
   +                            <module-option name="keyTab" value="/path/to/keytab"/>
   +                            <module-option name="principal" value="HTTP/${jboss.qualified.host.name}@EXAMPLE.NET"/>
   +                        </login-module>
   +                    </authentication>
   +                </security-domain>
   +                <security-domain name="SPNEGO" cache-type="default">
   +                    <authentication>
   +                        <login-module code="SPNEGO" flag="required">
   +                            <module-option name="serverSecurityDomain" value="host"/>
   +                            <module-option name="password-stacking" value="useFirstPass"/>
   +                        </login-module>
   +                    </authentication>
   +                    <mapping>
   +                        <mapping-module code="SimpleRoles" type="role">
   +                            <module-option name="admin@EXAMPLE.NET" value="admin"/>
   +                        </mapping-module>
   +                    </mapping>
   +                </security-domain>
                </security-domains>
            </subsystem>

Change the */path/to/keytab* to the real path of your
HTTP/www.example.com's keytab. It has to be accessible by the uid under
which you run the JBoss server.

Please note that for this test, we hardcode the admin@EXAMPLE.NET
principal (username) here to assign the admin role that is then used for
authentication. Adjust this value to match the user you will be able to
kinit to.

We then create application in **standalone/deployments/kerberos.war**:

``standalone/deployments/kerberos.war/index.html``:

::

   OK

``standalone/deployments/kerberos.war/WEB-INF/jboss-web.xml``:

::

   <jboss-web>
       <security-domain>SPNEGO</security-domain>
       <valve>
           <class-name>org.jboss.security.negotiation.NegotiationAuthenticator</class-name>
       </valve>
   </jboss-web>

``standalone/deployments/kerberos.war/WEB-INF/web.xml``:

::

   <?xml version="1.0" encoding="UTF-8"?>
   <web-app version="2.4" xmlns="http://java.sun.com/xml/ns/j2ee"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">

       <security-constraint>
           <web-resource-collection>
               <web-resource-name>Protect User data</web-resource-name>
               <url-pattern>/*</url-pattern>
           </web-resource-collection>
           <auth-constraint>
               <role-name>*</role-name>
           </auth-constraint>
       </security-constraint>

       <security-role>
           <role-name>admin</role-name>
       </security-role>
   </web-app>

``standalone/deployments/kerberos.war/META-INF/jboss-deployment-structure.xml``:

::

   <jboss-deployment-structure>
       <deployment>
           <dependencies>
               <module name="org.jboss.security.negotiation" />
           </dependencies>
       </deployment>
   </jboss-deployment-structure>

We then touch file ``standalone/deployments/kerberos.war`` and start the
server, we should see message like

| ``Register web context: /kerberos``
| ``Deployed "kerberos.war" (runtime-name : "kerberos.war")``

We then obtain a ticket:

``$ kinit admin@EXAMPLE.NET``

and we will run curl with Negotiate authentication enabled:

| ``$ curl --negotiate -u : ``\ ```http://$(hostname):8080/kerberos/`` <http://$(hostname):8080/kerberos/>`__
| ``OK``

We should see the ``OK`` (the content of the index.html file) printed.