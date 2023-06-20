One-way_trust
=============

Overview
========

Active Directory implementation of a trust between domains and forests
uses credentials of a trust domain object (TDO) to communicate across
the trust boundary. This is made possible on AD side because whole
domain controller implementation is seen as a monolith that doesn't pass
around the credentials for the trust domain object. This is purely
implementation detail though important one.

In early stages of a trust feature development FreeIPA also used trust
domain object to directly authenticate against Active Directory
services. However, as IPA is a combination of several loosely coupled
services, access to the trust domain object is highly guarded to prevent
unwanted elevation of privileges across the trust boundary. If FreeIPA
was to use TDO's credentials everywhere, it would mean most of
trust-related functionality would be limited to IPA admins or TDO object
in LDAP would have to be more accessible. Given that TDO credentials can
be used to compromise access to our domain, it is not advisable to give
a wider access to them.

As a side-effect of reducing exposure of TDO credentials, FreeIPA lost
ability to establish and use one-way trust to Active Directory. The
purpose of this feature is to regain the one-way trust support, yet
without giving an elevated access to TDO credentials.



Use cases
=========

A primary use case is the following one:

-  One-way trust to Active Directory where FreeIPA realm trusts Active
   Directory forest using cross-forest trust feature of AD but the AD
   forest does not trust FreeIPA realm. Users from AD forest can access
   resources in FreeIPA realm.

No other use cases exist at the moment.

Design
======

The one-way feature relies on an implementation of FreeIPA trust to AD
feature as released in FreeIPA v3.3. The difference between FreeIPA v3.3
and v3.0 is in the way how credentials to access information from a
trusted forest are used.



FreeIPA v3.0 and v3.3
---------------------

In FreeIPA v3.0 each IPA master initialized with ipa-adtrust-install
command was running Samba suite: smbd and winbindd daemons were used to
provide both capabilities to resolve AD users from trusted forests, to
manage trust forest topology, and to respond on NETLOGON interfaces as
Active Directory Domain Controllers expect to complete the sequence of
establishing trust relationships. The rest of clients in FreeIPA were
connecting to IPA masters through SSSD by means of an extended LDAP
control to resolve AD users and groups. FreeIPA LDAP server's plugin
which implemented the extended LDAP control was, in turn, talking to
winbindd daemon to complete the resolution of AD users and groups.

Additionally, in early FreeIPA v3.0 versions a management framework
(both CLI and web UI) was using credentials of TDO to directly resolve
AD users and groups against Active Directory Domain Controllers. The
consequence of this was that only IPA admins were able to map users and
groups from trusted Active Directory forests to local groups.

The trust by AD DCs means that FreeIPA framework can utilize existing
Kerberos service ticket it has (HTTP/ipa.master@IPA.REALM) to
authenticate to AD LDAP servers. AD LDAP servers allow access to its
information only to authenticated clients but the clients can provide
any proof of authenticity allowed by Active Directory. In the case of
cross-forest trust in AD, a properly issued Kerberos ticket from a
trusted forest is enough. In order to issue such ticket, FreeIPA KDC
does generate privilege attribute certificate data (MS-PAC) as required
by Microsoft's specification
`[MS-PAC <https://msdn.microsoft.com/en-us/library/cc237917.aspx>`__].
In order to limit which Kerberos services are allowed to authenticate
against services in a trusting AD forest, only
HTTP/ipa.master@IPA.REALM, cifs/ipa.master@IPA.REALM, and
host/ipa.master@IPA.REALM are given the MS-PAC in their TGT tickets
where the services are presented as members of a virtual Domain
Controllers group in FreeIPA domain.

FreeIPA v3.0 management framework was switched to use
HTTP/ipa.master@IPA.REALM Kerberos ticket with attached MS-PAC
information to directly resolve AD users and groups.

In FreeIPA v3.3 each IPA master initialized with ipa-adtrust-install
command still runs Samba suite: smbd and winbindd daemons. They are used
to respond on NETLOGON interfaces as Active Directory Domain Controllers
expect them, and to manage trust forest topology. However, users and
groups from trusted Active Directory forests are now resolved by SSSD
running on the IPA masters. SSSD has gained a so-called "IPA server
mode" which means the requests to resolve AD users and groups will go
directly to Active Directory Domain Controllers. The rest of clients in
FreeIPA are connecting to IPA masters through SSSD by means of the same
extended LDAP control to resolve AD users and groups. However, FreeIPA
LDAP server's plugin which implements the extended LDAP control now
talks to SSSD on the IPA master instead of winbindd daemon to complete
the resolution of AD users and groups. Finally, a management framework
also relies on the SSSD on IPA master to resolve users and groups from
trusted Active Directory forests. As a consequence of that, IPA admins
can delegate rights to map AD users and groups without giving access to
TDO credentials anymore.

In its own turn, SSSD 1.11 as used by FreeIPA v3.3, relies on the fact
that Active Directory Domain Controllers do trust FreeIPA realm and uses
host/ipa.master@IPA.REALM Kerberos ticket to directly resolve AD users
and groups.



Security of two-way trust solution
----------------------------------------------------------------------------------------------

It should be noted that existing two-way trust solution as implemented
by IPA is not giving any additional rights compared to one-way trust as
implemented by Active Directory. In fact, it is as secure, as one-way
trust on AD side, thanks to cross-forest trust SID filtering defaults.

While the second leg of the trust is used to provide SSSD ways to look
up users and groups in AD LDAP, thanks to ability to authenticate with
cross-realm ticket, this does not allow to elevate permissions of IPA
users and services in their access to AD resources. The only rights they
get are the ones given by AD administrators by default to 'all
authenticated users". An attempt to elevate rights would require IPA to
provide Global Catalog so that AD DCs would be able to resolve IPA users
and groups SIDs to their names. As IPA has no Global Catalog service, it
is not possible to assign additional rights to IPA users and groups
access the Active Directory resources, like logging into the Windows
machines or change any files other than those assigned for "all
authenticated users".

Note also, that with cross-forest trust, there are explicit SID filters
set up by both Active Directory and IPA. These filters disallow IPA
users and services to have MS-PAC record which includes SIDs from the
Active Directory domains. Even if an attacker would fake MS-PAC record
(to do so, it needs to completely own IPA KDC), it would not be able to
add the SIDs from Active Directory domains to the MS-PAC -- such MS-PAC
will be refused by Active Directory Domain Controllers at the forest
boundary.

Final consequence of v3.0 and v3.3 trust feature designs was the fact
that if IPA clients needed to resolve AD users and groups, they needed
to be enrolled to an IPA master which has been initialized with
ipa-adtrust-install. As a result, in major deployments all of IPA
masters had to run smbd and winbindd processes. While this is an
improvement over requirement to run winbindd to every single client as
with some other solutions, it is still too fragile for production. This
shortcoming is addressed by `V4/Trust agents <V4/Trust_agents>`__
feature design.



New design
----------

In order to support one-way trust to Active Directory, we need to switch
SSSD in IPA master mode to use TDO credentials when resolving AD users
and groups. This is a high level description of the design, and majority
of work to allow the switch will be done by SSSD team. Corresponding
ticket tracker on SSSD side is `ticket
2579 <https://fedorahosted.org/sssd/ticket/2579>`__, the text below is
an overview of the design.

On each IPA master SSSD runs in "IPA master mode". This mode means that
in case of existing trust to AD forest, SSSD will directly resolve AD
users and groups against Active Directory Domain Controllers. To perform
user/group resolution, SSSD needs to authenticate against AD LDAP
servers and it does so using Kerberos authentication based on a
host/ipa.master@IPA.REALM service ticket. The ticket towards AD LDAP
services is issued by FreeIPA KDC with the help of cross-realm trust
credentials.

For one-way trust SSSD cannot use this approach because Active Directory
Domain Controllers do not trust FreeIPA realm and, therefore, no
cross-realm trust credentials exist in AD for FreeIPA realm. However,
SSSD can use TDO object which always exists in AD for the trusting
domain (cross-forest trust is done by forest root domains' trust). This
means the ticket SSSD would need to request belongs to a different realm
(AD forest root realm) rather than to FreeIPA realm.

As FreeIPA supports multiple trusts to separate Active Directory
forests, a support for multiple separate tickets is required. SSSD will
need to gain ability to use different credentials caches to store TDO
tickets and use different keytabs with TDO credentials to obtain the
ticket from an Active Directory Domain Controllers.

In order to separate privilege access, FreeIPA masters have to provide
keytabs for SSSD running on IPA masters, one keytab per trusted AD
forest, so that SSSD could request the keys when required.

Additionally, FreeIPA management framework will need to change its
defaults from producing a two-way trust to a one-way trust. Two-way
trust will be added back when support for Global Catalog service will be
added so that Active Directory resources could be properly accessed and
access to them discretionally granted to FreeIPA users and groups.

Implementation
==============

Following changes will need to be done on FreeIPA side in order to
support one-way trust:

#. Switch two-way trust creation in ipaserver/dcerpc.py to one-way by
   default.

   #. The code needs to be changed to allow specifying either one- or
      two-way trust and should manipulate trust_direction property (by
      setting lsa.LSA_TRUST_DIRECTION_OUTBOUND or a combination of
      lsa.LSA_TRUST_DIRECTION_INBOUND and
      lsa.LSA_TRUST_DIRECTION_OUTBOUND) in
      TrustDomainInstance.establish_trust() method.
   #. One-way trust can be created with full AD administrator
      credentials too, while shared secret method will rely on the AD
      administrator creating the remote part of it in AD.

#. Make sure ipalib/plugins/trust.py passes properly a flag to enable
   two-way trust.
#. Change ipasam to create additional principal named IPA$@AD.REALM form
   when creating TDO object for AD.REALM forest trust. This principal
   has to be disabled so that KDC cannot use it to issue tickets.
#. Swtich IPA framework to perform out of band DBus call to external
   script that would use TDO credentials to populate information about
   trusted domains



Details about oddjobd-triggered script
--------------------------------------

Access to trusted domain object (TDO) is highly regulated (by us)
because possession of the TDO credentials impersonates whole trust link.
Thus, we want to avoid authenticating as TDO within Apache process.

To achieve this a scheme similar to oddjob-mkhomedir is used, by
providing a helper script which is executed by oddjobd on request from
Apache:

Apache process sends DBus request to oddjobd daemon. Oddjobd daemon
executes an IPA helper. IPA helper accesses /etc/samba/samba.keytab and
authenticates as cifs/ipa.master@IPA.REALM. It then fetches TDO
credentials from IPA LDAP and authenticates with them to AD DC. Once
operation is performed, it connects again to IPA LDAP and updates it.

There are several moving parts here:

#. /etc/samba/samba.keytab is root:root, 0600,
   unconfined_u:object_r:samba_etc_t:s0. It is created by
   /usr/sbin/ipa-adtrust-install
#. /var/lib/sss/keytabs/ad.test.keytab is sssd:sssd (or root:root),
   0600, unconfined_u:object_r:sssd_var_lib_t:s0. It can be created by
   IPA helper or by SSSD, whoever runs into need of the keytab first.
   The name is dependent on the AD forest root name (ad.test in for
   example). The ownership of the keytabs depends on the way SSSD runs
   --- either privileged (as root) or non-privileged (as sssd user).
#. /usr/libexec/ipa/com.redhat.idm.trust-fetch-domains is root:root,
   0755, system_u:object_r:ipa_helper_exec_t:s0 label. It is the IPA
   helper oddjobd daemon will be calling in response to Apache request.
   The helper is written in Python.
#. /var/run/ipa/krb5cc_oddjob_trusts{,_fetch} -- credential caches used
   by the helper. They are root:root, 0600,
   system_u:object_r:ipa_var_run_t:s0 label.
#. oddjobd daemon runs under system_u:system_r:oddjob_t:s0-s0:c0.c1023
   context.



Feature management
==================

CLI

The newly created trust will become one-way only, no additional options
will be needed. To force creating a two-way trust, pass **--two-way**
option to **ipa trust-add**.

**ipa-adtrust-install** utility gained **--add-agents** option. This
option adds IPA masters to the list that allows to serve information
about users from trusted forests. Starting with FreeIPA 4.2, a regular
IPA master can provide this information to SSSD clients. IPA masters
aren't added to the list automatically as restart of the LDAP service on
each of them is required. The host where **ipa-adtrust-install** is
being run is added automatically.

Note that IPA masters where ipa-adtrust-install wasn't run, can serve
information about users from trusted forests only if they are enabled
via ipa-adtrust-install run on any other IPA master. At least SSSD
version 1.13 on IPA master is required to be able to perform as a trust
agent.



Web UI----

No changes in Web UI are required if we wouldn't expose two-way trust
option.

Replication
-----------

Trust-related information is in the replicated subtree already.

Upgrades
--------

On upgrade sidgen and extdom plugins get enabled by default on all IPA
masters to help with AD trust agent mode.

SSSD will use ipa-getkeytab to obtain the keytabs if keytab is missing.



Test Plan
---------

http://www.freeipa.org/page/V4/One-way_trust/Test_Plan