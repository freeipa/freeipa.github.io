Directory_Server
================

This page contains troubleshooting advice for **directory server**
issues. For other issues, refer to the index at
`Troubleshooting <Troubleshooting>`__.



Replication issues
==================

-  If changes done on one FreeIPA master are not replicated to another
   master, always verify ``errors`` log on both master and replica. It
   most often contains exact error what is wrong
-  Make sure that there are no `DNS
   issues <Troubleshooting#DNS_Issues>`__ and both replicas can resolve
   each other's forward and reverse DNS records or that ``/etc/hosts``
   does not contain address of the remote DS with it's short them being
   the primary (first) name.
-  Make sure that the system time difference on the FreeIPA masters is
   not greater than 5 minutes
-  In case of Kerberos issues in the log, verify that the DS keytab is
   correct and can be used to query other master:

      :literal:`# kinit -kt /etc/dirsrv/ds.keytab ldap/`hostname\``
      ``# klist``
      :literal:`# ldapsearch -Y GSSAPI -h `hostname` -b "" -s base`
      ``# ldapsearch -Y GSSAPI -h the.other.master.fqdn -b "" -s base``

-  Make sure that DS communication is not failing because of wrong
   default for SASL communication buffer (`FreeIPA ticket with more
   information <https://fedorahosted.org/freeipa/ticket/4807>`__). This
   can be detected by seeing following error message in
   ``/var/log/messages`` or
   ``systemctl status dirsrv@YOUR-INSTANCE.service`` unit log in case of
   systemd:

      ``[01/Dec/2014:12:00:00 +0100] - sasl_io_recv failed to decode packet for connection xxxx``



Obsolete RUV records
====================

-  If the FreeIPA infrastructure keep getting obsolete RUV records
   (``ipa-replica-manage list-ruv``) which cannot be removed by
   ``ipa-replica-manage clean-ruv`` command and for example give status
   like
   ``RID XX Waiting to process all the updates from the deleted replica...``
   but never finish, make sure that IPv6 is not disabled on the FreeIPA
   replicas (related
   `freeipa-users <https://www.redhat.com/archives/freeipa-users/2015-September/msg00332.html>`__
   thread)
-  Obsolete CA RUV records. FreeIPA < 4.4 doesn't have
   `means <V4/Manage_replication_topology_4_4#ipa-replica-manage>`__ to
   remove obsolete CA RUVs. They usually manifest in directory server
   log as

      ::

         attrlist_replace - attr_replace (nsslapd-referral, ldap://my.ipa.test:389/o%3Dipaca) failed.

      recovery is described at `FreeIPA users list
      post <https://www.redhat.com/archives/freeipa-users/2016-May/msg00043.html>`__



Performance tuning
==================

See `hardware
recommendations <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html-single/Linux_Domain_Identity_Authentication_and_Policy_Guide/index.html#Preparing_for_an_IPA_Installation-Hardware_Requirements>`__
and `Directory Server 11
tuning <https://access.redhat.com/documentation/en-us/red_hat_directory_server/11/html/performance_tuning_guide/index>`__.



Advanced Troubleshooting
========================

See `troubleshooting
section <http://www.port389.org/docs/389ds/FAQ/faq.html#troubleshooting>`__
of `389 Directory Server <http://www.port389.org/>`__
`FAQ <http://www.port389.org/docs/389ds/FAQ/faq.html>`__ including the
information how to retrieve a core dump and a stacktrace if the
`Directory Server <Directory_Server>`__ is crashing.