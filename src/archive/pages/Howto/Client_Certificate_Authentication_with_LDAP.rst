We can use `user certificates <V4/User_Certificates>`__ to authenticate
our ldap session.

.. _generate_user_certificate_for_user_account:

generate user certificate for user account
------------------------------------------

Follow instructions `in this
blog <https://blog-ftweedal.rhcloud.com/2015/08/user-certificates-and-custom-profiles-with-freeipa-4-2/>`__.

Short version:

-  create csr (certificate signing request).

I usually create a new directory and name it after the name of the
user/host we want to create a certificate for. For user10, create a
user10 folder.

Inside this folder, create a text file user10.inf like this:

::

   [ req ]
   prompt = no
   encrypt_key = no

   distinguished_name = dn
   req_extensions = exts

   [ dn ]
   commonName = "user10"

   [ exts ]
   subjectAltName=email:user10@yourdomain.tld

-  generate a key:

::

   openssl genrsa -out user10.key 2048

-  generate the csr:

::

   openssl req -new -key user10.key -out user10.csr -config user10.inf

-  verify csr:

::

    
   openssl req -in user10.csr -text -noout 
   Certificate Request:
       Data:
           Version: 0 (0x0)
           Subject: CN=user10
           Subject Public Key Info:
               Public Key Algorithm: rsaEncryption
                   Public-Key: (2048 bit)
                   Modulus:
                       00:c2:d2:0c:44:c8:e3:8b:d7:e5:bc:b6:5d:fc:cf:
                       xxxxx
                   Exponent: 65537 (0x10001)
           Attributes:
           Requested Extensions:
               X509v3 Subject Alternative Name: 
                   email:user10@yourdomain.tld
       Signature Algorithm: sha1WithRSAEncryption
            05:7b:a7:51:1e:28:25:8d:78:fb:d9:08:43:6d:54:51:db:10:
            xxxxxxxxxxxxxxxxxxxxx

-  request the certificate (as the user self or as an admin user):

::

   $ ipa cert-request user10.csr --principal user10 
   ....

If everything goes according to plan, you know have a certificate
coupled to the user account

::

   $ ipa user-show user10
     User login: user10
     First name: ipa
     Last name: user
     Home directory: /home/user10
     Login shell: /bin/sh
     Email address: user10@yourdomain.tld
     UID: 1076200013
     GID: 1076200013
     Certificate: MIIEMjCCAxqgAwIBAgIBDjANBgkqhkiG9w0BAQsFADA5MRcwFQYDVQQKDA5VTklYxxxxxxxxxxxxxxxxxxxxxxxxxxxx==
     Account disabled: False
     Password: True
     Member of groups: ipausers
     Kerberos keys available: True

-  retrieve the certificate:

first we need to get the certificate's serial number.

::

   ipa cert-find
   ...
     Serial number (hex): 0xE
     Serial number: 14
     Status: VALID
     Subject: CN=user10,O=YOURDOMAIN.TLD
   <pre>
   So, number 14.

   <pre> 
   ipa cert-show 14 --out user10.pem 

-  eventually, verify certificate:

::

   openssl x509 -in user10.pem -noout -text

which will give you all the certificate output on screen.

.. _map_certificate_to_user_account:

map certificate to user account
-------------------------------

Canonical info:

http://directory.fedoraproject.org/docs/389ds/howto/howto-certmapping.html

https://access.redhat.com/documentation/en-US/Red_Hat_Directory_Server/9.0/html/Administration_Guide/Managing_SSL-Using_Certificate_Based_Authentication.html

-  verify /etc/dirsrv/slapd-INSTANCE-NAME/certmap.conf looks like this:

::

   certmap default         default
   #default:DNComps
   #default:FilterComps    e, uid
   #default:verifycert     on
   #default:CmapLdapAttr   certSubjectDN
   #default:library        <path_to_shared_lib_or_dll>
   #default:InitFn         <Init function's name>
   default:DNComps
   default:FilterComps     uid
   certmap ipaca           CN=Certificate Authority,O=SUB.DOMAIN.TLD
   ipaca:CmapLdapAttr      seeAlso
   ipaca:verifycert        on

As you see, there is a 'default' mapping and an 'ipaca' mapping.

WARNING!!!

Do not modify anything of the ipaca mapping unless you know what you are
doing. You risk messing up your pki tomcat service and plenty of things
will stop working.

WARNING!!!

As you see, the ipaca mapping is your ipa server PKI. It has a
CmapLdapAttr mapping attribute to the ldap object attribute seeAlso.

When I searched a test ipa environment, the only account with a seeAlso
attribute was the "DN: uid=pkidbuser,ou=people,o=ipaca" user, with this
value: "CN=CA Subsystem,O=SUB.DOMAIN.TLD" (substitute O=SUB.DOMAIN.TLD
with your own REALM name, obviously). This is an internal ipa user, do
not modify it! We cannot modify this mapping or the PKI subsystem will
stop working.

So the solution is quite simple. We need to populate the seeAlso
attribute of the user10 account with this value:

::

   cn=user10,o=SUB.DOMAIN.TLD

You can add this value to the seeAlso attribute using your favourite
ldap client, like the very nice `apache ds
studio <https://directory.apache.org/studio/>`__

.. _configure_ldap_client:

configure ldap client
---------------------

we can easily test this using ldapsearch. We need to set two environment
variables in ~/.ldaprc:

::

   TLS_CERT /path/to/user10.pem
   TLS_KEY /path/to/user10.key

And now search:

::

   $ ldapsearch -h kdc.domain.tld -ZZ -Y EXTERNAL objectclass=person -s sub -b dc=sub,dc=domain,dc=tld cn
   ...
   # search result
   search: 3
   result: 0 Success

   # numResponses: 1002
   # numEntries: 1001

And in the log files (/var/log/dirsrv/slapd-INSTANCE-NAME/access) of the
ldap server we see this:

::

   [04/Mar/2016:23:34:57 +0100] conn=100 fd=111 slot=111 connection from 192.168.0.124 to 192.168.5.10
   [04/Mar/2016:23:34:57 +0100] conn=100 op=0 EXT oid="1.3.6.1.4.1.1466.20037" name="startTLS"
   [04/Mar/2016:23:34:57 +0100] conn=100 op=0 RESULT err=0 tag=120 nentries=0 etime=0
   [04/Mar/2016:23:34:57 +0100] conn=100 TLS1.2 256-bit AES; client CN=user10,O=SUB.DOMAIN.TLD issuer CN=Certificate Authority,O=SUB.DOMAIN.TLD
   [04/Mar/2016:23:34:57 +0100] conn=100 TLS1.2 client bound as uid=user10,cn=users,cn=accounts,dc=sub,dc=domain,dc=tld
   [04/Mar/2016:23:34:57 +0100] conn=100 op=1 BIND dn="" method=sasl version=3 mech=EXTERNAL
   [04/Mar/2016:23:34:57 +0100] conn=100 op=1 RESULT err=0 tag=97 nentries=0 etime=0 dn="uid=user10,cn=users,cn=accounts,dc=sub,dc=domain,dc=tld"
   [04/Mar/2016:23:34:57 +0100] conn=100 op=2 SRCH base="dc=sub,dc=domain,dc=tld" scope=2 filter="(objectClass=person)" attrs="cn"
   [04/Mar/2016:23:34:57 +0100] conn=100 op=2 RESULT err=0 tag=101 nentries=1001 etime=0

.. _perl5_example:

perl5 example
-------------

you need the perl-LDAP and perl-IO-Socket-SSL packages for this
(fedora/rhel/centos).

::

   #!/usr/bin/env perl

   use strict;
   use warnings;
   use utf8;
   use Net::LDAP;

   my $base = "dc=sub,dc=domain,dc=tld";

   my $ldap = Net::LDAP->new( 'kdc.sub.domain.tld', debug => 0 ) || die "$@";

   my $msg = $ldap->start_tls(
       verify     => 'require',
       sslversion => 'tlsv1',
       clientcert => "/path/to/user10.pem",
       clientkey  => "/path/to/user10.key",
   );

   $msg->code && warn "could not starttls: " . $msg->error;

   # no bind needed, we are already authenticated!

   my $search = $ldap->search(
       base   => $base,
       scope  => "sub",
       filter => "(objectclass=person)",
       attrs => [ 'uid', ],
   );

   $search->code && warn "failed to get persons: " . $search->error;

   print "found " . $search->count . " persons\n";

   for my $entry ( $search->entries ) {
       print $entry->get_value('uid'), "\n";
       print $entry->dn, "\n";
   }

.. _disabling_access_to_the_user_certificate:

disabling access to the user certificate
----------------------------------------

if this certificate (or its key) has been compromised you need to
disable its access to the directory.

-  revoke it:

::

   ipa cert-revoke <serialnr>

-  remove the seeAlso attribute from the user account.

This is necessary because the DS apparently does not check the
revocation status of the certificate. Having revoked it, I can still use
it to access the ldap server. Removing the ldap value of seeAlso solves
this problem.

Author
------

Howto provided by Natxo Asenjo on
`freeipa-users <https://www.redhat.com/archives/freeipa-users/2016-March/msg00036.html>`__.

`Category:IPA <Category:IPA>`__
