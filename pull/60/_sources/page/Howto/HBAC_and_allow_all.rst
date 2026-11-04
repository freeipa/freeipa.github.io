HBAC_and_allow_all
==================



HBAC and the allow_all problem
------------------------------

The default setup of IPA server is to allow access from anywhere to
anywhere to any user and service. It is achieved by a catchall HBAC rule
**allow_all**:

::

   # ipa hbacrule-find
   -------------------
   1 HBAC rule matched
   -------------------
     Rule name: allow_all
     User category: all
     Host category: all
     Source host category: all
     Service category: all
     Description: Allow all users to access any host from any host
     Enabled: TRUE
   ----------------------------
   Number of entries returned 1
   ----------------------------

If we want to actively start using the HBAC feature, we need to disable
this rule, otherwise it will always apply even if none of our other
rules will apply and the access should be denied. However, if we disable
this rule, immediately any access of existing users to existing hosts
will be denied.

To avoid locking users out of their systems, we need to ensure that
there is other rule for the **existing hosts** that will allow access to
them, ensuring continuation of service. One possibility is the following
setup: all existing machines will become members of new group
**allow_all_hosts** and this in turn will become the target host group
for new HBAC rule **allow_all_users_services** which will grant access
to everyone on these machines. That way the existing behaviour will be
preserved for existing hosts and users.

The commands to create the setup are as follows:

::

   # ipa hostgroup-add --desc 'Host group which will have allow_all_users_services HBAC enabled.' allow_all_hosts
   ---------------------------------
   Added hostgroup "allow_all_hosts"
   ---------------------------------
     Host-group: allow_all_hosts
     Description: Host group which will have allow_all_users_services HBAC enabled.
   # ipa host-find --raw --pkey-only --sizelimit=0 \
       | awk '$1 == "fqdn:" { print "--hosts=" $2 }' | xargs -n100 ipa hostgroup-add-member allow_all_hosts
   [...]
   # ipa hbacrule-add allow_all_users_services --usercat=all --servicecat=all --desc='Allow access to hosts in group allow_all_hosts to anybody from anywhere.'
   ------------------------------------------
   Added HBAC rule "allow_all_users_services"
   ------------------------------------------
     Rule name: allow_all_users_services
     User category: all
     Service category: all
     Enabled: TRUE
   # ipa hbacrule-add-host allow_all_users_services --hostgroups=allow_all_hosts
     Rule name: allow_all_users_services
     User category: all
     Service category: all
     Enabled: TRUE
     Host Groups: allow_all_hosts
   -------------------------
   Number of members added 1
   -------------------------

Then the **allow_all** rule can be disabled:

::

   # ipa hbacrule-disable allow_all
   ------------------------------
   Disabled HBAC rule "allow_all"
   ------------------------------

From now on, for access to work like it used to do before, any new host
needs to be added to the **allow_all_hosts** using the
``ipa hostgroup-add-member`` or similar mechanism. Alternatively, it is
also possible to use automember and default automember features to set
the group membership automatically.

Note that there is ``ipa hbactest`` utility which can be used to test
policies -- use it to test your setup before locking your users out of
their systems.



Example of new service
----------------------

Once the individual systems are enumerated in the **allow_all_hosts**
host group, we can define new rules with possibly more targeted services
to align some of those hosts to.

Let us assume we plan to run application wikiapp and want to have PAM
service **wikiapp** with HBAC handled by the IPA server for the
authentication and authorization.

We can start by defining the service:

::

   # ipa hbacsvc-add wikiapp
   ----------------------------
   Added HBAC service "wikiapp"
   ----------------------------
     Service name: wikiapp

We then define the rule for this service:

::

   # ipa hbacrule-add allow_wikiapp
   -------------------------------
   Added HBAC rule "allow_wikiapp"
   -------------------------------
     Rule name: allow_wikiapp
     Enabled: TRUE

And we add the service:

::

   # ipa hbacrule-add-service allow_wikiapp --hbacsvcs=wikiapp
     Rule name: allow_wikiapp
     Enabled: TRUE
     Services: wikiapp
   -------------------------
   Number of members added 1
   -------------------------

At any point we can check the status of the rule:

::

   # ipa hbacrule-find allow_wikiapp
   -------------------
   1 HBAC rule matched
   -------------------
     Rule name: allow_wikiapp
     Enabled: TRUE
     Services: wikiapp
   ----------------------------
   Number of entries returned 1
   ----------------------------

We add user **bob** and host **wikiapp.example.com** to the rule:

::

   # ipa hbacrule-add-user allow_wikiapp --user=bob
     Rule name: allow_wikiapp
     Enabled: TRUE
     Users: bob
     Services: wikiapp
   -------------------------
   Number of members added 1
   -------------------------
   # ipa hbacrule-add-host allow_wikiapp --hosts=wikiapp.example.com
     Rule name: allow_wikiapp
     Enabled: TRUE
     Users: bob
     Hosts: wikiapp.example.com
     Services: wikiapp
   -------------------------
   Number of members added 1
   -------------------------

We now test the access to the service:

::

   # ipa hbactest --user=bob --host=wikiapp.example.com --service=wikiapp
   --------------------
   Access granted: True
   --------------------
     Matched rules: allow_all_users_services
     Matched rules: allow_wikiapp

We see that the rule **allow_wikiapp** matches which is good but
**allow_all_users_services** matches as well. We probably want to remove
the host from the hostgroup. But beware -- this might cut away our
access to the machine via ssh if ssh is configured to use IPA HBAC:

::

   # ipa hostgroup-remove-member allow_all_hosts --hosts=wikiapp.example.com
     Host-group: allow_all_hosts
     Description: Host group which will have allow_all_users_services HBAC enabled.
     Member hosts: ipa.example.com, smtp.example.com
     Member of HBAC rule: allow_all_users_services
   ---------------------------
   Number of members removed 1
   ---------------------------

On the **wikiapp.example.com** machine, we want to create
**/etc/pam.d/wikiapp** file with configuration specifying sssd as the
mechanism for authentication and authorization:

::

   auth    required   pam_sss.so
   account required   pam_sss.so