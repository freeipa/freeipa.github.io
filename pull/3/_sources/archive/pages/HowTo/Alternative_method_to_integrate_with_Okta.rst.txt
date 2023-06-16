.. _goals_of_this_methodology:

Goals of this methodology
-------------------------

Okta support for OpenLDAP very reduced, it just allows you to delegate
logins. When login succeeds a new user is imported int Okta. LDAP
support doesn't permit you to provision users in LDAP,
activate/deactivate a user, push a password, and so on.

Following this Howto you will be able to:

-  provision users in FreeIPA from Okta
-  push passwords from Okta
-  disabling accounts from Okta
-  Desktop SSO using Kerberos and Ipsilon using as Desktop clients:
   Linux, MAC OSX or Windows.

Bear in mind that as a bridge between Okta an FreeIPA **you will need a
Windows Server** that will be used just for provisioning/deprovisioning
purposes

.. _provisioning_integration_todo:

Provisioning integration (TODO)
-------------------------------

#. Install Okta Agent on Windows Server(s) you will need to create an OU
   for Okta created users
#. Integrate FreeIPA and Active Directory(or directories) as a winsync
   replica(s) specifying the OU created before
#. Install Passsync on Windows Server(s) to push passwords from Active
   Directory to FreeIPA

.. _desktop_sso_todo:

Desktop SSO (TODO)
------------------

#. Install ipsilon on a server, it can be the same as any of your
   freeipa replicas or another server
#. Enable Inbound SAML on Okta: Security -> Authentication -> Inbound
   SAML
#. Configure your desktops to initiate session using Kerberos
   credentials
#. Configure your browsers to pass kerberos tokens
#. Add a link on Okta signing page to initiate Kerberos Auth
