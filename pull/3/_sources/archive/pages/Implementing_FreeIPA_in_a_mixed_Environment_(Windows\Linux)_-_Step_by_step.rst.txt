**NOTE**:the information provided on this page was only tested against
FreeIPA 1.2 and should be considered deprecated for anything newer.

**Introduction**

--------------

The following documentation is a practical guide to implement freeIPA in
mixed environment (Windows/Linux Clients). You should also refer the
Administrator's as well as Installation and Deployment guide from the
main documentation page.

The Installation has been performed on the following environment.

Server: Single IPA server (Fedora 10 x86_64) with 2G RAM 1.6 GHz Intel
Dual Core processor.

Clients: Windows XP SP2, Fedora 10 x86_64 and RHEL5.2 x86_64

Note: Please be careful about the firewall and selinux policies before
continuing with the configuration. In windows also you should open the
necessary ports to communicate to the IPA Server or disable the firewall
if you are doing a test setup. Please refer the installation and
deployment guide to get more details about the ports required for IPA.

1. Installation of the IPA Server.

| ``# yum install ipa-*``
| ``# yum install bind bind-chroot``

The IPA server may show a conflict with mod_ssl package. IPA uses
mod_nss in apache. You can remove the mod_ssl for the time being.

2.Make sure that the host names are set properly

| ``# cat /etc/hosts``
| ``127.0.0.1               localhost.localdomain localhost``
| ``172.16.33.1             ds.example.com ds``

| ``# cat /etc/sysconfig/network``
| ``NETWORKING=yes``
| ``HOSTNAME=ds.example.com``

3. Run the following command to configure the IPA Server for you
environment and follow the instructions.

``# ipa-server-install --setup-bind``

Here the DNS Server is on the same machine. Please note that kerberos
server has very specific DNS requirements, if you have a DNS server
already on your network add the SRV records of the kerberos, ntp and
ldap server to that. A sample zone file will be created in your /tmp
directory after the ipa-server-install, do a copy paste of all the SRV
record from this file to your zone file.

If you have a chrooted bind installed, the named service start-up may
fail after the ipa-server-install . Do the following configuration to
setup DNS properly.

a. A minimal named.conf should look like

| ``# cat /etc/named.conf``
| ``options {``
| ``       directory "/var/named";``
| ``       dump-file               "data/cache_dump.db";``
| ``       statistics-file         "data/named_stats.txt";``
| ``       memstatistics-file      "data/named_mem_stats.txt";``
| ``};``

| ``zone "example.com" {``
| ``       type master;``
| ``       file "example.com.zone.db";``
| ``};``
| ``zone "33.16.172.in-addr.arpa" IN {``
| ``       type master;``
| ``       file "example.com.zone.rev.db";``
| ``};``

b. Copy the zone file to the proper location and create a reverse zone
file also.

``# cp /var/named/example.com.zone.db /var/named/chroot/var/named/``

No need to change anything in the forward zone file, create a reverse
zone as follows.

``#  cat example.com.zone.rev.db``

| ``$ORIGIN 33.16.172.in-addr.arpa.``
| ``$TTL    86400``
| ``@                       IN SOA  example.com. root.example.com. (``
| ``                               01              ; serial``
| ``                               3H              ; refresh``
| ``                               15M             ; retry``
| ``                               1W              ; expiry``
| ``                               1D )            ; minimum``

| ``                        IN NS                   ds.example.com.``
| ``1                       IN PTR                  ds.example.com.``

c. Restart the named service

4. Check whether the ntp time synchronization is proper, if you don't
want to sync to an external time server, configure a local time server
and synch all the clients to that.

| ``# ntpstat``
| ``# ntpq -p``

Sample configuration file for an ntp local server.

``# cat /etc/ntp.conf``

| ``restrict default nomodify notrap noquery``
| ``restrict 127.0.0.1``
| ``broadcast 224.0.1.1 ttl 4``
| ``broadcastdelay 0.004``

| ``server  127.127.1.0``
| ``fudge   127.127.1.0 stratum 10``

| ``driftfile /var/lib/ntp/drift``
| ``keys /etc/ntp/keys``

Sample Configuration for an ntp client

``# cat /etc/ntp.conf``

| ``restrict default kod nomodify notrap nopeer noquery``
| ``restrict -6 default kod nomodify notrap nopeer noquery``
| ``restrict 127.0.0.1``
| ``restrict -6 ::1``
| ``server  ds.example.com``
| ``driftfile /var/lib/ntp/drift``
| ``keys /etc/ntp/keys``

Please note that if the client time has much difference compared to ntp
server then do a force update using the following command. Also, the
first time synchronization will take some time (64 sec approx)

``# ntpdate -u ds.example.com``

To verify

| ``# ntpstat``
| ``# ntpq -p``

5. Make sure that all the required services are enabled in your run
level and reboot the IPA server (krb5kdc, ntp, named, httpd, dirserv
etc). This will be configured automatically when you run the
ipa-server-install, anyway just do a second check.

6. After the reboot test the IPA server configuration using the
following commands

| ``# kinit admin``
| ``# klist``
| ``# ipa-finduser admin``

**Configuring Windows Client**

--------------

Note: An alternative solution exists: `Windows authentication against
FreeIPA <Windows_authentication_against_FreeIPA>`__

| ``1. Add the host records in DNS, both forward and reverse``
| ``2. Make sure that the client is synchronized to the ntp server.``
| ``3. On the IPA Server add the host principal and set the password for the xp client.``

| ``#  ipa-addservice host/bmdata01.example.com``
| ``#  ipa-getkeytab -s ds.example.com  -p host/bmdata01.example.com -e des-cbc-crc -k krb5.keytab.txt -P``

4. On the Client (Windows XP)

a. Install Windows XP support tools
(WindowsXP-KB838079-SupportTools-ENU.exe, this can be found on the
Windows XP Media or download it from microsoft)

b. Create a user in Windows XP to map the kerberos principles (here it
is ipauser)

c. Configure kerberos authentication as follows (go to Start - Programs
- Windows Support Tools - Command Prompt )

| ``C:> ksetup /setrealm EXAMPLE.COM``
| ``C:> ksetup /addkdc EXAMPLE.COM dc.example.com``
| ``C:> ksetup /setmachpassword ``\ `` (the same password you have set in IPA server)``
| ``C:> ksetup /mapuser * ipauser``

d. Reboot the machine.

e. You will see "EXAMPLE.COM (Kerberos Realm)" in the windows logon drop
down menu.

Note: **CREATE A NEW USER ON THE IPA SERVER AND TRY TO LOGON TO THE
WINDOWS CLIENT. WINDOWS WILL TELL YOU THAT THE PASSWORD HAS BEEN
EXPIRED. IT WILL PROMPT YOU TO SET THE NEW PASSWORD ALSO. IF YOU ENTER
YOUR USER NAME, OLD PASSWORD AND NEW PASSWORDS, WINDOWS WILL SIMPLY TELL
YOU "DOMAIN NOT AVAILABLE**

**HERE IS THE TRICK, PLEASE NOTE THAT THE USER IS REQUIRED TO LOGIN
USING “USER@REALM” (testuser@EXAMPLE.COM) INSTEAD OF JUST THE USER NAME
FOR THE FIRST TIME.**

**Configuring RHEL 5.2 x86_64 Client**

--------------

1. Download and un-compress freeipa source,
http://freeipa.org/downloads/src/freeipa-1.2.1.tar.gz

| ``# tar -zxvf freeipa-1.2.1.tar.gz``
| ``# cd freeipa-1.2.1``

2. Install the following prerequisites

``# yum install autoconf automake pkgconfig.x86_64 libtool.x86_64 mozldap-devel.x86_64 krb5-devel.x86_64 openldap-devel.x86_64 python-ldap.x86_64``

3. You will also need to downloaded and install python-krbV package from
http://download.fedora.redhat.com/pub/epel/

4. Apply the patch

# patch -p1 < /path/to/make.patch 

``(patch can be found in ``\ ```https://www.redhat.com/archives/freeipa-users/2009-January/msg00022.html`` <https://www.redhat.com/archives/freeipa-users/2009-January/msg00022.html>`__\ ``, copy the contents and save it as make.patch)``

5. Make rpms, the rpms will be in dist/rpms

``# make IPA_VERSION_IS_GIT_SNAPSHOT=no local-dist``

--`viji <User:Viji>`__ 04:49, 15 January 2009 (EST)
