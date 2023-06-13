.. _ipa_with_integrated_bind_inside_chroot:

IPA with integrated BIND inside chroot
======================================

This how-to was created on RHEL 6.4 with IPA 3.0.

Please see this `note <#NOTE>`__ about chroots.

-  Install IPA as usual and add package bind-chroot:

   -  yum install ipa-server bind bind-chroot bind-dyndb-ldap -y
   -  ipa-server-install --setup-dns

-  Add required libraries to chroot environment (use do copy or link,
   whatever you prefer)

      Following script should copy all necessary files to "$CHROOT":

::

   CP="cp -Lpcvr"
   ARCH="64" # empty string should work for x86_32 architecture
   CHROOT="/var/named/chroot"
   LIB="$CHROOT/lib$ARCH/"
   USRLIB="$CHROOT/usr/lib$ARCH/"
   BINDLIB="$CHROOT/usr/lib$ARCH/bind/"
   ETC="$CHROOT/etc/"

   # urandom is required by Kerberos libraries
   $CP /dev/urandom "$CHROOT/dev/"

   # add libraries
   mkdir -p "$LIB"
   $CP /lib$ARCH/libldap-2.4.so.2 "$LIB"
   $CP /lib$ARCH/liblber-2.4.so.2 "$LIB"
   $CP /lib$ARCH/libplds4.so "$LIB"
   $CP /lib$ARCH/libplc4.so "$LIB"
   $CP /lib$ARCH/libnspr4.so "$LIB"
   $CP /lib$ARCH/libcrypt.so.1 "$LIB"
   $CP /lib$ARCH/libfreebl3.so "$LIB"
   mkdir -p "$USRLIB"
   $CP /usr/lib$ARCH/libssl3.so "$USRLIB"
   $CP /usr/lib$ARCH/libsmime3.so "$USRLIB"
   $CP /usr/lib$ARCH/libnss3.so "$USRLIB"
   $CP /usr/lib$ARCH/libnssutil3.so "$USRLIB"
   $CP /usr/lib$ARCH/libsasl2.so.2 "$USRLIB"
   $CP /usr/lib$ARCH/krb5 "$USRLIB"
   $CP /usr/lib$ARCH/sasl2 "$USRLIB"

   # add bind-dyndb-ldap library
   $CP /usr/lib$ARCH/bind/ldap.so "$BINDLIB"

   # add BIND and Kerberos configuration
   $CP /etc/named.* "$ETC"
   $CP /etc/rndc.* "$ETC"

   # add Kerberos configuration
   $CP /etc/krb5.conf "$ETC"
   mkdir -p /etc/ipa
   $CP /etc/ipa/ca.crt "$ETC/ipa"

   # Kerberos needs also name->IP mapping for KDC
   $CP /etc/hosts "$ETC"

-  In (chrooted) */etc/named.conf* and replace LDAPI socket with
   localhost:

::

   arg "uri ldap://127.0.0.1:389

-  In (chrooted) */etc/krb5.conf* comment out includedir line.

   -  Krb5 library initialization will fail if include directory is not
      accesible. **TODO**: Find some nicer way how to handle this
      problem.

::

   includedir /var/lib/sss/pubconf/krb5.include.d

-  BIND should be able to start now.

--------------

This article is enhanced version of Dale's `post on freeipa-users
list <https://www.redhat.com/archives/freeipa-users/2013-February/msg00411.html>`__.

--------------

.. _note_chroot_should_not_be_considered_a_security_feature:

NOTE: Chroot should not be considered a security feature
--------------------------------------------------------

A chroot jail is often described as being a security feature. In reality
a chrooted environment is no more secure than a non-chrooted
environment. The root user is able to "break out" of a chroot jail,
which is not unexpected behavior of the super user. Non root users are
not able to escape from a chroot jail, but due to the nature of
security, one has to assume that an attacker who is sophisticated enough
to gain local access to a chrooted environment is also sophisticated
enough to gain root privileges. A chrooted environment is ideal for
running a process in a minimal environement, it should not be considered
more secure than a non chrooted environment. Proper Discretionary Access
Control (DAC) permissions provide the same basic level of security as a
chrooted environment.
