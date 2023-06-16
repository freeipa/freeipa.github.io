This page contains troubleshooting advice for **integration** of FreeIPA
with other programs. For other issues, refer to the index at
`Troubleshooting <Troubleshooting>`__.

It is difficult to predict issues with interaction with other software,
the problem may be in bad configuration of the software, unexpected
environment configuration or, of course, a flaw in any of the FreeIPA
component. A general advise is to look for logs of the software and try
to narrow down the area where the problem is, check it's documentation
or know bugs or consult on the
`freeipa-users <https://lists.fedoraproject.org/archives/list/freeipa-devel@lists.fedorahosted.org/>`__
mailing list.

.. _sudo_does_not_work_for_hostgroups:

sudo does not work for hostgroups
=================================

SUDO requires netgroups to be working properly to use the hostgroups in
the FreeIPA sudo rules. If the SUDO rules only work with individual
hosts being set in SUDO rule, check the affected machine:

-  Make sure that NIS domain name matches your domain:

      ::

         nisdomainname

-  Make sure that netgroups can be read:

      ::

         getent netgroup hgroup1 

      if this command does not work, make sure that ``sss`` is listed
      for ``netgroup`` entry in ``/etc/nsswitch.conf``

If the advice above does not help, edit ``/etc/sudo-ldap.conf`` (or
``/etc/ldap.conf``, depending on what version of platform you're
running) and add the following line to the top:

::

   sudoers_debug 2

Then try another command, for example

::

   sudo -l
