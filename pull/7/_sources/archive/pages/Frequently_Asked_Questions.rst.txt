.. _general_faq:

General FAQ
===========

.. _whats_available_in_freeipa_now_whats_in_the_pipeline:

What's Available in FreeIPA Now? What's in the Pipeline?
--------------------------------------------------------

FreeIPA (so far) is an integrated solution combining

-  Linux (currently Fedora or Red Hat Enterprise Linux)
-  389 Directory Server
-  MIT Kerberos
-  NTP
-  DNS
-  Web and command line provisioning and administration tools
-  Dogtag Certificate System
-  `Active Directory Integration <IPAv3_AD_trust>`__
-  Integration with Weblogic server

**Version 1 focused on**

-  Allowing an administrator to quickly install, setup, and administer
   one or more FreeIPA servers for centralized authentication and user
   identity management.

**Version 2 focused on**

-  Adding DNS and Certificate Authority to the FreeIPA core
-  Allowing an admin to join a machine to an FreeIPA realm
-  Providing kerberos principal and cert to the joined machine
-  Providing service keytabs and service certificates to services
-  Managing the keytabs and certificates once provided
-  Plug-in architecture for FreeIPA extensibility
-  FreeIPA Client code for managing authentication, authorization,
   caching, connection
-  Centrally managed netgroups and automount

**Version 3 focused on**

-  `Active Directory Integration <IPAv3_AD_trust>`__ in form of Kerberos
   cross-realm trusts, allowing SSO from AD to Linux resources
-  Integration with Dogtag 10 or CA-less installation

**Version 4 focuses on**

-  `2FA Kerberos Authentication <V4/OTP>`__, Better Control Access,
   `Trusts <Trusts>`__ enhancements

Look for more detailed roadmap information at `Roadmap <Roadmap>`__

.. _domain_levels:

Domain levels
~~~~~~~~~~~~~

Please visit following page `Domain Levels <Domain_Levels>`__ to get
information about domain levels and features which are provided by them.

.. _why_use_freeipa:

Why Use FreeIPA?
----------------

For efficiency, compliance and risk mitigation, organizations need to
centrally manage and correlate vital security information including:

-  **Identity** (machine, user, virtual machines, groups, authentication
   credentials)
-  **Policy** (host based access control)
-  **Audit** (this component is deferred)

Because of its vital importance and the way it is interrelated, we think
identity, policy, and audit information should be open, interoperable,
and manageable. Our focus is on making identity, policy, and audit (some
day) easy to centrally manage for the Linux and Unix world. Of course,
we will need to interoperate well with Windows and much more.

We are looking to take concrete and useful steps and so have chosen
initially to focus on Identity solutions for the Unix/Linux world.

For policy we focus on the host based access control management and
enforcement. As for other aspects of the policy management related to
systems management and configuration management, after serious
evaluation we decided not to address these segments for now. There are
other projects that are working in this direction. We will closely
monitor those projects and integrate with them as interfaces become
available.

We did a lot of research and evaluation in the audit area and realized
that this is a significant effort and might require a project of its
own. For now we decided not to disperse our energy and work more on
improving the identity and authentication aspects of the system. But we
will continue to monitor open source projects in the audit related
space. One of such projects that was created as a result of our
evaluation is `ELAPI <https://fedorahosted.org/ELAPI>`__. Another recent
project is `Centralized Logging <Centralized_Logging>`__. We will
continue investing into these directions.

.. _what_are_the_problems_freeipa_is_trying_to_solve:

What are the problems FreeIPA is trying to solve?
-------------------------------------------------

-  Focus on solving identity management across the enterprise providing
   a reliable open source alternative to existing solutions
-  Vendor focus on Web identity management problems has meant less well
   developed solutions for central management of the Linux and Unix
   world's vital security info. Organizations are forced to maintain a
   hodgepodge of internal and proprietary solutions at high TCO.
-  Proprietary security products don't easily provide access to the
   vital security information they collect or manage. This makes it
   difficult to synchronize and analyze effectively.

.. _what_are_the_values_behind_the_freeipa_project:

What are the values behind the FreeIPA project?
-----------------------------------------------

Identity, policy, and audit information is vitally important and
interrelated. Therefore, we think it should be open, interoperable, and
manageable.

-  **Open** means the information is not held back as a proprietary
   value add, but is instead available to vendors and applications
   through standards wherever possible but always through
   well-documented and openly available protocols. It also means
   developing open source solutions and an open source community.

-  **Interoperable** means that systems storing or managing identity,
   policy, and audit information should provide backwards compatibility
   with existing systems and protocols, assume that infrastructure and
   systems will always be heterogeneous, and provide solutions that help
   heterogeneous systems work together rather than forcing migration to
   a single platform or technology.

-  **Manageable** means that systems managing this information should be
   easy to manage both centrally and locally (i.e a central server is
   not required) and should follow the principle of
   `subsidiarity <http://en.wikipedia.org/wiki/Subsidiarity>`__
   empowering individuals by enabling the delegation of administration
   to rights to the lowest level possible in an organization.

.. _what_license_does_freeipa_use:

What License does FreeIPA use?
------------------------------

See the `License <License>`__ page for more details

.. _technical_faq:

Technical FAQ
=============

.. _does_freeipa_support_cached_logins_for_example_for_laptops_at_home:

Does FreeIPA support cached logins, for example, for laptops at home?
---------------------------------------------------------------------

This is supported by FreeIPA's sister project,
`sssd <https://fedorahosted.org/sssd/>`__

.. _can_freeipa_replace_my_active_directory_server:

Can FreeIPA replace my Active Directory Server?
-----------------------------------------------

No. But with FreeIPA v2, you can replicate users and passwords from an
AD server to FreeIPA server.

With FreeIPA v3, you can create a `trust with Active
Directory <IPAv3_AD_trust>`__ and SSO (single sign on) from a Windows
machine to Linux machine.

.. _why_are_passwords_expired_after_reset:

Why are passwords expired after reset?
--------------------------------------

This is a security feature. For more information on the topic, see `New
Passwords Expired <New_Passwords_Expired>`__.

.. _why_freeipa_does_not_provide_a_self_service_password_reset_page:

Why FreeIPA does not provide a self-service password reset page?
----------------------------------------------------------------

This is a security feature. For more information on the topic, see
`Self-Service Password Reset <Self-Service_Password_Reset>`__.

.. _what_are_the_recommendations_for_freeipa_deployment:

What are the recommendations for FreeIPA deployment?
----------------------------------------------------

See `Deployment Recommendations <Deployment_Recommendations>`__.

.. _why_is_a_freeipa_client_not_backwards_compatible:

Why is a FreeIPA client not backwards compatible?
-------------------------------------------------

See `Client compatibility <Client#Compatibility>`__ article.

.. _when_will_we_implement_freeipa_to_freeipa_trusts:

When will we implement FreeIPA to FreeIPA trusts?
-------------------------------------------------

This is a feature in development (tracked in `ticket
4867 <https://fedorahosted.org/freeipa/ticket/4867>`__). FreeIPA to
FreeIPA trusts can be implemented right after we complete the second leg
of the Active Directory `Trusts <Trusts>`__, i.e. Active Directory
trusting FreeIPA users to access it's resources or log in. FreeIPA to
FreeIPA trusts will leverage the same interfaces (Global Catalog, which
is tracked in `ticket
3125 <https://fedorahosted.org/freeipa/ticket/3125>`__.

Until the feature is implemented, it would be technically possible to
create a Kerberos-only trust between two IPA realms in FreeIPA 4.2+, but
this is not supported with any native interface yet. There is a hacky
procedure described in `Red Hat Bugzilla
1035494 <https://bugzilla.redhat.com/show_bug.cgi?id=1035494#c16>`__ or
`ticket
4059 <https://fedorahosted.org/freeipa/ticket/4059#comment:5>`__. Such
trust would have no support from IPA tools and no ability to resolve
users, groups, support HBAC rules, sudo, etc. One could add additional
SSSD domains on IPA clients to represent other realms but this is not
tested by upstream and majority of features will may not work in the
intended ways.

It is important to understand, that `Kerberos <Kerberos>`__ trust is
only about authentication. Authorization decisions are
application-specific and mapping of Kerberos-authenticated identities to
POSIX application-visible identities has to happen somewhere (this is
part missing). Additionally, enforcement of IPA-specific rules (RBAC or
HBAC) is not ready for FreeIPA to FreeIPA trust yet.

We welcome any help with these engineering efforts! See
`Contribute <Contribute#Communication>`__ page for ways how to contact
us.

.. _active_directory_deprecated_identity_management_for_unix_idmu_what_should_i_do:

Active Directory deprecated Identity Management for Unix (IDMU), what should I do?
----------------------------------------------------------------------------------

With Windows Server 2012 R2, Microsoft announced the deprecation of the
Identity Management for Unix (IDMU) and NIS Server role which will not
be included starting with Windows Server 2016 Technical Preview (more
information on `TechNet
Blog <http://blogs.technet.com/b/activedirectoryua/archive/2015/01/25/identity-management-for-unix-idmu-is-deprecated-in-windows-server.aspx>`__).

This means that there will no longer be a UI to set POSIX attributes for
Active Directory users. Such users will no longer be able to
authenticate to FreeIPA clients, if FreeIPA ID Range is not configured
to automatically generate UID and GID for the AD users.

There are multiple options how to solve this issue on the FreeIPA side:

-  Generate POSIX attributes (especially UID, GID) automatically for AD
   users, based on their RID (recommended, especially for *green field*
   deployments)
-  Leverage `FreeIPA ID
   Views <V4/Migrating_existing_environments_to_Trust#ID_Views>`__ to
   assign POSIX attributes for the AD users

More information about user ID attributes mapping is for example in the
`RHEL
Guide <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Windows_Integration_Guide/sssd-ad-integration.html#about-id-mapping>`__.

.. _can_i_install_ipa_on_a_raspberry_pi:

Can I install IPA on a Raspberry Pi?
------------------------------------

We don't recommend using Raspberry Pi 3. With Raspberry Pi 4B+ it should
work. See `ARM <ARM>`__
