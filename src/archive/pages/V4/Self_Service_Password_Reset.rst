.. _self_service_password_reset:

Self Service Password Reset
===========================

Overview
--------

One of the most highly requested features of FreeIPA is self-service
password reset. While there is no truly secure way to implement
self-service password reset, for many users, a sufficiently secure
scheme can be devised. This proposal outlines an extension to the
self-service web portal that allows for self-service password reset,
without hard-coding into the complicated and delicate code that handles
password authentication in the core FreeIPA project.



Use Cases
---------

.. _community_sites:

Community Sites
~~~~~~~~~~~~~~~

Community Sites wishing to use FreeIPA might have user bases too large
or geographically diverse to make administrator-initiated resets
impractical. Other community sites using FreeIPA in a non-critical role
might accept the trade-off of security for convenience.

Design
------

.. _high_level_architecture_and_workflow:

High-Level Architecture and Workflow
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. A user has forgotten their password and navigates to the forgot
   password page.
#. The user inputs their uid to a form on the self-service portal
#. The self-service portal queries the FreeIPA server once to determine
   that the uid is valid and what the email address of the uid provided
   is. The user is redirected to a notice that a password reset has been
   initiated, but not given any confirmation as to whether or not the
   username is valid and the request can be completed.
#. The self-service portal generates a nonce (the password reset token),
   which is emailed to the user's email address. The token is also
   logged in a database, along with the uid of the requesting user and a
   time and date that the request is initiated and when it expires.
#. The user receive the token in the email. If the user did not request
   the reset, they can simply ignore the token and the reset request
   will expire. If the user did wish to reset their password, the user
   inputs the token into the self-service portal. If the token has not
   expired, the user asked for confirmation that they wish to reset
   their password.
#. When confirmed, the portal generates a second nonce. The portal calls
   to the FreeIPA server and sets the user's password to this second
   nonce. The user is displayed a new, temporary password which they use
   immediately.
#. The user logs into the FreeIPA WebUI with the new password and is
   immediately prompted to change the password to a permanent value. The
   user can now access the account regularly.

Implementation
--------------

The permissions for the self-service portal machine user account will
have to be expanded to allow the self-service portal to reset user
accounts. This is obviously risky, but the risk can be mitigated by
fine-tuning permissions so that the self-service user cannot reset the
password of administrator users. If the self-service account is found to
be compromised, then it can simply be disabled while the security hole
is repaired.

However, on the positive side, this external method of reseting user
passwords allows us to implement what is essentially an unsecure feature
without compromising on the security of the FreeIPA core for users who
opt not to enable self-service password reset.

Dependencies
~~~~~~~~~~~~

The self-service password reset will be built on the existing
self-service portal, and so adds few new dependencies.

The one very important dependency is the choice of persistant data
store, so that reset tokens can be preserved. The simplest choice would
be to use a SQLite database to store the token information. However, a
key-value database like Redis might also be suitable. Data integrity in
the database is not important, because if a token accidentally becomes
unavailable, then the user can simply generate another with no
consequence. Tokens can be periodically expunged from the database
through the use of a sweeper script, or the tokens can be allowed to
persist in the database forever as a log of request attempts.

.. _how_to_test34:

How to Test
-----------

Like the self-service portal, self-service password reset can be tested
very simply with automated unit and integration tests, and no bulk
testing configuration should be needed.

`Category:FreeIPA Community
Portal <Category:FreeIPA_Community_Portal>`__
