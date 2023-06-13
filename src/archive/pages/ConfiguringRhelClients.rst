Back to `FreeIPAv1:Client Configuration
Guide <FreeIPAv1:Client_Configuration_Guide>`__

\__TOC_\_

Introduction
============

This document describes the procedures required to configure Red Hat
Enterprise Linux as an IPA client. IPA 1.0 supports Red Hat Linux 2.1,
3, 4, and 5 as IPA clients.

This document distinguishes between commands to be run as root versus a
regular user. Commands to be run as root are prefixed with a # symbol.
Commands to be run as a regular user are prefixed with a $ symbol.

   |Note.png|\ **Note:**

      Before starting the freeIPA installation, ensure that you update
      your system with all the latest packages.

.. _configuring_rhel_5_as_an_ipa_client:

Configuring RHEL 5 as an IPA Client
===================================

.. _downloading_and_installing_the_ipa_packages:

Downloading and Installing the IPA Packages
-------------------------------------------

1. To download the IPA packages, log in to the Red Hat Network and
navigate to the "Downloads" area. Refer to `Red Hat Enterprise IPA
Release
Notes <http://www.redhat.com/docs/manuals/ipa/release-notes/1.0/index.html>`__
for information on the appropriate channels.

2. Download the appropriate ISO for your system, and then either mount
it or burn it to suitable media.

3. Use the following command to install the IPA client and tools:

::

   # yum install ipa-client ipa-admintools

This should install all of the dependencies as well.

4. Add the DNS server's IP address to the client's ``/etc/resolv.conf``
file.

.. _configuring_client_authentication:

Configuring Client Authentication
---------------------------------

   |Note.png|\ **Note:**

      The IPA client requires that an IPA server already exist.

1. Use the following command to set up the IPA client:

``# ipa-client-install``

   |Caution.png|\ **Caution:**

      Ensure that you use the correct script to set up the client.
      Separate scripts exist for Red Hat Enterprise Linux 4 and 5, and
      they are not interchangeable.

The script should set up the IPA client without prompting for any
further information. This includes configuring the name service cache
daemon to start at boot time. This will cache the most common name
service requests from the client, and reduce the load on the server.

When the script has finished configuring the IPA client, it will display
information about the realm, DNS domain, IPA server, etc. You should see
output similar to the following:

::

   Discovery was successful!
   Realm: EXAMPLE.COM
   DNS Domain: example.com
   IPA Server: ipaserver.example.com
   BaseDN: dc=example,dc=com

..

   |Note.png|\ **Note:**

      If your IPA server and client are not in the same domain, the
      setup script will prompt you for the information that it requires.

.. _configuring_kerberos:

Configuring Kerberos
--------------------

Only a basic Kerberos configuration file is created as part of the
client configuration procedures described here. You need to perform the
remaining configuration changes manually.

1. Modify the ``/etc/krb5.conf`` file on the client as shown below.
Ensure that you replace the example values with those that apply to your
deployment.

::

   [libdefaults]
    default_realm = EXAMPLE.COM
    dns_lookup_realm = true
    dns_lookup_kdc = true
    forwardable = yes
    ticket_lifetime = 24h

   [realms]
    EXAMPLE.COM = {
     kdc = ipaserver.example.com:88
     admin_server = ipaserver.example.com:749
     default_domain = example.com
    }
   [domain_realm]
    .example.com = EXAMPLE.COM
    example.com = EXAMPLE.COM

.. _configuring_client_tls:

Configuring Client TLS
----------------------

The SSL/TLS settings are only required if you want to use SSL between
the clients and the server when performing operations such as account
lookups.

TLS Client Configuration for Linux clients is detailed at
http://directory.fedora.redhat.com/wiki/Howto:SSL. Refer to this link if
any additional information is required. The basic steps required are:

1. Modify the following in the ``/etc/ldap.conf`` file:

::

   URI     ldap://ipaserver.example.com
   BASE dc=example,dc=com
   HOST ipaserver.example.com
   TLS_CACERTDIR /etc/cacerts/
   TLS_REQCERT allow

..

   |Note.png| **Note:**

      Ensure that the directory you specify for TLS_CACERTDIR actually
      exists.

2. Export your CA certificate to ASCII using the **certutil** utility
with the **-a** option.

3. Install this certificate in the ``/etc/cacerts`` directory as
follows:

::

   # cp cacert.asc /etc/cacerts/`openssl x509 -noout \ 
     -hash -in cacert.asc`.0

   The file name is the hash of the contents with a ".0" extension.

4. If the TLS_CACERTDIR directive does not work, set the cacert file
directly:

``# TLS_CACERT /etc/cacerts/cacert.asc``

   If more than one CA certificate is required, concatenate these
   certificates into a single file.

.. _configuring_nfs_v4_with_kerberos:

Configuring NFS v4 with Kerberos
--------------------------------

Use the following procedure to configure NFS on the IPA client:

1. Obtain a Kerberos ticket for the **admin** user.

::

    # kinit admin

2. Add an NFS service principal on the client.

::

    # ipa-addservice nfs/ipaclient.example.com

3. Get a keytab for the NFS service principal.

::

    # ipa-getkeytab -s ipaserver.example.com -p nfs/ipaclient.example.com -k /etc/krb5.keytab

..

   |Note.png|\ **Note:**

      The Linux NFS implementation still has limited encryption type
      support. You may need to use the **-e des-cbc-crc** option to the
      **ipa-getkeytab** command for any **nfs/<FQDN>** service keytabs
      you want to set up, both on the server and on all clients. This
      will instruct the KDC to generate only DES keys.

4. Add the following line to the ``/etc/sysconfig/nfs`` file:

**SECURE_NFS=yes**

5. Start the **rpcgssd** daemon.

::

    # service rpcgssd start

Your IPA client should now be fully configured to mount NFS shares using
your Kerberos credentials. You can use the following command to test the
configuration:

::

    # mount -v -t nfs4 -o sec=krb5 ipaserver.example.com:/ /mnt

.. _configuring_client_ssh_access:

Configuring Client SSH Access
-----------------------------

You can also configure the IPA client to accept incoming SSH requests
and authenticate with the user's Kerberos credentials. After installing
and configuring the IPA client, use the following procedure to configure
the IPA client for SSH connections. Remember to replace the example host
and domain names with your own host and domain name:

1. The IPA client installation process does not configure the NTP
protocol service by default, but it is a good idea to make sure that
time on the IPA client and server is synchronized. If it is not, you
should run the following command on the IPA client. You will need to
stop **ntpd** and restart it after the command is issued, because
**ntpdate** does not work if **ntpd** is running.

::

    # ntpdate -s -p 8 -u ipaserver.example.com

2. Obtain a Kerberos ticket for the **admin** user.

::

    # kinit admin

3. Add a host service principal on the IPA client.

::

    # ipa-addservice host/ipaclient.example.com

4. Retrieve the keytab.

::

    # ipa-getkeytab -s ipaserver.example.com -p host/ipaclient.example.com -k /etc/krb5.keytab

Your IPA client should now be fully configured to accept incoming SSH
connections and authenticate with the user's Kerberos credentials. Use
the following command from another machine to test the configuration.
This should succeed without asking for a password.

::

    # ssh admin@ipaclient.example.com

.. _configuring_host_based_access_control:

Configuring Host-Based Access Control
-------------------------------------

You can configure Red Hat Enterprise Linux and Fedora to allow or deny
access to IPA resources and services based on the configuration of the
host from which access is attempted. Refer to `Configuring Host-Based
Access
Control <Obsolete:Administrators_Guide#Configuring_Host-Based_Access_Control>`__
for more information on this topic.

.. _configuring_rhel_4_as_an_ipa_client:

Configuring RHEL 4 as an IPA Client
===================================

.. _downloading_and_installing_the_ipa_packages_1:

Downloading and Installing the IPA Packages
-------------------------------------------

1. To download the IPA packages, log in to the Red Hat Network and
navigate to the "Downloads" area. Refer to `Red Hat Enterprise IPA
Release
Notes <http://www.redhat.com/docs/manuals/ipa/release-notes/1.0/index.html>`__
for information on the appropriate channels.

2. Use the following command to install the IPA client and tools:

::

   # rpm -ivh ipa-client-<version>.rpm

This should install all of the dependencies as well.

2. If your IPA server was set up for DNS, and is in the same domain as
the client, add the server's IP address to the client's
``/etc/resolv.conf`` file.

.. _configuring_client_authentication_1:

Configuring Client Authentication
---------------------------------

   |Note.png|\ **Note:**

      The IPA client requires that an IPA server already exist.

1. Create the ``/etc/ipa/ipa.conf`` file.

2. Use the following command to set up the IPA client:

::

   # ipa-client-setup --server ipaserver.example.com

..

   |Caution.png|\ **Caution:**

      Ensure that you use the correct script to set up the client.
      Separate scripts exist for Red Hat Enterprise Linux 4 and 5, and
      they are not interchangeable.

The script should set up the IPA client without prompting for any
further information. This includes configuring the name service cache
daemon to start at boot time. This will cache the most common name
service requests from the client, and reduce the load on the server.

3. Reboot the client machine.

   |Note.png| **Note:**

      The RHEL 4 version of the IPA client installation script does not
      perform auto-discovery, and neither does it configure the client
      machine to perform auto-discovery.

.. _configuring_kerberos_1:

Configuring Kerberos
--------------------

Only a basic Kerberos configuration file is created as part of the
Client Configuration procedures described above. You need to perform the
remaining configuration changes manually.

1. Modify the ``/etc/krb5.conf`` file on the client as shown below.
Ensure that you replace the example values with those that apply to your
deployment.

::

   [libdefaults]
    default_realm = EXAMPLE.COM
    dns_lookup_realm = true
    dns_lookup_kdc = true
    forwardable = yes
    ticket_lifetime = 24h

   [realms]
    EXAMPLE.COM = {
     kdc = ipaserver.example.com:88
     admin_server = ipaserver.example.com:749
     default_domain = example.com
    }
   [domain_realm]
    .example.com = EXAMPLE.COM
    example.com = EXAMPLE.COM

.. _configuring_client_tls_1:

Configuring Client TLS
----------------------

The SSL/TLS settings are only required if you want to use SSL between
the clients and the server when performing operations such as account
lookups.

TLS Client Configuration for Linux clients is detailed at
http://directory.fedora.redhat.com/wiki/Howto:SSL. Refer to this link if
any additional information is required. The basic steps required are:

1. Modify the following in the ``/etc/ldap.conf`` file:

::

   URI     ldap://ipaserver.example.com
   BASE dc=example,dc=com
   HOST ipaserver.example.com
   TLS_CACERTDIR /etc/cacerts/
   TLS_REQCERT allow

..

   |Note.png| **Note:**

      Ensure that the directory you specify for TLS_CACERTDIR actually
      exists.

2. Export your CA certificate to ASCII using the certutil utility with
-a option.

3. Install this certificate in the ``/etc/cacerts`` directory as
follows:

::

   # cp cacert.asc /etc/cacerts/`openssl x509 -noout \ 
     -hash -in cacert.asc`.0

   The file name is the hash of the contents with a ".0" filename
   extension.

4. If the TLS_CACERTDIR directive does not work, set the cacert file
directly:

``# TLS_CACERT /etc/cacerts/cacert.asc``

   If more than one CA certificate is required, concatenate these
   certificates into a single file.

.. _system_login:

System Login
------------

-  On the RHEL 4 system console, log in as an IPA user. After you have
   logged in, open a terminal and try these commands:

| ``id (look for userid and group id correctness)``
| ``getent passwd``
| ``getent group``

.. _configuring_nfs_v4_with_kerberos_1:

Configuring NFS v4 with Kerberos
--------------------------------

Use the following procedure to configure NFS on the IPA client:

1. Obtain a Kerberos ticket for the **admin** user.

::

   # kinit admin

..

   |Note.png| **Note:**

      The **ipa-admintools** package is not available for RHEL 4.
      Consequently, you need to perform the following steps on the IPA
      server.

   2. Add an NFS service principal for the client.

   ::

      # ipa-addservice nfs/ipaclient.example.com

   3. Retrieve the NFS keytab.

   ::

      # ipa-getkeytab -s ipaserver.example.com -p nfs/ipaclient.example.com -k /tmp/krb5.keytab
      # klist -ket /tmp/krb5.keytab (to verify)

   4. Copy the keytab from the server to the client.

   ::

      # scp /tmp/krb5.keytab root@ipaclient.example.com:/tmp/krb5.conf

5. On the IPA client, use the **ktutil** command to import the keytab.

::

   # ktutil
    ktutil: read_kt /tmp/krb5.keytab
    ktutil: write_kt /etc/krb5/krb5.keytab
    ktutil: q

6. Add the following line to the ``/etc/sysconfig/nfs`` file:

   **SECURE_NFS=yes**

7. Start the **rpcgssd** daemon.

::

   # /etc/init.d/rpcgssd start

Your IPA client should now be fully configured to mount NFS shares using
your Kerberos credentials. You can use the following command to test the
configuration:

::

   # mount -v -t nfs4 -o sec=krb5 ipaserver.example.com:/ /mnt

.. _configuring_client_ssh_access_1:

Configuring Client SSH Access
-----------------------------

You can configure the IPA client to accept incoming SSH requests and
authenticate with the user's Kerberos credentials. After installing and
configuring the IPA client, use the following procedure to configure the
IPA client for SSH connections. Remember to replace the example host and
domain names with your own host and domain name:

1. The IPA client installation process does not configure the NTP
protocol service by default, but it is a good idea to make sure that
time on the IPA client and server is synchronized. If it is not, you
should run the following command on the IPA client. You will need to
stop **ntpd** and retsart it after the command is issued, because
**ntpdate** does not work if **ntpd** is running.

::

    # ntpdate -s -p 8 -u ipaserver.example.com

2. Obtain a Kerberos ticket for the **admin** user.

::

   # kinit admin
   # klist (to verify that you received a ticket)

..

   |Note.png| **Note:**

      The **ipa-admintools** package is not available for RHEL 4.
      Consequently, you need to perform the following commands on the
      IPA server.

   3. Add a host service principal.

   ::

      # ipa-addservice host/ipaclient.example.com

   4. Retrieve the host keytab.

   ::

      # ipa-getkeytab -s ipaserver.example.com -p host/ipaclient.example.com -k /tmp/krb5.keytab

   5. Copy the keytab from the server to the client.

   ::

      # scp /tmp/krb5.keytab root@ipaclient.example.com:/tmp/krb5.keytab

6. On the IPA client, use the **ktutil** command to import the keytab.

::

   # ktutil
    ktutil: read_kt /tmp/krb5.keytab
    ktutil: write_kt /etc/krb5/krb5.keytab
    ktutil: q

Your IPA client should now be fully configured to accept incoming SSH
connections and authenticate with the user's Kerberos credentials. Use
the following command from another machine to test the configuration.
This should succeed without asking for a password.

::

   # ssh admin@ipaclient.example.com

.. _configuring_host_based_access_control_1:

Configuring Host-Based Access Control
-------------------------------------

You can configure Red Hat Enterprise Linux and Fedora to allow or deny
access to IPA resources and services based on the configuration of the
host from which access is attempted. Refer to `Configuring Host-Based
Access
Control <Obsolete:Administrators_Guide#Configuring_Host-Based_Access_Control>`__
for more information on this topic.

.. _configuring_rhel_2.1_and_3_as_ipa_clients:

Configuring RHEL 2.1 and 3 as IPA Clients
=========================================

The following procedure describes how to configure RHEL 2.1 and RHEL 3
as IPA clients. The procedures are the same for each. There are no IPA
packages or installation scripts for either of these distributions; the
process is completely manual.

   |Note.png|\ **Note:**

      The IPA client requires that an IPA server already exist.

.. _configuring_kerberos_2:

Configuring Kerberos
--------------------

**To set up client authentication:**

1. Run **authconfig** as the root user.

2. On the **User Information Configuration** screen, select **LDAP**,
and enter the server name and Base DN.

   |Note.png|\ **Note:**

      The Base DN is the realm name translated into "dc" components. For
      example:

      EXAMPLE.COM -> dc=example,dc=com

      This step does not fully configure nss_ldap. Further configuration
      is described in the following procedure.

3. Navigate to the **Authentication Configuration** screen.

4. Ensure that **Use LDAP Authentication** is NOT selected.

5. Select **Use Kerberos 5** and enter the following details (modify to
suit your deployment):

::

       Realm: EXAMPLE.COM
       KDC: ipaserver.example.com:88
       Admin Server: ipaserver.example.com:749

6. Press **Enter** or click **OK** to save the configuration and exit
the **authconfig** utility.

Kerberos should now be correctly configured.

.. _configuring_ldap:

Configuring LDAP
----------------

You need to make the following configuration changes to the
``/etc/ldap.conf`` file to complete the client configuration for RHEL 3.
Modify the examples provided to suit your deployment. You may need to
add some of these entries if they do not exist in the original file.

::

   ldap_version 3
   host ipaserver.example.com
   base dc=example,dc=com

   nss_base_passwd cn=users,cn=accounts,dc=example,dc=com?sub
   nss_base_group cn=groups,cn=accounts,dc=example,dc=com?sub
   nss_schema rfc2307bis
   nss_map_attribute uniqueMember member
   nss_initgroups_ignoreusers root

   nss_reconnect_maxsleeptime 8
   nss_reconnect_sleeptime 1
   bind_timelimit 5
   timelimit 15

   ssl no

This completes the configuration steps for IPA.

.. |Note.png| image:: Note.png
.. |Caution.png| image:: Caution.png
