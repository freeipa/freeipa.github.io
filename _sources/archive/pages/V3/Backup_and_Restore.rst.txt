Overview
--------

`Backup and Restore <Backup_and_Restore>`__ is a loaded topic that means
different things to different people. The general principle that has
driven IPA to date is to run several masters as a way to ensure that
data is preserved in case of catastrophic failure. So back up has meant
continuity of service by keeping several copies of the data in multiple
servers.

Backup can also mean offline backups, for which our answer has been to
fully back up a machine. We have been resistant to providing a list of
files to back up because this has been a moving target as new services
are added and others are enhanced (like switching from the MIT LDAP
backend to our own).

This design will focus on the following scenarios:

#. Critical IPA system backup and restoration
#. LDAP data backup and restoration

To be done later:

#. Partial data restoration (pick and choose entries to restore)

Full metal backup is left as an exercise for the system administrator.
The only requirement is that the backup be done with the IPA services
offline.

.. _use_cases110:

Use Cases
---------

.. _catastrophic_hardware_failure_on_a_machine.:

Catastrophic hardware failure on a machine.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Backup would be full-metal or VM snapshot
-  Restore would be full-metal or restore VM snapshot

Basically, the machine dies. You use a full system backup, done using
your preferred method.

An optional method would be:

-  Reinstall the OS from scratch
-  Configure with same FQDN, hostname
-  Install IPA packages
-  Restore from an IPA full backup

This carries some limited risk that the packages do not match exactly,
or that the administrator forgets to install some optional packages like
bind, bind-dyndb-ldap, samba4-*, etc.

Once restored, replication will handle applying any missing changes. The
replication protocol will detect if a replica is too out of date. In
this case a re-init would be required.

.. _failed_upgrade_on_an_isolated_machine.:

Failed upgrade on an isolated machine.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The OS is still fine but the IPA data is somehow corrupted in such a way
that you want to restore to a known good state.

This one is a bit more complicated, as it is not predictable what an
upgrade might touch. One could in theory simply do a restoration of the
files that we back up and the IPA data but this wouldn't cover, for
example, any new services that were enabled as part of the upgrade.

In other words we could only restore the files we know about. The data
is another matter, see `Returning to a known good
state <V3/Backup_and_Restore#Returning_to_a_known_good_state>`__

.. _restore_accidentally_deleted_data:

Restore Accidentally deleted data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This will need to be covered separately. There are currently no tools or
processes for restoring individual LDAP entries. This will be
investigated with help from the 389-ds team. The issues are:

-  A tool to pick the entries to be restored
-  How to deal with uuid, modifiedby, creator, etc.
-  How to restore a deleted entry w/o the changelog intervening
-  How automatically to deal with membership and other relationships
-  What to do when restoring managed entries

.. _returning_to_a_known_good_state:

Returning to a known good state
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Because we lack the ability to restore individual entries, the most that
one can do is restore to a last known good state. This requires that ALL
masters be restored at the same time, and all offline, until they are
all restored and ready to go.

The issue is the 389-ds changelog. If one server is left with the
corrupted data it will replay that to the other servers after they have
been restored.

The restore program will disable all replication agreements on all
masters before restoring the data.

The act of re-initializing a master will re-enable its agreement.

Design
------

.. _basic_process:

Basic Process
~~~~~~~~~~~~~

As stated previously, there are four basic types of backup/restore, of
which we will provide scripts/process for two.

The four possibilities are:

-  Full system backup/restore
-  Offline full IPA server backup/restore (e.g monthly)
-  Online full LDAP data backup/restore (e.g. daily or weekly)
-  Individual LDAP data backup/restore

This design covers only the middle two.

.. _full_ipa_backup:

Full IPA backup
~~~~~~~~~~~~~~~

.. _files_for_full_ipa_backup:

Files for full IPA backup
^^^^^^^^^^^^^^^^^^^^^^^^^

IPA touches hundreds of files. We will need to back up a mix of specific
files and all files within a set of directories (e.g. certmonger, files
needed for IPA uninstall, etc).

It will be incumbent upon developers to maintain this list as new files
are modified/added to IPA.

Logs will be optionally backed up and restored.

The full list is included at the end of this document.

.. _full_system_backup_process_offline:

Full System Backup Process (offline)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is a raw file-based backup which is why it is done offline. All IPA
services need to be stopped in order to ensure a safe backup.

This will include the LDAP DB files so this is a standalone backup. A
scripted process would include this:

| ``# ipactl stop``
| ``# tar --xattrs --selinux -czf /path/to/backup ``
| ``# ipactl start``

Note that this a simplified view and doesn't include the metadata we
will package as well.

.. _full_system_restore_process:

Full System Restore Process
^^^^^^^^^^^^^^^^^^^^^^^^^^^

| ``# ipactl stop``
| ``# cd / && tar -xzf /path/to/backup``
| ``# ipactl start``
| ``# service sssd restart``

Note that this a simplified view and doesn't include the metadata we
will package as well.

We will not verify in advance of restoration that the system services
match the data (e.g. IPA is configured to start bind but the bind
packages aren't installed). This can be a future enhancement.

LDAP
~~~~

The db2bak method should be used to perform the LDAP backup. This will
also back up (and can restore) the changelog. The scripts provided by
389-ds are not adequate for our purposes because they require either an
operator to provide the DM password or to store it in a file.

The perl equivalents will do online backup/restore which is what we want
but we want it to be seamless. The solution is to provide our own
scripts which uses ldapi and autobind, allowing password-less
backup/restore.

The scripts will need to be robust enough automatically handle the case
of multiple instances (PKI-IPA and the IPA-REALM) as well as the case of
a single instance. For the case of a single instance we will need to
provide the list of backends to backup.

The 389-ds team recommends an LDIF back up as well because it is easier
to move to another machine and is human readable. Therefore we will
perform a db2ldif -r backup at the same time, and store the ldif with
the backup files. The restore command will provide an option to extract
the ldif.

Instances
^^^^^^^^^

Depending on the upgrade path of IPA, there will be one or two 389-ds
instances: one for IPA and one for the CA. Both will be backed up in all
cases. An option for ipa-restore will allow one to conditionally restore
instance data if needed. The possible instances are slapd-REALM and
slapd-PKI-CA

Backends
^^^^^^^^

Depending on the upgrade path of IPA, there will be one or two 389-ds
instances: one for IPA and one for the CA. Both will be backed up in all
cases. An option for ipa-restore will allow one to conditionally restore
instance data if needed. The possible backends are userRoot (basically
$SUFFIX) and o=ipaca.

.. _data_backup_restore_process_online:

Data Backup & Restore Process (online)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The connection will be made to the ldapi port and using autobind as root
we will not be required to provide the DM password.

The default should be to back up and restore both instances (if
installed) or both the IPA (userRoot) and dogtag (ipara) backends.

.. _file_naming_convention:

File Naming Convention
~~~~~~~~~~~~~~~~~~~~~~

Files will be stored in /var/lib/ipa/backup

.. _full_backups:

Full Backups
^^^^^^^^^^^^

ipa-full-%Y-%m-%d-%H-%M-%S.bak

.. _data_backups:

Data Backups
^^^^^^^^^^^^

ipa-data--%Y-%m-%d-%H-%M-%S.bak

.. _version_wrapper:

Version wrapper
~~~~~~~~~~~~~~~

We will want to maintain some metadata with each backup file. I propose
we include:

-  Date/time of backup, store as Generalized Time in GMT
-  FQDN
-  IPA version number (ipapython.version.NUM_VERSION)
-  A backup format version number (start with 1)
-  List of services configured for the master. This may help us now or
   in the future verify that the system can be properly restored.

The format of this file will be a set of name/value pairs separated by
=. Extra white space will be ignored. Comment is #.

We will prevent full restores on a different host.

Data restore should be allowed on any host.

.. _restore_validation:

Restore Validation
~~~~~~~~~~~~~~~~~~

Backing up is easy, restoring is hard, especially verifying that you
actually backed everything up (and restored it properly).

.. _full_restoration:

Full restoration
^^^^^^^^^^^^^^^^

#. Run full backup
#. ipa-server-install --uninstall -U
#. ipa-server-install
#. Restore backup

.. _data_restoration:

Data restoration
^^^^^^^^^^^^^^^^

In the case of a single IPA master you can:

#. Back up data
#. Delete some data
#. Restore from backup

Confirm that the data was restored. This will **not** automatically sync
the restored data to other masters. Any pending changes will not be
applied to the restored master but similarly any changes restored will
not be sent out to the other masters. After the restoration the other
masters will need to be reinitialized from the restored master:

``# ipa-replica-manage re-initialize --from=``

Replication
~~~~~~~~~~~

Because we are going to backup and restore the changelog we should be ok
when it comes to replication.

Agreements
^^^^^^^^^^

A very big issue will be what agreements exist at the time that a backup
is made and restored.

For example, lets say you have a single IPA server. You add a bunch of
records and then take a backup.

You add a replica, maybe even delete some records (oops).

So you do a restore.

Your data is back but your agreement is now gone because you restored a
backup from prior to the agreement! The remote server will need to
uninstalled and re-installed (no re-initialize is possible because the
restored server doesn't know about the replica at all).

This could potentially strand a number of servers.



External Impact
~~~~~~~~~~~~~~~

The sssd service will need a restart. If the assumption is that the
server is not in a known good state then it would be good practice to
restart this service after restoring its files.

In fact, we may want to consider recommending a reboot to be sure things
are in a good state, or we may need to think about extending ipactl to
include other daemons.

.. _more_on_partial_restores:

More on partial restores
~~~~~~~~~~~~~~~~~~~~~~~~

Quite a bit of infrastructure is required to be able to pick and choose
what to restore from a backup. In order to provide per-entry restoration
we would need the backup in a more readable form, say LDIF, then provide
a means to search for, pick and then execute restoration.

The restoration may take the form of:

-  an entry
-  a subtree
-  attributes within an entry, e.g. membership

Restoration of an entry may trigger other things to happen. Take the
case where a group is accidentally removed. Not only does the group need
to be restored but its membership needs to be recovered as well. Members
of the group will be managed automatically but since we handle nested
groups and groups can be members of other objects (HBAC, sudo, etc) we
need to restore that as well.

Qualifying
~~~~~~~~~~

Here is a list of some things to test

-  Run IPA unit tests
-  Create a new replica
-  Manage existing replicas
-  Enroll a client
-  Unenroll a client
-  Verify that replication is still working, and working with dogtag as
   well

.. _open_questions:

Open Questions
~~~~~~~~~~~~~~

.. _size_of_backup:

Size of backup?
^^^^^^^^^^^^^^^

Should we attempt to predict the resulting file size and try to
determine if there is adequate space before starting the backup? We may
be able to stat each file, sum the size, and check. It would just take a
bit of time and I/O.

.. _encrypt_backup_files:

Encrypt backup files?
^^^^^^^^^^^^^^^^^^^^^

Should we prompt for and/or encrypt with gpg the backup files? **Yes**

.. _should_i_delete_everything_before_doing_a_restore:

Should I delete everything before doing a restore?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For example, if you have a single master, you do a backup, then you add
a replica. If you then restore the backup and try to create another
replica it will fail because the changelog directory already exists. Who
knows what other problems might be lurking.

I'm inclined to suggest/force uninstalling the server first. We just may
not be in any position to do that depending on how hosed things are.

The other alternative is to create a list of these corner cases and test
for them on reinstall.

Implementation
--------------

.. _full_restore:

Full Restore
~~~~~~~~~~~~

If you do a full backup without the logs and do a restoration into a FS
that doesn't have an installed IPA server then tomcat will not stop.
This is because the log files needed by the CA are created on-the-fly by
the instance creation process. If the directory structure is created
manually then things will work.

Uninstall
~~~~~~~~~

The backup files are NOT removed on uninstall. When it comes to data, I
prefer not to delete things automatically.

.. _development_notes_semi_interesting_testing:

Development notes (semi-interesting testing)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As part of developing the backups I tried a couple of fairly outlandish
things. Here are those things and the outcomes. I'm not sure if these
will ever be eventually interesting or helpful, but I don't want to lose
anything.

.. _backup_uninstall_reinstall_restore_just_the_ldap_server:

Backup, uninstall, reinstall, restore JUST the LDAP server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

So I wanted to verify that the restoration actually worked, so what I
did was:

#. ipa-server-install ...
#. kinit admin
#. ipa user-add tuser1
#. ipa user-add tuser2
#. db2bak
#. ipa-server-install --uninstall -U
#. ipa-server-install (same options as above)
#. bak2db
#. ipa-getkeytab -k /etc/dirsrv/ds.keytab -p ldap/`hostname\` -s
   \`hostname\` -D 'cn=directory manager' -w password
#. service dirsrv restart
#. kdestroy
#. kinit admin
#. ipa-getkeytab -k /etc/httpd/conf/ipa.keytab -p HTTP/`hostname\` -s
   \`hostname\`
#. ipa-getkeytab -k /etc/krb5.keytab -p host/`hostname\` -s \`hostname\`
#. ipactl restart
#. service sssd restart
#. ipa user-show admin
#. ipa user-find (confirm that I have the 2 new users)
#. id tuser1 (to confirm that sssd is working)

So what does this do? Well, it replaces the CA for one. And it
invalidates all certificates.

It is also, if all you have is the data backup, is a way to restore an
IPA system.

A lot more work would be needed to actually make things work. All
clients and services would need new certs.

And we overwrote the 389-ds and Apache server certs when we reimported
the data, so those would need to be re-issued.

For the most part, any certificates in the data should be deleted
because they are for a CA that no longer exists, so revocation will
fail.

There may be quite a bit of certmonger rework needed, or it could be
that certmonger could fix all the certs for us using: ipa-getcert
resubmit.

References
~~~~~~~~~~

-  https://access.redhat.com/knowledge/docs/en-US/Red_Hat_Directory_Server/9.0/html/Administration_Guide/Populating_Directory_Databases-Backing_Up_and_Restoring_Data.html#Backing_Up_and_Restoring_Data-Restoring_Databases_That_Include_Replicated_Entries
-  https://access.redhat.com/knowledge/docs/en-US/Red_Hat_Directory_Server/9.0/html/Administration_Guide/Managing_Replication-Initializing_Consumers.html#Initializing_Consumers-Filesystem_Replica_Initialization



Feature Management
------------------

UI
~~

The backup/restore commands will need to be executed as root so it is
unlikely that system backup/recovery can be managed from the UI. It
could also represent a chicken-and-egg problem on restoration.

CLI
~~~

There will be two basic, standalone commands:

| ``ipa-backup OPTIONS``
| ``   --data    Back up just the data. Default is full system backup.``
| ``   --gpg     Encrypt the backup``
| ``   --gpg-keyring ``\ ``   The gpg key name to be used (or full path)``
| ``   --logs    Include logs in backup``
| ``   --online Perform the LDAP backups online, for data only.``

We will only encrypt the payload. The header will be in the clear.

| ``ipa-restore OPTIONS /path/to/backup``
| ``   --data             If the backup is a full backup, restore only the data``
| ``   --extract        Extract the backup files, do not restore (including the LDIF)``
| ``   --gpg-keyring ``\ ``    The key name to be used by gpg``
| ``   --data             Restore only the data``
| ``   --online          Perform the LDAP restores online, for data only.``
| ``   --instance=INSTANCE   The 389-ds instance to restore (defaults to all found)``
| ``   --backend=BACKEND     The backend to restore within the instance or``
| ``                                              instances``
| ``   --no-logs         Do not restore log files from the backup``
| ``   -U, --unattended      Unattended restoration never prompts the user``

ipa-restore will detect if the backup file provide contains only the
data, but if provided a full backup it should be able to restore just
the data component.

There are also common options:

| ``   --version             show program's version number and exit``
| ``   -h, --help            show this help message and exit``
| ``   -p PASSWORD, --password=PASSWORD``
| ``                       Directory Manager password``
| ``   -v, --verbose       print debugging information``
| ``   -d, --debug         alias for --verbose (deprecated)``
| ``   -q, --quiet         output only errors``
| ``   --log-file=FILE     log to the given file``

.. _full_list_of_files_and_directories_to_back_up:

Full list of files and directories to back up
---------------------------------------------

Directories
~~~~~~~~~~~

-  /usr/share/ipa/html
-  /etc/pki-ca
-  /etc/httpd/alias
-  /var/lib/pki-ca
-  /var/lib/ipa-client/sysrestore
-  /var/lib/sss/pubconf/krb5.include.d
-  /var/lib/authconfig/last
-  /var/lib/certmonger
-  /var/lib/ipa
-  /var/run/dirsrv
-  /var/lock/dirsrv

Files
~~~~~

-  /etc/named.conf
-  /etc/sysconfig/pki-ca
-  /etc/sysconfig/dirsrv
-  /etc/sysconfig/ntpd
-  /etc/sysconfig/krb5kdc
-  /etc/sysconfig/pki/ca/pki-ca
-  /etc/sysconfig/authconfig
-  /etc/resolv.conf
-  /etc/pki/nssdb/cert8.db
-  /etc/pki/nssdb/key3.db
-  /etc/pki/nssdb/secmod.db
-  /etc/nsswitch.conf
-  /etc/krb5.keytab
-  /etc/sssd/sssd.conf
-  /etc/openldap/ldap.conf
-  /etc/security/limits.conf
-  /etc/httpd/conf/password.conf
-  /etc/httpd/conf/ipa.keytab
-  /etc/httpd/conf.d/ipa-pki-proxy.conf
-  /etc/httpd/conf.d/ipa-rewrite.conf
-  /etc/httpd/conf.d/nss.conf
-  /etc/httpd/conf.d/ipa.conf
-  /etc/ssh/sshd_config
-  /etc/ssh/ssh_config
-  /etc/krb5.conf
-  /etc/group
-  /etc/passwd
-  /etc/ipa/ca.crt
-  /etc/ipa/default.conf
-  /etc/named.keytab
-  /etc/ntp.conf
-  /etc/dirsrv/ds.keytab
-  /etc/sysconfig/dirsrv-REALM
-  /etc/sysconfig/dirsrv-PKI-IPA
-  /root/ca-agent.p12
-  /root/cacert.p12
-  /var/kerberos/krb5kdc/kdc.conf
-  /etc/dirsrv/slapd-REALM
-  /var/lib/dirsrv/scripts-realm
-  /var/lib/dirsrv/slapd-realm
-  /usr/lib64/dirsrv/slapd-PKI-IPA
-  /etc/dirsrv/slapd-PKI-IPA
-  /var/lib/dirsrv/slapd-PKI-IPA

Logs
~~~~

This is a mix of files and directories

-  /var/log/pki-ca
-  /var/log/dirsrv/slapd-REALM-COM
-  /var/log/dirsrv/slapd-PKI-IPA
-  /var/log/httpd
-  /var/log/ipaserver-install.log
-  /var/log/kadmind.log
-  /var/log/pki-ca-install.log
-  /var/log/messages
-  /var/log/ipaclient-install.log
-  /var/log/secure
-  /var/log/ipaserver-uninstall.log
-  /var/log/pki-ca-uninstall.log
-  /var/log/ipaclient-uninstall.log
-  /var/named/data/named.run

.. _gpg_encryption:

GPG encryption
--------------

The backup can be optionally encrypted using GPG. To create a key you
can run:

::

   # cat >keygen <<EOF
        %echo Generating a standard key
        Key-Type: RSA
        Key-Length: 2048
        Name-Real: IPA Backup
        Name-Comment: IPA Backup
        Name-Email: root@example.com
        Expire-Date: 0
        %pubring /root/backup.pub
        %secring /root/backup.sec
        %commit
        %echo done
   EOF
   # gpg --batch --gen-key keygen
   # gpg --no-default-keyring --secret-keyring /root/backup.sec \
         --keyring /root/backup.pub --list-secret-keys

This will create the key ``backup`` and can be passed to ipa-backup
using:

``# ipa-backup --gpg --gpg-keyring=/root/backup ...``

Troubleshooting
---------------

gpg2 now requires an external program to enter pins to make it "easier"
for desktop folks.

To run purely from a console add
``"pinentry-program /usr/bin/pinentry-curses"`` to
``.gnupg/gpg-agent.conf`` before generating a key.

.. _how_to_test9:

How to Test
-----------

.. _general_test_outline:

General test outline
~~~~~~~~~~~~~~~~~~~~

-  Install server
-  Do a LDAP search for ``uid=admin,cn=users,cn=accounts,$SUFFIX``. Note
   the result.
-  Verify that the commands ``ipa user-show admin``, ``id admin``,
   ``ipa cert-find``, ``host``\ *``$HOSTNAME``*\ ``localhost``,
   ``kinit admin`` work. This checks basic functionality of IPA client,
   PAM, CA, DNS and Kerberos. Note the output of these commands
-  (Do backup & restore)
-  Do a LDAP search on admin again; check that all attributes except
   ``krbLastSuccessfulAuth`` match
-  Run the above commands again, check that they are successful and the
   output matches.
-  Uninstall server

.. _test_full_backup_and_restore:

Test Full Backup and Restore
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The "Do backup & restore" steps are:

-  ``ipa-backup -v``
-  Uninstall server
-  ``ipa-restore``\ *``$BACKUP_PATH``*

.. _test_backup_and_restore_with_removed_users:

Test Backup and Restore with Removed Users
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The "Do backup & restore" steps are:

-  ``ipa-backup -v``
-  Uninstall server
-  Remove users ``dirsrv`` and ``pkiuser``
-  Add system user ``ipatest_user1`` (to claim the UID of a removed
   user)
-  ``ipa-restore``\ *``$BACKUP_PATH``*

At the end of the test, remove user ``ipatest_user1``

.. _test_backup_and_restore_with_selinux_booleans_off:

Test Backup and Restore with SELinux Booleans Off
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The "Do backup & restore" steps are:

-  ``ipa-backup -v``
-  Uninstall server
-  Turn SELinux booleans ``httpd_can_network_connect`` and
   ``httpd_manage_ipa`` off
-  ``ipa-restore``\ *``$BACKUP_PATH``*

After restoring, check that the above booleans are on.

.. _test_backup_and_restore_from_heavily_upgraded_instance:

Test Backup and Restore from heavily upgraded instance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Start with a master that has been in-place-upgraded since there was a
separate 389-ds instance for IPA.

-  ``ipa-backup -v``
-  Uninstall server
-  ``ipa-restore ...``

Backup and Restore is NOT a method of eliminating that extra instance.

.. _data_backup_only:

Data backup only
~~~~~~~~~~~~~~~~

-  ``ipa-backup --data``
-  Add a new user
-  ``ipa-restore /var/lib/ipa/ipa-data-...``
-  Ensure that the new user is gone

.. _online_data_restore:

Online data restore
~~~~~~~~~~~~~~~~~~~

-  ``ipa-backup --data``
-  Add a new user
-  ``ipa-restore --online /var/lib/ipa/ipa-data-...``
-  Ensure that the new user is gone
-  Ensure IPA is still functioning properly

.. _encryptiondecryption_of_backup_files:

Encryption/decryption of Backup files
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  ``ipa-backup --gpg --gpg-keyring=/path/to/keyring``
-  ``ipa-server-install --uninstall -U``
-  ``ipa-restore --gpg-keyring=/path/to/keyring /var/lib/ipa/ipa-...``

.. _clientreplica_installation_with_restored_master:

Client/Replica installation with restored Master
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  ``ipa-backup``
-  ``ipa-restore``
-  Create new replica
-  Enroll client
