Overview
--------

A notification system for changes or warnings in FreeIPA. The system
should be general enough to be able to send messages through various
outputs (like email, IRC) and maybe not only limited to the FreeIPA
framework.



Use Cases
---------

-  Create mounted home directories on user creation
-  Recieve notifications about expiring passwords/certificates

.. _open_questions:

Open questions
--------------

.. _hook_locations:

Hook locations
~~~~~~~~~~~~~~

Where to place the hooks that would be in charge of sending
notifications?

Related to the granularity of the notification system. Hooks in the
components instead of the framework would allow for a better coverage of
events we're interested in (eg. user creation directly through LDAP).

.. _method_of_transport:

Method of transport
~~~~~~~~~~~~~~~~~~~

What method of transport for messages generated on changes/warnings
would be the best?

If multiple sources of information were needed for the notification
system (Kerberos, 389-DS, certmonger) a universal system like dbus might
be the best method to use. However some components would have to be
extended first. Maybe different kinds of transport for different
components (fedmsg for python, dbus for C)?

.. _expiring_passwords:

Expiring passwords
~~~~~~~~~~~~~~~~~~

How to poll for expiring passwords and certificates?

Probably using some daemon that would periodically check the records.
How often? Performance implications?

.. _user_scripts:

User scripts
~~~~~~~~~~~~

Should the system be general enough to allow for running user scripts?

.. _reaction_to_hooks:

Reaction to hooks
~~~~~~~~~~~~~~~~~

Do we want for FreeIPA to be able to react to the hooks?

If yes, should the hooks be synchronous/asynchronous? Performance
implications?

.. _performance_issues:

Performance issues
~~~~~~~~~~~~~~~~~~

How will the hooks impact performance when they are/are not used?
