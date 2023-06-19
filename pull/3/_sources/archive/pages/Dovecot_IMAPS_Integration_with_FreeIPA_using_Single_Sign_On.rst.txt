**HOWTO: Configure Dovecot to authenticate IPA users using Kerberos with
Single Sign On.**

**Provided by Dale Macartney**

The below details will walk you through how to add a Red Hat Enterprise
Linux 6.2 system to an IPA domain, and then configure Dovecot to allow
single sign on to user mailboxes with IMAP/S.

Details of this example are as follows

| ``   Domain name: example.com``
| ``   IPA Server: ds01.example.com``
| ``   Dovecot Server: mail01.example.com``
| ``   IPA Client: workstation01.example.com``
| ``   IPA User: user1 and user2``

**Please Note: This guide describes using SSL combined with Dovecot to
deliver IMAPS support. This guide is not designed to cover how to create
a valid SSL certificate. This guide uses the default dovecot generated
certificate and it is HIGHLY recommended that if you wish to deploy this
into a production environment, that you replace this certificate with
your own trusted/validated certificate**

Gotcha's
--------

I encounter a few gotcha's around mailbox storage which would be
worthwhile remembering, or at least keep in mind for deployment
considerations.

Storing MailDir content in home directories is a very common method for
managing mailboxes.

If you use automount for your IPA based users (e.g. NFS Home
Directories), you need to keep in mind that Dovecot does not manage an
interactive login which would be when the automount normally occurs.

The result of this will mean that trying to store the MailDir folders in
a users network based home dir will fail as in the eyes of Dovecot, the
folder simply won't exist. Basically, Dovecot will not be able to write
to your automount directory.

This guide will show you how to use local based storage to manage this
issue and create any directories which aren't currently present.

.. _add_system_to_ipa_domain_ensure_dns_is_working_correctly_otherwise_this_step_will_fail:

Add system to IPA Domain (ensure DNS is working correctly otherwise this step will fail)
----------------------------------------------------------------------------------------

``# ipa-client-install -U -p admin -w mysecretpassword``

.. _install_dovecot_and_set_service_to_start_on_boot:

Install Dovecot and set service to start on boot
------------------------------------------------

| ``# yum install dovecot``
| ``# chkconfig dovecot on``

.. _edit_etcdovecotdovecot.conf_to_allow_imap:

Edit /etc/dovecot/dovecot.conf to allow imap
--------------------------------------------

Find

``#protocols = imap pop3 lmtp``

and replace with

``protocols = imap``

.. _edit_etcdovecotconf.d10_auth.conf_to_configure_kerberos_authentication:

Edit /etc/dovecot/conf.d/10-auth.conf to configure kerberos authentication
--------------------------------------------------------------------------

Enter the below lines at the end of the file
/etc/dovecot/conf.d/10-auth.conf

| ``userdb {``
| ``  driver = static``
| ``  args = uid=dovecot gid=dovecot home=/var/spool/mail/%u``
| ``}``

Next, find the below lines (these will be in various locations inside
the file)

| ``auth_mechanisms = plain``
| ``#auth_gssapi_hostname =``
| ``#auth_krb5_keytab =``
| ``#auth_realms =``
| ``#auth_default_realm =``

and replace with

| ``auth_mechanisms = gssapi``
| ``auth_gssapi_hostname = mail01.example.com``
| ``auth_krb5_keytab = /etc/dovecot/krb5.keytab``
| ``auth_realms = example.com``
| ``auth_default_realm = example.com``

.. _create_new_ipa_group_for_mailbox_access:

Create new IPA group for mailbox access
---------------------------------------

From your IPA server, create a new group for your users to store their
mailbox

| ``[root@ds01 ~]# ipa group-add``
| `` Group name: mailusers``
| `` Description: Mail User Group``
| `` --------------------``
| `` Added group "mailusers"``
| `` --------------------``
| `` Group name: mailusers``
| `` Description: Mail User Group``
| `` GID: 1427200003``
| ``[root@ds01 ~]# ``

.. _add_users_to_mailusers_group:

Add users to "mailusers" group
------------------------------

Add your users to the new group

| ``[root@ds01 ~]# ipa group-add-member mailusers``
| ``[member user]: user1``
| ``[member group]: ``
| ``  Group name: mailusers``
| ``  Description: Mail User Group``
| ``  GID: 1427200003``
| ``  Member users: user1``
| ``-------------------------``
| ``Number of members added 1``
| ``-------------------------``
| ``[root@ds01 ~]# ``

.. _create_new_directory_for_user_mailboxes:

Create new directory for user mailboxes
---------------------------------------

Create a new directory to be used as your mail store for the server.
Also remember to change the group membership to allow your "mailusers"
to be able to write to the folder.

| ``mkdir /mail``
| ``chmod 770 /mail``
| ``chgrp mailusers /mail``
| ``chcon -t user_home_t /mail``

Note: If you wish to use file system quotas or add high availability to
your solution, having this folder on a shared file system would be very
beneficial.

.. _edit_etcdovecotconf.d10_mail.conf_to_configure_the_mailbox_location:

Edit /etc/dovecot/conf.d/10-mail.conf to configure the mailbox location
-----------------------------------------------------------------------

Find

``#mail_location =``

and replace with

``mail_location = mbox:/mail/%u/:INBOX=/var/mail/%u``

.. _generate_a_kerberos_keytab_for_dovecot_imap_access:

Generate a kerberos keytab for Dovecot IMAP access
--------------------------------------------------

On the IPA server run:

| ``# kinit admin``
| ``Password for admin@EXAMPLE.COM:``
| ``# ipa service-add imap/mail01.example.com``

If successful, you will see the below output

| ``----------------------------------------------------``
| ``Added service "imap/mail01.example.com@EXAMPLE.COM"``
| ``----------------------------------------------------``
| ``  Principal: imap/mail01.example.com@EXAMPLE.COM``
| ``  Managed by: mail01.example.com``

On the Dovecot server run:

| ``# kinit admin``
| ``# ipa-getkeytab -s ds01.example.com -p imap/mail01.example.com -k /etc/dovecot/krb5.keytab``

if successful, you will see the below output:

``Keytab successfully retrieved and stored in: /etc/dovecot/krb5.keytab``

.. _change_the_permissions_of_the_keytab_to_allow_dovecot_to_read_the_file_note_this_should_be_kept_secure_so_only_grant_enough_privileges_as_absolutely_necessary.:

Change the permissions of the keytab to allow Dovecot to read the file (Note, this should be kept secure, so only grant enough privileges as absolutely necessary.)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------

| ``# chown root:dovecot /etc/dovecot/krb5.keytab``
| ``# chmod 640 /etc/dovecot/krb5.keytab``

.. _restart_dovecot:

Restart Dovecot
---------------

| ``# service dovecot restart``
| ``Stopping Dovecot IMAP: ................                           [  OK  ]``
| ``Starting Dovecot IMAP: .                                          [  OK  ]``

.. _send_a_test_email_to_your_user:

Send a test email to your user
------------------------------

From your Dovecot server, run the following command:

``# echo Hello | mail -s Hello user1@example.com``

.. _configure_thunderbird_to_connect_to_imap_server:

Configure Thunderbird to connect to IMAP Server
-----------------------------------------------

#. Open Thunderbird
#. Click the Edit Menu and select Account Settings
#. Under Account Actions (Bottom left), select "Add Mail Account"
#. Enter Name (user1), Email Address(user1@example.com) and leave
   password blank, then click continue
#. Verify the username is user1 (not user1@example.com, Set the imcoming
   server to mail01.example.com, select IMAP, Set port to 993, and
   select SSL/TLS. Then click Manual Setup
#. Select Server Settings under your new mail account
#. Select Kerberos/GSSAPI as the Authentication Method, then click OK
#. Click Get Mail and you will be presented to accept an SSL
   Certificate.
#. Once you have accepted the SSL Certificate, you will see your test
   email you sent in the previous step.

.. _verify_your_authentication_on_the_dovecot_server:

Verify your authentication on the Dovecot server
------------------------------------------------

| ``# tail /var/log/maillog``
| ``Feb 10 13:31:22 mail01 dovecot: imap-login: Login: user=<user1@example.com>, method=GSSAPI, rip=192.168.122.51, lip=192.168.122.63, mpid=1835, TLS``

If everything has worked successfully, you will see in your logs that
your user has connected using the method GSSAPI and has validated their
session over TLS.
