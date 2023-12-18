ARM
===

There has been periodic interest expressed in installing IPA on a SoC
computer like the Raspberry or Banana Pi. While technically possible
there are a number of caveats.

We do not have specific SoC recommendations but in general the minimum
requirements are 2 GB of RAM with a CA-full install and 1 GB RAM with a
CA-less or a replica without a CA installed. We have no figures on how
well, if at all, this will scale.

SoC chips tend to have SDcard-based disks which are slow. This can cause
service timeouts during installation. One way to mitigate this is to
create /etc/ipa/installer.conf prior to calling ipa-server-install:

| ``$ cat /etc/ipa/installer.conf``
| ``[global]``
| ``startup_timeout=900``

Note that it isn't uncommon for the CA to take 10 minutes or more to
start on a Pi.

A user contributed his experience installing IPA on a BananaPi at
`HowTo/FreeIPA_on_banana_pi <HowTo/FreeIPA_on_banana_pi>`__

As of pki-core 10.8.3 (IPA 4.8.x) the above workaround is reported to no
longer work. An override file can be passed to the installer to set a
new timeout. Add an override file and pass this in via
--pki-config-override. The override file will consist of:

| ``[DEFAULT]``
| startup_timeout=900 

This alone is not sufficient. You'll also need to increase the startup
timeout for systemd.

Add TimeoutStartSec=900 to /usr/lib/systemd/system/pki-tomcatd@.service

Alternatively if the first change doesn't work you can change the python
installer code directly by modifying
/usr/lib/python3.*/site-packages/pki/server/deployment/scriptlets/configuration.py
and set PKISPAWN_STARTUP_TIMEOUT_SECONDS to 900.

Revised: Apr 7, 2020