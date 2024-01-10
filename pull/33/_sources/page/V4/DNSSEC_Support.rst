DNSSEC_Support
==============

Overview
--------

Please note that **DNSSEC zone signing** and **DNSSEC records
validation** are two different features, which can coexist without each
other.



DNSSEC zone signing
----------------------------------------------------------------------------------------------

DNSSEC zone signing allows protect your DNS records. IPA supports zone
signing without additional manual configuration. You just need to set up
just one IPA DNS server as **DNSSEC key master** using
``ipa-dns-install --dnssec-master`` command, and enable particular zones
to be signed using ``dnszone-mod example.com. --dnssec=true`` command.



DNSSEC records validation
----------------------------------------------------------------------------------------------

DNSSEC records validation allows to validate signed DNS records from
other servers to protect you against spoofed addresses. IPA installer
enables DNSSEC validation by default. However, for successful validation
it is crucial to have properly DNSSEC configured forwarders. IPA checks
global forwarders and if forwarders don't support DNSSEC, DNSSEC
validation on particular server will be disabled (during installation)
or a warning message to disable DNSSEC validation manually will be shown
(``dnsconfig-mod``).

Note: **IPA 4.2** checks forwarders in forwarding zones too
(``dnsforwardzone-*``).



Use Cases
---------



Zone records signing
----------------------------------------------------------------------------------------------

IPA allows to enable zones signing per zone. IPA automatically does the
most of steps that had to be done manually before.

All required steps are:

-  install exactly the one DNSSEC master server
   (``ipa-dns-install --dnssec-master``)
-  enable zone signing (``ipa dnszone-mod example.com --dnssec=true``)
-  create DS record in the parent zone
-  verify DNSSEC chain of trust
   (``drill -TD example.com. -k /etc/trusted-key.key``)

More information can be found here: `Howto/DNSSEC <Howto/DNSSEC>`__.

Troubleshooting: `DNSSEC signing does not
work <Troubleshooting#DNSSEC_signing_does_not_work>`__.



Records validation
----------------------------------------------------------------------------------------------

Record validation is done by the BIND server itself.

IPA enables records validation by default during installation if all
global forwarders passed DNSSEC compability test that is done by
installer. If any of configured forwarders does not support DNSSEC,
installer disables records validation.

If a global forwarder or a forward zone that does not support DNSSEC is
added later, records validation must be manually disabled on all IPA
servers.

To manually enable/disable record validation, option
**dnssec-validation** in **/etc/named.conf** must be configured in
following way:

-  enable validation:

| ``options {``
| ``    dnssec-validation yes;``
| ``}``

-  disable validation:

| ``options {``
| ``    dnssec-validation no;``
| ``}``



Adding forwarders
----------------------------------------------------------------------------------------------

If the DNSSEC validation is enabled, forwarders must support DNSSEC.

New check was implemented for global forwarders and forward zones (**IPA
4.2**) that detects improper DNSSEC configuration on forwarders.

Design
------

`Exhausting information about DNSSEC
Support <https://fedorahosted.org/bind-dyndb-ldap/wiki/BIND9/Design/DNSSEC/Keys/Shortterm>`__
(Low level)

Schema
----------------------------------------------------------------------------------------------

Logical structure of entries is listed
`here <https://fedorahosted.org/bind-dyndb-ldap/wiki/BIND9/Design/DNSSEC/Keys/Shortterm#LDAPschema>`__.

Schema for DNSSEC metada is defined
`here <https://fedorahosted.org/bind-dyndb-ldap/wiki/BIND9/Design/DNSSEC/Keys/Shortterm#DNSSECmetadata>`__.

Schema for keys is defined `here <PKCS11_in_LDAP/Schema>`__.



Detection if forwarders are DNSSEC capable
----------------------------------------------------------------------------------------------

Detection was improved in **IPA 4.2**, this section describes IPA 4.2
only.



Global forwarders
----------------------------------------------------------------------------------------------

Check is done against the root zone.

#. check if the record ". IN SOA" @forwarder is resolvable (flags: RD)

   #. failed: forwarder is not working
   #. pass: continue with next step

#. check if the record ". IN SOA" @forwarder with EDNS0 is resolvable
   (flags: RD):

   #. failed: forwarder does not support EDNS0 -> does not support
      DNSSEC
   #. pass: continue with next step

#. check if the records ". IN SOA" @forwarder with EDNS0 has DNSSEC
   signatures (flags: RD, DO)

   #. failed: forwarder does not support DNSSEC
   #. pass: forwarder supports DNSSEC



Forward zones
----------------------------------------------------------------------------------------------

Check is done against the forward zone. Check consists of 2 steps:

#. check before forwarder is added into LDAP/DNS
#. check after forwarder was added into LDAP/DNS

This check is not 100% reliable, but should catch the most issues. Also
there can be false positive results and DNS administrator should check
each forwarder. Check is executed just against one of many possible IPA
DNS servers. This may cause, if any of the untested IPA DNS servers are
configured different then these servers may not resolve forward zone
properly.

Steps:

#. check if the record "fwzone IN SOA" @forwarder is resolvable (flags:
   RD)

   #. failed: forwarder is not working
   #. pass: continue with next step

#. check if the record "fwzone IN SOA" @forwarder with EDNS0 is
   resolvable (flags: RD):

   #. failed: forwarder does not support EDNS0 -> does not support
      DNSSEC
   #. pass: continue with next step

#. check if the record "fwzone IN SOA" @forwarder with EDNS0 has DNSSEC
   signatures (flags: RD, DO)

   #. failed: forwarder does not support DNSSEC
   #. pass: forwarder supports DNSSEC

#. add forwarder into LDAP/DNS
#. get answer "fwzone IN SOA" @IPA_DNS with EDNS0 (flags: RD, DO,
   **CD**); store result into ans_cd
#. check if the record "fwzone IN SOA" @IPA_DNS with EDNS0 (flags: RD,
   DO) is resolvable; store result into ans_do

   #. NXDOMAIN: DNSSEC validation error, records was marked as not
      trusted.
   #. pass: continue with next step

#. compare if ans_cd and ans_do contains the same answer (same values)

   #. failed: values differ, zone is probably "shadowed", DNSSEC
      validation may not work
   #. pass: DNSSEC validation seems to be working with this forwarder
      and forward zone

Implementation
--------------



How DNS zone gets signed in IPA
----------------------------------------------------------------------------------------------

#. **All DNS replicas** During replica installation, a keypair is
   generated in local HSM and public key is then stored into LDAP.

   #. **DNSSEC key master** If the replica is DNSSEC key master, it also
      generates DNSSEC master key and stores this key into local HSM.

#. User enables DNSSEC signing for given zone
   (``$ ipa dnszone-mod --dnssec=true zone.example``). This sets boolean
   attribute idnsSecInlineSigning in zone object to TRUE.
#. **DNSSEC key master** On the DNSSEC key master replica, change in
   idnsSecInlineSigning attribute is detected (using SyncRepl protocol)
   by *ipa-dnskeysyncd* daemon running in MASTER mode. (Master mode =
   environment variable ISMASTER is set to '1'.) Master
   *ipa-dnskeysyncd* daemon calls command ``ods-enforcer zone add``
   (``ods-ksmutil zone add`` on RHEL 7) and adds the new zone to
   OpenDNSSEC Enforcer's database running on the same machine.
#. **DNSSEC key master** OpenDNSSEC generates keys according to policy
   */etc/opendnssec/kasp.xml*. When the key is generated or changed,
   OpenDNSSEC calls ``ods-signer`` command which through socket
   */var/run/opendnssec/engine.sock* activates *ipa-ods-exporter* on the
   DNSSEC key master replica.
#. **DNSSEC key master** ipa-ods-exporter is executed on DNSSEC key
   master. It does following:

   #. Key material is synchronized:

      #. Downloads replica keys from LDAP into local HSM on the master.
      #. Encrypt master keys in local HSM using replica keys. Copy of
         master key encrypted using each replica public key is stored in
         LDAP.
      #. All new or modified zone keys from local HSM are encrypted
         using the master key and uploaded to LDAP.

   #. Key metadata like key validity etc. are sychronized from
      OpenDNSSEC database to LDAP.

#. **Other replicas which are not DNSSEC key master** *ipa-dnskeysyncd*
   daemon detects using SyncRepl protocol that key metadata were
   changed:

   #. executes *ipa-dnskeysync-replica*

      #. Downloads all master keys from LDAP and decrypts them using
         replica private key. Master keys are stored in local HSM.
      #. Downloads all zone keys from LDAP, decrypts them using DNS
         master key, and stores them in local HSM.

   #. *ipa-dnskeysyncd* daemon generates BIND key files (calls
      ``dnssec-keyfromlabel`` for each key)
   #. *ipa-dnskeysyncd* daemon informs BIND about new keys by calling
      ``rndc sign``

#. **All DNS replicas** BIND (bind-pkcs11 required) reads key metadata
   and uses local HSM to sign the DNS data.



SoftHSM configuration
----------------------------------------------------------------------------------------------

Each replica creates own local SoftHSM storage. IPA uses own
configuration of SoftHSM. To access right database you need to configure
environment variable **SOFTHSM2_CONF**.

``$ export SOFTHSM2_CONF=/etc/ipa/dnssec/softhsm2.conf``

SoftHSM database is initialized during installation (or upgrade) with
following command:

| ``$ softhsm2-util --init-token --slot=0 --label=ipaDNSSEC --pin=``\ `` --so-pin=``

and values are stored in files:

| ``/var/lib/ipa/dnssec/softhsm_pin``
| ``/etc/ipa/dnssec/softhsm_pin_so``

SoftHSM tokens are stored in directory:

``/var/lib/ipa/dnssec/tokens``



OpenDNSSEC configuration
----------------------------------------------------------------------------------------------

OpenDNSSEC is required only at IPA DNSSEC master server.

Default key parameters:

-  KSK

   -  Key Length: 3072
   -  Lifetime: 2 years
   -  Algorithm: 8 (RSASHA256)

-  ZSK

   -  Key Length: 2048
   -  Lifetime: 90 days
   -  Algorithm: 8 (RSASHA256)

Default values can be changed in *kasp.xml* file
(*/etc/opendnssec/kasp.xml*).



Directory permissions
----------------------------------------------------------------------------------------------

DNSSEC related files has to be accessible for several daemons, under
**ods** (openddnssec) and **named** user. Following list shows required
file modes, owner and group per directory/file:

| ``drwxr-x---.  ods named    /var/lib/ipa/dnssec``
| ``-rwxrwx---.  ods named    /var/lib/ipa/dnssec/softhsm_pin``
| ``drwxrws---.  ods named    /var/lib/ipa/dnssec/tokens``
| ``drwxrws---.  ods named    /var/lib/ipa/dnssec/tokens/*``
| ``-rwxrwx---.  ods named    /var/lib/ipa/dnssec/tokens/``\ ``/*``
| ``-rw-rw----.  root ods     /etc/opendnssec/*``
| ``-rw-rw----.  ods ods      /var/opendnssec/kasp.db``
| ``drwxrwx---.  ods ods      /var/opendnssec/signconf``
| ``drwxrwx---.  ods ods      /var/opendnssec/signed``
| ``drwxrwx---.  ods ods      /var/opendnssec/tmp``

**Note:** Tokens created during installation (upgrade) has root:root
owner group. Is required to modify all files and subdirs in token's
directory to proper mode, owner and group.



LDAP default PKCS#11 values
----------------------------------------------------------------------------------------------

-  IPA PKCS#11 schema: `V4/PKCS11 in
   LDAP/Schema <V4/PKCS11_in_LDAP/Schema>`__

If any LDAP attribute is not present in entry, then a particular default
value is used.



DNSSEC Master Key
^^^^^^^^^^^^^^^^^

===================== ==================== =============
PKCS#11 attribute     LDAP attribute       default value
===================== ==================== =============
CKA_COPYABLE          ipk11Copyable        true
CKA_DECRYPT           ipk11Decrypt         false
CKA_DERIVE            ipk11Derive          false
CKA_ENCRYPT           ipk11Encrypt         false
CKA_EXTRACTABLE       ipk11Extractable     true
CKA_MODIFIABLE        ipk11Modifiable      true
CKA_PRIVATE           ipk11Private         true
CKA_SENSITIVE         ipk11Sensitive       true
CKA_SIGN              ipk11Sign            false
CKA_UNWRAP            ipk11Unwrap          true
CKA_VERIFY            ipk11Verify          false
CKA_WRAP              ipk11Wrap            true
CKA_WRAP_WITH_TRUSTED ipk11WrapWithTrusted false
===================== ==================== =============



Replica Private Key
^^^^^^^^^^^^^^^^^^^

======================= ======================= =============
PKCS#11 attribute       LDAP attribute          default value
======================= ======================= =============
CKA_ALWAYS_AUTHENTICATE ipk11AlwaysAuthenticate false
CKA_COPYABLE            ipk11Copyable           true
CKA_DECRYPT             ipk11Decrypt            false
CKA_DERIVE              ipk11Derive             false
CKA_EXTRACTABLE         ipk11Extractable        false
CKA_MODIFIABLE          ipk11Modifiable         true
CKA_PRIVATE             ipk11Private            true
CKA_SENSITIVE           ipk11Sensitive          true
CKA_SIGN_RECOVER        ipk11Sign               false
cka_sign_recover        ipk11SignRecover        false
CKA_UNWRAP              ipk11Unwrap             true
CKA_WRAP_WITH_TRUSTED   ipk11WrapWithTrusted    false
======================= ======================= =============



Replica Public Key
^^^^^^^^^^^^^^^^^^

================== ================== =============
PKCS#11 attribute  LDAP attribute     default value
================== ================== =============
CKA_COPYABLE       ipk11Copyable      true
CKA_DERIVE         ipk11Derive        false
CKA_ENCRYPT        ipk11Encrypt       false
CKA_MODIFIABLE     ipk11Modifiable    true
CKA_PRIVATE        ipk11Private       true
CKA_TRUSTED        ipk11Trusted       false
CKA_VERIFY         ipk11Verify        false
CKA_VERIFY_RECOVER ipk11VerifyRecover false
CKA_WRAP           ipk11Wrap          true
================== ================== =============



DNSSEC Master Key Attributes
----------------------------------------------------------------------------------------------

Master key is generated only by IPA DNSSEC Master server, with following
values (default values are not listed):

-  CK_MECHANISM: CKM_AES_KEY_GEN
-  CKA_LABEL: "dnssec-master"
-  CKA_ID: 16B pseudo-random value (unique per secret key)
-  CKA_VALUE_LEN: 16 (keylength)
-  CKA_TOKEN: true
-  CKA_WRAP: true
-  CKA_UNWRAP: true



Disabling Old Master Key
^^^^^^^^^^^^^^^^^^^^^^^^

If new master key is generated, the old key must be disable by setting
attribute **CKA_WRAP** to **false**.



Replica Keys Attributes
----------------------------------------------------------------------------------------------

Each replica generates own replica key pair during install (upgrade)
with following values (attributes with default values are not listed):

-  **Both (Private and Public Key):**

   -  CK_MECHANISM: CKM_RSA_PKCS_KEY_PAIR_GEN
   -  CKA_LABEL: a canonicalized absolute replica domain name
   -  CKA_ID: 16B pseudo-random value (same for value for private and
      public key), this value is unique per a key pair
   -  CKA_TOKEN: true

-  **Private Key:**

   -  CKA_UNWRAP: true
   -  CKA_SENSITIVE: true
   -  CKA_EXTRACTABLE: false

-  **Public Key:**

   -  CKA_PUBLIC_EXPONENT: 65537 (RFC 6376 section 3.3.1)
   -  CKA_MODULUS_BITS: 2048
   -  CKA_VERIFY: false
   -  CKA_VERIFY_RECOVER: false
   -  CKA_WRAP: true

Public replica key is also stored in LDAP database:

::

   | ``dn: ipk11UniqueId=``\ ``,cn=keys,cn=sec,cn=dns,dc=example,dc=com``
   | ``objectclass: ipk11Object``
   | ``objectclass: ipk11PublicKey``
   | ``objectclass: ipaPublicKeyObject``
   | ``objectclass: top``
   | ``ipk11UniqueId: ``
   | ``ipk11Label: ``
   | ``ipaPublicKey: <public key in SubjectPublicKeyInfo (RFC 5280) form>``
   | ``ipk11Id': ``\ ``,``
   | ``ipk11Wrap: true``
   | ``ipk11Verify: false``
   | ``ipk11VerifyRecover: false``



Disabling old replica keys
^^^^^^^^^^^^^^^^^^^^^^^^^^

If a new key is generated, old public keys must be disabled. This is
achieved by setting **CKA_WRAP (ipk11Wrap)** attribute to **false** in
both LDAP and local SoftHSM database.

Private keys should stay unchanged, to allow unwrap already wrapped keys
in LDAP.

Dependencies
----------------------------------------------------------------------------------------------

-  Softhsm > 2
-  opendnssec



Backup and Restore
----------------------------------------------------------------------------------------------

Following directories/files must be backed up:

-  IPA DNSSEC directory (*/var/lib/ipa/dnssec*)

   -  directory containing tokens (*tokens/*)
   -  file containing softhsm user pin (*softhsm_pin*)

-  system configuration of ipa-dnskeysyncd daemon
   (*/etc/sysconfig/ipa-dnskeysyncd*)
-  system configuration of ipa-ods-exporter
   (*/etc/sysconfig/ipa-ods-exporter*)
-  system configuration of OpenDNSSEC (*/etc/sysconfig/ods*)
-  security officer PIN (*/etc/ipa/dnssec/softhsm_pin_so*)
-  OpenDNSSEC configuration file (*/etc/opendnssec/conf.xml*)
-  OpendDNSSEC KASP database configuration file
   (*/etc/opendnssec/kasp.xml*)
-  KASP database (*/var/opendnssec/kasp.db*)
-  zone list file (*/etc/opendnssec/zonelist.xml*)
-  softhsm configuration file (*/etc/ipa/dnssec/softhsm2.conf*)
-  ipa-ods-exporter keytab (*/etc/ipa/dnssec/ipa-ods-exporter.keytab*)
-  ipa-dnskeysyncd keytab (*/etc/ipa/dnssec/ipa-dnskeysyncd.keytab*)



Feature Management
------------------

UI

N/A

CLI

Enabling DNSSEC signing:

-  ``ipa dnszone-add --dnssec=true``
-  ``ipa dnszone-mod --dnssec=true``

Disabling DNSSEC signing:

-  ``ipa dnszone-mod --dnssec=false``

Installers
----------------------------------------------------------------------------------------------

Install DNSSEC master:

-  ``ipa-dns-install --dnssec-master``

Disable DNSSEC master (**IPA 4.2**):

-  ``ipa-dns-install --disable-dnssec-master``

Reenable DNSSEC master (**IPA 4.2**):

-  ``ipa-dns-install --dnssec-master --kasp-db=<path to kasp.db file from disabled master>``

Configuration
----------------------------------------------------------------------------------------------

N/A

Upgrade
-------

Required enabling/configuring ipa-dnskeysyncd service instance on each
DNS replica



How to test
-----------

Follow instructions in this `howto <Howto/DNSSEC>`__, to test DNSSEC:

-  install DNSSEC master
-  enable signing per zone
-  create root zone, create zone and verify chain of trust



Test Plan
---------

Prerequisites
----------------------------------------------------------------------------------------------

QE is going to need DNS servers which has option "dnssec-enable yes;" in
named.conf. This option has to be enabled on the whole chain of
forwarders used by testing machines.

Tests
----------------------------------------------------------------------------------------------

Integration `DNSSEC
test <https://git.fedorahosted.org/cgit/freeipa.git/tree/ipatests/test_integration/test_dnssec.py>`__
covers:

-  Test if synchronization of keys works between replicas

   -  Master server is DNSSEC key master
   -  New replica is DNSSEC key master

-  Test if zones in IPA are signed on all replicas:

   -  Zone created on master (with --dnssec=true)
   -  Zone created on replica (with --dnssec=true)
   -  Disable zone signing & re-enable DNSSEC signing for zone on master
   -  Disable zone signing & re-enable DNSSEC signing for zone on
      replica

-  Test DNSSEC chain of trust

   -  Create root zone (with --dnssec=true) and test if it is signed
   -  Create example.zone. (with --dnssec=true) and test if it is
      signed.
   -  Export DS record of example.zone. to IPA's root zone
   -  test if DNSSEC chain of signatures is trusted (using ``drill``
      command)

-  Test migration DNSSEC master



RFE Author
----------

Martin Basti <mbasti@redhat.com>

Petr Spacek <pspacek@redhat.com>