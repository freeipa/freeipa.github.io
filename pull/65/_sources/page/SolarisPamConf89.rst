SolarisPamConf89
================

Back to `Configuring UNIX Clients <ConfiguringUnixClients>`__

::

   #*************************************
   # pam.conf for Solaris 8 and 9
   #*************************************

   # pam.conf being used on trout
   # should work for all solaris machines from 2.6 to 10.
   # only exception is that sol 10 must have pam_unix_cred.so.1 enabled
   # while the others mush have it disabled
   # telnet/ssh work.  try ftp, dtlogin, etc. later

   # /etc/pam_conf for pam_ldap for solaris.
   # from: http://docs.sun.com/app/docs/doc/816-4556/6maort2te?a=view
   #
   # Authentication management
   #
   # login service (explicit because of pam_dial_auth)
   #
   login   auth requisite        pam_authtok_get.so.1
   login   auth required         pam_dhkeys.so.1
   #login   auth required         pam_unix_cred.so.1                       # sol 10 req this.
   login   auth required         pam_dial_auth.so.1
   login   auth binding          pam_unix_auth.so.1 server_policy
   login   auth required         pam_ldap.so.1
   #
   # rlogin service (explicit because of pam_rhost_auth)
   #
   rlogin  auth sufficient       pam_rhosts_auth.so.1
   rlogin  auth requisite        pam_authtok_get.so.1
   rlogin  auth required         pam_dhkeys.so.1
   #rlogin  auth required         pam_unix_cred.so.1
   rlogin  auth binding          pam_unix_auth.so.1 server_policy
   rlogin  auth required         pam_ldap.so.1
   #
   # rsh service (explicit because of pam_rhost_auth,
   # and pam_unix_auth for meaningful pam_setcred)
   #
   rsh     auth sufficient       pam_rhosts_auth.so.1
   #rsh     auth required         pam_unix_cred.so.1
   rsh     auth binding          pam_unix_auth.so.1 server_policy
   rsh     auth required         pam_ldap.so.1
   #
   # PPP service (explicit because of pam_dial_auth)
   #
   ppp     auth requisite        pam_authtok_get.so.1
   ppp     auth required         pam_dhkeys.so.1
   ppp     auth required         pam_dial_auth.so.1
   ppp     auth binding          pam_unix_auth.so.1 server_policy
   ppp     auth required         pam_ldap.so.1
   #
   # Default definitions for Authentication management
   # Used when service name is not explicitly mentioned for authentication
   #
   other   auth requisite        pam_authtok_get.so.1
   other   auth required         pam_dhkeys.so.1
   #other   auth required         pam_unix_cred.so.1
   other   auth binding          pam_unix_auth.so.1 server_policy
   other   auth required         pam_ldap.so.1
   #
   # passwd command (explicit because of a different authentication module)
   #
   passwd  auth binding          pam_passwd_auth.so.1 server_policy
   passwd  auth required         pam_ldap.so.1
   #
   # cron service (explicit because of non-usage of pam_roles.so.1)
   #
   cron    account required      pam_unix_account.so.1
   #
   # Default definition for Account management
   # Used when service name is not explicitly mentioned for account management
   #
   other   account requisite     pam_roles.so.1
   other   account binding       pam_unix_account.so.1 server_policy
   other   account required      pam_ldap.so.1
   #
   # Default definition for Session management
   # Used when service name is not explicitly mentioned for session management
   #
   other   session required      pam_unix_session.so.1
   #
   # Default definition for  Password management
   # Used when service name is not explicitly mentioned for password management
   #
   other   password required     pam_dhkeys.so.1
   other   password requisite    pam_authtok_get.so.1
   other   password requisite    pam_authtok_check.so.1
   other   password required     pam_authtok_store.so.1 server_policy
   #
   # Support for Kerberos V5 authentication and example configurations can
   # be found in the pam_krb5(5) man page under the "EXAMPLES" section.
   #