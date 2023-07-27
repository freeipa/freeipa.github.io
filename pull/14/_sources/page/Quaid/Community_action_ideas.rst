Community_action_ideas
======================

Based on my research so far, there are several areas that could be
improved that will lower barriers to participation.

The goal is, make it easy to participate, and from those participants,
contributors emerge.



Community enablement ideas
--------------------------

-  Make CLA process easier.

   -  It's all manual; how to ensure CLA is signed, etc.?
   -  Can we hook in to Fedora Infra? Why? Why not?

-  Explain PRD process up front
-  Ensure that all design work is done on devel list, explain.
-  Make it easier to give wiki access

   -  Encourage freeipa-users here

-  Update contribution pages to reflect needs beyond code



How to solve the problems
-------------------------

In summary, I am thinking of tying \*.freeipa.org in to Fedora
Infrastructure. This is to fix a few problems and to accomplish related
goals. The idea is in the spirit of Fedora Hosted, that is, being an
umbrella over worthwhile projects and letting those projects be uniquely
branded upstreams.



The problems
----------------------------------------------------------------------------------------------

-  The CLA is print, sign, and FAX/mail. This raises barriers for easy
   participation in what should be the easiest way to participate. It
   also can be difficult for people in other countries.
-  The lowest possible barrier to entry, editing the wiki, requires a
   manual CLA and sending an email to freeipa-\* list requesting an
   account. For a contrast, fedoraproject.org/wiki enables people to
   create an account, sign the CLA, and gain wiki edit access without
   joining any further groups.
-  Login to the wiki is over HTTP only. By contrast, login and editing
   to fedoraproject.org/wiki is HTTPS, and HTTP is heavily cached.
-  The main requested contribution is writing code for the *BIG SCARY*
   code base. Oh, and documentation, which is either done via the wiki
   way or ...
-  All documentation is written in DocBook using Publican by a single
   employee of Red Hat. On the face of it to a contributor, there is a
   high barrier of entry for writing documentation and an established
   team and methods where work is not done openly outside of Red Hat.

   -  Wiki content does not have an easy path or process for going to
      DocBook XML.

-  The wiki pages hide the edit and discussion buttons at the bottom, in
   a small font.
-  Wike page naming and some other stylistic choices can be redone to
   make search easier, intuitive page finding, and l10n.
-  Is it an explicit choice to have the favicon be Shadowman?



Questions to ponder
----------------------------------------------------------------------------------------------

-  What are the contribution goals?
-  What have FreeIPA's experience been with the CLA?

   -  What elements do they want in a contribution policy?
   -  Does FreeIPA need a CLA? What works for Fedora may not need to
      work the same way where there is one or a small handful of
      licensing choices and source repositories. Would a simple
      contribution policy work?
   -  Would the project want documentation contributions via the wiki
      with a



Bold solutions
----------------------------------------------------------------------------------------------

-  Gain a number of advantages and no real negatives by switching
   hosting to Fedora Infrastructure.

   -  Underpinnings supported by professional contributor sysadmins via
      Fedora Infra project.

      -  FreeIPA enthusiasts can participate and contribute via Fedora
         Infra project as a volunteer sysadmins.

   -  Signing of FreeIPA-specific CLA can be done via Fedora Account
      System (FAS).
   -  Account used for fedorahosted.org/freeipa can be the same account
      used on the wiki and for other systems.
   -  Easier tie-in to the build and release systems of Fedora, for the
      packagers.

      -  Attracts contributors to FreeIPA via packaging.

   -  Use the same configuration for FreeIPA.org as Fedora's MediaWiki
      instance, which is hooked in to FAS; the sysadmin team has worked
      to make the install not deviate from the upstream by using
      modules.

      -  There are good MediaWiki savvy sysadmins within Fedora who may
         be able to help.

   -  Making the FreeIPA wiki easy to edit encourages enthusiasts and
      sysadmin users to participate in contributing content.

      -  Fedora Docs has tools and methods for converting wiki content
         to DocBook XML to be packaged, translated, and so forth.
      -  For example, moving the work for the FreeIPA release notes to
         the wiki can gain the same level of contribution and work-load
         split that Fedora Docs gains.
      -  Can rely upon Fedora's wiki style and how-to documentation as
         canonical, where Fedora uses MediaWiki norms + specific, common
         conventions. I.e., get a better wiki, faster.

   -  Gain access to any further Fedora web tool enhancements, such as
      Fedora Community.
   -  Be that much closer to a wider community of interest that might
      want to contribute l10n, design, documentation, etc.
   -  FreeIPA contributors may benefit from the `Fedora
      Community <https://fedoraproject.org/wiki/FedoraCommunity>`__
      portal tools.



How quaid is going to do this
----------------------------------------------------------------------------------------------

#. Volunteer to run migration and future maintainence via existing
   involvement in Fedora Infrastructure

   -  *This puts the pressure on me to get further participation from
      FreeIPA sysadmin users to help via Fedora Infra*

#. Modify freeipa.org in test environment, then roll out new instance
   tied to FAS
#. Create ``cla_freeipa`` group via FAS and hook that in to auth for
   freeipa.org, or otherwise resolve CLA situation
#. ...
#. Profit!

`Category:Community enablement
plans <Category:Community_enablement_plans>`__