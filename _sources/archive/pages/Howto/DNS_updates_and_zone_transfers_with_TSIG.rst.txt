.. _dns_updates_and_zone_transfers_with_tsig:

DNS updates and zone transfers with TSIG
========================================

FreeIPA doesn't have support for TSIG in user interface but it can be
configured to use TSIG for dynamic updates and zone transfers.

.. _tsig_key_configuration:

TSIG key configuration
----------------------

.. _generate_a_new_tsig_key:

Generate a new TSIG key
~~~~~~~~~~~~~~~~~~~~~~~

| ``$ dnssec-keygen -a HMAC-SHA512 -b 512 -n HOST keyname``
| ``Kkeyname.+165+03160``

.. _copy_and_paste_key_from_key_file_to_named.conf:

Copy and paste key from key file to named.conf
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| ``$ cat Kkeyname.+165+0316.private``
| ``Private-key-format: v1.3``
| ``Algorithm: 165 (HMAC_SHA512)``
| ``Key: keyvalue``
| ``Bits: AAA=``

| ``$ vim /etc/named.conf``
| ``key "keyname" {``
| ``       algorithm hmac-sha512;``
| ``       secret "keyvalue";``
| ``};``

You have to repeat this copy&paste step for every FreeIPA server.

.. _dynamic_updates:

Dynamic updates
---------------

Server
~~~~~~

Normal rules for `BIND dynamic update policies
apply <http://ftp.isc.org/isc/bind9/cur/9.9/doc/arm/Bv9ARM.ch06.html#dynamic_update_policies>`__.
Just use name of the key you defined in named.conf:

``$ ipa dnszone-mod example.com. --update-policy="grant keyname name example.com A;"``

One of FreeIPA specifics is that dynamic updates can be completely
disabled by switch even if update policy is non-empty. Make sure that
DNS dynamic updates are enabled for your zone:

``$ ipa dnszone-mod example.com. --dynamic-update=1``

Client
~~~~~~

For ``nsupdate`` from ``bind-utils`` package you have to either use
option ``-y algorithm:keyname:keyvalue`` or ``-k keyfilename`` option.
E.g.

``$ nsupdate -y hmac-sha512:keyname:keyvalue``

or

``$ nsupdate -k Kkeyname.+165+0316.private``

.. _zone_transfers:

Zone transfers
--------------

.. _server_1:

Server
~~~~~~

FreeIPA user interface will not allow you to configure allow-transfer
policy directly because it expects that allow-transfer consists only of
IP addresses. You have to modify LDAP directly.

Run this on one of FreeIPA servers:

| ``$ kinit admin``
| ``$ ldapmodify -Y GSSAPI << EOF``
| ``dn: idnsname=example.com.,cn=dns,dc=ipa,dc=example``
| ``changetype: modify``
| ``replace: idnsAllowTransfer``
| ``idnsAllowTransfer: key keyname;``
| ``-``
| ``EOF``

Don't forget to replace zone name in ``idnsname`` component of DN and
realm name in ``dc=ipa,dc=example`` components.

.. _client_1:

Client
~~~~~~

The syntax for ``dig`` from ``bind-utils`` package is the same as for
``nsupdate``. You have to either use option
``-y algorithm:keyname:keyvalue`` or ``-k keyfilename`` option. E.g.

``$ dig -y hmac-sha512:keyname:keyvalue``

or

``$ dig -k Kkeyname.+165+0316.private``
