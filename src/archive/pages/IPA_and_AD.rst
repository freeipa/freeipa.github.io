**'NOTE: This page is obsolete, we have since decided to change strategy
and use a slightly different trust model to interoperate with AD/Windows
machines.**'

See `IPAv3_Architecture <IPAv3_Architecture>`__ for the current
proposal.

.. _overview_and_problem_statement:

Overview and Problem Statement
------------------------------

The expansion of corporate networks has revealed a need for central
management of identities in the enterprise. Starting in the early
2000's, Microsoft's Active Directory (AD) very successfully met this
need. Since then, Microsoft has been the only central player providing
authentication, authorization and identity services for client machines
and users in the enterprise. There were alternatives, to be sure, but
they were not as robust as AD, and could not seriously be considered as
competitors. These solutions sometimes required too much in-depth
product knowledge, and were much harder to administer than AD.
Meanwhile, the AD position was so dominant that customers started to
standardize on AD as a centralized solution for authentication and
identity services in the enterprise. AD's dominance stimulated the
creation of a series of projects and solutions for integrating
non-Microsoft OSes into the AD domain. Among these are:

-  Samba - an open source project
-  SerNet - provides support for Samba as a stand-alone product
-  Red Hat Inc. - provides support for Samba as a part of Red Hat
   Enterprise Linux platform.
-  Quest (via Vintela acquisition) - provides a "One Identity Solution"
-  Likewise - a fork of the Samba code with additional features,
   provided as a separate product.
-  Centrify - provides a suit of products for central
   identity-management using AD.

These solutions are good ones, and enterprises are deploying them
successfully. For some customers, the solutions work extremely well, for
others the solution is acceptable but not perfect, especially where the
share of non-Windows systems is high. The main issue usually is that in
such mixed deployments, the UNIX/Linux systems are somewhat like
second-class citizens, and must pretend to AD to be Windows clients. But
as corporate deployments of UNIX/Linux systems increase, so also grows
the need to leverage UNIX/Linux' own native capabilities. AD can't
cleanly accommodate some of these UNIX-native features, at least not in
a convenient and easy-to-manage way.

The IPA project was started to provide a solution for those companies
that see UNIX/Linux computing as a strategic part of the enterprise and
see a value in using the native features of those systems. The IPA
project offers that IPA should be viewed as an AD surrogate for Windows
clients in the UNIX/Linux world.

The IPA project started in 2007 and so far has attracted great interest
in the Linux community. The freeipa-user mailing-list is active, and
frequently discusses users' deployment work. This means that IPA is
right on target, providing value for companies that see benefit in
having non-Windows clients authenticated and managed through a
UNIX-native domain controller with an integrated directory server.

But these two worlds, the world of AD and the world of IPA, can't exist
independently. They are parts of the same enterprise, and current
security-compliance guidelines mandate that AD and IPA must not only be
aware of each other, but must also be integrated for security
compliance. This is why IPA v.1.1 offered basic user-account
synchronization functionality. That said, IPA's account-synch solution
is limited, and shouldn't be viewed as a long-term solution for a
compliance and interoperability problem. On the one hand, many companies
have invested heavily in AD infrastructure, and will rely on AD for
years to come, protecting their investment. On the other hand, the
emerging share of UNIX/Linux servers requires a native solution like
IPA, that can manage these systems in a natural way. IPA developers have
thought about this situation, and have suggested a solution that would
provide a native integration between IPA and AD:

-  without changing anything on AD, and
-  without continuing to rely on a cumbersome synchronization mechanism.

This page is dedicated to that solution.

.. _solution_in_a_nutshell:

Solution In a Nutshell
----------------------

IPA's proposed solution calls for using Samba 4 together with IPA,
working from one LDAP back-end and sharing a kerberos server. Samba 4
will present IPA to the AD world as a separate domain forest and will be
responsible for establishing a cross-forest domain trust between the
IPA/Samba domain and the Windows part of the enterprise.

.. _what_is_samba_4:

What is Samba 4?
----------------

Samba 4 is a sub-project under the Samba umbrella, based on UNIX/Linux,
and focussing on creation of a security server fully equivalent in
functionality to AD. Samba 4's goal is to displace AD from the heart of
every Windows network, by providing an open-source alternative to AD.
Unfortunately, Samba 4 is still far from its final goal. In our opinion
there are several interrelated reasons for this:

-  The Samba 4 project (at least until recently) had no corporate
   sponsors, so early adopters had to support the software themselves.
   For such a core piece of security infrastructure, lack of
   product-support is a big barrier to adoption.
-  Lacking corporate sponsorship, the Samba 4 development community has
   remained small, and must struggle to balance between
   feature-development and product-support.
-  Some customers worry that it's nearly impossible to make a 100%
   compatible version of such a complex piece of proprietary technology.
   Such risk-averse customers prefer to pay Microsoft for AD
   product-support from Redmond, rather than to rely on an unsupported
   OSS replacement.
-  These customer-support issues are important even for early adopters.

We think that for Samba 4 to advance to its goal it needs to:

-  Get official support from some company
-  Grow its development community
-  Get more early adopters that will do testing, provide ideas, define
   requirements and shape the project growth.

In sum, the Samba 4 project, despite being a very promising piece of
technology, still struggles to gain mind share among software vendors,
the open source community and early adopters. Samba4 needs a boost, and
we think that integration with IPA could be just the boost that would
push Samba 4 over the top.

.. _high_level_architecture:

High Level Architecture
-----------------------

The diagram below shows a high level architecture for Samba4 with IPA.
In this diagram:

-  The Windows part of the enterprise (Windows clients) will continue to
   be served by multiple instances of AD, grouped in domains,
-  UNIX/Linux will be served by an IPA/Samba domain, with data shared
   between IPA and Samba.
-  IPA/Samba might even serve some of the Windows clients, too (there
   might be use cases when this makes sense).
-  "Cross-forest trust" means that users from the Windows part of the
   enterprise will securely be able to access resources in the IPA/Samba
   domain, and user-authentication and access-rights will properly be
   enforced.
-  The same will hold when users from the IPA/Samba domain access
   Windows resources.
-  At the core of this trust will be inter-realm kerberos trust.

As usual, the devil is in the details; there are several additional
requirements and constraints that need to be factored in.

.. _choosing_a_common_ldap_server:

Choosing a Common LDAP server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The architecture requires that Samba and IPA share the same data store.
This means that both of them need to be able to work using the same
Directory Server as a back end.

-  IPA is locked onto the "389" directory server and it would be a huge
   effort to make 389 DS swappable for some other DS server.
-  Samba is a bit more DS agnostic, but requires some directory features
   that OpenLDAP DS offers and 389 DS lacks.
-  Until recently Samba 4 could still use 389 as a DS, but the latest
   Samba 4 functionality takes advantage of some features not currently
   provided by DS 389.
-  Since the Samba 4 project seems not to mind to continue being DS
   agnostic, and since some feature enhancements would not harm DS 389,
   the path of least resistance seems to be to add these new features to
   DS 389.

.. _choosing_a_shared_kdc:

Choosing a Shared KDC
~~~~~~~~~~~~~~~~~~~~~

The diagram's architecture also assumes that Samba and IPA will share
the same KDC; this KDC must be able to serve both Windows and UNIX/Linux
clients. This task is much more complex.

-  IPA takes advantage of MIT's Kerberos. The IPA KDC runs as a separate
   process, listens on the kerberos port and does the packet processing.
-  Samba takes advantage of the Heimdal Kerberos with multiple
   additional extensions, some of which were added to Heimdal explicitly
   for Samba.
-  Heimdal Kerberos runs as a library linked into the Samba server,
   instead of running as a stand-alone process.

The integration effort has tough choices to make: which version of
Kerberos to use, and how? One of the constraints the Samba 4 + IPA
project needs to take into account is the supportability of the product
based on the integration. The most flexible solution would be to have an
option to choose any Kerberos implementation, allowing software vendors
to choose which KDC to support. Unfortunately, IPA is heavily dependent
on MIT and it will be a big effort to make IPA pluggable. The IPA
project would be interested in the customer benefits of such
functionality, but unfortunately, Kerberos-pluggability isn't a priority
for the IPA project right now. Similarly, Samba used to be able to plug
in different Kerberos back ends, but over time, the level of
functionality provided by MIT server became insufficient, and Samba
became more attached to the Heimdal implementation.

Here is a comparison of the possible approaches:

-  Approach 1

   -  Action: Make IPA take advantage of Heimdal instead of MIT Kerberos
   -  Pros:

      -  IPA would allow pluggable Kerberos implementations - (Low
         priority and value for IPA project)

   -  Cons:

      -  Big effort
      -  Low benefit
      -  Would require software vendors that would want to take
         advantage of the IPA project and turn it into a product to
         support Heimdal. Currently the vendor that supports IPA as a
         product supports MIT Kerberos and not Heimdal. Requiring
         Heimdal would lead to support of two kerberos implementations
         which will be a huge cost for the vendor.

-  Approach 2

   -  Action: Make Samba be able to use either MIT's or Heimdal's
      Kerberos
   -  Pros:

      -  Help and guidance from MIT, leveraging good relations with
         MIT's Kerberos Consortium
      -  Samba 4 / IPA Project, and its product, can both be released
         faster
      -  Additional functionality and flexibility added to Samba. Since
         Samba 4 is not yet a supported product, this change makes it
         more more appealing for software vendors to support Samba 4 as
         a product.
      -  No extra cost for any vendor to support IPA

   -  Cons:

      -  This is still a relatively big effort

After some evaluation, the IPA project team came to the conclusion that
it would be easier and mostly beneficial to enhance MIT Kerberos to meet
Samba 4 requirements, so as to restore Samba's ability to be Kerberos
independent. This approach seems to benefit not only the IPA project but
the Samba 4 project as well.

Conclusion
----------

By integrating Samba with IPA, Samba would become a supported component
via the same vendor. That would significantly grow Samba's exposure.
Samba will end up in many more hands, growing the community of adopters.
This usually leads to the growth of developer interest and thus
development community, bringing closer Samba 4's ultimate goal of
replacing AD.

The IPA project sees mutual benefit in this, and hopes that this view
would be shared in the IPA and Samba communities, creating a productive
environment where ideas developed by the IPA team get checked and
evaluated by the Samba community, with a shared sense of urgency and
importance. The IPA project community plans to conduct nearly all the
development work related to this project. However, the collaboration
with Samba developers is very important, and the IPA project community
hopes that Samba developers will be open and responsive to questions and
patches that arise during this effort.
