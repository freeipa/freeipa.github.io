NIS_accounts_migration_preserving_Passwords
===========================================

**HOWTO: Migrate NIS accounts to IPA preserving user's passwords.**

*Provided by Giulio Fidente*

The below details will walk you through how to migrate some existing NIS
accounts over IPA preserving the NIS passwords.

Details of this example are as follows:

| ``Domain name: example.com``
| ``IPA Server: ipa1.example.com``

To have a user auth successfully on IPA, you need to get the proper
Kerberos hashes in LDAP, but you can't just paste the NIS passwords as
Kerberos keys; SSSD can work instead with FreeIPA to solve this and
mitigate the user impact on migration by generating the required user
keys.



Enable IPA migration mode
-------------------------

This step is needed to have FreeIPA regenerate the hashes and store them
in the user entry when users successfully bind via LDAPs.

``# ipa config-mod --enable-migration=true``



Export the NIS accounts details
-------------------------------

Consider a single line of your YP passwd map:

``gfidente:v4sL5vFQjQWQ2:4009:20:,,,:/home/gfidente:/bin/bash``

the first field is a username and the second field is a DES encrypted
password. You may want to script the export saving the id/pwd couples in
a temporary file.



Import users into IPA
---------------------

From your export file, import the users into IPA using the admin tools
and set the original hashed password:

``# ipa user-add [username] --setattr userpassword={crypt}yourencryptedpass``

You should be able now to bind on the LDAP using your migrated NIS
credentials. FreeIPA will intercepts the bind request and if the user
also has a Kerberos principal but no Kerberos hashes, then it will
rewrite create the principal key.

Obviously you don't know the passwords of your users so you can't bind
manually on LDAP for each of them, but SSSD can do this for you, if
configured properly, when users log in for the first time.

**NOTE**: Use {crypt} for any password taken from an NIS dump or
/etc/shadow. It defines the password storage scheme that crypt(3) uses.



Add systems to the IPA Domain
-----------------------------

First, ensure DNS is working correctly otherwise this step will fail.

Configure manually an IPA client os use the ipa-client package and run
the install utility. You can even run that on a system where the NIS
client tools are already set in place, it won't break things.

``# ipa-client-install -U -p admin -w mysecretpassword``

Double check your nsswitch.conf file, it should have added the 'sss'
keyword to a number of lines:

``# passwd: files nis sss``



Deploy the server CA cert on the client systems
-----------------------------------------------

The IPA CA cert is provided by the IPA httpd server and you just need to
download it in /etc/openldap/cacerts:

| ``# wget ``\ ```http://ipa1.example.com/ipa/config/ca.crt`` <http://ipa1.example.com/ipa/config/ca.crt>`__\ `` -O /etc/openldap/cacerts/ipa.ca``
| ``# cacertdir_rehash /etc/openldap/cacerts``

When users will try to log on the client, SSSD will try to acquire a
kerberos ticket but if that doesn't work, it will retry then with a bind
over LDAPs; if this works the IPA server will rewrite the kerberos
principal key for you and the user will actually be logged in with his
old password.

Troubleshooting
---------------

If users still can't login, you can enable the SSSD debug mode editing
/etc/sssd/sssd.conf and adding the following in your
[domain/example.com] stanza:

| ``debug_level = 6``
| ``ldap_tls_reqcert = never (or allow)``

You will get the details in /var/log/sssd/sssd_example.com.log