Overview
========

To support SELinux as an application the IPA shall have several
different kinds of plug-ins implemented. This article talks about these
plug-ins from the prospective of FreeIPA v2 plans.

Design
======

We agreed that SELinux application would heavily leverage the policy
engine, role engine and pluggable PAM responder. The fact that PAM
responder is not planned to be pluggable in v2 complicates things a bit.
We agreed that we will hard code SELinux related functionality in PAM
responder in v2 and refactor it into a sepaFreeIPAv2:rate plug-in in v3
time frame.

.. _policy_engine:

Policy Engine
-------------

So far we have identified three different areas where the IPA policy
engine can help to manage a large number of hosts running SELinux.

-  download, install or remove SELinux policy modules
-  configure SELinux policy modules via SELinux policy booleans
-  manage the relation of Linux users to SELinux users (role
   management?)

These three areas map well to the policy section, action, configuration
and role respectively, we have identified. See `FreeIPAv2:Overall Design
of Policy Related
Components <FreeIPAv2:Overall_Design_of_Policy_Related_Components>`__
for more details.

.. _download_install_or_remove_selinux_policy_modules_with_ipa_action_policy:

Download, install or remove SELinux policy modules with IPA action policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An IPA action policy has three parts, , and (see `FreeIPAv2:Overall
Design of Policy Related
Components <FreeIPAv2:Overall_Design_of_Policy_Related_Components>`__
for more details). In the context of SELinux we can for example check
with the if SELinux is enabled at all and if yes download a SELinux
policy module and install it. The main section of a corresponding IPA
policy file might look like

::

     <ipaaction>
       <condition>
        <command>/bin/cat /etc/redhat-release</command>
        <expected_output>Fedora release 9 (Sulphur)</expected_output>
       </condition>
       <file>
         <url>http://my.server.org/my_selinux_policy.pp</url>
         <path>/tmp/my_selinux_policy.pp</path>
         <owner>root</owner>
         <group>root</group>
         <access>0400</access>
         <cleanup>yes</cleanup>
       </file>
       <run>
         <command>/usr/sbin/semodule -i /tmp/my_selinux_policy.pp</command>
         <user>root</user>
       </run> 
     </ipaaction>

This example shows how to download the binary SELinux policy module and
calling semodule to install it.

.. _xslt_processing:

XSLT Processing
^^^^^^^^^^^^^^^

This IPA action policy will be processed as any other IPA action policy.

.. _configure_selinux_policy_modules_via_selinux_policy_booleans_with_ipa_config_policy:

Configure SELinux policy modules via SELinux policy booleans with IPA config policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

SELinux policies can include conditional section to allow runtime
modification without having to load a new policy. These conditional
sections can be enable or disabled with the help of SELinux policy
booleans.

SELinux policy booleans are simple boolean variable which can be
manipulated with semanage or get/set/togglesebool. Call 'semanage
boolean -l' to get a list and a short description of all SELinux policy
booleans currently know to the system.

To make the SELinux policy booleans accessible to IPA a configuration
policy, selinux_booleans, is used. The main section of an IPA XML policy
file may look like this:

::

     <ipaconfig>
       <selinux_boolean>
         <name>webadm_manage_user_files</name>
         <value>true</value>
       </selinux_boolean>
       <selinux_boolean>
         <name>ssh_sysadm_login</name>
         <value>false</value>
       </selinux_boolean>
     </ipaconfig>

.. _xslt_processing_1:

XSLT Processing
^^^^^^^^^^^^^^^

With the help of a XSL template the IPA config policy will be
transformed to a format suitable as a command line argument for
setseboot. The policy downloader will than call setsebool with the
result as defined by XSL metadata

::

     <md:output_handler>
       <exec_with_args command_name="/usr/sbin/setsebool" user="root"/>
     </md:output_handler>

.. _manage_the_relation_of_linux_users_to_selinux_users_with_ipa_role_policy:

Manage the relation of Linux users to SELinux users with IPA role policy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To give or take certain privileges to/from a user with the help of
SELinux we have to connect three SELinux element, namely

-  the Linux user, e.g. luser
-  the SELinux user, e.g. seuser_u
-  the SELinux role, e.g. serole_r

(It is a widespread convention to use suffix \_u for SELinux users and
\_r for SELinux roles.)

SELinux roles are defined by the SELinux policy or SELinux policy
modules. SELinux users can be connected to SELinux role with semanage:

::

   /usr/sbin/semanage user -a -R "serole_r guest_r" seuser_u

Linux user can mapped to SELinux user with semanage, too.

::

   /usr/sbin/semanage login -s seuser_u luser

While the relation of SELinux users to roles should be created when the
corresponding SELinux policy modules is installed (see above), the
management of Linux users and SELinux users can be handled by IPA
policies. The simple mapping of a Linux user to a SELinux user can be
done with the help of an IPA action policy or using role defintion
policy. We will concentrate here an a role based approach where a Linux
user can be mapped with the help of pam_selinux to different SELinux
users depending if he logs in via an insecure, e.g. telnet, or secure,
e.g. console login, channel. The following IPA role definition policy
specifies three roles: "guest", "user" and "admin", with different
settings. Users which have the "guest" role on a specific host will
always associated with guest_u, independent of the channel they are
using to access the host. If a user logs in via ssh and is associated to
the admin role he will be mapped to staff_u.

::

     <iparole>
       <role>
         <name>guest</name>
         <default_context>
           <selinux_user>guest_u</selinux_user>
           <mls>S0</mls>
         </default_context>
       </role>

       <role>
         <name>user</name>
         <default_context>
           <selinux_user>guest_u</selinux_user>
           <mls>S0</mls>
         </default_context>
         <context>
           <service>ssh</service>
           <service>console</service>
           <selinux_user>user_u</selinux_user>
           <mls>S0</mls>
         </context>
       </role>

       <role>
         <name>admin</name>
         <default_context>
           <selinux_user>guest_u</selinux_user>
           <mls>S0</mls>
         </default_context>
         <context>
           <service>ssh</service>
           <selinux_user>staff_u</selinux_user>
           <mls>S0</mls>
         </context>
         <context>
           <service>login</service>
           <selinux_user>staff_u</selinux_user>
           <mls>S0-S15</mls>
         </context>
       </role>
     </iparole>

.. _xslt_processing_2:

XSLT Processing
^^^^^^^^^^^^^^^

The IPA role policy will be transformed to an LDIF format and written
into the LDB of the IPA client. Further processing has to be done
elsewhere, because the XSLT engine has no knowledge of the
user-host-role association.

.. _pam_ipa_and_pam_responder:

pam_ipa and PAM Responder
^^^^^^^^^^^^^^^^^^^^^^^^^

All the work for the pam_ipa is actually done in the PAM responder. At
the login moment the PAM responder will implement a specific SELinux
logic. Later in v3 it will be replaced with a proper plug-in. The IPA's
PAM module will be higher in the stack than pam_selinux and will prepare
information for it to consume.

This logic will follow the following algorithm:

-  If the user logging in is not known to IPA, e.g. a local account,
   return PAM_USER_UNKNOWN and let the the following PAM modules decide
-  Get IPA pam_selinux role for user logging into the machine (this is
   done via same internal calls as used to satisfy requests from EXT
   library).
-  If there is no role for the user return PAM_USER_UNKNOWN and let the
   the following PAM modules decide (maybe it would make sense to have
   an configurable switch to force role association on certain host, in
   this case PAM_AUTH_ERR/PAM_ABORT can be returned).
-  Get IPA pam_selinux role definition from the LDB (or other file if we
   decide it is better to use a different storage)
-  Create a file for this user in the following format:

``   ``\ ``:``\ ``:``

with the example above it would look like this:

::

      guest_u:*:S0
      staff_u:ssh:S0
      staff_u:login:S0-S15

-  Put this file into

``   /etc/SELinux/targeted/login/``\ ``/seusers``

pam_selinux
^^^^^^^^^^^

The current version of pam_selinux will only use
/etc/selinux/targeted/seusers but coming version of SELinux PAM will be
modified to look for

``   /etc/selinux/targeted/logins/``\ ``/seusers``

file before falling back to default

``   /etc/selinux/targeted/seusers``

If there is no seusers file under specific user name directory or
directory does not exist at all then the default seusers files from
**/etc/selinux/targeted/seusers** will be used. If the file exists and
pam_selinux cannot decide what to do based content of the file the
authentication should fail and not use the default from
/etc/selinux/targeted/seusers.
