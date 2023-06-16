Getting Windows clients that are not enrolled to FreeIPA domain to be
able to mount a CIFS share using NTMLSSP.

1) Install required package:

``yum install ipa-server-trust-ad``

2) Run ipa-adtrust-install

**NOTE** Let the installer overwrite smb.conf

3) Create directory to be shared and configure ACL and Selinux

``mkdir /srv/testshare``

chcon -t samba_share_t /srv/testshare

setfacl -m g:admins:rwx /srv/testshare

4) Add a share

``net conf addshare testshare /srv/testshare writeable=y guest_ok=N``

**NOTE** do not try to add the share configurations to your smb.conf

5) From a Windows client that is not enrolled to IPA or AD domain, test
the share
