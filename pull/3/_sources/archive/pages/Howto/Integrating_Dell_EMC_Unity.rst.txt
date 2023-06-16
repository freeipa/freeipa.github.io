.. _howto_integrate_freeipa_with_nfs_share_hosted_on_dell_emc_unity:

HowTo Integrate FreeIPA with NFS share hosted on Dell EMC Unity
---------------------------------------------------------------

This guide provides the steps that were taken to integrate our FreeIPA
instance with our Dell EMC Unity storage appliance. After following this
guide your FreeIPA clients will be able to authenticate with Kerberos
and seamlessly connect to a NFS share provided by the Dell EMC Unity
system.

Assumption
~~~~~~~~~~

FreeIPA installation already configured (4.x) CentOS/Fedora/Ubuntu
FreeIPA client already configured

.. _freeipa_configuration:

FreeIPA Configuration
^^^^^^^^^^^^^^^^^^^^^

This section will describe the parts you will need to perform on your
FreeIPA server for the integration.

#. Add Dell EMC Unity host (you can also do this step through the
   FreeIPA GUI)

      ``ipa host-add emc-nas-server.example.com --ip-address 10.75.37.2``

#. Add Dell EMC Unit host as a service principal (you can also do this
   step through the FreeIPA GUI)

      ``ipa service-add NFS/emc-nas-server.example.com@EXAMPLE.COM``

#. Create a keytab from your IPA server for your newly created service

      ``ipa-getkeytab -s ipaserver.example.com -p nfs/emc-nas-server.example.com -k /tmp/emc-nas-server.keytab``

.. _dell_emc_unit_configuration:

Dell EMC Unit Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section will describe the parts you need to modify on your Dell EMC
Unity. These settings were performed on a Dell EMC Unity 300 with
UnityOS 4.5.1.0.5.001.

#. Create a new NAS Server (you can use an existing NAS Server but YMMV)

   #. General - Provide Server Name (name for your NAS server) -
      emc-nas-server.
   #. Select your Pool
   #. Select your Storage Processor
   #. Enter IP address you provided to FreeIPA (10.75.37.2) in our case
   #. Subnet address (255.255.255.0)
   #. Gateway (10.75.37.1)

#. Select your Sharing Protocols in this case select "Enable NFSv4"

   #. Select "Configure Secure NFS"
   #. Provide your Host Name: emc-nas-server
   #. Enable "Secure NFS (with Kerberos)"
   #. Specify your REALM name, this is your primary domain. Realm:
      EXAMPLE.COM
   #. Specify your "SPN:". Upload the keytab file you generate from your
      FreeIPA server

#. Configure Kerberos by selecting "Configure KDC Servers for custom
   Kerberos realm

   #. Your Realm is your primary domain (e.g. EXAMPLE.COM) - it should
      be automatically filed in if you are going through the main NAS
      Servers setup in the Dell EMC NAS Servers creation wizard.
   #. Add your KDC Servers.

         ipaserver.example.com - for our example

#. For your Unix Directory service Select "LDAP"

   #. If you have your environment setup correctly (DNS/Discovery) you
      can leaveObtain LDAP servers IPs automatically enabled.
   #. Select Kerberos for your "Authentication"

      #. Realm: EXAMPLE.COM
      #. Principal: "nfs/emc-nas-server.example.com" (exactly as this,
         the form will take this format)
      #. Password: password you gave from above
      #. Base DN: dc=example,dc=.com -

#. Enter your domain and your DNS Servers

   #. example.com
   #. add - 10.75.35.66
   #. add - 10.75.35.77

#. Finish
#. Go back into the NAS Server configuration and select "Naming
   services" --> LDAP/NIS

   #. THIS IS KEY TO THIS ENTIRE PROCESS. If you have not changed your
      Schema click "Retrieve Current Schema" and save and edit it.

         It will look like

      :

nss_base_passwd ou=people,dc=example,dc=com nss_base_group
ou=group,dc=example,dc=com nss_base_hosts ou=hosts,dc=example,dc=com
nss_base_netgroup ou=netgroup,dc=example,dc=com

##:Change to:

##:

::

   nss_base_passwd cn=users,cn=accounts,dc=example,dc=com 
   nss_base_group cn=groups,cn=accounts,dc=example,dc=com 
   nss_base_hosts cn=computers,cn=accounts,dc=example,dc=com 
   nss_base_netgroup cn=ng,cn=alt,dc=example,dc=com

#. Select File Systems from Storage --> File --> File Systems
#. Add new Protocol Linux/Unix Share (NFS)
#. Provide Name, FLR, Storage Tier etc.
#. Select NFS Share (Linux Unix) and provide it a name
#. Apply any other settings (Snapshots replications, etc.) you need for
   your environment.
#. Select NFS Shares from Storage --> File menu.
#. Select your Share you just created and Edit.
#. Make note of your Export Paths on the General Tab
#. Select "Host Access" tab

   #. Select your "Minimum security" you want to allow and default
      access
   #. Add the hosts that you want to allow to access the NFS Share if
      you do not provide any Default Access

#. Apply settings and you are done on the Dell EMC Unity system

.. _centosubuntu_configuration:

CentOS/Ubuntu Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section describes what you need to do on your client side to access
the NFS share.

Ubuntu
''''''

#. Run and install

      ``sudo apt-get install -y nfs-common nfs-kernel-server``

#. Mount the NFS file share

      ``mount -o sec=krb5 emc-nas-server.example.com:/datastore /mnt``

CentOS
''''''

#. Run and install

      ``sudo apt-get install -y nfs-utils``

#. Mount the NFS file share

      ``mount -o sec=krb5 emc-nas-server.example.com:/datastore /mnt``
