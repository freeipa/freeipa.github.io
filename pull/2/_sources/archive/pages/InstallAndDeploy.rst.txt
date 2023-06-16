.. _installing_the_ipa_server:

Installing the IPA Server
=========================

Introduction
------------

This page provides instructions on how to download the freeIPA server
software, and to get it installed and configured on your system. It also
provides information on common problems and possible solutions.

Assumptions
~~~~~~~~~~~

FreeIPA relies on a script to automate many of the installation tasks.
This script makes a number of assumptions.

-  **System:**

   That you are performing the installation on a "clean" system. The
   script overwrites a number of files without prompting for
   confirmation. `Configuring the FreeIPA
   Server <#Configuring_the_IPA_Server>`__ lists the services whose
   scripts and configuration files are modified.

-  **Directory Server:**

   That there are no existing Directory Server instances. There is
   currently no automatic upgrade facility from an existing Directory
   Server. Any migration requirements should be referred to Red Hat
   Global Professional Services.

-  **DNS:**

   -  That the server's machine name is set, and that it resolves to its
      public IP address (not to localhost).
   -  That DNS is correctly configured to resolve forward and reverse
      addresses. The DNS does not need to be on the same machine as the
      IPA Server, but it does need to be fully functional.

.. _required_ports:

Required Ports
~~~~~~~~~~~~~~

FreeIPA makes use of the following ports:

-  TCP

   -  80, 443: HTTP/HTTPS
   -  389, 636: LDAP/LDAPS
   -  88, 464: Kerberos

-  UDP

   -  88, 464: Kerberos
   -  123: NTP

You need to ensure that these ports are available; that is, that they
are not assigned to another service. If these ports are not available,
IPA will not function correctly.

.. _file_systems:

File Systems
~~~~~~~~~~~~

The default root directory for users' home directories is ``/home``, but
it is the responsibility of the system administrator to ensure that
whatever value is specified for this attribute is actually available.

Red Hat Enterprise Linux and most other Linux distributions include a
pam module called *pam_mkhomedir* that can be used to automatically
create a home directory if one does not exist for the user
authenticating against the system. IPA does not force the use of this
module because it may try to create home directories even when the
shared storage is simply not available. It is the responsibility of the
system administrator to activate this module on the clients if needed.

If a suitable directory and mechanism are not available for the creation
of home directories, users may not be able to log in successfully.

DNS
~~~

It is recommended that you use DNS to facilitate Service Discovery in
IPA. Service Discovery refers to the way that IPA clients find (or
*discover*) IPA servers. You can use the basic DNS configuration that is
provided with IPA to configure an existing DNS to work with IPA, or use
``--setup-bind`` option to configure a new DNS. The DNS does not need to
be on the same machine as the IPA server, but it does need to be
correctly configured and fully functional.

   |Note.png|\ **Note:**

      The ``--setup-bind`` option is an optional parameter that can be
      passed to the ``ipa-server-install`` script. This is provided for
      convenience only; it is not a supported aspect of IPA. The
      following article may help you to configure your DNS server: `How
      to set up a DNS
      server <http://www.redhat.com/magazine/025nov06/features/dns/>`__

To aid in the creation and configuration of a suitable DNS setup, the
IPA installation creates a sample zone file. During the installation you
will see a message similar to the following:

   ``Sample zone file for bind has been created in /tmp/sample.zone.F_uMf4.db``

You should use this file in your zone file in DNS. Further, you need to
ensure that your FQDN does not resolve to your loopback address.

.. _ipa_dns_and_nscd:

IPA, DNS, and NSCD
^^^^^^^^^^^^^^^^^^

It is recommended that you avoid or restrict the use of nscd (Name
Service Caching Daemon) in an IPA deployment. While nscd is extremely
useful for reducing the load on the server, and for making clients more
responsive, there are also drawbacks to its use.

nscd performs caching operations for all services that perform queries
via the nsswitch interface, including getent. Because it performs both
positive and negative caching, if it determines that a specific IPA user
does not exist when a request is made, it marks this as a negative
cache. Values stored in the cache remain until the cache expires,
regardless of any changes that may have occurred on the server.

The results of such caching is that new users and memberships may not be
visible, and users and memberships that have been removed may still be
visible.

To alleviate the effects of this, you can avoid the use of nscd
altogether, or change the default settings to use a shorter cache time.
In particular, consider changing the following values in the
``/etc/nscd.conf`` file to suit the usage patterns of your deployment:

::

   positive-time-to-live   group           3600
   negative-time-to-live   group           60
   positive-time-to-live   hosts           3600
   negative-time-to-live   hosts           20

.. _dns_and_kerberos:

DNS and Kerberos
^^^^^^^^^^^^^^^^

Kerberos, too, has very specific DNS requirements. Your Kerberos server
requires a proper DNS A record, and reverse DNS needs to work correctly.
Do not use CNAME or DDNS names, as it can cause major problems later.
The IPA installation process includes checks to ensure that the IPA
server name is a DNS A record and that its reverse matches its forward
address.

Refer to `IPA and
DNS <http://www.freeipa.com/page/IpaConcepts#IPA_and_DNS>`__ for more
information on how these technologies work together.

.. _configuring_etchosts:

Configuring /etc/hosts
~~~~~~~~~~~~~~~~~~~~~~

You need to ensure that your ``/etc/hosts`` file is configured
correctly, or the **ipa-\*** commands may not work correctly.

The ``/etc/hosts`` file should list the FQDN for your IPA server
*before* any aliases. You should also ensure that the hostname is not
part of the localhost entry. The following is an example of a valid
hosts file:

::

   127.0.0.1       localhost.localdomain   localhost
   ::1     localhost6.localdomain6 localhost6
   192.168.1.1     ipaserver.example.com      ipaserver

.. _hardware_requirements:

Hardware Requirements
~~~~~~~~~~~~~~~~~~~~~

The following table contains guidelines for Directory Server disk space
and memory requirements based on on the number of entries that your
organization requires. The values shown here assume that the entries in
the LDIF file are approximately 100 bytes each and that only the
recommended indices are configurable.

The system requirements for both 32-bit and 64-bit platforms are the
same.

.. table:: **Operating system hardware requirements for IPA 1.0 Server**

   +-----------------------------------+-----------------------------------+
   | **Criteria**                      | **Requirements**                  |
   +===================================+===================================+
   | Operating System                  | Red Hat Enterprise Linux 5.1      |
   |                                   | Server or later, with the latest  |
   |                                   | patches and upgrades Fedora 7 or  |
   |                                   | later, with the latest patches    |
   |                                   | and upgrades                      |
   +-----------------------------------+-----------------------------------+
   | CPU Type                          | Pentium 3 or higher; 500MHz or    |
   |                                   | higher                            |
   +-----------------------------------+-----------------------------------+
   | Required Memory                   | +--------------+--------------+   |
   |                                   | | **Entries**  | **RAM**      |   |
   |                                   | +==============+==============+   |
   |                                   | | 10,000 -     | 256 MB       |   |
   |                                   | | 250,000      | minimum      |   |
   |                                   | +--------------+--------------+   |
   |                                   | | 250,000 -    | 512 MB       |   |
   |                                   | | 1,000,000    | minimum      |   |
   |                                   | +--------------+--------------+   |
   |                                   | | 1,000,000 +  | 1 GB minimum |   |
   |                                   | +--------------+--------------+   |
   +-----------------------------------+-----------------------------------+
   | Hard Disk                         | +--------------+--------------+   |
   |                                   | | **Entries**  | **Disk       |   |
   |                                   | |              | Space**      |   |
   |                                   | +==============+==============+   |
   |                                   | | 10,000 -     | 2GB          |   |
   |                                   | | 250,000      |              |   |
   |                                   | +--------------+--------------+   |
   |                                   | | 250,000 -    | 4GB          |   |
   |                                   | | 1,000,000    |              |   |
   |                                   | +--------------+--------------+   |
   |                                   | | 1,000,000 +  | 8GB          |   |
   |                                   | +--------------+--------------+   |
   +-----------------------------------+-----------------------------------+
   |                                   |                                   |
   +-----------------------------------+-----------------------------------+

.. _installing_the_ipa_server_packages:

Installing the IPA Server Packages
----------------------------------

This document distinguishes between commands to be run as root versus a
regular user. Commands to be run as root are prefixed with a # symbol.
Commands to be run as a regular user are prefixed with a $ symbol.

   |Note.png|\ **Note:**

      Before starting the freeIPA installation, ensure that you update
      your system with all the latest packages.
      If you are installing on 64-bit Red Hat Enterprise Linux 5.1, you
      need to update the **krb5libs** package *before* you install the
      **ipa-server** package.

1. freeIPA is currently only in the Fedora 7 and 8 updates-testing
repository. It is in the regular repository for rawhide (Fedora 9). To
install freeIPA you will need to enable the updates-testing repository.
You can do this either by editing the
``/etc/yum.repos.d/updates-testing.repo`` file, or on the command line,
as shown in step 2:

2. Run the following command to install the IPA server packages:

::

   # yum install --enablerepo=updates-testing ipa-server

For Fedora 9 you do not need to include ``--enablerepo=updates-testing``

This will install a large number of dependencies, including
**TurboGears**, **fedora-ds-base** and **krb5-server**. Approximately 40
dependencies are required, depending on what is already installed.

3. freeIPA no longer requires a special **mod_auth_kerb** package but it
does require a specific version of **krb5-libs** which contains a fix
for Kerberos ticket delegation (look for "spnego" in the Changelog). The
dependency should be handled by the **ipa-server** rpm, but if you want
to verify manually, you need:

-  Fedora 7: 1.6.1-7 or higher
-  Fedora 8: 1.6.2-11 or higher
-  Fedora 9: all versions should be ok

If you previously installed the IPA-specific **mod_auth_kerb** package,
you can remove it and replace it with the default Fedora version and
ensure you have the minimum version of **krb5-libs** as listed above.

   |Note.png|\ **Note:**

      If you are installing on Fedora 9, it is strongly recommend that
      you *not* use **NetworkManager**. Instead, run the following
      commands to use **network** to manage the network service:

   ::

      # chkconfig NetworkManager off
      # chkconfig network on
      # service network start

Now you are ready to configure your IPA server.

.. _configuring_the_ipa_server:

Configuring the IPA Server
--------------------------

Use the **ipa-server-install** command to install the IPA server, which
includes:

-  Configuring the Network Time Daemon (ntpd)
-  Creating and configuring an instance of Directory Server
-  Creating and configuring a Kerberos Key Distribution Center (krb5kdc)
-  Configuring Apache (httpd)
-  Configuring TurboGears
-  Updating the SELinux targeted policy

You can install the server interactively by running the command with no
options, or by passing options directly to the **ipa-server-install**
command. To view the available command-line options, run
``/usr/sbin/ipa-server-install --help``

   |Note.png|\ **Note:**

      If you are running IPA as a virtualized guest, you should not run
      the ntp daemon. In this case, you should pass the *-N* (no ntp)
      option to the **ipa-server-install** command.

**To install the freeIPA server interactively:**

1. Run the following command:

   ``# ipa-server-install``

2. When prompted, enter the server host name, realm name and other
details.

   The installation script compares the hostname returned by DNS to the
   hostname found in the ``/etc/hosts`` file. If the
   non-fully--qualified domain name appears first, the script aborts.

..

   |Note.png|\ **Note:**

      The hostname that you enter into the ipa-server-install script
      must be the same as that returned by the hostname command,
      otherwise the Directory Server cannot use its own keytab. This
      results in commands such as ipa-finduser to fail.

3. Wait until the configuration script completes. Note that it can take
several minutes to set up and configure all of the freeIPA requirements.

4. When the configuration script has completed, you should either reboot
the server or at least restart the ssh server so that the Name Server
Switch (nss) configuration is read when the service restarts.

   To restart the ssh service, run the following command (existing
   connections are not terminated):

   ``# service sshd restart``

You can now proceed to test the configuration.

.. _testing_the_configuration:

Testing the Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~

The following examples assume that you are using EXAMPLE.COM as your
realm.

   |Note.png| **Note:**

      The realm is used as the base DN in the directory instance, so it
      will be *dc=example,dc=com*.

When the installation is complete, all of the services should be
running. You can test your installation as follows:

1. Use the **kinit** command to request a Kerberos ticket:

::

    $ kinit admin
    Password for admin@EXAMPLE.COM:

2. Use the **klist** command to display the list of Kerberos tickets:

::

    $ klist
    Ticket cache: FILE:/tmp/krb5cc_0
    Default principal: admin@EXAMPLE.COM
    Valid starting     Expires            Service principal
    03/05/08 02:47:53  03/06/08 02:47:50  krbtgt/EXAMPLE.COM@EXAMPLE.COM
    Kerberos 4 ticket cache: /tmp/tkt0
    klist: You have no tickets cached

3. Use the **ipa user-find** command to search for the admin user:

::

    $ ipa-finduser admin
    cn: Administrator
    homedirectory: /home/admin
    loginshell: /bin/bash
    uid: admin

If you receive output similar to the following, ensure that DNS is
configured correctly:

| ``Could not initialize GSSAPI: Unspecified GSS failure.``
| ``Minor code may provide more information/Server not found in Kerberos database.``

.. _configuring_your_browser:

Configuring your Browser
------------------------

Firefox can use your Kerberos credentials for authentication, but you
need to specify which domains you want to communicate with, and using
which attributes.

1. Open Firefox, and type "about:config" in the Address Bar.

2. In the Search field, type "negotiate".

3. Ensure the following lines reflect your setup. Replace "example.com"
with your own IPA server's domain, with a preceding period (.):

::

    network.negotiate-auth.trusted-uris  .example.com
    network.negotiate-auth.delegation-uris  .example.com
    network.negotiate-auth.using-native-gsslib true

4. In Firefox, navigate to your IPA server (use the fully-qualified
domain name, for example, http://ipaserver.example.com). Ensure that
there are no Kerberos authentication errors, and that you can see and
interact with the Web interface.

.. _using_a_browser_on_another_system:

Using a Browser on Another System
---------------------------------

Use the following procedure to set up a browser on another system that
already has Kerberos set up for a different realm.

1. Copy the ``/etc/krb5.conf`` file from the IPA server to the client
system. Do not overwrite the existing ``krb5.conf`` file.

::

   # scp /etc/krb5.conf root@ipaclient:/etc/krb5_ipa.conf

2. On the IPA client, open a terminal and run the following commands:

::

   $ export KRB5_CONFIG=/etc/krb5_ipa.conf
   $ kinit user@EXAMPLE.COM
   $ /usr/bin/firefox

3. Configure the Firefox **negotiate** attributes as described in the
`Configuring your Browser <#Configuring_your_Browser>`__ section.

Now you should be able to connect to the IPA Web interface remotely.

.. _setting_up_multi_master_replication:

Setting up Multi-Master Replication
===================================

Replication is the mechanism by which directory data is automatically
copied from one Directory Server to another. Updates of any kind —
adding, modifying, or deleting entries — are automatically mirrored to
other Directory Servers using replication.

IPA 1.0 uses a number of scripts to install, configure, and manage
replica servers and replication agreements. These are discussed in the
following sections.

.. _preparing_the_replica_servers:

Preparing the Replica Servers
-----------------------------

Replica servers require much the same preparation as IPA servers. That
is, there should be no existing Directory Server installations, the
ports required by IPA must be free and available, and the server's
machine name be set and resolve to its public IP address (not to
localhost). The replica server must also be able to contact the master
LDAP server, so DNS or a similar lookup system must be working
correctly.

Refer to `Introduction to Installing IPA <#Introduction>`__ for more
information about these and other considerations for installing an IPA
server.

.. _installing_the_server_packages:

Installing the Server Packages
------------------------------

Follow the steps in `Installing the IPA Server
Packages <#Installing_the_IPA_Server_Packages>`__ to install all of the
required packages for the replica server.

   |Note.png|\ **Note:**

      Do not run the **ipa-server-install** script on the replica
      servers.

.. _creating_the_replica_information_file:

Creating the Replica Information File
-------------------------------------

You need to create a *replica information file* for each replica that
you intend to create. This file contains various realm information
required to correctly configure the replica server.

Before you create the replica information file, you need to ensure that
the master IPA server is correctly configured and functioning properly.
The master IPA server is the server from which all IPA replica servers
will be created.

**To create the replica information file:**

Run the following command on the master IPA server, where
*ipareplica.example.com* is the FQDN of the server where you are going
to create the replica:

::

   # ipa-replica-prepare ipareplica.example.com

This should produce output similar to the following:

::

   Determining current realm name
   Getting domain name from LDAP
   Preparing replica for ipareplica.example.com from ipaserver.example.com
   Creating SSL certificate for the Directory Server
   Creating SSL certificate for the Web Server
   Copying additional files
   Finalizing configuration
   Packaging the replica into replica-info-ipareplica.example.com

..

   |Note.png| **Note:**

      Each replica information file is created in the ``/var/lib/ipa/``
      directory, and named specifically for the replica server for which
      it is intended. You cannot use the same replica information file
      for multiple replicas.

.. _configuring_an_ipa_replica:

Configuring an IPA Replica
--------------------------

After you have created the replica information file, you need to copy it
to the replica server and run the required script to configure the
replica.

**To configure an IPA replica:**

1. Copy the replica information file to the replica server:

   ``# scp /var/lib/ipa/replica-info-ipareplica.example.com root@ipareplica:/var/lib/ipa/``

2. On the replica server, run the replica installation script, passing
it the replica information file you copied from the master:

   ``# ipa-replica-install /var/lib/ipa/replica-info-ipareplica.example.com``

   The replica installation script runs a test to ensure that the
   replica file being installed matches the current hostname. If they do
   not match, the script returns a warning message and asks for
   confirmation. This could occur on a multi-homed machine, for example,
   where mismatched hostnames may not be an issue.

3. Enter the Directory Manager (DM) password when prompted.

The script then configures a directory server instance based on
information in the replica information file, and initiates a replication
process. When this has successfully completed, the script continues to
set up a complete master replica of the IPA server.

   |Note.png| **Note:**

      You can only have a single Directory Server instance on an IPA
      server, the one used by IPA itself. If the replica installation
      script detects an existing Directory Server instance, you will be
      prompted to remove it.

.. _updating_dns_for_ipa_replicas:

Updating DNS for IPA Replicas
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After you have configured a new IPA replica, you should update your DNS
entries so that IPA clients can discover the new server. For example,
for an IPA replica with a server name of $HOST, you should add the
following entries to your zone file:

::

   _ldap._tcp             IN SRV 0 100 389 $HOST
   _kerberos._tcp         IN SRV 0 100 88 $HOST
   _kerberos._udp         IN SRV 0 100 88 $HOST
   _kerberos-master._tcp  IN SRV 0 100 88 $HOST
   _kerberos-master._udp  IN SRV 0 100 88 $HOST
   _kpasswd._tcp          IN SRV 0 100 464 $HOST
   _kpasswd._udp          IN SRV 0 100 464 $HOST
   _ntp._udp              IN SRV 0 100 123 $HOST

.. _managing_multi_master_replication:

Managing Multi-Master Replication
---------------------------------

You can use the ``ipa-replica-manage`` command to manage certain aspects
of replication between IPA servers. This includes listing, adding, and
deleting replication agreements, and also performing manual replication
initialization and updates.

Initialization is typically only required when you first set up
replication, or if a problem arises that causes replication to fail.
Initialization erases all data on the target replica and re-copies all
data from the master. That is, it completely destroys the database on
the consumer and rebuilds it with data from the master.

Sending updates is the regular incremental replication protocol.
Typically, this is not needed because the server sends changes when
required, provided that the replication agreement schedule allows it.

Refer to the ``ipa-replica-manage`` man page for a full description of
the available options.

   |Note.png| **Note:**

      There is no WebUI facility for managing IPA replicas. You need to
      use the command line.

Refer to the `Managing
Replication <http://www.redhat.com/docs/manuals/dir-server/ag/8.0/Managing_Replication.html>`__
section of the Red Hat Directory Server Administration Guide for
information about managing replication.

.. _troubleshooting_multi_master_replication:

Troubleshooting Multi-Master Replication
----------------------------------------

Refer to the following sections in the Red Hat Director Server
Administration Guide for information about troubleshooting replication:

-  `Solving Common Replication
   Conflicts <http://www.redhat.com/docs/manuals/dir-server/ag/8.0/Managing_Replication-Solving_Common_Replication_Conflicts.html>`__
-  `Troubleshooting Replication-Related
   Problems <http://www.redhat.com/docs/manuals/dir-server/ag/8.0/Managing_Replication-Troubleshooting_Replication_Related_Problems.html>`__

.. _running_ipa_in_a_virtual_host:

Running IPA in a Virtual Host
=============================

If you have a standard Apache instance running on port 80, you can
configure IPA to run on a secondary port, for example 8089. You should
be aware, however, that running IPA in this configuration does not use
SSL; all requests will go over standard HTTP.

.. _converting_your_ipa_configuration_to_run_as_a_virtualhost:

Converting Your IPA Configuration to run as a VirtualHost
---------------------------------------------------------

The following procedure assumes you already have IPA configured to run
on port 80, and wish to move it to a different port.

1. Log in as the root user.

2. Edit the ``/etc/httpd/conf.d/ipa.conf`` file. Add the following three
lines to the top of the file:

::

   Listen 8089
   NameVirtualHost *:8089
   <VirtualHost *:8089>

3. Add the following line to the end of the file:

::

   </VirtualHost>

This wraps the entire IPA configuration in a VirtualHost, and ensures
that Apache is listening to that port.

   |Note.png|\ **Note:**

      You can not use port 8080. This port is used by the **ipa_webgui**
      service.

4. Comment out the following rewrite rules from the
``/etc/httpd/conf.d/ipa.conf`` file:

::

   ----------------------------------------------------------------------
   # Redirect to the fully-qualified hostname. Not redirecting to secure
   # port so configuration files can be retrieved without requiring SSL.
   RewriteCond %{HTTP_HOST}    !^host.foo.com$ [NC]
   RewriteRule ^/(.*)          http://host.foo.com/$1 [L,R=301]

   # Redirect to the secure port if not displaying an error or retrieving
   # configuration.
   RewriteCond %{SERVER_PORT}  !^443$
   RewriteCond %{REQUEST_URI}  !^/(errors|config|favicon.ico)
   RewriteRule ^/(.*)          https://host.foo.com/$1 [L,R=301,NC]
   ---------------------------------------------------------------------

5. Reload the **httpd** service.

::

   # service httpd reload

IPA should now be running on port 8089, leaving port 8080 free for your
normal web site.

`Category:Obsolete <Category:Obsolete>`__

.. |Note.png| image:: Note.png
