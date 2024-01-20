AD_User_Short_Names
===================

Overview
--------

Currently the ability of FreeIPA/SSSD to resolve and authenticate AD
users by their short names is quite limited. While
``default_domain_suffix`` option in ``sssd.conf`` can be used to resolve
short names incoming from single AD domain, it quickly becomes unusable
if the same functionality is desired for users from multiple trusted
forests. Moreover, being a local setting, each node has to be configured
manually which does not scale in larger deployments.

In order to further improve the integration with Active
Directory-managed deployments, this design proposes a centrally managed
mechanism that can be used by clients (SSSD) to resolve short names to
fully-qualified identifiers, use them to authenticate users and cache
the results.



Use Cases
---------

-  FreeIPA Administrator manages access to Unix resources for AD users
   coming from multiple forests/domains. In order to improve UX for
   these users he wishes to have a mechanism that permits them to log
   into FreeIPA-enrolled machines using their short name only

-  The administrator also wishes to have the ability to manually set the
   trusted forest/domain priority during user lookup so that the AD
   forest whose users are accessing IPA-managed machines most often is
   resolved first thus increasing performance

-  The administrator also wishes to manage the domain list centrally on
   FreeIPA masters. The changes should be immediately seen and consumed
   by all SSSD clients in the topology which have the capability to
   resolve AD users by short name.

-  the administrator wishes to have both a global settings and a local
   overrides applicable to selected hosts/hostgroups when a more
   fine-grained short name resolution is required

Design
------

NOTE: This design concerns mostly the server-side implementation and
management of data consumable by SSSD client.

The ordered list of domains used for resolution of user short name to
the full-qualified form will be referred to as domain resolution order
in the rest of this document.



Domain resolution order storage and interpretation
----------------------------------------------------------------------------------------------

The domain resolution order will be stored as a part of Global
configuration object. If the attribute exists and is not empty ( i.e.
contains at least one domain) the client shall extract the ordered list
of domains from it and used them to resolve an incoming short name
iteratively until a matching user is found. If the short name occurs in
multiple forests/domains (e.g. Administrator), the first matching entry
is used during authentication. If such behavior is undesired, the
affected user must specify his fully-qualified name upon login.

Any domain which is not contained in domain resolution order will not be
used in short name resolution. Any user belonging to such domain has to
use his fully-qualified name to login to IPA domain.

By the extension of the previous rule, empty domain resolution order can
imply that all users (even those in FreeIPA domain) have to use
fully-qualified names to log into enrolled machines. The exact
interpretation of these rule can, however, be left on the client
application.



Global vs. local domain resolution order settings
----------------------------------------------------------------------------------------------

There will be a means to manage a global domain resolution order
applicable for all enrolled hosts in the IPA domain. The global settings
can be overriden for selected hosts/hostgroups by attaching a local
configuration to a specific ID view object. The override can be created
also when no global definition is set so that the feature can first be
tested on selected hostgroup and then applied globally when desired.



Domain resolution order management
----------------------------------------------------------------------------------------------

The feature will provide a simple API for the management of the domain
resolution order. More sophisticated API may be provided later if demand
for better user experience arises. For example, the following high-level
operations may be considered in the future:

-  prepending and appending selected domains to the list
-  inserting the domains before and after other domain which is already
   in the list
-  moving the domains at the beginning and the end of the list
-  moving the domains before another domain already in the list
-  removing domain names present in the list
-  refreshing the domain list: this operation will check whether all
   domains present are active or exist in the trust objects and remove
   stale entries

Additionally, the commands that retrieve trusted domain information
(``trust-fetch-domains``) and manipulate trusted domains
(``trustdomain-disable``, ``trustdomain-del``) may query domain
resolution order and remove stale domains from it when the trusted
domain was removed or disabled.



Domain resolution order validation
----------------------------------------------------------------------------------------------

Finally, any modification of the domain resolution order must ensure
that each of the specified domain names corresponds either to that of
FreeIPA domain or to one of the trusted AD domains stored in LDAP
backend. In the case of trusted domains, the domain must not be marked
as disabled.

Implementation
--------------

The LDAP schema of the global config object
(``cn=ipaConfig,cn=etc,$SUFFIX``) and ID view objects will be extended
by a new objectclass providing the attribute which stores the domain
resolution:

| ``   objectclasses: ( 2.16.840.1.113730.3.8.12.39  NAME 'ipaNameResolutionData' DESC 'Data used to resolve short names``
| ``   to fully-qualified form' SUP top AUXILIARY MAY ( ipaDomainResolutionOrder ) X-ORIGIN 'IPA v4.5' )``

| ``   attributeTypes: ( 2.16.840.1.113730.3.8.11.77 NAME 'ipaDomainResolutionOrder' DESC 'List of domains used to resolve ``
| ``   a short name' EQUALITY caseIgnoreIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 SINGLE-VALUE X-ORIGIN 'IPA v4.5' )``

The value of the ``ipaDomainResolutionOrder`` consist of a
colon-separated list of domain names (e.g
``domain1.test:ad.domain2.test:ad.domain3.test``. The client is
responsible to retrieve and parse this value in order to decode the list
of domains it should consult for short name resolution.



Feature Management
------------------

UI

The WebUI should provide a way to view and manipulate the domain
resolution order under the ``Configuration`` tab.

CLI

The ``config`` object will gain ``domain-resolution-order`` option which
contains the raw attribute value which can be displayed by
``config-show`` and (re-)set by ``config-mod``.

Higher level commands may be considered in the future if there is a
demand for better user experience.

Configuration
----------------------------------------------------------------------------------------------

The feature is considered disabled if the domain resolution order is
absent in the configuration and applied ID view. In this case the client
shall retain the default behavior when handling incoming users.

If the domain resolution order (or its override) is present and empty,
then the client may either keep the default behavior or force all users
to use fully qualified names to access resources in FreeIPA domain.

Upgrade
-------

Upon upgrade the LDAP schema will be updated and the ipaConfig object
will be augmented by the new objectclass.

Since updating the objectclasses of all ID View objects can potentially
be costly, the existing ID views will be updated on-demand by the
framework code. ID views created after the upgrade will include the new
objectclass automatically.

The feature is considered backwards compatible since the old client
which do not understand domain resolution order will simply ignore it
and keep original behavior.



How to Use
----------

Consider the following scenario as an example:

FreeIPA domain 'ipa.test' is trusting a forest 'ad.forest.test' which
has two child domains ('child1.ad.forest.test',
'child2.ad.forest.test').

To allow users from both FreeIPA domain and from the trusted AD forest
log in using short name, we may do the following:



Example 1: Create a global resolution order
----------------------------------------------------------------------------------------------

just directly set the value of ``--domain-resolution-order`` attribute
to the desired value:

::

   $ ipa config-mod --domain-resolution-order='ipa.test:ad.forest.test:child1.ad.forest.test:child2.ad.forest.test'
     Maximum username length: 32
     Home directory base: /home
     ...
     Domain Resolution Order: ipa.test:ad.forest.test:child1.ad.forest.test:child2.ad.forest.test
     ...



Example 1 more conductive to automation
----------------------------------------------------------------------------------------------

-  store to FreeIPA domain name in the temporary file which will store
   the entries of interest:

::

   $ ipa env domain | awk '{print $2}' > domain_list.txt 

-  append the list of trusted domains to the file:

::

    $ ipa trustdomain-find ad.forest.test --pkey-only --raw | grep 'cn:' | awk '{ print $2}' >> domain_list.txt

NOTE: if you wish the AD users to be resolved first you can just reverse
the order of operations.

-  now set the ``domain-resolution-order`` attribute value:

::

   $ ipa config-mod --domain-resolution-order=$(cat domain_list.txt | tr '\n ':')
     Maximum username length: 32
     Home directory base: /home
     ...
     Domain Resolution Order: ipa.test:ad.forest.test:child1.ad.forest.test:child2.ad.forest.test
     ...



Example 2: creating local override of global resolution order
----------------------------------------------------------------------------------------------

Let's say that we have a machine named 'special.ipa.test' and we wish
that just users coming from the child domains of trusted forest
('child1.ad.forest.test', 'child2.ad.forest.test'). Since we observe
much more logins from the latter than from the former, we wish to have
this one tried out first when resolving short names.

-  first we create an ID view which will hold the modified resolution
   order:

::

   $ ipa idview-add special_host_view --desc 'ID view for custom shortname resolution on special hosts' --domain-resolution-order 'child2.ad.forest:test:child1.ad.forest'
   ---------------------------------
   Added ID View "special_host_view"
   ---------------------------------
     ID View Name: special_host_view
     Description: ID view for custom shortname resolution on special hosts
     Domain Resolution Order: child2.ad.forest:test:child1.ad.forest

-  then we apply the view on the host

::

   $ ipa idview-apply special_host_view --hosts special.ipa.test
   -----------------------------------
   Applied ID View "special_host_view"
   -----------------------------------
     hosts: special.ipa.test
   ---------------------------------------------
   Number of hosts the ID View was applied to: 1
   ---------------------------------------------



Test Plan
---------

Test scenarios that will be transformed to test cases for FreeIPA
`Continuous Integration <V3/Integration_testing>`__ during
implementation or review phase. This can be also link to `source in
cgit <https://git.fedorahosted.org/cgit/freeipa.git/>`__ with the test,
if appropriate.