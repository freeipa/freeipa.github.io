Requirements
============

Kerberos as authentication mechanism allows authenticating different
kinds of principals: users, host, services etc. As a result of the
authentication a so called TGT (ticket granting ticket) is issued. This
ticket is used to obtain tickets to contact other services, so that they
do not need to contact the central server to confirm authentication. The
TGT is valid for a limited period of time. When the TGT expires a new
one need to issued. If allowed by the TGT and requested on issue, the
ticket can also be renewed before it is expired. The ticket cannot be
renewd past the maximum renewal time. This page deals with acquiring
and/or renewing TGTs.

.. _host_ticket:

Host Ticket
-----------

In IPA v2 every host enrolled with IPA will have a kerberos keytab that
will be provisioned to the machine during the machine enrollment. This
keytab will be used to authenticate the host when the host itself needs
to perform an operation against the IPA server or any other client in
the network.

Because the keytab contains the secret it is always possible to acquire
a new TGT at any time. If a TGT is expired a new one will be requested
before performing an operation that requires exchange of kerberos
credentials.

.. _user_ticket:

User Ticket
-----------

Users logging into the IPA enrolled client are going to use a PAM module
that will proxy requests down to the IPA Data Provider. The result will
be the generation of a credential cache for the user containing the User
TGT.

.. _credentials_expiration:

Credentials Expiration
~~~~~~~~~~~~~~~~~~~~~~

Sometimes the user may need to run long jobs that take hours to
complete. If the job tries to access other kerberos enabled services
like NFS on behalf of the user, access will be denied if the Ticket is
expired, and the job may abort. This situation is causes a lot of
problems in the customer environment and is one of the show stoppers for
broader kerberos adoption.

The requirement to solve this problem was added to the IPA PRD.

.. _other_considerations:

Other Considerations
--------------------

There several important considerations that should be taken into the
account:

-  The solution should not jeopardize security
-  The solution should not require end user intervention
-  The user ticket should not be renewed by default, but only if
   explicitly configured to do so.

Design
======

There are several different approaches that address the requirements.

.. _kerberos_renewal_approach:

Kerberos Renewal Approach
-------------------------

The Kerberos protocol allows to renew a ticket if it is marked as
renewable (and original ticket was requested as renewable). The TGTs in
addition to the “renewable” flag has a max renew time – when asking for
renewals the expiration time will not be set beyond this time limit.
Renewal must occur before the ticket expires. Potentially this can be
defined on per users bases and stored, per principal, in LDAP using this
objectclass.

| ``  objectClasses: ( 2.16.840.1.113719.1.301.6.16.1 ``
| ``  NAME 'krbTicketPolicyAux' ``
| ``  AUXILIARY ``
| ``  MAY ( krbTicketFlags $ krbMaxTicketLife $ krbMaxRenewableAge ) ``
| ``  X-ORIGIN 'user defined' ) ``

| Currently in IPA v2 there is no UI to add this information, but it can
  be manipulated using LDAP operations.
| It would be possible to leverage this class to set per principal
  defaults, to allow or disallow renewals on a per principal basis. A
  possible scenario to manage these options is this:

-  Any kerberos principal entry would be populated with a
   krbTicketPolicyAux objectclass.
-  There would be a management interface that would allow bulk
   manipulation of kerberos ticket flags, ticket maximum time and
   renewable age.
-  There would be an option in the IPA client policy that will indicate
   that the clients should attempt to renew the tickets automatically.
   The IPA clients affected by this policy will attempt automatic
   renewal of the tickets.
-  The IPA data provider will have additional logic:

   -  If the client is configured to automatically renew tickets, a
      “check kerberos renewal” task will be created.
   -  The task will periodically generate an event and when that happens
      it will check if it is time to renew tickets.
   -  If a ticket is close to expiration it will try to renew it.
   -  Only ticket for logged in users are checked for, a list of
      references to the credential caches for each user is kept up to
      date based on login/logout events.
   -  The renewal must happen before the ticket is expired, it is
      usually wise to renew a ticket when half the time of validity is
      passed:

``     (now > releasetime + (expiretime - releasetime)/2)``

or:

``     (now > expiretime - configured_amount_of_time)``

The exact method can be configured by a specific policy.

If we implement this functionality there will be no need to capture and
store user passwords on the client (but will be limited to renewal
time). We can also make the IPA client policy to specify a group of
users for which the tickets should be renewed. In this case the handler
first will check if the user who owns a ticket is a member of a group.
It is pretty simple and doable. The only long pole in this solution is
management of the options and ages in the IPA UI. But may be it is not
that needed. The policy management effectively provides the capability
of specifying the set of hosts (by making a policy destined to these
hosts) and a user group (in the policy) for which the tickets need to be
automatically renewed. It might then be sufficient not to use explicit
attributes on a per user entry but rather configure the IPA data
provider to always request renewable tickets with renewal period a long
enough to complete nightly jobs or long "over the weekend jobs". With
the current configuration in v1 the renewal age is set to 14 days
maximum which should be more than enough for any unattended job in
normal environments. Another factor to keep in mind is that in normal
circumstances the user can obtain a completely new ticket simply
performing an authentication. A common method to force a new
authentication is to lock the machine and provide the user password to
the screensaver. This will generate a new PAM authentication which will
allow us to obtain a completely new TGT extending renewal time to 14
days from that point on.

.. _security_concerns:

Security Concerns
~~~~~~~~~~~~~~~~~

The only drawback of this solution is that an arbitrary program can be
developed to automatically renew the kerberos tickets bypassing IPA
policy checks since they are enforced on a client. But still this seems
to be a viable solution for v2 even considering this issue.

.. _alternative_approach:

Alternative Approach
--------------------

The alternative approach calls for caching the user passwords on the
client. During the authentication the user password will be captured and
stored locally on the client to allow offline authentication. Normally
the user password is hashed before being stored, so that it can be used
only to verify authentication, but not be used to impersonate the user
in case the machine is compromised. In case renewal of kerberos
credentials is more important than other security concerns, then the
clear text password can be stored in protected kernel memory. This way
it will be automatically cleared when the machine is shut off or
hibernated, and the system will never swap it to disk as the memory
pages will be locked. As an extra measure it may be reversibly encrypted
with an appropriate secret.

In this case the logic will be the same as the previous one. the only
differences being that the user passwords are captured and stored on the
client, and that renewal could happen even if the ticket is actually
expired as we posses the user secret.

.. _security_concerns_1:

Security Concerns
~~~~~~~~~~~~~~~~~

The main issue with this approach is the need to cache passwords on the
client, this is a quite relevant security concern. This approach allows
to overcome the problem of being able to renew credentials past the
renewal time and before a ticket is expired but should not be used
unless there is a critical need for it (very long, completely unattended
jobs on very secure systems.

.. _suggested_solution:

Suggested Solution
------------------

After some discussion we decided that we will use a slightly modified
ticket renewal approach.

.. _solution_for_host_use_case:

Solution for Host Use Case
~~~~~~~~~~~~~~~~~~~~~~~~~~

There are several components of the IPA client that would require
kerberos authentication:

-  IPA Data Provider – used to do LDAP lookups
-  XML-RPC client – (formerly known as the Policy Downloader) used to
   download policies and execute certificate related requests
-  Audit client – component responsible for uploading the logs collected
   from different processes

Each component will implement its own independent kerberos
authentication logic. This authentication logic will be capable of:

-  Authenticating with keytab if the TGT is not available or expired
-  Renewing TGT if it is about to expire

There is no need to use a shared ticket cache. For simplicity each
process will keep its own ticket cache in memory and re-authenticate or
renew ticket as needed.

To avoid the re-implementation of the same logic multiple times a common
library will be created. This library will be implemented SSSD
developers and shared with others team members who are working on the
client components.

.. _solution_for_user_use_case:

Solution for User Use Case
~~~~~~~~~~~~~~~~~~~~~~~~~~

For the user case we decided that it would be an overhead to implement
the logic described above inside the IPA client. We agreed that by
default the IPA Data Provider when performs authentication will not
request renewable kerberos tickets for users. If the user needs a
renewable ticket he would be able to request it using “kinit -r ”. Then
one can use a cron job or some other periodic mean to request a renewal
of the ticket using “kinit -R”. This can be documented in the IPA v2 as
a solution. Later we might create a convenient utility that would
combine the functionality of the “kinit -r ...” and “kinit -R” into one
utility. Such utility would be explicitly used by the users that are
about to start a long job. It will request a renewable ticket, demonize
and continue renewing the ticket until the renewal age elapses. If we do
not implement the utility we will at least document how the same task
can be accomplished using the current existing means.

For this approach to work the kerberos ticket policies should be
enforced on the server side. There are several places where and how the
policies can be specified.

The research showed the following rules:

-  One can specify maximum ranges for renewal and lifetime in the
   /ver/kerberos/krb5kdc/kdc.conf file. It is in the realms section.
   **max_life** setting defines maximum life of the ticket.
   **max_renewable_life** setting defines the period during which the
   ticket is renewable. In IPA v1 the values are 7d and 14 days. This
   means that if the client ask for a ticket valid for the 7d it will
   get it.
-  If the settings are removed from the kdc.conf file the default hard
   coded values are 24h for lifetime and 7d for renewal.
-  There is an entry in the kerberos hive with cn equal name of the
   realm. This entry has krbTicketPolicyAux object class applied to it
   but no attributes that specify the timeouts. If those attributes
   added they can only further restrict the rules defined in the
   configuration file. They can't extend the lifetime beyond what has
   been set in kds.conf or, if entries are not defined, the hard coded
   values.
-  Each individual principal can have the krbTicketPolicyAux object
   class applied to it. Some principals already have it applied but not
   users. So in v2 we will add the krbTicketPolicyAux object class to
   user object and expose the ticket lifetime attributes in UI and CLI.
   This would allow to alter the policies defined at the higher level
   but up to the limits imposed by configuration.

Based on the rules above we will use the following defaults:

-  kdc.conf as current (no changes)

   -  max_life = 7d
   -  max_reneable_life = 14d

-  Kerberos realm entry

   -  krbMaxTicketLife = 86400 *(it is 1 day)*
   -  krbMaxRenewableAge = 604800 *(it is 7 days)*

-  User entries will have

   -  objectClass = krbTicketPolicyAux *(applied during upgrade)*
   -  krbMaxTicketLife *- missing*
   -  krbMaxRenewableAge *- missing*

By adding the attributes to the user entry the customer would be able to
override (extend) the ticket policies on per user basis up to 7 days and
14 days. On the client side the IPA client will always ask for the
renewable ticket with 7 days lifetime and 14 days renewable life time.
This is equivalent to:

`` kinit -r 14d -l 7d ``

These two values will be defined in the IPA client policy and will be
centrally changeable if ever customer would want to alter values in
kdc.conf and match the new kdc,conf values with the values used on the
client.

In UI the kerberos fields will be special “protected” fields non
editable until explicitly requested. The CLI can be used to effectively
build a “bulk update” of the attributes if such functionality is needed.
We might explore using same approach for the management of the password
policy on per user basis instead of one size fits all as it is currently
in v1.x.
