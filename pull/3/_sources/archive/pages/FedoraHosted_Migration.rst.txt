We need to migrate from Fedora Hosted to other solution. This page lists
what we currently use and what are our requirements for the next tools.

SSSD is undergoing the same migration:
`discussion <https://lists.fedorahosted.org/archives/list/sssd-devel@lists.fedorahosted.org/thread/WGK3LXHLVUJHCFIGMOTPH5DE7ZXL6VKN/#WGK3LXHLVUJHCFIGMOTPH5DE7ZXL6VKN>`__,
`used trac fields <https://fedorahosted.org/sssd/wiki/ticket_fields>`__

Why
---

fedorahosted.org shuts down.

-  Shutdown date: 2017-02-28
-  Announcement:
   https://communityblog.fedoraproject.org/fedorahosted-sunset-2017-02-28/

.. _current_state:

Current state
-------------

FreeIPA project uses:

-  git hosting: https://git.fedorahosted.org/cgit/freeipa.git
-  Trac issue tracker: https://fedorahosted.org/freeipa

.. _issue_tracker:

Issue tracker
-------------

.. _used_fields:

Used fields
~~~~~~~~~~~

-  **Reported by**
-  **Owned by**
-  **Priority**
-  **Milestone**
-  Component
-  Version
-  **Keywords**
-  **Cc**
-  **Blocked By**
-  **Blocking**
-  Affects Documentation
-  **Patch link**
-  **Red Hat Bugzilla**
-  **Patch review by**
-  **External tracker**
-  **Design link**
-  Test coverage
-  Test by
-  Test case
-  Needs UI design
-  Feature
-  Source
-  Expertise
-  Release Notes

Purpose of non-standard fields is documented in `Trac
wiki <https://fedorahosted.org/freeipa/wiki/TicketFields>`__.

Some of the fields are not used so often but in general a support for
custom fields are o way to mimic them is needed.

API
~~~

Our process uses linking of various resources together, e.g. in "Patch
link", "Red Hat Bugzilla'", "Design link" fields. These fields are often
processed/set by automated tools(ipatool, clone tool). For that a public
API is needed. Our Trac instance doesn't support authentication by using
an API token, so it is not a strict requirement, but would be very much
welcome.

.. _ticket_numbers:

Ticket numbers
~~~~~~~~~~~~~~

FreeIPA Trac instance has over 6000 ticket in total and around 1200
opened ticket. When a ticket is fixed a commit message contains a link
to the ticket. Ticket usually contains additional useful information
like related commits, reason why certain aspects were implemented in
certain way, or simply general discussion. Therefore it is essential to
keep this information by migrating to new tracker system and ideally
keep the same ticket numbers so that redirection rule can be made .

Hosting
~~~~~~~

Trac was maintain by Fedora Infra, it worked well. Would be nice to
avoid self-hosted solution.

Requirements
~~~~~~~~~~~~

-  support of custom fields
-  support of API
-  ability migrate all tickets with comments in automated fashion
-  keep ticket numbers

.. _nice_to_have:

Nice to have
~~~~~~~~~~~~

-  API token support
-  hosted solution

.. _possible_replacements:

Possible Replacements
---------------------

GitHub
~~~~~~

Pros
^^^^

-  widely used
-  a lot of tooling around it

Cons
^^^^

-  closed

.. _missing_features:

Missing features
^^^^^^^^^^^^^^^^

-  custom fields

Notes
^^^^^

-  Board https://www.zenhub.com/

Pagure.io
~~~~~~~~~

.. _pros_1:

Pros
^^^^

-  open

.. _cons_1:

Cons
^^^^

-  immature

.. _missing_features_1:

Missing features
^^^^^^^^^^^^^^^^

-  custom fields
