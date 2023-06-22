This short tutorial will teach you how to setup your name server so that
you can dynamically update the resource records with the help of
FreeIPA.

Introduction
------------

Our network looks like this:

| ``Subnet:         192.0.2.0/24``
| ``Domain:         example.com``
| ``Kerberos realm: EXAMPLE.COM``
| ``IPA server:     ipaserver.example.com``
| ``DNS server:     dns.example.com``
| ``Client:         client.example.com``

.. _configuring_the_name_server:

Configuring the name server
---------------------------

FreeIPA configures DNS server for you (on the same machine as FreeIPA
replica). You can safely skip step "Configuring GSS-TSIG" if you ran
command ``ipa-server-install --setup-dns`` or ``ipa-dns-setup``.

.. _configuring_gss_tsig:

Configuring GSS-TSIG
~~~~~~~~~~~~~~~~~~~~

First, we have to configure the BIND on our DNS server to use GSS-TSIG
for authenticating dynamic updates:

``/etc/named.conf`` must contain this:

-  For BIND `version >=
   9.8.0 <https://lists.isc.org/pipermail/bind-announce/2011-March/000691.html>`__:

| ``options {``
| ``   ...``
| ``   tkey-gssapi-keytab  "DNS/dns.example.com";``
| ``   ...``
| ``};``

-  For BIND version < 9.8.0:

| ``options {``
| ``   ...``
| ``   tkey-gssapi-credential  "DNS/dns.example.com";``
| ``   tkey-domain             "dns.example.com";``
| ``   ...``
| ``};``

Environment variable ``KRB5_KTNAME`` has to contain path to keytab file.
On RHEL and Fedora systems it is possible to add line
``KRB5_KTNAME=/etc/named.keytab`` to ``/etc/sysconfig/named``. If this
is not done, BIND older than 9.8.0 will not start and will exit with
very misleading error messages.

Next we must add the DNS service principal and acquire the keytab. We'll
also add the host service for our client.

| ``# kinit admin``
| ``Password for admin@EXAMPLE.COM: ``
| ``# ipa-addservice DNS/dns.example.com``
| ``# ipa-getkeytab -s ipaserver.example.com -p DNS/dns.example.com -k /etc/named.keytab``
| ``Keytab successfully retrieved and stored in: /etc/named.keytab``

Note: Set appropriate file permissions for file ``/etc/named.keytab``.
This file is usually owned by user ``named`` with mode ``400`` on RHEL
and Fedora systems.

Now we can start named and set it up to always start when booting:

| ``# service named start``
| ``Starting named:                                            [  OK  ]``
| ``# chkconfig named on``

.. _dynamic_update_policy:

Dynamic update policy
~~~~~~~~~~~~~~~~~~~~~

As a next step, configure `dynamic update
policies <http://ftp.isc.org/isc/bind9/cur/9.9/doc/arm/Bv9ARM.ch06.html#dynamic_update_policies>`__
according to your requirements.

**Example:** All machines belonging to Kerberos realm EXAMPLE.COM are
allowed to update own A record. Update requests have to be signed by
Kerberos principal ``host/``\ *``machine.fqdn``*\ ``@EXAMPLE.COM``.
(Naturally, name *machine.fqdn* has to belong to the zone example.com.)

| ``zone "example.com" {``
| ``   update-policy {``
| ``       grant EXAMPLE.COM krb5-self * A;``
| ``   };``
| ``   ...``
| ``};``

**Example:** Allow Kerberos principal
``SERVICE/ipaserver.example.com@EXAMPLE.COM`` to do any updates in whole
zone. Note the escaping trick, symbol "/" was replaced with "\047".

| ``zone "example.com" {``
| ``   update-policy {``
| ``       grant SERVICE\047ipaserver.example.com@EXAMPLE.COM wildcard * ANY;``
| ``   };``
| ``   ...``
| ``};``

**Example:** Machine is allowed to update own PTR record in reverse
zone. Update is allowed only if it came over TCP connection and source
IP address matches updated name (in reverse tree).

| ``zone "2.0.192.in-addr.arpa" IN {``
| ``   ...``
| ``   update-policy {``
| ``       grant * tcp-self * PTR;``
| ``   };``
| ``   ...``
| ``};``

.. _configuring_the_client:

Configuring the client
----------------------

We will now get the keytab on the client and use it right away with
kinit. (This step is not required if the client was enrolled by
``ipa-client-install`` script or host keytab is already in place for
other reasons.)

| ``# kinit admin``
| ``Password for admin@EXAMPLE.COM: ``
| ``# ipa-addservice host/client.example.com``
| ``# ipa-getkeytab -s ipaserver.example.com -p host/ipaserver.example.com -k /etc/named.keytab``
| ``# kinit -k -t /etc/named.keytab host/client.example.com@EXAMPLE.COM``

Notice that we aren't required to type any password during ``kinit``.
All actions from now will be done under account
``host/client.example.com@EXAMPLE.COM``.

Now we are ready to use ``nsupdate`` utility to update resource records.
``nsupdate`` can be used as a shell-type utility with prompt, or we can
place all the commands in a file and then give the file to ``nsupdate``.

See ``nsupdate(8)`` for more information about other ``nsupdate``
commands. The -g option we use is not documented in older man pages.

In following examples, the "``server dns.example.com``" command tells
``nsupdate`` to update the specified DNS server, but be aware that when
doing lookups, it will still use the default server as specified in
``/etc/resolv.conf``. Updates will be sent to master server of the
correct zone if no ``server`` command is used.

Examples
--------

-  File ``a_update``:

| ``server dns.example.com``
| ``zone example.com.``
| ``prereq yxrrset client.example.com.                            IN      A``
| ``update delete client.example.com.                             IN      A``
| ``send``
| ``update add client.example.com.                86400           IN      A       192.0.2.120``
| ``send``

If we will now want to update our A record, we will execute ``nsupdate``
like this:

``nsupdate -g a_update``

-  File ``ptr_update``:

| ``server dns.example.com``
| ``zone 2.0.192.in-addr.arpa.``
| ``prereq yxrrset 120.2.0.192.in-addr.arpa.                    IN      PTR``
| ``update delete 120.2.0.192.in-addr.arpa.                     IN      PTR``
| ``send``
| ``update add 120.2.0.192.in-addr.arpa.        86400           IN      PTR     client.example.com.``
| ``send``

If we want to update our PTR record we'll use ``ptr_update`` file as an
argument and add ``-v`` option to force update over TCP. Sometimes
``-g`` option enforces TCP usage, but the Kerberos authentication is not
necessary in this case (because of ``tcp-self`` option).

``nsupdate -v ptr_update``

Troubleshooting
---------------

If you have troubles with ``nsupdate``, try some additional debugging
flags, for example:

``nsupdate -d -D 99 a_update``

You can also add ``debug`` command to separate line:

| ``debug``
| ``zone 2.0.192.in-addr.arpa.``
| ``update add 120.2.0.192.in-addr.arpa.        86400           IN      PTR     client.example.com.``
| ``send``

If you have problems with Kerberos, you can try to use the -l flag in
order to communicate with local DNS server and get GSS-API major and
minor error messages.

The -D and -l flags were not documented.
