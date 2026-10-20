Replica_Setup
=============

This text should be straightforward guide to users who want to setup and
test FreeIPA replica feature.

-  Please note that used host names (ipa-server.example.test,
   replica1.example.test, replica2.example.test) are only for better
   orientation and these names do not take effect on setup.



IPA server configuration
------------------------

First of all we need to install FreeIPA server to one of our machines.
This should be easily done with command:

::

   dnf install -y freeipa-server freeipa-server-dns

The *freeipa-server-dns* is recommended to install but you will **not**
be notified until the ``ipa-server-install`` command has been run and
you will try to configure integrated DNS. You can try to set up entire
FreeIPA server but before installation process it will most likely fail
with *missing freeipa-server-dns* message. We will predict this and
install freeipa-server-dns too.

--------------

After this we can set up our server defaults using command:

::

   ipa-server-install --domain=example.test --realm=EXAMPLE.TEST

This installation will ask for some additional information including
server administrator and Directory Manager password which you can see
down below:

::

   Do you want to configure DNS forwarders? [yes]: yes
   Following DNS servers are configured in /etc/resolv.conf: 10.10.160.1, 10.16.101.41, 10.11.5.19
   Do you want to configure these servers as DNS forwarders? [yes]: yes
   All DNS servers from /etc/resolv.conf were added. You can enter additional addresses now:
   Enter an IP address for a DNS forwarder, or press Enter to skip: 
   Checking DNS forwarders, please wait ...
   Do you want to search for missing reverse zones? [yes]: yes

   The IPA Master Server will be configured with:
   Hostname:       ipa-server.example.test
   IP address(es): 10.16.4.23
   Domain name:    example.test
   Realm name:     EXAMPLE.TEST

   BIND DNS server will be configured to serve IPA domain with:
   Forwarders:    10.10.160.1, 10.16.101.41, 10.11.5.19
   Reverse zone(s):  No reverse zone

   Continue to configure the system with these values? [no]: yes

   The following operations may take some minutes to complete.
   ...

At this point we can say that we have basic FreeIPA server installation
done.

--------------

Now we could get credentials as server administrator to test and later
configure ipa-server

::

   kinit admin

--------------



IPA replica configuration
-------------------------

There are three ways to get it done. First is to use *admin's* account
and enroll host using it, but we do not want to and we should not use
this administrator's account or password like this. Instead we can
create user with privileges to enrol host and promote it with one time
password to replica so only his password will be used in configuration
(or even installation scripts) and second one is to add another host
into special hostgroup *ipaservers* and promote client into replica
without any password needed. We will try both of them.

--------------



IPA replica server in ipaservers group
----------------------------------------------------------------------------------------------

This is may be the easiest way to get it done. The "only" thing to be
done is to have already enrolled replica machine added into
``ipaservers`` group on main IPA server. To get it done you have to
enroll machine to IPA server first (if you had one please skip this
step). **On the replica1 machine run:**

::

   dnf install -y ipa-server

Configure client side components and use admin's password:

::

   ipa-client-install --domain=example.test --realm=EXAMPLE.TEST --server=ipa-server.example.test
   ...
   Proceed with fixed values and no DNS discovery? [no]: yes
   Client hostname: replica1.example.test
   Realm: EXAMPLE.TEST
   DNS Domain: example.test
   IPA Server: ipa-server.example.test
   BaseDN: dc=example,dc=test

   Continue to configure the system with these values? [no]: yes
   Skipping synchronizing time with NTP server.
   User authorized to enroll computers: admin
   Password for admin@EXAMPLE.TEST: 
   Successfully retrieved CA cert
       Subject:     CN=Certificate Authority,O=EXAMPLE.TEST
       Issuer:      CN=Certificate Authority,O=EXAMPLE.TEST
       Valid From:  Tue Aug 23 10:24:58 2016 UTC
       Valid Until: Sat Aug 23 10:24:58 2036 UTC

   Enrolled in IPA realm EXAMPLE.TEST
   ...

-  Please note that if host is already IPA enrolled and have client side
   components installed we can skip these steps

--------------

**On the ipa-server** we should now add our replica1 host into
ipaservers group with command:

::

   ipa hostgroup-add-member ipaservers --hosts replica1.example.test
     Host-group: ipaservers
     Description: IPA server hosts
     Member hosts: ipa-server.example.test, replica1.example.test
   -------------------------
   Number of members added 1
   -------------------------

Obviously the result of command ``ipa hostgroup-find`` should be:

::

   ipa hostgroup-find
   -------------------
   1 hostgroup matched
   -------------------
     Host-group: ipaservers
     Description: IPA server hosts
     Member hosts: ipa-server.example.test, replica1.example.test
   ----------------------------
   Number of entries returned 1
   ----------------------------

and result of ``ipa host-find``:

::

   ---------------
   2 hosts matched
   ---------------
     Host name: ipa-server.example.test
     Principal name: host/ipa-server.example.test@EXAMPLE.TEST
     Password: False
     Member of host-groups: ipaservers
     Keytab: True
     Managed by: ipa-server.example.test
     SSH public key fingerprint: 4B:F4:EB:0E:6E:03:45:EF:C6:13:4E:E3:2C:F0:0B:42 (ssh-ed25519), 2B:82:7E:2B:07:72:46:CA:7F:93:10:A0:F0:8B:1B:D1 (ecdsa-sha2-nistp256), DB:1B:67:E9:2D:D9:29:77:B2:33:A3:DD:8A:B3:A8:5A
                                 (ssh-rsa)

     Host name: replica1.example.test
     Principal name: host/replica1.example.test@EXAMPLE.TEST
     Password: False
     Member of host-groups: ipaservers
     Keytab: True
     Managed by: replica1.example.test
     SSH public key fingerprint: 17:B0:CF:8E:02:E8:5E:F0:BE:7E:DC:4D:9F:7B:63:BB (ecdsa-sha2-nistp256), FE:33:03:48:F8:40:51:DD:30:29:BD:08:BF:81:1A:18 (ssh-ed25519), 70:D8:67:27:7E:7A:02:AA:83:61:D0:ED:2A:DF:84:A0
                                 (ssh-rsa)
   ----------------------------
   Number of entries returned 2
   ----------------------------

--------------

If host **replica1** is successfully enrolled and in host group
ipaservers then we just run command ``ipa-replica-install`` and there is
no need for admin's password as you can see:

::

   ipa-replica-install
   WARNING: conflicting time&date synchronization service 'chronyd' will
   be disabled in favor of ntpd

   ipa         : ERROR    Reverse DNS resolution of address 10.16.4.23 (ipa-server.example.test) failed. Clients may not function properly. Please check your DNS setup. (Note that this check queries IPA DNS directly and ignores /etc/hosts.)
   Continue? [no]: yes
   Run connection check to master
   Connection check OK
   Configuring NTP daemon (ntpd)
     [1/4]: stopping ntpd
     [2/4]: writing configuration
     [3/4]: configuring ntpd to start on boot
     [4/4]: starting ntpd
   Done configuring NTP daemon (ntpd).
   Configuring directory server (dirsrv). Estimated time: 1 minute
   ...

Now we have IPA replica and we have get it done only by adding this host
into *ipaservers* group. Hosts in this group automatically gets
credentials to become replica and when ``ipa-replica-install`` command
used, you do not need to use administrator password or other users
privileged to promote host into replica.

We can now ``kinit`` as admin on **replica1** and add new user:

::

    ipa user-add csantana --first=Carlos --last=Santana
   ---------------------
   Added user "csantana"
   ---------------------
     User login: csantana
     First name: Carlos
     Last name: Santana
     Full name: Carlos Santana
     Display name: Carlos Santana
     Initials: CS
     Home directory: /home/csantana
     GECOS: Carlos Santana
     Login shell: /bin/sh
     Kerberos principal: csantana@EXAMPLE.TEST
     Email address: csantana@example.test
     UID: 1217300000
     GID: 1217300000
     Password: False
     Member of groups: ipausers
     Kerberos keys available: False

Now the ``ipa user-find`` command should display same output **on both
ipa-server and replica1 machine**:

::

   ipa user-find
   ---------------
   2 users matched
   ---------------
     User login: admin
     Last name: Administrator
     Home directory: /home/admin
     Login shell: /bin/bash
     UID: 1217200000
     GID: 1217200000
     Account disabled: False
     Password: True
     Kerberos keys available: True

     User login: csantana
     First name: Carlos
     Last name: Santana
     Home directory: /home/csantana
     Login shell: /bin/sh
     Email address: csantana@example.test
     UID: 1217300000
     GID: 1217300000
     Account disabled: False
     Password: False
     Kerberos keys available: False
   ----------------------------
   Number of entries returned 2
   ----------------------------

--------------



IPA client enrolled and promoted to replica with One Time Password in one step
----------------------------------------------------------------------------------------------

**On the IPA server** we should add new host and copy *OTP* (Random
password):

::

   ipa host-add replica2.example.test --random
   ----------------------------------------------
   Added host "replica2.example.test"
   ----------------------------------------------
     Host name: replica2.example.test
     Random password: huN@Nk5k9vjg
     Password: True
     Keytab: False
     Managed by: replica2.example.test

Then we make him member of host group ipaservers:

::

   ipa hostgroup-add-member ipaservers --hosts replica2.example.test 
     Host-group: ipaservers
     Description: IPA server hosts
     Member hosts: ipa-server.example.test, replica1.example.test, replica2.example.test
   -------------------------
   Number of members added 1
   -------------------------

--------------

Now we have to install freeipa-server **on replica2**.example.test :

::

   dnf install -y freeipa-server

To configure replica on replica2.example.test machine just run:

-  make sure that OTP is closed in quotes, there may be some special
   characters which might be interpreted by shell

::

   ipa-replica-install -p 'huN@Nk5k9vjg' --server ipa-server.example.test --domain example.test --realm EXAMPLE.TEST

--------------

After successful configuration on all three machines the output of
command ``ipa-host-find`` should be:

::

   ipa host-find
   ---------------
   3 hosts matched
   ---------------
     Host name: ipa-server.example.test
     Principal name: host/ipa-server.example.test@EXAMPLE.TEST
     Password: False
     Member of host-groups: ipaservers
     Keytab: True
     Managed by: ipa-server.example.test
     SSH public key fingerprint: 4B:F4:EB:0E:6E:03:45:EF:C6:13:4E:E3:2C:F0:0B:42 (ssh-ed25519), 2B:82:7E:2B:07:72:46:CA:7F:93:10:A0:F0:8B:1B:D1 (ecdsa-sha2-nistp256), DB:1B:67:E9:2D:D9:29:77:B2:33:A3:DD:8A:B3:A8:5A
                                 (ssh-rsa)

     Host name: replica1.example.test
     Principal name: host/replica1.example.test@EXAMPLE.TEST
     Password: False
     Member of host-groups: ipaservers
     Keytab: True
     Managed by: replica1.example.test
     SSH public key fingerprint: 17:B0:CF:8E:02:E8:5E:F0:BE:7E:DC:4D:9F:7B:63:BB (ecdsa-sha2-nistp256), FE:33:03:48:F8:40:51:DD:30:29:BD:08:BF:81:1A:18 (ssh-ed25519), 70:D8:67:27:7E:7A:02:AA:83:61:D0:ED:2A:DF:84:A0
                                 (ssh-rsa)

     Host name: replica2.example.test
     Principal name: host/replica2.example.test@EXAMPLE.TEST
     Password: False
     Member of host-groups: ipaservers
     Keytab: True
     Managed by: replica2.example.test
     SSH public key fingerprint: 11:E6:02:AB:0D:BB:A4:28:BE:CB:0F:68:B1:4A:EB:B8 (ssh-ed25519), 82:78:5E:14:4C:B7:92:D1:F4:C1:6D:D1:8E:C0:87:84 (ssh-rsa), 46:FA:6A:03:BD:32:89:5B:58:A4:1B:C2:4A:C1:22:77 (ecdsa-
                                 sha2-nistp256)
   ----------------------------
   Number of entries returned 3
   ----------------------------

We just add one other user for example again **on** new
**replica2**.example.test to test functionality

::

   ipa user-add sclaus --first=Santa --last=Claus
   -------------------
   Added user "sclaus"
   -------------------
     User login: sclaus
     First name: Santa
     Last name: Claus
     Full name: Santa Claus
     Display name: Santa Claus
     Initials: SC
     Home directory: /home/sclaus
     GECOS: Santa Claus
     Login shell: /bin/sh
     Kerberos principal: sclaus@EXAMPLE.TEST
     Email address: sclaus@example.test
     UID: 1217250000
     GID: 1217250000
     Password: False
     Member of groups: ipausers
     Kerberos keys available: False

And all ipaservers should display same info:

::

   ipa user-find
   ---------------
   3 users matched
   ---------------
     User login: admin
     Last name: Administrator
     Home directory: /home/admin
     Login shell: /bin/bash
     UID: 1217200000
     GID: 1217200000
     Account disabled: False
     Password: True
     Kerberos keys available: True

     User login: csantana
     First name: Carlos
     Last name: Santana
     Home directory: /home/csantana
     Login shell: /bin/sh
     Email address: csantana@example.test
     UID: 1217300000
     GID: 1217300000
     Account disabled: False
     Password: False
     Kerberos keys available: False

     User login: sclaus
     First name: Santa
     Last name: Claus
     Home directory: /home/sclaus
     Login shell: /bin/sh
     Email address: sclaus@example.test
     UID: 1217250000
     GID: 1217250000
     Account disabled: False
     Password: False
     Kerberos keys available: False
   ----------------------------
   Number of entries returned 3
   ----------------------------
