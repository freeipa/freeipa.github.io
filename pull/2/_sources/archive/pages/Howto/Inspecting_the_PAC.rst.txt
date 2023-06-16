.. _inspecting_the_pac:

Inspecting the PAC
------------------

The `PAC <http://msdn.microsoft.com/en-us/library/cc237917.aspx>`__ is a
part of a `Kerberos <Kerberos>`__ ticket, the so called `Authorization
Data <https://tools.ietf.org/html/rfc4120#section-5.2.6>`__. For more
details see:

-  RFC 4120: The Kerberos Network Authentication Service (V5)
   ` <https://tools.ietf.org/html/rfc4120>`__\ https://tools.ietf.org/html/rfc4120
-  Micosoft Technical Document [MS-PAC]: Privilege Attribute Certificate
   Data Structure
   ` <http://msdn.microsoft.com/en-us/library/cc237917.aspx>`__\ http://msdn.microsoft.com/en-us/library/cc237917.aspx
-  Microsoft Technical Document [MS-KILE]: Kerberos Protocol Extensions
   ` <http://msdn.microsoft.com/en-us/library/cc233855.aspx>`__\ http://msdn.microsoft.com/en-us/library/cc233855.aspx

In general the PAC is processed by applications like
`Samba <http://www.samba.org>`__ or
`SSSD <https://fedorahosted.org/sssd/>`__ to retrieve information about
the authenticated user and e.g. assign access rights or group
memberships. Since the PAC is buried NDR encoded inside the encrypted
part of the Kerberos ticket it is not easy for a user to access it and
inspect the content.

Luckily there is a largely unknown tool in the Samba treasure chest
called **net ads kerberos pac**. It was originally developed with only
the Samba use case in mind because at that time there was only Samba
using the PAC.

.. _ad_client:

AD client
~~~~~~~~~

By default **net ads kerberos pac** tries to get a Kerberos service
ticket for the local host (HOSTNAME$@AD.DOMAIN) for an AD user given by
*-U username*. The keys needed to encrypt the PAC are the Kerberos host
keys which means that the user calling **net ads kerberos pac**. must
have access to the host keytab. Typically this is only the *root* user.

If the Linux client is joined to the AD domain with Samba/Winbind you
can call:

::

   # net ads kerberos pac dump -U tu1
   Enter tu1's password:
   The Pac:     info: struct PAC_LOGON_INFO
           info3: struct netr_SamInfo3
               base: struct netr_SamBaseInfo
   .....

If SSSD is used to connect to the AD domain and no Samba configuration
file *smb.conf* is available you have to give two addition options:

::

   # net ads kerberos pac dump --option='realm = AD.DOMAIN' --option='kerberos method = system keytab' -U tu1
   Enter tu1's password:
   The Pac:     info: struct PAC_LOGON_INFO
           info3: struct netr_SamInfo3
               base: struct netr_SamBaseInfo

The restriction to the hardcoded service name (HOSTNAME$@AD.DOMAIN) made
it hard to use this tool in other environments. But recently Samba Team
member Günther Deschner enhanced the tool and opened it for other use
cases.

.. _freeipa_client:

FreeIPA client
~~~~~~~~~~~~~~

A client enrolled to a FreeIPA server has a host keytab as well but the
AD-style HOSTNAME$@KRB.REALM host principals are not available. With the
recent enhancements to **net ads kerberos pac** an alternative service
principal can be given which makes it possible to use it with FreeIPA as
well.

::

   Usage:
   net ads kerberos pac dump [impersonate=string] [local_service=string] [pac_buffer_type=int]
       Dump the Kerberos PAC

You can call

::

   net ads kerberos pac dump --option='realm = IPA.DOMAIN' --option='kerberos method = system keytab' -s /dev/null local_service=host/ipa-client.ipa.domain@IPA.DOMAIN -U ipa_user
   Enter ipa_user's password:
   The Pac:     pac_data_ctr->pac_data: struct PAC_DATA
           num_buffers              : 0x00000004 (4)
           version                  : 0x00000000 (0)
           buffers: ARRAY(4)
               buffers: struct PAC_BUFFER
                   type                     : PAC_TYPE_LOGON_INFO (1)
                   _ndr_size                : 0x000001c8 (456)
                   info                     : *
                       info                     : union PAC_INFO(case 1)
   ...

This is basically the same scheme as for the SSSD AD client example
above with the exception of the *-s /dev/null* option. You can drop this
option if there is no Samba configuration on your system, e.g. on a
typical FreeIPA client. However, on FreeIPA server with
`Trusts <Trusts>`__ configured *ipa-adtrust-install* creates a Samba
configuration which might have unexpected side effects and *-s
/dev/null* will start **net ads kerberos pac** with an empty Samba
configuration.

As you can see the new version not only prints LOGON_INFO buffer but all
buffers and tries to decode as much as possible. The new version also
provides additionally to the *dump* option a *save* option. This is
useful if you need a real PAC blob to test PAC processing in unit tests.
If you have to decode the binary PAC later you can use the **ndrdump**
utility form the samba-test package

::

   # ndrdump krb5pac decode_pac in /tmp/pac.bla 
   pull returned NT_STATUS_OK
       decode_pac: struct decode_pac
           in: struct decode_pac
               pac: struct PAC_DATA
                   num_buffers              : 0x00000004 (4)
                   version                  : 0x00000000 (0)
                   buffers: ARRAY(4)
                       buffers: struct PAC_BUFFER
                           type                     : PAC_TYPE_LOGON_INFO (1)
                           _ndr_size                : 0x000001c8 (456)
                           info                     : *
                               info                     : union PAC_INFO(case 1)
                               logon_info: struct PAC_LOGON_INFO_CTR
   ...

If you want to use a different Kerberos service where the keys are not
stored in the host keytab you can use the following options:

::

   net ads kerberos pac dump --option='realm = IPA.DOMAIN' --option='kerberos method = dedicated keytab' --option='dedicated keytab file = FILE:/etc/dirsrv/ds.keytab' -s /dev/zero local_service=ldap/ipa-server.ipa.domain@IPA.DOMAIN -U admin
