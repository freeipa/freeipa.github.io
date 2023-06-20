NetApp_integration_in_a_mixed_environment
=========================================

**NetApp integration with IPA in a mixed environment**

--------------

Provided by: Sigbjorn Lie



Configure the DNS client
------------------------

Configure the NetApp's DNS client to point to a set of DNS servers that
knows both your AD and your IPA DNS domain. Configure the DNS search
path to point at both the IPA domain, and the AD domain (if you have a
different DNS domain for your IPA and AD instances)



Configure the CIFS server
-------------------------

Join the CIFS server to the AD domain:

``# cifs setup``

Follow the prompts.



Configure the LDAP client
-------------------------

Setup the LDAP client To list:

``# options ldap``

To configure each value:

``# options ldap.option value``

I use authenticated simple binds. I have created an account for the
NetApp filers under cn=sysaccounts,cn=etc,$BASE for this purpose. The
LDAP attribute mapping options can be left as standard. You need to
specify the compat tree for group, and netgroup lookups. Users need to
be pointed to the accounts tree.

Specify each user/group/ng lookup path fully (e.g. I do NOT specify the
base DN and request subtree for lookups).

Configure the "options ldap.enabled" after setting up the account used
for the LDAP bind and after configuring all the other options. Leave
"ldap.ADdomain" blank.

This is an example of my configuration:

::

   ldap.ADdomain                           
   ldap.base                    dc=example,dc=com 
   ldap.base.group              cn=groups,cn=compat,dc=example,dc=com 
   ldap.base.netgroup           cn=ng,cn=compat,dc=example,dc=com 
   ldap.base.passwd             cn=users,cn=accounts,dc=example,dc=com 
   ldap.enable                  on         
   ldap.minimum_bind_level      simple     
   ldap.name                    uid=netapp,cn=sysaccounts,cn=etc,dc=example,dc=com 
   ldap.nssmap.attribute.gecos  gecos      
   ldap.nssmap.attribute.gidNumber gidNumber  
   ldap.nssmap.attribute.groupname cn         
   ldap.nssmap.attribute.homeDirectory homeDirectory 
   ldap.nssmap.attribute.loginShell loginShell 
   ldap.nssmap.attribute.memberNisNetgroup memberNisNetgroup 
   ldap.nssmap.attribute.memberUid memberUid  
   ldap.nssmap.attribute.netgroupname cn         
   ldap.nssmap.attribute.nisNetgroupTriple nisNetgroupTriple 
   ldap.nssmap.attribute.uid    uid        
   ldap.nssmap.attribute.uidNumber uidNumber  
   ldap.nssmap.attribute.userPassword userPassword 
   ldap.nssmap.objectClass.nisNetgroup nisNetgroup 
   ldap.nssmap.objectClass.posixAccount posixAccount 
   ldap.nssmap.objectClass.posixGroup posixGroup 
   ldap.passwd                  ******     
   ldap.port                    389        
   ldap.servers                 ipa01.example.com ipa02.example.com ipa03.example.com 
   ldap.servers.preferred       ipa01.example.com ipa02.example.com 
   ldap.ssl.enable              off        
   ldap.timeout                 20         
   ldap.usermap.attribute.unixaccount uid        
   ldap.usermap.attribute.windowsaccount ntUserDomainId 
   ldap.usermap.base            cn=users,cn=accounts,dc=example,dc=com 
   ldap.usermap.enable          on     

*NOTE: I have been unable to get the LDAP SSL client of NetApp to work
with IPA as of yet. I have opened a support case with NetApp for this
issue. Not really a big issue as users password are not being
transmitted. To make of of SSL NetApp's documentation is to upload the
CA certificate in PEM format into /etc on the filer and use the keymgr
command to import it. After uploading the CA cert SSL is enabled using
"options ldap.ssl.enable on".*



Testing the LDAP client
-----------------------

Grant yourself advanced privileges on the filer "priv set advanced", and
use the "getXXbyYY" command to verify that the LDAP naming services
works as expected for users, groups and netgroups.



Configure nsswitch.conf
-----------------------

If the previous test was successful: Configure the NetApp's
nsswitch.conf (using the filer webui is the easiest, you can also edit
the file using NFS: filer:/vol/rootvol/etc/nsswitch.conf or by using
CIFS). Specify "files" before "ldap" for passwd, group, and netgroup.
Configure hosts to use "files" before "dns".

You should now have a working AD (CIFS) and IPA (NFS) setup.



AD - IPA username mapping
-------------------------

If you syncronize IPA with AD, the ntUserDomainId attribute in IPA will
be set to AD's sAMAccountName. You may also use a script to sync this
attribute manually. Having the ntUserDomainId attribute available will
allow for automatic user mapping in the NetApp filer when Windows CIFS
users connect. The username may be the same, but the NetApp's user
mapping has been seen to be case sensitive in our environment. Syncing
the sAMAccountName from AD into IPA's ntUserDomainId attribute fixed
these issue for us.

You have to enable the "ldap.usermap.enable" option on the NetApp filer
to enable username mapping lookup from LDAP.

`Category:How to <Category:How_to>`__ `Category:Draft
documentation <Category:Draft_documentation>`__