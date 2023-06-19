Back to `FreeIPAv1:Client Configuration
Guide <FreeIPAv1:Client_Configuration_Guide>`__

\__TOC_\_

Introduction
============

This document describes the procedures required to configure Fedora as
an IPA client. IPA 1.2 supports Fedora 8, 9, 10 and 11 as IPA clients.
Previous versions may work as well if manually configured (or the
freeIPA src.rpm rebuilt on that platform).

This document distinguishes between commands to be run as root versus a
regular user. Commands to be run as root are prefixed with a # symbol.
Commands to be run as a regular user are prefixed with a $ symbol.

   |Note.png|\ **Note:**

      Before starting the freeIPA installation, ensure that you update
      your system with all the latest packages.

.. _installing_the_ipa_client:

Installing the IPA Client
=========================

1. Install the client and tools with:

::

   # yum install ipa-client

This should install all the dependencies as well.

2. If your IPA server was set up for DNS, and is in the same domain as
the client, add the server's IP address to the client's
``/etc/resolv.conf`` file.

So add a line that looks something like (depending on the IP address of
your IPA server).

::

   nameserver 192.168.100.1

Note: this is not typical.

.. _configuring_the_ipa_client:

Configuring the IPA Client
==========================

.. _configuring_client_authentication:

Configuring Client Authentication
---------------------------------

   |Note.png| **Note:**

      The IPA client requires that an IPA server already exist.

1. Use the following command to set up the freeIPA client:

::

   # ipa-client-install

The script should set up the IPA client without prompting for any
further information. This includes configuring the name service cache
daemon to start at boot time. This will cache the most common name
service requests from the client, and reduce the load on the server.

When the script has finished configuring the freeIPA client, it will
display information about the realm, DNS domain, IPA server, etc. You
should see output similar to the following:

::

   Discovery was successful!
   Realm: EXAMPLE.COM
   DNS Domain: example.com
   IPA Server: ipaserver.example.com
   BaseDN: dc=example,dc=com

..

   |Note.png| **Note:**

      If your IPA server and client are not in the same domain, the
      setup script will prompt you for the information that it requires.

.. _configuring_kerberos:

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

   The file name is the hash of the contents with a ".0" filename
   extension.

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

3. Add the following line to the ``/etc/sysconfig/nfs`` file:

**SECURE_NFS=yes**

4. Start the **rpcgssd** daemon.

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
stop **ntpd** and retsart it after the command is issued, because
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

.. |Note.png| image:: Note.png
