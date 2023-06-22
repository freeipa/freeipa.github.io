.. _enable_dnssec_support_in_freeipa:

Enable DNSSEC support in FreeIPA
--------------------------------

Steps to enable DNSSEC support:

-  choose one server which will act as DNSSEC key master
-  enable DNSSEC signing for individulal DNS zones

.. _dnssec_key_master:

DNSSEC key master
~~~~~~~~~~~~~~~~~

To enable DNSSEC in FreeIPA topology, exactly one FreeIPA replica has to
act as *the DNSSEC key master*. This replica is responsible for proper
key generation and rotation.

Zone signing will not work without DNSSEC key master replica.

Following command will install DNSSEC key master role to a replica. This
command can be run on FreeIPA server which is already configured as
FreeIPA DNS server:

| ``# ipa-dns-install --dnssec-master``
| ``The log file for this installation can be found in /var/log/ipaserver-install.log``
| ``==============================================================================``
| ``This program will setup DNS for the FreeIPA Server.``
| ``This includes:``
| ``  * Configure DNS (bind)``
| ``  * Configure SoftHSM (required by DNSSEC)``
| ``  * Configure ipa-dnskeysyncd (required by DNSSEC)``
| ``  * Configure ipa-ods-exporter (required by DNSSEC key master)``
| ``  * Configure OpenDNSSEC (required by DNSSEC key master)``
| ``  * Generate DNSSEC master key (required by DNSSEC key master)``
| ``NOTE: DNSSEC zone signing is not enabled by default``
| ``DNSSEC support is experimental!``
| ``Plan carefully, current version doesn't allow you to move DNSSEC``
| ``key master to different server and master cannot be uninstalled``
| ``To accept the default shown in brackets, press the Enter key.``
| **``Do``\ ````\ ``you``\ ````\ ``want``\ ````\ ``to``\ ````\ ``setup``\ ````\ ``this``\ ````\ ``IPA``\ ````\ ``server``\ ````\ ``as``\ ````\ ``DNSSEC``\ ````\ ``key``\ ````\ ``master?``\ ````\ ``[no]:``\ ````\ ``yes``**
| ``Existing BIND configuration detected, overwrite? [no]: yes``
| ``...``

Please verify the file *kasp.xml* (*/etc/opendnssec/kasp.xml*) if it
satisfies your DNSSEC configuration requirements.

.. _enable_zone_signing:

Enable zone signing
~~~~~~~~~~~~~~~~~~~

Zones are not signed by default. Zone signing has to be enabled per zone
using following command:

| ``$ ipa dnszone-add example.test. --dnssec=true``
| ``  Zone name: example.test.``
| ``  Active zone: TRUE``
| ``. . .``
| `` Allow query: any;``
| `` Allow transfer: none;``
| `` ``\ **``Allow``\ ````\ ``in-line``\ ````\ ``DNSSEC``\ ````\ ``signing:``\ ````\ ``TRUE``**

or

``$ ipa dnszone-mod example.test. --dnssec=true``

To disable zone signing use command:

``$ ipa dnszone-mod example.test. --dnssec=false``

.. _signing_zones_in_freeipa:

Signing zones in FreeIPA
------------------------

.. _add_a_zone_to_be_signed:

Add a zone to be signed
~~~~~~~~~~~~~~~~~~~~~~~

``ipa dnszone-add example.test. --dnssec=true``

or

| ``ipa dnszone-add example.test.``
| ``ipa dnszone-mod --dnssec=true``

.. _verify_if_zone_is_signed:

Verify if zone is signed
~~~~~~~~~~~~~~~~~~~~~~~~

Verify using *dig* utility if answer contains RRSIG record. This may
take a few minutes until proper key are distributed to all replicas in
topology.

| ``$ dig @server.ipa.test. +dnssec example.test. SOA``
| ``;; Got answer:``
| ``;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 41379``
| ``;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 3, ADDITIONAL: 2``
| ``;; OPT PSEUDOSECTION:``
| ``; EDNS: version: 0, flags: do; udp: 4096``
| ``;; QUESTION SECTION:``
| ``;example.test.         IN  SOA``
| ``;; ANSWER SECTION:``
| ``example.test.      86400   IN  SOA    server.ipa.test. hostmaster.example.test. 1426005184 3600 900 1209600 3600``
| ``example.test.      86400   IN  ``\ **``RRSIG``**\ ``  SOA 8 2 86400 20150409163304 20150310153304 30144 example.test. 8Q1g1wXlJ0647pTF7rhGsZDrkxzq8QGdcviraEEityhS9/2lvMz6tem6 ...``

.. _key_types_ksk_and_zsk:

Key types: KSK and ZSK
~~~~~~~~~~~~~~~~~~~~~~

DNSSEC zones typically contain two types of DNSSEC keys.

KSK
   Key Signing Key-pair (long term)
   Private key is used to sign ZSK
   Public key exposed in **DNSKEY 257** record is used to verify
   signatures over ZSK. Hash of the public KSK is stored in DS record in
   the parent zone to create the chain of trust from parent to child
   zones.
   This key is used for signing DNSKEY records in zone.

ZSK
   Zone Signing Key-pair
   Private key is used to sign records
   Public key exposed in **DNSKEY 256** record is used to verify
   signatures over other resource records in the DNS zone.

| ``$ dig +rrcomments example.test. DNSKEY``
| ``...``
| ``;; ANSWER SECTION:``
| ``example.test.      86400   IN  ``\ **``DNSKEY``\ ````\ ``257``**\ `` 3 8 AwEAAbxszl5h9Mag1AG2uTsBCoR7oIgfTm3bU8H10bcaNiUrkqpPUXq+ ... ; KSK; alg = RSASHA256; key id = 60466``
| ``example.test.      86400   IN  ``\ **``DNSKEY``\ ````\ ``256``**\ `` 3 8 AwEAAfxpqvJhHDzNwH9Lhm0H9qyzxRSG8Kpt2AGpg6J6RqHtBtZrYB1J ... ; ZSK; alg = RSASHA256; key id = 30144``

On **DNSSEC key master** all currently used keys can be shown using
following command (replace ``ods-enforcer`` by ``ods-ksmutil`` on RHEL
7):

| ``$ sudo -u ods SOFTHSM2_CONF=/etc/ipa/dnssec/softhsm2.conf ods-enforcer key list --verbose``
| ``SQLite database set to: /var/opendnssec/kasp.db``
| ``Keys:``
| ``Zone:           Keytype:  State:  Date of next transition (to):  Size:   Algorithm: CKA_ID:                           Repository:               Keytag:``
| ``example.test    ZSK       active  2015-06-08 12:33:00 (retire)   2048    8          069ee3ece56beee7129ea18494331b35  SoftHSM                   30144``
| ``example.test    ``\ **``KSK``**\ ``      ``\ **``ready``**\ ``   ``\ **``waiting``\ ````\ ``for``\ ````\ ``ds-seen``\ ````\ ``(active)``**\ ``   2048    8          7d44dc987ef258ce0b88c81550d4e319  SoftHSM                   ``\ **``60466``**

.. _get_the_ds_record:

Get the DS record
~~~~~~~~~~~~~~~~~

The DS record of the zone, has to be uploaded to parent zone, otherwise
chain of trust can not be completed.

| ``$ dig example.test. DNSKEY > dnskey.txt``
| ``$ dnssec-dsfromkey -f dnskey.txt -2 example.test``
| ``example.test. IN DS ``\ **``60466``**\ `` 8 2 0A758A8B28B7D1A9467D3E91E9699C0ECA381E18AFFCF7C4EB7955E24ED87956``

Output of the *dnssec-dsfromkey* is the DS record for zone
*example.test.*, which has to be uploaded to parent zone, e.g. *test.*.

.. _add_ds_record_into_parent_zone:

Add DS record into parent zone
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Following example shows how to add DS record of *example.test.* zone
into a parent zone *test.* which is managed by IPA:

``$ ipa dnsrecords-add test. example.test. --ns-rec=ns.example.test.  ``\ **``--ds-rec="60466``\ ````\ ``8``\ ````\ ``2``\ ````\ ``0A758A8B28B7D1A9467D3E91E9699C0ECA381E18AFFCF7C4EB7955E24ED87956"``**

DS record has to be added to the same name as NS record (delegation)
**in the parent zone**.

The procedure to add DS record will be different if you are not using
FreeIPA for managing the parent zone but the end goal is the same - you
need to get DS records added to the parent zone to establish chain of
trust from the parent zone.

.. _confirm_ds_record_upload:

Confirm DS record upload
~~~~~~~~~~~~~~~~~~~~~~~~

Verify that DS record is available from the parent zone:

| ``$ dig +rrcomments example.test DS ``
| ``example.test       86400   IN  DS  ``\ **``60466``**\ `` 8 2 0A758A8B ...``

After successfull DS record upload to the parent zone, the following
command has to be executed on DNSSEC key master server to enable key
rotation. Keytag value has to match KSK keytag as shown in outputs
above:

``$ sudo -u ods SOFTHSM2_CONF=/etc/ipa/dnssec/softhsm2.conf ods-enforcer key ds-seen --zone example.test --keytag ``\ **``60466``**

*ds-seen* command will allow the KSK to proceed to the next state:

| ``$ sudo -u ods SOFTHSM2_CONF=/etc/ipa/dnssec/softhsm2.conf ods-enforcer key list --verbose``
| ``SQLite database set to: /var/opendnssec/kasp.db``
| ``Keys:``
| ``Zone:           Keytype:  State:  Date of next transition (to):  Size:   Algorithm: CKA_ID:                           Repository:               Keytag:``
| ``example.test    ZSK       active  2015-06-08 12:33:00 (retire)   2048    8          069ee3ece56beee7129ea18494331b35  SoftHSM                   30144``
| ``example.test    ``\ **``KSK``**\ ``       ``\ **``ready``**\ ``   ``\ **``2016-03-09``\ ````\ ``11:34:38``\ ````\ ``(retire)``**\ ``   2048    8          7d44dc987ef258ce0b88c81550d4e319  SoftHSM                   ``\ **``60466``**

.. _verify_dnssec_chain_of_trust:

Verify DNSSEC chain of trust
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If DS record was successfully uploaded to parent zone, the check if
chain of trust can be established should follow, to make sure the
records from zone will pass the DNSSEC validation on DNS servers.

For example this can be done via ``drill`` utility:

| ``drill -TD example.test. -k /etc/trusted-key.key``
| ``drill -TD example.test. SOA -k /etc/trusted-key.key``
| ``drill -TD host.example.test. A -k /etc/trusted-key.key``

All keys/records should be marked as [T] trusted.

.. _dnssec_in_isolated_networks:

DNSSEC in isolated networks
---------------------------

.. _create_signed_root_zone:

Create signed root zone
~~~~~~~~~~~~~~~~~~~~~~~

How to create the root zone is explained in article `DNS in isolated
networks <Howto/DNS_in_isolated_networks>`__. Please note that update of
root hints will be required on all recursive clients as noted in the
linked article.

Do not forget to install DNSSEC key master before you enable DNSSEC
signing.

You can enable DNSSEC zone signing for it:

``$ ipa dnszone-mod . --dnssec=true``

.. _configure_trusted_key_on_clients:

Configure trusted key on clients
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Local resolvers need to know KSK of your root zone because it is entry
point to the chain of trust from root zone to all other zones.

Get the KSK key of your root zone:

| ``$ dig @localhost  . DNSKEY``
| ``...``
| ``;; QUESTION SECTION:``
| ``;.             IN  DNSKEY``
| ``;; ANSWER SECTION:``
| ``.          86400   IN  DNSKEY  256 3 8 AwEAAdsQWj6AM8dVdvgRPw87DaSWRa2w7oknABSepVwhDlOLpxicOS+n ...``
| **``.``\ ````\ ``86400``\ ````\ ``IN``\ ````\ ``DNSKEY``\ ````\ ``257``\ ````\ ``3``\ ````\ ``8``\ ````\ ``AwEAAdsNYeNTZMVgvWYAEIv+w0PujAmWtcSF15rvsPP25X2lFkgIg+QT``\ ````\ ``JLqHzaughLdjduMUCGJwLfG7O4IUIIhqApwLAbQ+GYfrRSaETPPc9z/X``\ ````\ ``AGtqiOn/EYj3BcO95wJPcubXxOukHrXcZ/Pt153EkMHyBGTHcsYDA1rD``\ ````\ ``qwN5S+IY4PxlhilSth0e427bSJx18huQogR/O0iu6hkKNoFUAflG697P``\ ````\ ``a88FJMwL0l6BSJR3WCi/lT0HuX4c4nNKpolaJX3dJoZphGiCsFRmZ67l``\ ````\ ``Vswrk88vkVKeD4JLZAq5wJd78IFO8Jd0gSwQY5Q0LxnArcl2yn1d2uSt``\ ````\ ``Fcs8Xgl7E1s=``**
| ``...``

Put your root zone KSK (denoted by flag value **257**) into
*trusted-key.key* file on all DNSSEC clients:

| ``$ cat /etc/trusted-key.key``
| ``.          86400   IN  DNSKEY  257 3 8 AwEAAdsNYeNTZMVgvWYAEIv+w0PujAmWtcSF15rvsPP25X2lFkgIg+QT JLqHzaughLdjduMUCGJwLfG7O4IUIIhqApwLAbQ+GYfrRSaETPPc9z/X AGtqiOn/EYj3BcO95wJPcubXxOukHrXcZ/Pt153EkMHyBGTHcsYDA1rD qwN5S+IY4PxlhilSth0e427bSJx18huQogR/O0iu6hkKNoFUAflG697P a88FJMwL0l6BSJR3WCi/lT0HuX4c4nNKpolaJX3dJoZphGiCsFRmZ67l Vswrk88vkVKeD4JLZAq5wJd78IFO8Jd0gSwQY5Q0LxnArcl2yn1d2uSt Fcs8Xgl7E1s=``

.. _migrate_dnssec_master_to_another_ipa_server:

Migrate DNSSEC master to another IPA server
-------------------------------------------

Supported on version: **IPA 4.2+**

Migration is not recommended. In case of failure DNSSEC caused by
migration, DNSSEC signing may be broken and you may need to recreate new
keys.

Requirements
~~~~~~~~~~~~

-  only one DNSSEC master can be active in topology
-  DNSSEC master can be migrated only to IPA server where
   *ipa-dnskeysyncd* is running (IPA 4.1+ with installed DNS)
-  you have zones with enabled DNSSEC signing

   -  if you do not have any zones with DNSSEC signing enabled, you can
      just disable dnssec master

Steps
~~~~~

.. _disable_current_dnssec_key_master:

Disable current DNSSEC key master
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To disable current DNSSEC master, please reinstall IPA DNS with
``--disable-dnssec-master`` option.

| ``# ipa-dns-install --disable-dnssec-master``
| ``The log file for this installation can be found in /var/log/ipaserver-install.log``
| ``==============================================================================``
| ``This program will setup DNS for the FreeIPA Server.``
| ``This includes:``
| ``  * Configure DNS (bind)``
| ``  * Configure SoftHSM (required by DNSSEC)``
| ``  * Configure ipa-dnskeysyncd (required by DNSSEC)``
| ``  * Unconfigure ipa-ods-exporter``
| ``  * Unconfigure OpenDNSSEC``
| ``No new zones will be signed without DNSSEC key master IPA server.``
| ``Please copy file from /var/lib/ipa/ipa-kasp.db.backup after uninstallation. This file is needed on new DNSSEC key ``
| ``master server``
| ``NOTE: DNSSEC zone signing is not enabled by default``
| ``To accept the default shown in brackets, press the Enter key.``
| ``Do you want to disable current DNSSEC key master? [no]: ``\ **``yes``**
| ``Existing BIND configuration detected, overwrite? [no]: ``\ **``yes``**
| `` ``
| ``...``

.. _copy_kasp.db_to_safe_location:

Copy kasp.db to safe location
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This file will be needed on target server.

`` # scp /var/lib/ipa/ipa-kasp.db.backup me@my.happy.place:/safe/location/ipa-kasp.db.backup``

.. _install_dnssec_key_master_on_target_ipa_server:

Install DNSSEC key master on target IPA server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You need kasp.db file from disabled DNSSEC key master, to be able
restore proper key rotation for existing zones.

With option ``--kasp-db=<path to original kasp.db file>`` installer does
several additional steps, which. Please do not copy this file to
location where OpenDNSSEC is expecting to find this file, this will not
work.

| ``# ipa-dns-install --dnssec-master --kasp-db=/safe/place/ipa-kasp.db.backup``
| ``The log file for this installation can be found in /var/log/ipaserver-install.log``
| ``==============================================================================``
| ``This program will setup DNS for the FreeIPA Server.``
| ``This includes:``
| ``  * Configure DNS (bind)``
| ``  * Configure SoftHSM (required by DNSSEC)``
| ``  * Configure ipa-dnskeysyncd (required by DNSSEC)``
| ``  * Configure ipa-ods-exporter (required by DNSSEC key master)``
| ``  * Configure OpenDNSSEC (required by DNSSEC key master)``
| ``  * Generate DNSSEC master key (required by DNSSEC key master)``
| ``NOTE: DNSSEC zone signing is not enabled by default``
| ``DNSSEC support is experimental!``
| ``Plan carefully, replacing DNSSEC key master is not recommended``
| ``To accept the default shown in brackets, press the Enter key.``
| ``Do you want to setup this IPA server as DNSSEC key master? [no]: ``\ **``yes``**
| ``Existing BIND configuration detected, overwrite? [no]: ``\ **``yes``**
| ``...``

.. _check_if_dnssec_signing_still_works:

Check if DNSSEC signing still works
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  show status if DNSSEC/DNS related services are running (except
   *ipa-ods-exporter* service which is run only on-demand)
-  check if signed zones are present in OpenDNSSEC ( `howto
   here <Troubleshooting#DNS_keys_are_not_generated_by_OpenDNSSEC>`__).
-  test DNSSEC signatures of current zones using ``dig +dnssec``
-  try to add new test zone with enabled DNSSEC signing and test if it
   works
