Overview
--------

While using Single-Sign-on authentication provided via an
`MIT <http://web.mit.edu/kerberos/>`__
`Kerberos <http://en.wikipedia.org/wiki/Kerberos_%28protocol%29>`__ KDC
there must not be any significant clock skew between server and clients
so a time synchronization service is required.

When FreeIPA started there was only ntpd that offered hooks to sign NTP
packets using kerberos keys. Now as the change is required we think
about chronyd as replacement for ntpd as there will not be a ntpd in new
fedora releases.

FreeIPA server supports only a configuration of ntpd. ntpd is being
deprecated in Fedora 28 therefore IPA must deprecate it as well. FreeIPA
needs network time synchronization to have all clients and servers
synchronized because of time based services so there should be
replacement for it.

NTP - network time protocol ntpd, chronyd - time service implementations
based on NTP



Use Cases
---------

-  ipa-{server,replica}-install

Install first check for NTP services already running and disable all in
favor of ntpd. After this check if there is NO option -N set installer
will force ntpd configuration. It alters ntpd configuration file enable
service and add record to LDAP.

-  ipa-client-install

FreeIPA client install first tries to configure itself as client to
existing ntpd instance on network. Same behavior can be reproduced by
chrony.

With -N FreeIPA is not configuring ntpd server nor client. Option
--force-ntp is now used to disable any other NTP services.

Design
------

The idea of ntpd replacement with *chronyd* played with idea not using
FreeIPA as NTP server but only as NTP client.

Installer deprecates --force-ntp option and forces chronyd service by
default if there is any conflicting time service running on host.

Unless we do not specify the -N option described down below FreeIPA
installer will enable chronyd daemon with respective client
configuration (default or user-defined).

Installer used with -N option will not configure chronyd nor enable
chronyd service if it is not running. This option should be used in case
administrator would like to have other NTP service than chronyd.

-  **Dependencies**:

   -  - ntpd
   -  + chrony

Implementation
--------------

File ipaclient/install/ntpconf.py has been replaced with
ipaclient/install/timeconf.py. The configuration of chronyd service
handled by methods in the ipaclient/install/timeconf.py file and it is
done same way in client and master installation using Augeas tool.

Note that different StateStores for chronyd in {master, replica DL0} and
{replica DL1} are used. This is caused by installation of master using
master's StateStore and client using client's StateStore.

Due to change that FreeIPA is no longer a NTP server providing clock
only client the removal of the NTP server role from the FreeIPA has been
done. There is no need for instalation to add \_ntp._udp record with the
FreeIPA master's hostname to DNS as well.

If there is not specified --ntp-server option and no --ntp-pool option
as well the FreeIPA installer will try use autodiscovery for \_ntp._udp
record in DNS and configure chrony with discovered server address. When
DNS has no \_ntp._udp record default chrony configuration will be used.

The --ntp-server (knob ntp_servers) is used to specify a exactly one
time server and may be used multiple times. The --ntp-pool option is
meant to be pool of multiple servers resolved as one hostname. This
pool's servers may vary but pool address will be still same and chronyd
daemon will choose only one server randomly from this pool.

After succesfull configuration of chrony client we run waitsync to
synchronize with given time source.

Note that FreeIPA will force chronyd service therefore disable any other
conflicting time synchronization daemon when the -N option is not used.

All changes related to --force-ntp option and ntpd replacement should be
reflected into man pages.

CLI
~~~

The ipa-{server,replica}-install did not support the --ntp-server option
so support has been added with the new option --ntp-pool as well. The
ipa-client-install supports the new option --ntp-pool as well Overview
of the CLI commands. Example:

+---------------------+----------------------+----------------------+
| Command             | Options              | Behavior             |
+=====================+======================+======================+
| ipa-server-install  | -N                   | will NOT configure   |
|                     |                      | NOR start a local    |
|                     |                      | time service(chronyd |
|                     |                      | client)              |
+---------------------+----------------------+----------------------+
| ipa-server-install  |                      | will use default     |
|                     |                      | chrony configuration |
|                     |                      | when autodiscovery   |
|                     |                      | for \_ntp._udp       |
|                     |                      | record fails(enables |
|                     |                      | chronyd)             |
+---------------------+----------------------+----------------------+
| ipa-server-install  | --                   | will configure       |
|                     | ntp-server=1.foo.bar | chrony client using  |
|                     |                      | foo.bar NTP server   |
|                     |                      | and enable chronyd   |
|                     |                      | daemon               |
+---------------------+----------------------+----------------------+
| ipa-server-install  | --                   | will configure       |
|                     | ntp-server=1.foo.bar | chrony client using  |
|                     | --                   | 1.foo.bar and        |
|                     | ntp-server=2.foo.bar | 2.foo.bar NTP        |
|                     |                      | servers and enable   |
|                     |                      | chronyd daemon       |
+---------------------+----------------------+----------------------+
| ipa-server-install  | --n                  | will configure       |
|                     | tp-pool=pool.foo.bar | chrony client using  |
|                     |                      | pool.foo.bar NTP     |
|                     |                      | pool and enable      |
|                     |                      | chronyd daemon       |
+---------------------+----------------------+----------------------+
| ipa-replica-install | -N                   | will NOT configure   |
|                     |                      | NOR start a local    |
|                     |                      | time service(chronyd |
|                     |                      | client)              |
+---------------------+----------------------+----------------------+
| ipa-replica-install |                      | will use default     |
|                     |                      | chrony configuration |
|                     |                      | when autodiscovery   |
|                     |                      | for \_ntp._udp       |
|                     |                      | record fails(enables |
|                     |                      | chronyd)             |
+---------------------+----------------------+----------------------+
| ipa-replica-install | --                   | will configure       |
|                     | ntp-server=1.foo.bar | chrony client using  |
|                     |                      | foo.bar NTP server   |
|                     |                      | and enable chronyd   |
|                     |                      | daemon               |
+---------------------+----------------------+----------------------+
| ipa-replica-install | --                   | will configure       |
|                     | ntp-server=1.foo.bar | chrony client using  |
|                     | --                   | 1.foo.bar and        |
|                     | ntp-server=2.foo.bar | 2.foo.bar NTP        |
|                     |                      | servers and enable   |
|                     |                      | chronyd daemon       |
+---------------------+----------------------+----------------------+
| ipa-replica-install | --n                  | will configure       |
|                     | tp-pool=pool.foo.bar | chrony client using  |
|                     |                      | pool.foo.bar NTP     |
|                     |                      | pool and enable      |
|                     |                      | chronyd daemon       |
+---------------------+----------------------+----------------------+
| ipa-client-install  | -N                   | will NOT configure   |
|                     |                      | NOR start a local    |
|                     |                      | time service(chronyd |
|                     |                      | client)              |
+---------------------+----------------------+----------------------+
| ipa-client-install  |                      | will use default     |
|                     |                      | chrony configuration |
|                     |                      | when autodiscovery   |
|                     |                      | for \_ntp._udp       |
|                     |                      | record fails(enables |
|                     |                      | chronyd)             |
+---------------------+----------------------+----------------------+
| ipa-client-install  | --                   | will configure       |
|                     | ntp-server=1.foo.bar | chrony client using  |
|                     |                      | foo.bar NTP server   |
|                     |                      | and enable chronyd   |
|                     |                      | daemon               |
+---------------------+----------------------+----------------------+
| ipa-client-install  | --                   | will configure       |
|                     | ntp-server=1.foo.bar | chrony client using  |
|                     | --                   | 1.foo.bar and        |
|                     | ntp-server=2.foo.bar | 2.foo.bar NTP        |
|                     |                      | servers and enable   |
|                     |                      | chronyd daemon       |
+---------------------+----------------------+----------------------+
| ipa-client-install  | --n                  | will configure       |
|                     | tp-pool=pool.foo.bar | chrony client using  |
|                     |                      | pool.foo.bar NTP     |
|                     |                      | pool and enable      |
|                     |                      | chronyd daemon       |
+---------------------+----------------------+----------------------+

Upgrade
-------

While upgrading IPA server there is need for:

-  complete removal of ntpd service related configuration files
-  disablement of ntpd service
-  removal of \_ntp._udp DNS record

(taken care with method ntpd_cleanup(fqdn, fstore)



Test Plan
---------

fresh install with -N and ntpd already setup

fresh install with -N and chrony already setup

fresh install with -N and without any time service configuration

fresh install without any time option and ntpd already setup

fresh install without any time option and chrony already setup

fresh install without any time option and without any time conf

upgrade on instance installed with -N and ntpd already setup

upgrade on instance installed with -N and chrony already setup

upgrade on instance installed with -N and without any time conf

upgrade on instance installed without any time option and ntpd already
setup

upgrade on instance installed without any time option and chrony already
setup

upgrade on instance installed without any time option and without any
time conf
