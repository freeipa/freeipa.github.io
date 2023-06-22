.. _windows_authentication_against_freeipa:

Windows authentication against FreeIPA
======================================

This article describes direct integration between FreeIPA and Windows
machine, i.e. **without involving Active Directory** server. This
article **does not** apply to configurations where `trust between AD and
FreeIPA <Trust>`__ was established. Note also that the described
configuration is not supported by FreeIPA development team and also is
not supported by Red Hat Enterprise Linux Identity Management product. A
work on making possible to login to Windows machines already enrolled
into a trusted Active Directory forest is ongoing and is not available
yet in any released FreeIPA version.

#. If you already have AD we recommend using this system with AD and
   using trusts between AD and IPA.
#. If you do not have AD then use Samba 4 instead of it. As of Samba
   4.3, Samba AD can establish cross-realm trusts. The feature is still
   incomplete and lacks proper access controls but it can be configured
   to trust FreeIPA.
#. If neither of the two options work for you you can configure Windows
   system to work directly with IPA as described below. It is an option
   of last resort because IPA does not provide the services windows
   client expects and FreeIPA development team does not support this
   mode. If this is good enough for you, fine by us.
#. Build a native Windows client (cred provider) for IPA using latest
   Kerberos. This would be really useful if someone does that because we
   don't have capacity to not build this ourselves. With the native OTP
   support in IPA it becomes a real business opportunity to provide a
   native 2FA inside enterprise across multiple platforms. But please do
   it open source way otherwise we would not recommend you ;-)

.. _freeipa_is_not_an_active_directory_server:

FreeIPA is not an Active Directory server
-----------------------------------------

FreeIPA is not a re-implementation of Microsoft Active Directory.
FreeIPA is focused on Linux (and other standards compliant) systems.

For this reason FreeIPA without configured `AD trust <Trusts>`__ can
provide only
`authentication <http://en.wikipedia.org/wiki/Authentication>`__ service
for Windows hosts (via standard `Kerberos
protocol <http://en.wikipedia.org/wiki/Kerberos_%28protocol%29>`__).
FreeIPA can't provide account database for Windows hosts in the same way
as AD does. You have to create local Windows account and appropriate
account mapping for each user if you select direct Windows<=>FreeIPA
integration. (This limitation doesn't apply if you use `AD
trust <Trusts>`__.)

`Project pGina <http://pgina.org/>`__ could help you to overcome some
limitations.

.. _configure_freeipa:

Configure FreeIPA
-----------------

| ``1. Create the host principal in the web interface``
| ``2. Create IPA users to correspond to Windows users``
| ``3. Reset the user's IPA password to a known password using the web interface or CLI,``
| ``   the user will be prompted to change at first log in.``
| ``4. On the IPA server run``
| `` ipa-getkeytab -s [kdc DNS name]``
| ``               -p host/[machine-name]``
| ``               -e  arcfour-hmac``
| ``               -k krb5.keytab.[machine-name]``
| ``               -P``
| `` At the prompt enter a random MACHINE_PASSWORD``
| `` (you will enter this later on the windows machine too).``
| `` ``\ **``Note:``\ ````\ ``you``\ ````\ ``can``\ ````\ ``change``\ ````\ ``the``\ ````\ ``-e``\ ````\ ``argument``\ ````\ ``to``\ ````\ ``include``\ ````\ ``also``**
| `` ``\ **``AES``\ ````\ ``enctypes``\ ````\ ``from``\ ````\ ``FreeIPA``\ ````\ ``2.1.4``\ ````\ ``and``\ ````\ ``higher.``**\ `` (FreeIPA ticket ``\ ```2038`` <https://fedorahosted.org/freeipa/ticket/2038>`__\ ``)``

| `` ``\ **``Note:``\ ````\ ``Windows``\ ````\ ``machines``\ ````\ ``names``\ ````\ ``cannot``\ ````\ ``exceed``\ ````\ ``15``\ ````\ ``characters``**
| ``  -- pointed out by Han Boetes on 2013-01-03 on freeipa-users mailing list``

.. _configure_windows_ksetup:

Configure Windows (ksetup)
--------------------------

| ``1. ksetup /setdomain [REALM NAME]``
| ``2. ksetup /addkdc [REALM NAME] [kdc DNS name]``
| ``3. ksetup /addkpasswd [REALM NAME] [kdc DNS name]``
| ``4. ksetup /setcomputerpassword [MACHINE_PASSWORD] (the one used above)``
| ``5. ksetup /mapuser * *``
| ``6. Run gpedit.msc, open the key called:``
| `` "Network Security: Configure encryption types allowed for Kerberos”``
| `` under:``
| ``   Computer Configuration``
| ``     Windows Settings``
| ``       Security Settings``
| ``         Local Policies``
| ``           Security Options``
| `` and deselect everything except RC4_HMAC_MD5``
| ``7. *** REBOOT ***``
| ``8. Add local user accounts for all users that need to be able to log in.``
| ``9. Log in as [user]@[REALM] with the initial password, you will be prompted``
| ``to change the password then logged in.``

**Note: Configuring encryption types is not needed from FreeIPA 2.1.4
and higher.** (FreeIPA ticket
`2038 <https://fedorahosted.org/freeipa/ticket/2038>`__)

--------------

The FreeIPA team thanks 'Jimmy' for providing this information on the
`freeipa-users <https://www.redhat.com/archives/freeipa-users/2011-November/msg00156.html>`__
mailing list. See mailing list archives for the original text. Several
amendments were made.
