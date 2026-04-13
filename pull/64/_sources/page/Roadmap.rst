Roadmap
=======

From time perspective, FreeIPA upstream releases are tied to `Fedora
release schedule <https://fedoraproject.org/wiki/Schedule>`__. The
development is coordinated so that the most recent FreeIPA bits can be
released and delivered before next Fedora version is finished and
released so that it's available to all it's users. In parallel with
development of the next bleeding-edge FreeIPA version we do support and
maintenance versions present in the stable Fedora releases (see `Fedora
releases page <https://fedoraproject.org/wiki/Releases>`__).



Requests for upstream
---------------------

A primary and authoritative source of enhancement/bug tickets is our
upstream `Pagure page <https://pagure.io/freeipa/>`__. It contains the
planned fixes for both current development and maintenance branch (see
`FreeIPA roadmap <https://pagure.io/freeipa/roadmap>`__ for more
details).



Where to get FreeIPA
--------------------

Upstream releases are always released both as source tarball and as a
build for current Fedora versions. There is also a
`Copr <https://copr.fedoraproject.org/>`__ build, providing the builds
for other platforms, including EPEL. See `Downloads <Downloads>`__ or
`Quick Start Guide <Quick_Start_Guide>`__ pages for details where to get
the developed FreeIPA versions.



Current Efforts
---------------

-  **FreeIPA 4.7**: development of new major version focused on
   stability, supportability and reflecting Fedora changes
-  **FreeIPA 4.6.x**: stabilization of version focused on Python3
   `latest release <Releases/4.6.0>`__
-  **FreeIPA 4.5.x**: stabilization of the `latest feature
   release <Releases/4.5.0>`__



Postponed/Abandoned efforts
---------------------------

-  Described on `Postponed <Postponed>`__ page.



Future Direction
----------------

-  FreeIPA versions 4.8+ will focus primarily on robustness, stability,
   testing and troubleshooting capabilities. Work on new features is
   deprioritized till the aforementioned aspects are good enough.
-  Next major feature bucket is called `"Future
   Releases" <https://pagure.io/freeipa/roadmap?status=Open&no_stones=&milestone=Future+Releases>`__.
   The tickets in the `"Future
   Releases" <https://pagure.io/freeipa/roadmap?status=Open&no_stones=&milestone=Future+Releases>`__
   bucket will be reviewed and considered first to be pulled to next
   FreeIPA release.
-  Tickets in the `"Ticket
   Backlog" <https://pagure.io/freeipa/roadmap?status=Open&no_stones=&milestone=Ticket+Backlog>`__
   is the main backlog for tickets we think are a good idea but we do
   not plan to complete any time soon due to our resource planning.
   However, it does not mean we do not welcome our community to contact
   us and `contribute <contribute>`__! It may still happen though, that
   we will pick a ticket from this milestone to next FreeIPA release,
   but it is much less likely than with *Future Releases*.
-  The tickets in the
   `"Deferred" <https://pagure.io/freeipa/roadmap?status=Open&no_stones=&milestone=Tickets+Deferred>`__
   bucket are tasks we surely do not plan to complete ourselves and we
   do not recommend community to look at those either (unless strongly
   justified). Discuss your plans on implementing these tickets on
   `freeipa-devel
   list <http://www.redhat.com/mailman/listinfo/freeipa-devel>`__ first
   please.



Notes about P and A of IPA
--------------------------

-  The general direction of the project is to continue adding identity
   and policy related changes to IPA.
-  The audit component of the project was deferred, the reasoning can be
   found on the `About <About>`__ page. However, there are ongoing
   efforts how to close this gap, see `Centralised
   Logging <Centralised_Logging>`__.