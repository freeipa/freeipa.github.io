**NexentaStor integration with IPA in a mixed environment**

--------------

Provided by: Sigbjorn Lie

Making NexentaStor speak to AD and LDAP/IPA is easy, adding krb5 for NFS
is a bit more tricky.

.. _first_part_cifs_nfs_no_kerberos:

First part: CIFS + NFS no-kerberos
----------------------------------

Configure the CIFS service to join to AD, and configure the LDAP client
to point at IPA. Use the following configuration for LDAP,

REMEBER to edit /etc/nsswitch.ldap before applying LDAP configuration.
Failing to do so will freeze NexentaStor as all services will be
configured to use LDAP, and maps such as protocols and service is not
served by IPA. You'll do this in a shell using expert_mode in the NMC.
The only maps in /etc/nsswitch.ldap that should be configured for ldap
lookup is passwd, group, and netgroup.

Make the following changes under Settings -> Misc Services -> LDAP
client -> Configure.

::

   LDAP config type: manual
   Profile name: <none>
   Groups Service Descriptor: cn=groups,cn=compat,dc=ix,dc=test,dc=com
   Netgroup Service Descriptor: cn=ng,cn=compat,dc=ix,dc=test,dc=com
   Credential Level: anonymous
   Domain name: <none>
   Base DN: dc=ix,dc=test,dc=com
   LDAP Authentication password: <none>
   LDAP Servers: ipa01.ix.test.com, ipa02.ix.test.com
   Authentication Method: none
   Proxy DN: <none>
   Proxy Password: <none>
   Users Service Descriptor: cn=users,cn=compat,dc=ix,dc=test,dc=com

In a shell using expert_mode, edit the nsswitch.conf as the following:

::

   passwd:   files ldap ad
   group:    files ldap ad

This will make Nexenta look for Unix accounts and ground in IPA first,
before looking up the rest from Active Directory.

.. _second_part_adding_kerberos_to_nfs4_nfs_krb5:

Second part, adding kerberos to NFS4: NFS + KRB5
------------------------------------------------

-  After the server has been joined to AD, scp the /etc/krb5/krb5.keytab
   file from the NexentaStor server to the IPA server.
-  Add a host entry for the NexentaStor server to IPA, and retrieve the
   kerberos keytab and add them to the krb5.keytab file copied from the
   NexentaStor machine. This is required as the NFS service and the CIFS
   service share the same krb5.keytab file.

``# ipa-getkeytab -s ipa-server -p nexentastorserver.fqdn -k /path/to/nexentastor/krb5.keytab``

-  scp the modified krb5.keytab file back into the NexentaStor server at
   /etc/krb5/krb5.keytab.

-  Edit /etc/nsswitch.conf again, make sure "ad" is still present for
   passwd and group, if not, add it back in.

-  Remove /etc/krb5/krb5.conf,v (bug in NexentaStor makes this file
   re-appear with old contents, even if it's edited trough the NMC.

-  Edit /etc/krb5/krb5.conf, add a sections under [realms] for your IPA
   domain. I've specified admin_server, kdc, and kpasswd_server for all
   my IPA servers.
-  Add "allow_weak_crypto = true" under libdefaults to widen the support
   for Linux clients.
-  Set "default_realm = IPA-REALM-CAPITAL-LETTERS"
-  Add a section for the IPA domain under [domain_realm]:

::

   .ipa-domain.com = IPA-REALM-CAPITAL-LETTERS
   ipa-domain.com = IPA-REALM-CAPITAL-LETTERS

-  Edit /etc/defaultdomain, create the file if it does not exist
   already, and add the IPA domain.

-  Edit /etc/resolv.conf:

::

   search addomain.com ipadomain.com
   domain addomain.com
   nameserver <ipa-dns-ip>
   nameserver <ad-dns-ip>

I have also configured my IPA DNS server to forward any requests for my
AD domain directly to the AD dns servers. This should not be required if
your domains is delegated properly, but it speeds AD requests up a bit.
:)

The addomain must be the first domain listed to make the nss_ad module
work.

-  Switch back to the NMC, and edit the nfs defaults file:

``# NMC: $ setup network service nfs-server edit-settings``

-  Uncomment and modify:

::

   NFSMAPID_DOMAIN=ipa-domain

-  Restart the NFS service.

``# NMC $ setup network service nfs-server restart``

That's it. Your NexentaStor server will now look up LDAP/IPA users and
groups first, and then generate UID/GID's for any other users/groups
only found in AD.

`Category:How to <Category:How_to>`__ `Category:Draft
documentation <Category:Draft_documentation>`__
