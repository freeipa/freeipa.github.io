.. _howtofreeipa_and_policykit:

Howto/FreeIPA and PolicyKit
---------------------------

Making PolicyKit recognize certain FreeIPA user as administrator on
local system requires following simple step
(https://fedorahosted.org/freeipa/ticket/3203), until more advanced
solution is provided, see https://fedorahosted.org/freeipa/ticket/5350:

-  create file called 40-freeipa.rules in /etc/polkit-1/rules.d/ with
   contents:

| ``polkit.addAdminRule(function(action, subject) {``
| ``    return ["unix-group:admins", "unix-group:wheel"];``
| ``});``

-  (optional) make sure that file has at least 644 permissions (I'm
   using default umask 077 and that caused me a bit of delay to make it
   work)

-  (alternate approach would be) to use password-less access, fill the
   same file with this contents instead:

| ``polkit.addRule(function(action, subject) {``
| ``    if (action.id == "org.freedesktop.policykit.exec" &&``
| ``        subject.isInGroup("admins")) {``
| ``        return polkit.Result.YES;``
| ``    }``
| ``});``

where 'wheel' is default local 'administrative' group on RHEL-based
systems ('adm' on Debian-based systems, I think) and admins is default
'administrative' group in FreeIPA. Without this custom rule
polkit/pkexec/... will be asking for password of root account, if there
is no user in local wheel group. If there is (I'm using 1 local
administrative account in case something with FreeIPA goes wrong), it
will be asking for that user(s) password instead, not involving root
user at all.

So, to sum up, step above turns this:

| ``$ id``
| ``uid=1293400001(ipauser) gid=1293400001(ipauser) groups=1293400001(ipauser),1293400000(admins) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023``
| ``$ pkexec id``
| ``==== AUTHENTICATING FOR org.freedesktop.policykit.exec ===``
| :literal:`Authentication is needed to run `/usr/bin/id' as the super user`
| ``Authenticating as: root``
| ``Password: ``
| ``==== AUTHENTICATION COMPLETE ===``
| ``uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023``

or this:

| ``$ id``
| ``uid=1293400001(ipauser) gid=1293400001(ipauser) groups=1293400001(ipauser),1293400000(admins) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023``
| ``$ pkexec id``
| ``==== AUTHENTICATING FOR org.freedesktop.policykit.exec ===``
| :literal:`Authentication is needed to run `/usr/bin/id' as the super user`
| ``Authenticating as: SysAdmin (sysadmin)``
| ``Password: ``
| ``==== AUTHENTICATION COMPLETE ===``
| ``uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023``

into this:

| ``$ id``
| ``uid=1293400001(ipauser) gid=1293400001(ipauser) groups=1293400001(ipauser),1293400000(admins) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023``
| ``$ pkexec id``
| ``==== AUTHENTICATING FOR org.freedesktop.policykit.exec ===``
| :literal:`Authentication is needed to run `/usr/bin/id' as the super user`
| ``Multiple identities can be used for authentication:``
| `` 1. SysAdmin (sysadmin)  ``
| `` 2. Administrator (admin)``
| `` 3. IPA User (ipauser)``
| `` ...``
| ``Choose identity to authenticate as (1-X): 3``
| ``Password:``
| ``==== AUTHENTICATION COMPLETE ===``
| ``uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023``

It works also for graphical user interfaces (GNOME Shell in my case),
etc. Only when it comes to that GNOME Shell, user is shown as 'Standard'
and not as 'Administrator' in Settings -> Users.

.. _hbac_rule:

HBAC Rule
~~~~~~~~~

If FreeIPA has not been configured to *allow_all* for any service on any
host, you will have to add a HBAC Service named **polkit-1**, if this
does not already exist, and create an appropriate HBAC rule for users
accessing hosts with the above rule definition via the polkit-1 service.

Symptoms requiring this HBAC Rule include when running;

``$ pkexec id``

resulting in log messages such as;

| ``$ journalctl -xe``
| ``...``
| ``polkit-agent-helper[3130]: pam_sss(polkit-1:account): Access denied for user ``\ ``: 6 (Permission denied)``
| ``...``
| ``gnome-shell.desktop[2372]: polkit-agent-helper-1: pam_acct_mgmt failed: Permission denied``
