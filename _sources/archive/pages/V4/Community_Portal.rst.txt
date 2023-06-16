Overview
--------

There are several use cases where the owners of a FreeIPA server might
want to allow anonymous users to interact with the FreeIPA server, such
as for self-service user registration. This proposal outlines a web
application that can interact with the FreeIPA server on behalf of an
anonymous user.

Source code: `Community Portal on
GitHub <https://github.com/freeipa/freeipa-community-portal>`__

.. _use_cases4:

Use Cases
---------

.. _self_service_user_registration:

Self-service user registration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If the FreeIPA server owner wishes to allow anonymous registration, the
self-service portal allows users to input a set of preconfigured fields
and submit them to the FreeIPA server, creating a staged user with these
field values. An administrator can then review account sign ups and
individually approve or reject accounts.

Design
------

.. figure:: Community_portal_arch.png
   :alt: Community_portal_arch.png

   Community_portal_arch.png

.. _stand_alone_web_application:

Stand Alone Web Application
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The self-service portal is designed as a stand-alone application that
communicates with the FreeIPA server using only ipalib and the RPC. The
server sees the self-service portal as it does any other client, and
never needs to directly interact with anonymous users.

.. _self_service_registration:

Self-Service Registration
~~~~~~~~~~~~~~~~~~~~~~~~~

Self-service registration is designed to make use of FreeIPA's new
staged user feature. When users sign up for FreeIPA, the self-service
portal calls the ipalib Python API to issue a stageuser-add command.
Submission to the self-service portal is protected with a CAPTCHA. The
user's input is then validated. If there are any errors in the user's
inputs, the user is dropped back to the sign-up form with values
pre-filled, and a message outlining the error is displayed. If the
user's input is valid, then the command is sent and committed on the
FreeIPA server, and the user is dropped to a completion page.

.. _administrator_notification:

Administrator Notification
~~~~~~~~~~~~~~~~~~~~~~~~~~

When a user has registered, the adminstrator is sent an email describing
the new user. Confirmation or rejection of new users should be performed
manually.

There is talk of using D-Bus to publish a notification that a user has
signed up and then have another script subscribed to that notification
actually dispatch the email/notice/carrier pigeon. I think this solution
is over-engineered and will increase development time.

Extension
~~~~~~~~~

With the core framework of the Self-Service Portal built, the software
should be easily extensible so that other self-service use cases
involving the participation of an anonymous user, such as self-service
password reset, can be covered.

Implementation
--------------

The Self-Service Portal will be implemented as a stand-alone Python web
application. User input will be taken via forms, and client-side
scripting will be limited to visual enhancements only, in order to
greatly simplify the application and ensure compatibilty.

The self-service portal will be configured to use a special user account
on the FreeIPA server with very limited permissions, which means in the
event that the Community Portal is compromised, the scope of the damage
that can be done is limited to the authorized commands required by the
portal. For the self-service registration, the only permission needed is
the ability to execute "stageuser-add".

The portal will use the ipalib Python API to issue commands. Validation
is performed entirely by the ipalib API, as well as the actual sending
of the command over RPC.

Dependencies
~~~~~~~~~~~~

The portal should be built using a Python web framework, in order to
simplify development and avoid having to write code directly interacting
with WSGI. A prototype is built in Flask, a lightweight Python web
framework, but Flask's absence from the RHEL repos may mean an alternate
choice is needed.

The pages themselves are rendered using the Jinja2 template engine.

.. _feature_management4:

Feature Management
------------------

The Self-Service portal exists almost completely decoupled from the
FreeIPA core and by design cannot be interacted with via the FreeIPA
WebUI or CLI.

Configuration
~~~~~~~~~~~~~

The application will require some configuration to use. A user account
on the FreeIPA server with the permissions required by the self-service
portal (permission to perform stageuser-add) will need to be created.
The self-service host server will need to be configured with Kerberos
credentials so that it can access the FreeIPA server. Disabling the
self-service portal in case of a breach or coordinated attack is as
simple as shutting down the web application.

Scripts will be written to automatically configure and deploy the
self-service portal.

.. _how_to_test4:

How to Test
-----------

As a stand-alone application, unit and integration tests will be
written, with the boundaries between the application and ipalib mocked
out. No special testing configuration should be needed, and the
application should fit in nicely with existing continous integration
systems.

`Category:FreeIPA Community
Portal <Category:FreeIPA_Community_Portal>`__
