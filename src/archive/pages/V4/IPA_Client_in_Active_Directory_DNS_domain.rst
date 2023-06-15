Overview
--------

In the ideal world, FreeIPA clients should be deployed in DNS zones
owned by FreeIPA. However, in many environments where FreeIPA is being
deployed, Active Directory is the dominant identity management solution
owning not only the identities, but also the DNS domains. Currently, the
only solution how to migrate a Linux client in such AD owned DNS domain
to FreeIPA was to move it to FreeIPA owned domain. While this procedure
works well when migrating 10 client systems, it is less desirable when
migrating 10k client systems.



Use Cases
---------



User Story
~~~~~~~~~~

As an Administrator with a big number of Linux machines in a DNS domain
controlled by Active Directory, I want to join them to the IdM Server so
that they can benefit from it’s Linux focused features.

Details
~~~~~~~

Allow FreeIPA client to respond a host name in a DNS domain belonging to
a domain from a trusted Active Directory forest.

If Active Directory forest example.com uses DNS zone *example.com*, and
FreeIPA is deployed at *ipa.example.com*, then it is desirable to have
some FreeIPA machines accessible as machine-foo.example.com, not just
*machine-foo.ipa.example.com*.

In many cases FreeIPA client machines are used as servers for hosting
applications in the same DNS name space as existing Active Directory
environment. While Active Directory enforces ownership of resources (DNS
domain is owned by corresponding Active Directory domain) and FreeIPA
cannot be part of the Active Directory forest by itself, it should be
possible to have a DNS host name for FreeIPA client as a part of a DNS
domain of existing Active Directory domain and still allow single
sign-on operations.

.. _theory_and_practice_of_a_forest_trust_interaction:

Theory and practice of a forest trust interaction
-------------------------------------------------

There are several concepts needs to be understood for the setup when
FreeIPA machine is a part of the DNS zone belonging to the domain of
Active Directory forest:

-  Active Directory has a concept of relationship between the domain and
   DNS zones. An Active Directory domain *owns* DNS zone of the same
   name and no other Active Directory domain may claim the same DNS
   zone.

-  When forest trust is established between FreeIPA and an Active
   Directory forest, Active Directory Domain Controller enforces
   non-conflict check of the DNS name spaces claimed by FreeIPA. If
   there is any conflict between what FreeIPA claims to own and what the
   Active Directory Domain Controller knows to belong to any of the
   Active Directory domain in the same forest, a link between FreeIPA
   and AD would be disabled and no authentication would be possible
   across the trust link.

-  FreeIPA automatically adds DNS domains it manages to the list of DNS
   domains associated with FreeIPA realm. This list is then presented to
   Active Directory when trust is established to allow proper routing of
   authentication requests when talking to servers in these DNS domains.

As a consequence, Active Directory will never refer authentication
request to FreeIPA domain controller for a Kerberos service principal on
a host within DNS zone owned by Active Directory. This means no Kerberos
authentication is possible against such FreeIPA machines from Windows
systems.

.. _a_subdomain_of_ad_that_only_has_ipa_machines:

A subdomain of AD that only has IPA machines
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

What if there is a subdomain of Active Directory that only contains
Linux machines enrolled into FreeIPA realm? In such case Active
Directory needs to be told that this DNS domain belongs to FreeIPA. This
is done with the help of \`ipa realmdomains-mod
--add-domain=sub.ad.domain\` and re-establishing trust to AD.

.. _enrolling_to_freeipa_realm:

Enrolling to FreeIPA realm
~~~~~~~~~~~~~~~~~~~~~~~~~~

When FreeIPA client *ipa-client.ipa.domain* is enrolled into FreeIPA
realm, following is done:

#. Host object *ipa-client.ipa.domain* is created in FreeIPA to hold
   references to the new FreeIPA client
#. Kerberos principal is created based on the host object,
   *host/ipa-client.ipa.domain@IPA.DOMAIN*
#. Keys for this principal retrieved and stored in */etc/krb5.keytab* on
   the FreeIPA client
#. Kerberos configuration is added to */etc/krb5.conf* to refer
   *IPA.DOMAIN* Kerberos realm to FreeIPA KDC and map *ipa.domain* DNS
   zone to *IPA.DOMAIN* Kerberos realm
#. SSSD daemon on FreeIPA will attempt to update DNS record for
   *ipa-client.ipa.domain* using the host Kerberos principal created
   above.

Host object in FreeIPA is decoupled from the corresponding DNS record.
Creating the host object with host name from non-FreeIPA DNS zone does
not cause adding that DNS zone to the list of DNS zones associated with
FreeIPA realm.

The way how Kerberos configuration for FreeIPA realm in */etc/krb5.conf*
is added depends on the way how *ipa-client-install* tool discovered the
FreeIPA realm. When automated discovery via DNS SRV records was
successful, no explicit configuration for the FreeIPA realm would be
added. Instead, DNS-based realm and KDC discovery will be activated.

If FreeIPA client is placed in a DNS domain of Active Directory,
automated discovery would be run against Active Directory's DNS domain
and FreeIPA would not be discovered. To aid the proper discovery,
*--domain* option has to be passed to *ipa-client-install* tool.

A mapping between DNS domain and Kerberos realm is created where the
FreeIPA client's domain is explicitly mapped to point to the FreeIPA
realm.

To sum up, for FreeIPA realm IPA.EXAMPLE.COM and Active Directory
EXAMPLE.COM, when FreeIPA client has host name ipa-client.example.com,
following configuration would be written if *ipa-client-install --domain
ipa.example.com* was used to configure the FreeIPA client (abbreviated):

/etc/krb5.conf

| ``   [libdefaults]``
| ``   dns_lookup_realm = true``
| ``   dns_lookup_kdc = true``
| ``   ``
| ``   [realms]``
| ``   IPA.EXAMPLE.COM = {``
| ``       pkinit_anchors = ``\ ```FILE:/etc/ipa/ca.crt`` <FILE:/etc/ipa/ca.crt>`__
| ``   }``
| ``   ``
| ``   [domain_realm]``
| ``   .ipa.example.com = IPA.EXAMPLE.COM``
| ``   ipa.example.com = IPA.EXAMPLE.COM``
| ``   .example.com = IPA.EXAMPLE.COM``
| ``   example.com = IPA.EXAMPLE.COM``

As can be seen above, look up for any service principal on the hosts in
DNS zone *example.com* will be forced to belong to realm
*IPA.EXAMPLE.COM*. This means the client will not be able correctly
communicate with services enrolled into Active Directory because all
Kerberos requests for *EXAMPLE.COM* realm would be instead sent to the
KDC of *IPA.EXAMPLE.COM*.

It is, however, possible to change

| ``   .example.com = IPA.EXAMPLE.COM``
| ``   example.com = IPA.EXAMPLE.COM``

to explicit configuration for the FreeIPA hostname:

``   ipa-client.example.com = IPA.EXAMPLE.COM``

and leave out any other explicit mapping for *.example.com* to have it
discovered via DNS SRV record lookups.

Note that the setup above will not allow machines from realm
*EXAMPLE.COM* to properly obtain a service ticket towards
*ipa-client.example.com* because they will be thinking
*ipa-client.example.com* belongs to realm *EXAMPLE.COM*. On Linux
machines it would be possible to extend *[domain_realm]* mapping the
same way to force a single machine to map to the right realm but in
Active Directory it is not possible to do so.

For Kerberos-based authentication and access to services running on
FreeIPA machines to work, two conditions must be satisfied:

#. Client A must be able to talk to the KDC of its own realm to request
   a service ticket to server B or a cross-realm TGT for realm of the
   server B and then request a service ticket to server B
#. Server B must be able to talk to the KDC of its own realm

Condition (1) is needed so that client A could present the service
ticket to the service running on the server B to mutually authenticate.
Condition (2) is needed for SSSD on server B to be able to transform an
incoming Kerberos principal identity to an identity understood by the
underlying POSIX environment.

As result, KDC of the client's realm must know either Kerberos principal
for a service on the server B, or should be able to issue a cross-realm
referral ticket to the KDC of the realm where the Kerberos principal is
located. In practice, this means that either server B is enrolled to
Active Directory domain, or it is enrolled to FreeIPA domain \_and\_ a
cross-forest trust is established between the FreeIPA and the Active
Directory forest root domain.

However, if server B is enrolled to the FreeIPA domain, its DNS host
name cannot be part of the *example.com* DNS zone because this is
prohibited by MS-ADTS specification, `section 6.1.6.9.3.2 "Building
Well-Formed msDS-TrustForestTrustInfo
Message" <https://msdn.microsoft.com/en-us/library/cc223787.aspx>`__. An
abridged version of these rules is available in MS-LSAD, `section
3.1.4.7.16.1 "Forest Trust Collision
Generation" <https://msdn.microsoft.com/en-us/library/cc234372.aspx>`__:

The rules for top-level name entries are as follows:

-  An enabled (that is, non-conflict) top-level name record must not be
   equal to an enabled top-level name for another trusted domain object
   or to any of the DNS tree names within the current forest. Equality
   is computed using case-insensitive string comparison. If the strings
   differ only by one trailing '.' character, the difference is ignored.
-  The top-level name must not be subordinate to an enabled top-level
   name for another trusted domain object, unless the other trusted
   domain object has a corresponding exclusion record.
-  A top-level name must not be superior to an enabled top-level name
   for another trusted domain object, unless the current trusted domain
   object has a corresponding exclusion record.

If any of these rules are violated, a top-level name is considered in
conflict.

The solution for Kerberos-based authentication and access to resources
in DNS zone owned by an Active Directory domain relies on the fact that
Kerberos libraries use a specific logic to discover actual service
principal for host- based services.

MIT Kerberos as an implementation of Kerberos protocol follow `these
rules <http://web.mit.edu/Kerberos/krb5-latest/doc/admin/princ_dns.html>`__:
MIT Kerberos clients currently always do forward resolution (looking up
the IPv4 and possibly IPv6 addresses using getaddrinfo()) of the
hostname part of a host-based service principal to canonicalize the
hostname. They obtain the “canonical” name of the host when doing so.

In practice this also means any CNAME record will be resolved to the
corresponding A/AAAA record and the result is then used to construct
host- based Kerberos principal (e.g. *nfs/ipa-client.example.com*).

The same logic is used by Active Directory:

-  If FreeIPA client is enrolled as *ipa-client.ipa.example.com* (A/AAA
   records set using this hostname) and
-  there is CNAME record *ipa-client.example.com* pointing to
   *ipa-client.ipa.example.com*,
-  then Windows client will attempt to request a Kerberos service ticket
   for a host-based service on the host *ipa-client.ipa.example.com*

As result, no machine with A/AAAA DNS record *ipa-client.example.com*
can operate properly with Kerberos in Active Directory while being part
of a Kerberos realm different to *EXAMPLE.COM* but a CNAME record
*ipa-client.example.com* can point to A/AAAA DNS record
*ipa-client.ipa.example.com* to allow Kerberos authentication.

.. _possible_solutions:

Possible solutions
------------------

Depending on what is required to achieve, there are two solutions
possible. In both cases we assume proper enrollment of the client to
FreeIPA by means of *ipa-client-install* tool which would set up SSSD
with 'ipa' identity provider.

.. _no_single_sign_on_required:

No single sign-on required
~~~~~~~~~~~~~~~~~~~~~~~~~~

When no single sign-on (Kerberos authentication) required, we still
should make sure Kerberos configuration is set up to allow SSSD to
communicate with FreeIPA masters.

FreeIPA client should be configured with *ipa-client-install
--domain=ipa.example.com* so that auto-detection of Active Directory
domain via SRV records in DNS domain *example.com* will not be done.

Kerberos configuration in */etc/krb5.conf* should be modified to add:

| ``   [domain_realm]``
| ``     ipa-client.example.com = IPA.EXAMPLE.COM``

This configuration change will ensure that the host itself is associated
with FreeIPA realm on this machine.

Only password-based logon will work for accessing resources on this
machine. Any Kerberos or GSSAPI based access will fail from both other
FreeIPA machines or Active Directory clients as long as originating
machines have no mapping in their Kerberos configuration for
*ipa-client.example.com* to *IPA.EXAMPLE.COM* realm. As described in the
previous sections, on Active Directory side it is not possible to add
such configuration.

If AD users logged in with password using SSH session or GNOME Desktop
manager, they might get valid Kerberos credentials in their credentials
cache. To use these credentials against any other Active
Directory-enrolled Windows resources one needs to remove Kerberos
domain-realm mapping that forces *.example.com* to be associated with
*IPA.EXAMPLE.COM* realm:

/etc/krb5.conf

| ``   [domain_realm]``
| ``   .ipa.example.com = IPA.EXAMPLE.COM``
| ``   ipa.example.com = IPA.EXAMPLE.COM``
| ``   .example.com = EXAMPLE.COM``
| ``   example.com = EXAMPLE.COM``

Once *.example.com* is associated with *EXAMPLE.COM* realm, actual
Kerberos credentials obtained on the FreeIPA client as part of the
OpenSSH logon can be used to authenticate against other Active Directory
resources.

.. _handling_of_ssl_certificates:

Handling of SSL certificates
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For SSL-based service protection (HTTPS, IMAPS, etc), a certificate with
dNSName extension records covering all system hostnames is required due
to the fact that both original (A/AAAA) and CNAME record names need to
be in the certificate.

Currently FreeIPA only issues certificates to host objects presenting in
FreeIPA database. For the case when single sign-on is not required, it
is assumed that the host *ipa-client.example.com* is enrolled into
FreeIPA realm.

This means there is already a host object for *ipa-client.example.com*
in FreeIPA and Certmonger can already request for the certificate in its
name:

| ``   ipa-getcert request -r \``
| ``      -f /etc/httpd/alias/server.crt \``
| ``      -k /etc/httpd/alias/server.key \``
| :literal:`      -N CN=`hostname --fqdn` \\`
| :literal:`      -D `hostname --fqdn` \\`
| ``      -K host/ipa-client.example.com@IPA.EXAMPLE.COM \``
| ``      -U id-kp-serverAuth``
| ``   ``

This example allows to request an SSL certificate from FreeIPA CA to
store it in *server.crt* (public key) and *server.key* (private key)
files.

Certmonger uses default host key stored in */etc/krb5.keytab* to
authenticate against FreeIPA CA. This means Kerberos authentication
against *IPA.EXAMPLE.COM* realm should be properly working which is why
*ipa-client.example.com = IPA.EXAMPLE.COM* was added to *[domain_realm]*
mapping in */etc/krb5.conf* above.

.. _single_sign_on_required:

Single sign-on required
~~~~~~~~~~~~~~~~~~~~~~~

When single sign-on is required, moving FreeIPA client outside DNS zone
*example.com* is the pre-requisite. A CNAME record
*ipa-client.example.com* can then be created to point to the A/AAAA
record of the FreeIPA client. E.g., *ipa-client.ipa.example.com*.

For Kerberos-based application servers MIT Kerberos supports a method to
allow accept any host-based principal available in the application's
keytab. When Kerberos client would connect to a Kerberos application
server, such server typically does strict check on what Kerberos
principal was used to target it (so-called, 'acceptor check'). This can
be relaxed:

| ``   [libdefaults]``
| ``    ignore_acceptor_hostname = true``

For OpenSSH server there is a specific option *GSSAPIStrictAcceptorCheck
no* to achieve the same.

.. _handling_of_ssl_certificates_1:

Handling of SSL certificates
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For SSL-based service protection (HTTPS, IMAPS, etc), a certificate with
dNSName extension records covering all system hostnames is required due
to the fact that both original (A/AAAA) and CNAME record names need to
be in the certificate.

Currently FreeIPA only issues certificates to host objects presenting in
FreeIPA database. This means one would need to create host object for
*ipa-client.example.com* in FreeIPA and make sure the real FreeIPA
machine's host object is able to manage this host:

| ``   ipa host-add ipa-client.example.com --force``
| ``   ipa host-add-managedby ipa-client.example.com --hosts=ipa-client.ipa.example.com``

We have to use *--force* option here because *ipa-client.example.com* is
a CNAME, not an A/AAAA DNS record as required by FreeIPA.

With this setup *ipa-client.ipa.example.com* would be able to request an
SSL certificate with dNSName extension record for
*ipa-client.example.com*.

| ``  ipa-getcert request -r \``
| ``      -f /etc/httpd/alias/server.crt \``
| ``      -k /etc/httpd/alias/server.key \``
| :literal:`      -N CN=`hostname --fqdn` \\`
| :literal:`      -D `hostname --fqdn` \\`
| ``      -D ipa-client.example.com \``
| ``      -K host/ipa-client.ipa.example.com@IPA.EXAMPLE.COM \``
| ``      -U id-kp-serverAuth``
