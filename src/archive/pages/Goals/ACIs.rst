.. _access_control_in_freeipa:

Access control in FreeIPA
=========================

Assumptions
-----------

.. _read_permissions_for_normal_users_do_not_harm:

Read permissions for normal users do not harm
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When work on IPA started most of the information stored was needed
without authentication by machines, it was all (NIS maps equivalent
data).

Later more functionality that falls into a grey area where it could be
restricted was added. It was decided on a case by case basis, but by
default it was not restricted because we do not believe in security
through obscurity or because the information was still easily available
if someone escalated privileges on any single client.

Goals
-----

-  FreeIPA components should have least possible privilege. When
   possible, user-facing components should use delegation and operate
   using user's credentials (to limit attack surface).

`Category:Goals <Category:Goals>`__
