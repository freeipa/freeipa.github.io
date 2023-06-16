**KVM Virtualization Integrating Libvirt and VNC Authentication with
Kerberos in IPA**

Introduction
------------

The 'default' behaviour of KVM with libvirt is to use a UNIX socket
rather than network and to only allow root read/write access to the
system session with others only having user-session access read/write
and the system session being read only - with this controlled via
PolicyKit.

Leveraging the SASL mechanisms in libvirt allows GSSAPI for an
authentication mechanism (X509 certificates are also possible but since
'user certificates' are not yet in IPA and I've not tested that in this
workplace I'll leave that out of scope of this document).

The prerequisites to work through this guide are a working IPA topology
and a working libvirt/kvm server (which only needs to allow root to
manage the guests) that has been joined to the IPA domain.

For the purposes of this document the IPA server will be
ipa01.example.com in the realm EXAMPLE.COM, the virtualization server
will be kvmhost01.example.com and the guest will be
kvmguest01.example.com.

.. _creating_the_services_and_obtaining_the_keytabs:

Creating the services and obtaining the keytabs
-----------------------------------------------

In order to allow for all this to work keytabs need to be arranged for
libvirt and VNC. On ipa01.example.com (or any system with ipa-admintools
installed or the GUI):

| `` ipa service-add libvirt/kvmhost01.example.com``
| `` ipa service-add vnc/kvmhost01.example.com``

On the KVM host server:

| `` ipa-getkeytab -s ipa01.example.com -p libvirt/kvmhost01.example.com -k /etc/libvirt/krb5.tab``
| `` ipa-getkeytab -s ipa01.example.com -p vnc/kvmhost01.example.com -k /etc/qemu/krb5.tab``

**Notes**

-  On CentOS 7, the VNC keytab should be located in /etc/qemu-kvm
   instead of /etc/qemu.
-  The keytabs are 600 root:root for libvirt and 600 qemu:root for qemu.
-  The standard etc_t selinux type appeared to work fine.

With the keytabs in place libvirt and qemu now need to be configured to
make use of them and enable TCP connections.

.. _configuring_libvirt_to_use_kerberos_via_sasl:

Configuring libvirt to use kerberos via SASL
--------------------------------------------

In order to tell kerberos that GSSAPI is an available mechanism and to
allow connections over tcp the following configuration needs to be
carried out:

The appropriate changes in /etc/libvirt/libvirtd.conf are:

| `` listen_tls = 0``
| `` listen_tcp = 1``
| `` auth_tcp = "sasl"``
| `` sasl_allowed_username_list = ["*@EXAMPLE.COM" ]``

To configure the SASL auth for libvirtd following this make sure you
have the following in /etc/sasl2/libvirt.conf:

| `` mech_list: gssapi``
| `` keytab: /etc/libvirt/krb5.tab``

Finally listening on TCP needs to be enabled in /etc/sysconfig/libvirtd:

`` LIBVIRTD_ARGS="--listen"``

.. _configuring_qemu_to_allow_sasl_authentication_for_vnc:

Configuring qemu to allow SASL authentication for VNC
-----------------------------------------------------

With libvirt listening qemu needs to be configured to allow connections
to console. This part of things it's the qemu process running the guest
rather than the overall libvirt daemon that's responsible.

To enable the VNC side of things edit /etc/libvirt/qemu.conf:

| `` vnc_listen = "0.0.0.0"``
| `` vnc_tls = 0``
| `` vnc_sasl = 1``

If SELinux is enabled on the host, it may prevent qemu to use GSSAPI.
This is the case with CentOS 7.1.1503, but other distributions may be
impacted. Either disable SELinux completely or add the following line to
/etc/libvirt/qemu.conf to disable SELinux integration:

`` secure_driver = "none"``

Then configure SASL2 appropriately in /etc/sasl2/qemu.conf:

| ``mech_list: gssapi``
| ``keytab: /etc/qemu/krb5.tab``

On CentOS 7, the file is named /etc/sasl2/qemu-kvm.conf, and the keytab
location is different:

| ``mech_list: gssapi``
| ``keytab: /etc/qemu-kvm/krb5.tab``

On CentOS 7, qemu will try to read VNC SASL configuration from the file
/etc/sasl2/spice.conf. Other distributions may exhibit the same
behaviour. In that case, just create a symbolic link
/etc/sasl2/spice.conf pointing to /etc/sasl2/qemu-kvm.conf:

| `` cd /etc/sasl2``
| `` ln -s qemu-kvm.conf spice.conf``

The final note to make is an `outstanding
bug <https://bugzilla.redhat.com/show_bug.cgi?id=718377>`__ that affects
this setup currently where the kerberos key cache that gets made by
libvirt has the range set to the guest VM being connected to and then
consequently connecting to another guest on the same host fails due to
the selinux range on the file preventing it from being read.

As a workaround until the bug is fixed, as an example, I have the
following in cron:

`` */5 * * * * chcon -l s0 /var/tmp/vnc_*``

This will set the range to s0 and thus all VMs can then use this replay
cache for the credentials.

.. _firewall_configuration:

Firewall configuration
----------------------

Although libvirtd should now be listening remotely at this point
iptables should still be blocking connections. Assuming default ports
for libvirt and VNC the following lines should be put in
/etc/sysconfig/iptables:

| `` -A INPUT -m state --state NEW -m tcp -p tcp --dport 5900:5999 -j ACCEPT``
| `` -A INPUT -m state --state NEW -m tcp -p tcp --dport 16509 -j ACCEPT``

If you don't want to deal with iptables directly, you may use
firewall-cmd instead. Assuming your network interface is bound to the
'public' zone:

| `` firewall-cmd --zone=public --add-service=libvirt``
| `` firewall-cmd --zone=public --add-service=libvirt --permanent``
| `` firewall-cmd --zone=public --add-port=5900:5999/tcp``
| `` firewall-cmd --zone=public --add-port=5900:5999/tcp --permanent``

This should be the only requirement left for connectivity.

.. _client_usage:

Client usage
------------

At this point everything should be functional. Either a remote
connection should be possible or ssh -X to kvmhost01.example.com and
then opening virt-manager.

To connect to libvirt over TCP use a connection string such as :

`` virsh -c qemu+tcp://kvmhost01.example.com/system ``

In the alternative add the environment variable to the client (useful on
the server itself where it won't need to vary) of:

`` LIBVIRT_DEFAULT_URI="qemu+tcp://kvmhost01.example.com/system"``

To connect to the VNC instance a client that is capable of GSSAPI for
VNC should be used such as virt-viewer or the console view of
virt-manager.

Conclusion
----------

If all the steps have been followed then as long as there is a valid
kerberos token in the realm EXAMPLE.COM connecting with a libvirt client
to the libvirt daemon should work without any additional credentials
being requested.
