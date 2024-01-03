LDAP
====



LDAP Overview
=============

This guide is meant to provide general guidance on configuring an LDAP
client to connect to IPA. There are specific guides/Howtos for some
clients/servers.



Data layout (DIT)
=================

The basedn in an IPA installation consists of a set of domain components
(dc) for the initial domain that IPA was configured with. If you
installed IPA with the domain example.com then your basedn is
``dc=example,dc=com``. We often refer to this as $SUFFIX. Don't confuse
this with DNS domains. You will only ever have one basedn, the one
defined during installation.

You can find your basedn, and other interesting things, in
``/etc/ipa/default.conf``

IPA uses a flat structure, storing like objects in what we call
containers. Some container examples are:

-  Users: ``cn=users,cn=accounts,$SUFFIX``
-  Groups: ``cn=groups,cn=accounts,$SUFFIX``



System Accounts
===============

There are some LDAP clients that need a pre-configured account. Some
examples are the LDAP autofs client and sudo. Using a user's credentials
is generally preferable to creating a shared system account but that is
not always possible. Do **not** use the Directory Manager account to
authenticate remote services to the IPA LDAP server. Use a system
account, created like this:.

::

   # ldapmodify -x -D 'cn=Directory Manager' -W
   dn: uid=system,cn=sysaccounts,cn=etc,dc=example,dc=com
   changetype: add
   objectclass: account
   objectclass: simplesecurityobject
   uid: system
   userPassword: secret123
   passwordExpirationTime: 20380119031407Z
   nsIdleTimeout: 0
   <blank line>
   ^D

Be sure to change the password to something more secure, and the uid to
something reasonable.

The reason to use an account like this rather than creating a normal
user account in IPA and using that is that the system account exists
only for binding to LDAP. It is not a real POSIX user, can't log into
any systems and doesn't own any files.

This use also has no special rights and is unable to write any data in
the IPA LDAP server, only read.

Note: IPA 4.0 is going to change the default stance on data from nearly
everything is readable to nothing is readable, by default. You will
eventually need to add some Access Control Instructions (ACI's) to grant
read access to the parts of the LDAP tree you will need.



Handy helper for system accounts management
-------------------------------------------

Noah Bliss
`created <https://lists.fedorahosted.org/archives/list/freeipa-devel@lists.fedorahosted.org/message/AI4WSAMPKF4OSV6DFMKKTDEK4P7Y33SF/>`__
a shell helper to manage system accounts:
`freeipa-sam <https://github.com/noahbliss/freeipa-sam>`__.

SSL/startTLS
============

When possible, configure your LDAP client to communicate over SSL/TLS.
You can either use port 389 and enable startTLS in the client or
configure to use the ldaps port, 636. The IPA CA certificate can be
found in /etc/ipa/ca.crt on all enrolled hosts.



Tool configuration
==================

Since IPA 3.0 we've configured /etc/openldap/ldap.conf with some bare
defaults:

::

   URI ldaps://ipaserver.example.com
   BASE dc=example,dc=com
   TLS_CACERT /etc/ipa/ca.crt

Setting these defaults means you don't need to pass as many options to
tools like ldapsearch.

So you can do this:

``$ ldapsearch -x uid=admin``

Rather than:

``$ ldapsearch -x -h ipa.example.com  -b dc=example,dc=com uid=admin``



Unix clients
============

For specific information on configuring Unix clients to authenticate
against IPA, see `ConfiguringUnixClients <ConfiguringUnixClients>`__

As a general rule, we recommend using RFC 2307bis when possible. If this
is not possible, we provide a compatibility layer that provides the same
information in an RFC 2307-compatible way. The only change you need to
make is to set the ``basedn`` to ``cn=compat,$SUFFIX``