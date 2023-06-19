.. _ose_ipa_with_ad_trust:

OSE + IPA with AD Trust
=======================

Disclaimer
----------

This is just a proof of concept for review!

Purpose
-------

To allow users setup in a Windows Active Directory server to be able to
access OpenShift Enterprise through establishing a trust between RHEL
IdM (IPA) and the AD server.

Prerequisites
-------------

-  4 RHEL 6.4 Machines:

   #. IPA Server
   #. OSE Broker (an IPA Client)
   #. OSE Node (an IPA Client)
   #. User/Developer (an IPA Client)

-  1 Windows Active Directory Machine (assumption is that AD is already
   setup)
-  Optional: 2nd Windows non-AD machine that is already enrolled in
   Windows AD for a Windows developer

**Note**: The trust setup only supports two domain setups:

#. Parallel domains (e.g. ``ipaserver.example.com`` and
   ``addomain.example.com``)
#. IPA as a subdomain of AD (e.g. ``ipaserver.addomain.example.com`` and
   ``addomain.example.com``)

AD as a subdomain of IPA does not yet work per the instructions above,
and is being looked into.

.. _setup_idm_ose_machines:

Setup IdM + OSE Machines
------------------------

Follow the following instructions to setup OpenShift Enterprise with
Identity Management:

#. `IPA Server
   Setup <https://wiki.idm.lab.bos.redhat.com/export/idmwiki/IPA_Server_Setup>`__
#. `Broker and Node
   Setup <http://etherpad.corp.redhat.com/puddle-1-2-2013-06-26>`__ -
   the most up-to-date puddle
#. `User/Developer
   Setup <https://wiki.idm.lab.bos.redhat.com/export/idmwiki/User/Developer_Setup>`__

.. _ssh_configuration:

SSH Configuration
~~~~~~~~~~~~~~~~~

For SSH with keytabs configuration, follow
`these <https://wiki.idm.lab.bos.redhat.com/export/idmwiki/SSH_with_Keytabs_for_OpenShift>`__
instructions.

**Note**, however, that ssh with keytabs does *not* work out-of-the-box
between Windows <-> OpenShift Enterprise. If the user/developer uses
MinGW for the command line, GSSAPI is not compiled. MinGW can be
recompiled with GSSAPI, but this has not been tested. If the
user/developer uses PuTTy, configuration for Keytabs has not been
confirmed to work.

.. _setup_trust_between_ipa_server_and_windows_ad_server:

Setup Trust between IPA Server and Windows AD Server
----------------------------------------------------

Follow
`these <http://www.freeipa.org/page/Active_Directory_trust_setup#Configure_IPA_server_for_cross-realm_trusts>`__
instructions (starting specifically on "Configure IPA server for
cross-realm trusts").

**Note**: It may be helpful to install MinGW/MSYS for a Bash-like
environment on a Windows machine. This
`git <http://msysgit.github.io>`__ installation comes with both git and
MinGW, labeled as “Git Bash”. If you install this, then you may use this
shell to test
`ssh <http://www.freeipa.org/page/Active_Directory_trust_setup#Using_SSH>`__
instead of using PuTTY.

.. _setup_windows_for_userdeveloper:

Setup Windows for user/developer
--------------------------------

To be done on a Windows developer machine, or can be tested on the
Windows AD Server itself:

Follow step #2 of the command line instructions found
`here <https://www.openshift.com/get-started>`__ to get git, the
RubyInstaller and RHC installed.

.. _additional_configuration:

Additional Configuration
------------------------

krb5.conf
~~~~~~~~~

On both the Broker and Node machines, edit ``/etc/krb5.conf`` for the
following lines:

::

   # &lt;--snip--&gt;
   dns_lookup_realm = true
   dns_lookup_kdc = true
   # &lt;--snip--&gt;

This allows the clients to defer to DNS if/when a user has a different
realm, e.g. user@ADDOMAIN.EXAMPLE.COM.

Restart the KDC: ``service krb5kdc restart``.

mod_auth_kerb
~~~~~~~~~~~~~

On the Broker machine, within both broker & console apps
``/var/www/openshift/[broker | console]/httpd/conf.d/``, edit
``openshift-origin-auth-remote-user-kerberos.conf`` file, edit to match
the following with the appropriate parameters. Notice there are **two**
realms for ``KrbAuthRealms``, one is the IPA Realm, and one is the
$AD_DOMAIN:

::

   # Provided by the mod_auth_kerb package
   LoadModule authz_user_module modules/mod_authz_user.so
   LoadModule auth_kerb_module modules/mod_auth_kerb.so
   &lt;Location /broker&gt;
       AuthName &quot;OpenShift broker API&quot;
       AuthType Kerberos
       KrbMethodNegotiate On
       KrbMethodK5Passwd On
       KrbServiceName HTTP/broker.ipadomain.example.org
       KrbAuthRealms EXAMPLE.ORG ADDOMAIN.EXAMPLE.ORG
       Krb5KeyTab /var/www/openshift/broker/httpd/conf.d/http.keytab
       require valid-user

   # &lt;--snip--&gt;

This allows Apache to accept AD users when username is
user@ADDOMAIN.EXAMPLE.COM.

Restart OpenShift Broker and Console:

.. code:: bash

   $ service openshift-broker restart
   $ service openshift-console restart

.. _test_debug_setup_configuration:

Test & Debug Setup & Configuration
----------------------------------

RHEL
~~~~

On an IPA Client/Developer machine:

#. Get a new ticket as the AD user: ``kinit aduser@ADDOMAIN`` (notice
   that the ADDOMAIN is all caps)
#. Access the Broker:
   ``curl -Ik --negotiate -u :``\ ```https://$BROKER_FQDN/broker/rest/api/`` <https://$BROKER_FQDN/broker/rest/api/>`__
   - You should see two responses: a 401, and a 200.
#. Access the Node via ssh: ``ssh aduser@ADDOMAIN@NODE_FQDN``. Note the
   two ``@`` - the ADDOMAIN must be capitalized like before. If you
   setup ssh with keytabs, you should *not* be prompted for a password.

If you can not kinit as an AD User, then the configuration of the trust
is wrong. Check the logs on the server: ``/var/log/krb5kdc.log``.

If you have two 401 responses, check the http logs on the Broker at
``/var/log/openshift/broker/httpd/error_log``. You may need to restart
the broker (``service openshift-broker restart``), or re-edit the
configuration for kerberos in
``/var/www/openshift/broker/httpd/conf.d/openshift-origin-auth-remote-user-kerberos.conf``
like `above <#modauthkerb>`__.

If you are prompted for your password, on the Node machine, edit
``/etc/ssh/sshd_config`` and set ``UsePAM on``. Restart ssh via
``services sshd restart`` and try again. If the issue persists, add
verbosity in two places:

#. On the Node, edit ``/etc/sssd/sssd.conf`` and under
   ``[domain/ipadomain]`` add a line for ``debug_level = 9``. You will
   need to restart sssd: ``service sssd restart``. The logs will be
   available in ``/var/log/sssd/``.
#. On the IPA Client, you can add up to 3 ``v``'s when ssh'ing:
   ``ssh -v aduser@ADDOMAIN@NODE_FQDN``.

If the above works on the IPA Server but **not** the IPA client, then
revisit the ``/etc/krb5.conf`` configuration `above <#krbconf>`__.

.. _with_rhc:

With RHC
^^^^^^^^

If all of the above work, then:

#. Install RHC per OpenShift installation `command-line
   instructions <https://www.openshift.com/get-started>`__.
#. Run ``rhc setup --server=$BROKER_FQDN`` and follow the setup
   instructions. When prompted for login, use the pattern of
   ``aduser@ADDOMAIN``, specifically the ADDOMAIN being in caps.

**NOTE**: Logging in with Kerberos ticket is not yet supported. You will
be prompted for your password.

If you can not login, and within the broker logs, you find an error
stating ``Specified realm 'ADDOMAIN' not allowed by configuration``,
then review your Apache configuration `above <#modauthkerb>`__.

Windows
~~~~~~~

cURL
^^^^

Within either the Windows Command Line (with the ``C:\>`` prompt) or
with Git Bash/MinGW/MSYS shell, run
``curl -Ik -u $ADUSER@$ADDOMAIN:$ADPASSWORD $BROKER_FQDN``. You should
receive two responses: 401 and a 200.

**Note**: On Git Bash/MinGW/MSYS, GSSAPI is *not* compiled; therefore,
``curl --negotiate`` will *not* work. You can, however, recompile MinGW
with GSSAPI (directions below under *SSH*). Supposedly, the
``--negotiate`` would work after recompiling, but this has not been
tested.

SSH
^^^

Follow
`these <http://www.freeipa.org/page/Active_Directory_trust_setup#Using_SSH>`__
instructions with PuTTY to test SSH from Windows to the Node. If ssh'ing
into the Node machine does not work, but does work for IPA server, then
revisit the ``/etc/krb5.conf`` configuration `above <#krbconf>`__.

Windows does not have GSSAPI compiled for it, so similar to
``curl --negotiate`` not working on Windows, ssh with keytabs will not
work out of the box. While this has *not* been tested, you can
try/follow these instructions to set it up:

#. If you are using PuTTY, follow
   `these <http://www.ncsa.illinois.edu/UserInfo/Resources/Software/kerberos/windows_kfw_ssh.html#putty>`__
   setup instructions.

2. If you are using MinGW/MSYS/Git Bash, you can follow
   `these <http://www.nomachine.com/ar/view.php?ar_id=AR01J00621>`__
   instructions to recompile OpenSSH and GSSAPI for ssh with keytabs and
   curl with ``--negotiate``.

RHC
^^^

-  Install RHC per OpenShift installation `command-line
   instructions <https://www.openshift.com/get-started>`__, if not
   already.
-  Within either the Windows Command Line (with the ``C:\>`` prompt) or
   the Git Bash/MinGW shell, run ``rhc setup --server=$BROKER_FQDN`` and
   follow the setups prompted to complete setup.

**NOTE**: Logging in with Kerberos ticket is not yet supported. You will
be prompted for your password.
