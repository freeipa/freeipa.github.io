**HowTo/Setup FreeIPA Services for MacOS X 10.12 and 10.13**

.. _dns_setup:

DNS Setup
---------

Either,

#. Go to System Preferences>Network
#. Select top priority network and click Advanced…
#. Select DNS
#. Add your IPA server’s IP Address
#. Click OK
#. Click Apply

Or, configure your DHCP service to set your IPA server as primary DNS.

.. _ssl_setup:

SSL Setup
---------

-  Download the ca.crt from the IPA server

#. Open terminal
#. cd ~/desktop
#. curl -OL http://yourserver.yourdomain.com/ipa/config/ca.crt

-  Doubleclick the ‘ca.crt’ file
-  Add to the System keychain
-  Locate certificate within Keychain Access
-  Doubleclick the certificate
-  Expand Trust
-  Change System Default to Always Trust
-  Exit Keychain Access and authenticate to apply changes
-  Move the ca.crt file to /etc/ipa

.. _kerberos_setup:

Kerberos Setup
--------------

Edit/create the file /etc/krb5.conf as shown below:

::

   [domain_realm]
       .yourdomain.com = YOURDOMAIN.COM
       yourdomain.com = YOURDOMAIN.COM

   [libdefaults]
       default_realm = YOURDOMAIN.COM
       allow_weak_crypto = yes 
       dns_lookup_realm = true
       dns_lookup_kdc = true
       rdns = false
       ticket_lifetime = 24h
       forwardable = yes 
       renewable = true
    
   [realms]
       YOURDOMAIN.COM = {
           kdc = tcp/ipa-server.yourdomain.com
           admin_server = tcp/ipa-server.yourdomain.com
           pkinit_anchors = FILE:/etc/ipa/ca.crt
       }

-  Edit /etc/pam.d/authorization as shown below:

::

   # authorization: auth account
   auth          optional       pam_krb5.so use_first_pass use_kcminit default_principal
   auth          sufficient     pam_krb5.so use_first_pass default_principal
   auth          required       pam_opendirectory.so use_first_pass nullok
   account    required       pam_opendirectory.so

-  Edit screensaver and passwd as shown below

::

   #cat > /etc/pam.d/screensaver << 'EOF'
   auth       optional       pam_krb5.so use_first_pass use_kcminit
   auth       optional       pam_krb5.so use_first_pass use_kcminit default_principal
   auth       sufficient     pam_krb5.so use_first_pass default_principal
   auth       required       pam_opendirectory.so use_first_pass nullok
   account    required       pam_opendirectory.so
   account    sufficient     pam_self.so
   account    required       pam_group.so no_warn group=admin,wheel fail_safe
   account    required       pam_group.so no_warn deny group=admin,wheel ruser fail_safe
   EOF
   #cat > /etc/pam.d/passwd << 'EOF'
   password   sufficient     pam_krb5.so
   auth       required       pam_permit.so
   account    required       pam_opendirectory.so
   password   required       pam_opendirectory.so
   session    required       pam_permit.so 
   EOF

-  Verify by running “kinit username”

.. _ipa_enrollment:

IPA Enrollment
--------------

.. _name_workstation:

Name workstation
~~~~~~~~~~~~~~~~

-  Open terminal
-  sudo scutil --set HostName workstation.yourdomain.com

.. _add_via_freeipa_web_console:

Add via freeIPA web console
~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Open IPA web console (https://yourserver.yourdomain.com)
#. Sign on as a Directory Manager
#. Go to Identity > Hosts
#. Click the + Add button
#. Enter the workstation’s hostname (e.g., Book001)
#. Add current primary IP address (terminal > # ifconfig)
#. Click the Add and Edit button.
#. Add the workstation’s MAC addresses

.. _generate_keytab_on_ipa_server:

Generate keytab on IPA server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. su root
#. kinit admin
#. ipa-getkeytab -s yourserver.yourdomain.com -p
   host/workstation.yourdomain.com -k ~/workstation.keytab
#. To test that the keytab successfully retrieved and stored in
   ~/workstation.keytab, run *ipa host-show workstation*
#. The previous should return,

::

   Host name: workstation.yourdomain.com   Principal name: host/workstation.yourdomain.com@YOURDOMAIN.COM
       MAC address: 00:00:00:AA:1B:14
       Password: False
       Keytab: True
       Managed by: workstation.yourdomain.com

.. _retrieve_keytab_from_server:

Retrieve keytab from server
~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. From the workstation run *sftp admin@yourserver.yourdomain.com*
#. sftp> get workstation.keytab /etc/krb5.keytab
#. sftp> exit
#. chown root:wheel /etc/krb5.keytab
#. chmod 0600 /etc/krb5.keytab
#. Verify on freeIPA web GUI that Kerberos key is present (Identity >
   Host > workstation)

.. _directory_utility_setup:

Directory Utility Setup
-----------------------

#. On workstation, go to System Preferences > Users & Groups > Login
   Options
#. Set the following:

::

   Automatic login: off
   Display login window as: Name and password
   Show fast user switching menu as Full Name

#. Click Join… beside Network Account Server
#. Enter “yourserver.yourdomain.com”
#. Click Continue
#. Verify *Allow network users to log in at login window* is selected
#. Click on Options... beside the previous setting
#. Verify All network users is selected
#. Next to Network Account Server, click Edit…
#. Click Open Directory Utility
#. Edit LDAPv3
#. Select yourserver.yourdomain.com and choose Edit…
#. Set the following:

::

   Open/close times out in 5 seconds
   Query times out in 5 seconds
   Connection idles out in 1 minute
   Encrypt using SSL (selected)

Mappings
--------

#. From the edit window opened in previous step (Connection), click
   Search & Mappings
#. Add record type Groups and map to ‘ipausergroup’
#. Add PrimaryGroupID attribute to Groups and map to ‘gidNumber’
#. Add RecordName attribute to Groups and map to ‘cn’
#. Add record type Users and map to the following:

   -  inetOrgPerson
   -  posixAccount
   -  shadowAccount
   -  apple-user

#. Within the record type Users add the following with the mappings
   shown on the right:

======================= =============
Attribute               Mapping
======================= =============
AuthenticationAuthority uid
GeneratedUID            GeneratedUID
HomeDirectory           #/Users/$uid$
NFSHomeDirectory        #/Users/$uid$
PrimaryGroupID          gidNumber
RealName                cn
RecordName              uid
UniqueID                uidNumber
UserShell               loginShell
======================= =============

#. Verify the search base for both Record Types is
   “dc=yourdomain,dc=com”
#. Verify all subtrees is selected for both Record Types
#. Click OK button to save and return to server list
#. Click OK again
#. Click on Search Policy
#. Verify “/LDAPV3/yourserver.yourdomain.com” is listed beneath
   “/Local/Default”
#. Close open windows
#. Open terminal and run test “dscacheutil -q user -a name
   *yourusername*\ ”

.. _allow_mobile_accounts_a_user_profile:

Allow Mobile Accounts A User Profile
------------------------------------

-  From a terminal, run 'chmod 0777 /Users'

.. _make_accounts_mobile_off_network_access:

Make Accounts Mobile (Off-network Access)
-----------------------------------------

-  From a terminal, run 'sudo
   /System/Library/CoreServices/ManagedClient.app/Contents/Resources/createmobileaccount
   -n *username*'

.. _if_filevault_already_enabled:

If FileVault already enabled
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  fdesetup add -usertoadd username
-  Enter user’s password at prompt

.. _migrate_user_profile_for_mobile_account:

Migrate User Profile for Mobile Account
---------------------------------------

#. sudo su root
#. ditto old_userprofile new_userprofile
#. chown -R new_username:staff new_userprofile
#. After login as the new mobile account, update keychain password to
   mobile account’s

**Alternative method: backup user profile with time machine and migrate
user profile to network account**

.. _mobilenetwork_account_known_issues:

Mobile/Network Account Known Issues
-----------------------------------

-  On OS X 10.13.x, a mobile account fails to build a profile at initial
   login

   -  Workaround: create a local account, build profile, change local
      account to mobile

-  Cannot change mobile/network account password from login desktop
-  Changing password from IPA website does not sync with keychain:

   -  The keychain may not update if Update selected
   -  Run Keychain Access and manually set password from edit menu
   -  Workaround: change password from System Preferences > Users &
      Groups

-  Changing password from IPA website does not update passphrase for
   disk encryption via FileVault

   -  Workaround: change password from System Preferences > Users &
      Groups

-  Mobile users created after enabling FileVault cannot log in until
   another account decrypts the drive

   -  Fix: Enable User from System Preferences > Security & Privacy >
      FileVault

-  Cannot login to mobile account while offline with OS X 10.11

   -  Fix: Upgrade to OS X 10.12

.. _migrate_user_profile_issues:

Migrate User Profile Issues
---------------------------

-  Chrome extensions shortcuts are broken, if absolute path used to
   define
-  Chrome download folder requires manual adjustment, if absolute path
   defines
-  Cloud storage services generally fail to sync due to file path change

   -  Fix path within application
   -  DropBox requires the old path before allowing access to settings

-  Adobe CC products require re-install

.. _configuring_multiple_workstations:

Configuring Multiple Workstations
---------------------------------

This process can be significantly faster after an initial workstation
setup:

-  Instead of manually configuring authorization, passwd, and
   screensaver, just copy working versions of them to /etc/pam.d
-  Similarly, copy a working version of krb5.conf to /etc.
-  Instead of manually configuring the LDAP mappings, copy the contents
   of /Library/Preferences/OpenDirectory from a workstation with the
   desired mappings and paste to the same location on a workstation
   being configured **after joining the workstation to your IPA
   server**.

References
----------

http://pig.made-it.com/ldap-mac.html

https://annvix.com/using_freeipa_for_user_authentication#Configuring_IPA_Clients

`Category:How to <Category:How_to>`__
