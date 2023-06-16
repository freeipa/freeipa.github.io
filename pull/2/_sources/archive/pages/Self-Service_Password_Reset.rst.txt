Self-service password reset feature is often requested by FreeIPA users
as it is not part of the default user management module. Users with
forgotten password are expected to contact helpdesk or FreeIPA
administrator to reset the password manually, after proving user's
identity to them (see `New Passwords Expired <New_Passwords_Expired>`__
for more information). While it may seem to be insufficient, it is a
feature that FreeIPA team does not plan to include as a FreeIPA core
feature.

.. _password_self_service_weaknesses:

Password self-service weaknesses
================================

The FreeIPA project makes strong security standards and encryption
available for regular users and environments, without a need to be a
security expert to be able to configure and use it. This approach
however requires all it's parts to maintain a certain level of security
that users can trust to avoid undermining it's purpose. A system is as
strong as it's weakest part and it was found that a self-service
password reset service may indeed be the weak spot.

The most common approach to password self-service may be security
questions (vulnerable to social engineering), reset by an e-mail (may be
sent in plain text between mail servers) or others. Such approaches are
vulnerable and can be abused. While they are fine for a low security
system like a mailing list or a free mail service, we do not see it as
secure enough for FreeIPA and we do not plan to include it in the core.

.. _possible_solutions:

Possible solutions
==================

FreeIPA administrators are expected and welcome to take advantage of
FreeIPA internal API or `LDAP <Directory_Server>`__ interface to prepare
and integrate their own solution for password reset, where they are
aware of it's disadvantages and can tailor such system to meet their
infrastructure and security level requirements. An Extending FreeIPA
guide with examples on how to extend FreeIPA can be found in
`Documentation <Documentation>`__ page.

We welcome external parties to take on this task and effort. We will
help, advise, collaborate and advertise them. We can even host them in a
special repository. We encourage those efforts to produce variety of the
solutions for different use cases. We will be interested in making sure
we do not break them by running contributed integration test. We
encourage these plugins to provide the same user experience and look and
feel as the rest of the system. We will not, however, maintain, support
or distribute any of those solutions ourselves as a part of main FreeIPA
project.

At some point in future we might develop a self-service portal solution
but it will be treated as a separate project following the principals
outlined above.

.. _rd_party_projects:

3rd party projects
------------------

-  `pwm <https://github.com/pwm-project/pwm>`__ - Open Source Password
   Self Service for LDAP directories
