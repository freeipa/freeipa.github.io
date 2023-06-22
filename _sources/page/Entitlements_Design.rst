Entitlements_Design
===================

Introduction
------------

IPA v2 will include support for counting the number of enrolled clients
and comparing that with a number of *entitlements* and logging it is out
of compliance.

The entitlement server is the `Candlepin
server <https://fedorahosted.org/candlepin/wiki>`__.

An enrolled client is defined as a machine with a keytab.

IPA will provide 25 entitlements with the base server install.

Entitlements
------------

Entitlements take the form of an X.509 v3 certificate. Certificate
extensions are used to store bits of information about the entitlement
such as the product being entitled, the quantity, etc. The validity of
the certificate represents the validity of the entitlement. When the
certificate expires, the entitlement expires.

Bootstrapping
-------------

When one obtains an entitlement a username and password is provided.
This username and password is sent to the Cancelpin server and registers
as a consumer (of entitlements). The result of this registration is an
SSL client certificate and private key. All subsequent operations
against the Candlepin server are done using this cert and key. This cert
and key will be packaged as a PKCS#12 object and stored in LDAP so that
all IPA servers will have access to it. The password to decrypt it is
currently TBD but I'm leaning towards using the Apache NSS database
password as we have easy access to it.



Entitlement CLI Operations
--------------------------

The ipa command-line will have an entitlements plugin. This plugin will
only be enabled if a setting is True in /etc/ipa/default.conf. By
default this will be False, hiding the plugin.

We will support the following operations:

-  Obtain new entitlement for *n* clients
-  Return an existing entitlement back to the Candlepin server
-  Count of current entitlements
-  List all entitlements including date, quantity and product
-  Import an entitlement Certificate. This loads a raw entitlement
   certificate and is only allowed if we are not enrolled to a Candlepin
   server.



Cron Operations
---------------

A cronjob will pull the entitlements out of IPA and Candlepin and first
compare that we have the right entitlements stored in IPA. Next it will
compare the quantity to the number of enrolled clients (meaning those
entries in cn=computers,cn=accounts that have a krbprincipalkey). We
will bind using the host keytab, meaning that all replicas need to be
able to write the entitlements.



Counting Hosts
--------------

Since the number of entitlements is linked to the number of hosts users
will want to be able to find those hosts that aren't being used. A
simple LDAP query can be used to determine this:

| ``$ ldapsearch -x -b 'cn=computers,cn=accounts,dc=example,dc=com' \``
| ``"(&(krbLastSuccessfulAuth < 201004110000Z)(krbPrincipalName=*))" fqdn``

This looks for all hosts that have a principal and haven't logged in
since April 11, 2010.