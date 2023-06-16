Introduction
============

The IPA team lacks expertise on configuring non-Linux systems. The
following instructions worked at one time to provide a basic roadmap on
how to configure different operating systems to authenticate against an
IPA server. For additional information and support we suggest contacting
the operating system vendor.

.. _solaris_8910:

Solaris 8/9/10
==============

.. _configuring_client_authentication:

Configuring Client Authentication
---------------------------------

#. Apply the latest kernel and LDAP patches.

   #. For Solaris8: kernel patch + 108993. The latest version (version
      55) can be downloaded by entering 108993 at
      http://sunsolve.sun.com/pub-cgi/show.pl?target=patches/patch-access.
   #. For Solaris 9, the required patch is 112960 (version 37)
   #. For Solaris 10, the required patch is 119073-3.

#. Add the following files to the /var/ldap directory:

   #. /var/ldap/ldap_client_file: The important parameters in this file
      are the ldapservers and the name of the client profile. At
      startup, the ldapclient extracts the profile from the server.
   #. /var/ldap/ldap_client_cred: These are the proxy settings. You can
      encrypt the password in the file by using an online web tool such
      as http://jpirr.nic.ad.jp/crypt_gen_web.html

   Sample files are available
   `here <ConfiguringUnixClients#Client_Configuration_Files>`__. These
   files need to be configured specifically for each client.

#. Ensure that the above files have 0400 permissions.
#. Configure nsswitch.conf. A sample file is provided
   `here <SolarisNsswitchConf>`__. In this file, passwd, group and
   automounts are set to be read from ldap.
#. Set the Solaris NIS domain name:

   #. echo “example.com” > /etc/defaultdomain
   #. domainname \`cat /etc/defaultdomain\`

#. Configure pam.conf. Solaris 10 uses the same pam.conf file as Solaris
   8 and 9, except that certain lines for pam_unix_cred.so.1 have been
   uncommented. Sample files are provided
   `here <ConfiguringUnixClients#Client_Configuration_Files>`__.
#. Restart the ldap_cachemgr and ncsd.

.. _configuring_client_tls:

Configuring Client TLS
----------------------

The SSL/TLS settings are only required if you want to use SSL between
the clients and the server when performing operations such as account
lookups.

The SUNWtlsu package is needed to provide /usr/sfw/bin/certutil.

**To set up TLS on the clients:**

1. Copy the cert7.db, key3.db and secmod.db databases to the /var/ldap
directory. Make sure these files have permissions 0444.

2. Change the profile reference in /var/ldap/ldap_client_file to the
correct TLS profile. The TLS profile should have:

::

   authenticationMethod: tls:simple
   serviceAuthenticationMethod:pam_ldap:tls:simple

3. Restart the client:

::

   /usr/lib/ldap/ldap_cachemgr -K
   mv /var/ldap/cachemgr.log /var/ldap/cachemgr.log.old
   /usr/lib/ldap/ldap_cachemgr
   /etc/init.d/nscd stop
   /etc/init.d/nscd start

.. _additional_resources:

Additional Resources
--------------------

.. _freeipa_users_mailing_list_threads:

freeipa-users mailing list threads
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  `Solaris 10 client configuration using
   profile <https://www.redhat.com/archives/freeipa-users/2014-October/msg00132.html>`__
-  `Patch to update Solaris
   documentation <https://www.redhat.com/archives/freeipa-devel/2014-April/msg00286.html>`__

.. _documentation_bugs:

Documentation bugs
~~~~~~~~~~~~~~~~~~

-  `Update the DUA profile included in
   IPA <https://bugzilla.redhat.com/show_bug.cgi?id=815515>`__
-  `Update the Solaris 10 client
   documentation <https://bugzilla.redhat.com/show_bug.cgi?id=815533>`__

.. _solaris_67:

Solaris 6/7
===========

.. _building_nss_ldap_openldap_and_pam_ldap:

Building nss_ldap, OpenLDAP, and pam_ldap
-----------------------------------------

Solaris 6 does not have a native LDAP client. Therefore, it is necessary
to provide a solution using nss_ldap.so. The following steps describe
how to compile nss_ldap.so on a Solaris 6 machine. This module is binary
compatible with Solaris 7.

1. Download and install the following packages from
http://sunfreeware.com/indexsparc26.html:

::

   openssl.0.9.8a.SPARC.Solaris.2.6.pkg.tgz
   make-3.80-sol26-sparc-local.gz
   gzip-1.3.5-sol26-sparc-local
   gcc-3.4.2-sol26-sparc-local

   pkgadd -d gzip-1.3.5-sol26-sparc-local
   pkgadd -d gcc-3.4.2-sol26-sparc-local
   gunzip make-3.80-sol26-sparc-local.gz
   pkgadd -d make-3.80-sol26-sparc-local
   gunzip openssl.0.9.8a.SPARC.Solaris.2.6.pkg.tgz |tar -xvf -pkgadd -d. openssl

There will be messages about conflicting files. Ignore them and proceed
with the installation.

2. Download openldap-stable-20060127.tar from the openldap site, and
extract as follows:

::

   tar -xvf openldap-stable-20060127.tar
   cd openldap-2.3.19

3. Compile openldap:

::

   ./configure --prefix=/opt/ldap --enable-syslog --disable-slapd -with-tls CC=/usr/local/bin/gcc
   make depends
   make
   make install

4. Download pam_ldap from the padl site, and configure as follows:

::

   tar -xvf pam_ldap.tar
   cd pam_ldap-180
   ./configure --prefix=/opt/ldap –with-ldap-dir=/opt/ldap

5. Modify the Makefile and add -L/usr/local/lib to the LD_FLAGS, and
then run make and make install:

::

   pam_ldap_so_LDFLAGS = -B dynamic -M $(srcdir)/exports.solaris -G -B group -lc\ -L/opt/ldap/lib -L/usr/local/lib -R/opt/ldap/lib

   make
   make install

   make[1]: Entering directory `/export/pam_ldap-180'
   /bin/sh ./mkinstalldirs /opt/ldap/lib/security
   mkdir /opt/ldap/lib/security
   ./install-sh -c  -o root -g root pam_ldap.so
   /opt/ldap/lib/security/pam_ldap.so.1
   (cd /opt/ldap/lib/security; rm -f pam_ldap.so; ln -s pam_ldap.so.1
   pam_ldap.so)
   make  install-man5
   make[2]: Entering directory `/export/pam_ldap-180'
   /bin/sh ./mkinstalldirs /opt/ldap/man/man5
    ./install-sh -c -m 644 ./pam_ldap.5 /opt/ldap/man/man5/pam_ldap.5
   make[2]: Leaving directory `/export/pam_ldap-180'
   make[1]: Leaving directory `/export/pam_ldap-180'

6. Download nss_ldap from the padl.com site, and extract as follows:

::

   tar -xvf nss_ldap.tar
   cd nss_ldap-248

7. Compile nss_ldap

::

   ./configure --prefix=/opt/ldap --with-ldap-dir=/opt/ldap --enable-rfc2307bis
   /usr/local/bin/make
   /usr/local/bin/make install

.. _installing_pam_ldap_and_nss_ldap:

Installing pam_ldap and nss_ldap
--------------------------------

This procedure describes how to install the nss_ldap and pam_ldap
binaries on Solaris 6 and Solaris 7. The binaries have been delivered as
a tar file that extract into /opt/ldap.

1. Copy the tar file to the root directory and untar.

::

   cp nss_ldap_solaris_6_7.tar /; tar -xvf nss_ldap_compiled.tar

2. Install openssl 0.9.8 on the system. You may get messages about
conflicting files; ignore these and continue with the installation.

3. Copy /opt/ldap/lib/security/pam_ldap.so to
/usr/lib/security/pam_ldap.so.

4. Create an appropriate symlink as follows:

::

   cd  /opt/ldap/lib/security; ln -s pam_ldap.so ./pam_ldap.so.1

5. For Solaris 7, save the version of pam_unix.so, and copy over the
Solaris 6 version from /opt/ldap/lib/security/pam_unix.so.

::

   cp /usr/lib/security/pam_unix.so /usr/lib/security/pam_unix.so.sol7
   cp /opt/ldap/lib/security/pam_unix.so  /usr/lib/security/pam_unix.so

6. Check if the pam module can be loaded. That is, see if the dynamic
linker can resolve all the dependencies by running ldd.

::

   ldd /usr/lib/security/pam_ldap.so

7. Check that all the libraries can be found. The libraries under
/usr/local/lib may not be found. To put them in the search path, create
symbolic links. You may have to make the following links:

::

   libssl.so.0.9.8 =>       /usr/lib/libssl.so.0.9.8
   libcrypto.so.0.9.8 =>    /usr/lib/libcrypto.so.0.9.8
   libgcc_s.so.1 =>         /usr/lib/libgcc_s.so.1

For example: ln -s /usr/local/lib/libssl.so.0.9.8
/usr/lib/libssl.so.0.9.8

For Solaris 7, the openssl libraries can be found in the
/usr/local/lib/ssl directory.

8. Copy nss_ldap from /opt/ldap/lib/nss_ldap.so to /usr/lib/nss_ldap.so
and then create the following link:

::

   ln -s /usr/lib/nss_ldap.so /usr/lib/nss_ldap.so.1

9. Check if all dynamic libraries are resolved, as follows:

::

   ldd /usr/lib/nss_ldap.so

10. Copy over nsswitch.conf and pam.conf (refer to `Client Configuration
files <ConfiguringUnixClients#Client_Configuration_Files>`__). These
files are same as the ones used for Solaris 8+.

.. _configuring_client_authentication_1:

Configuring Client Authentication
---------------------------------

The following files need to be configured correctly. Sample files are
provided in the client directories:

-  /etc/ldap.conf
-  /etc/pam.conf
-  /etc/nsswitch.conf

.. _configuring_client_tls_1:

Configuring Client TLS
----------------------

The SSL/TLS settings are only required if you want to use SSL between
the clients and the server when performing operations such as account
lookups.

Because nss_ldap and pam_ldap have been compiled with TLS, it is
possible to do authentication with these clients using TLS. Due to time
constraints, this implementation is untested.

.. _hp_ux_11.0:

HP-UX 11.0
==========

The HP-UX clients are configured to read a profile from the Directory
Server. While this profile is of the same object class as that used by
the Solaris servers, its attributes have slightly different content and
usage. It is therefore necessary to create a separate profile for HP-UX
machines. This profile is automatically created when the first HP-UX
client on a domain is created. Subsequent machines on that domain are
configured by copying over the relevant configuration files.

Example profiles are provided
`here <ConfiguringUnixClients#Sample_Profiles>`__.

   **Note:** The client uses a proxy user to connect to the Directory
   Server. In this case, the proxy user is
   uid=proxyagent,ou=profile,dc=example,dc=com. This user needs to exist
   on the Directory Server.

.. _configuring_client_authentication_2:

Configuring Client Authentication
---------------------------------

**To create a new profile:** Ensure that all clients have the following
software installed:

-  LDAP-UX Integration version. You can obtain B03.30.02 from
   https://h20293.www2.hp.com/portal/swdepot/try.do?productNumber=J4269AA

The following steps are required to complete the installation of the
first HP-UX 11.0 client. Subsequent clients can (and should) be
configured by copying over the relevant files.

1. Create the /etc/ldap.conf file if it does not exist.

2. Configure a proxy user on the client. This user must exist on the
Directory Server.

::

   cd /opt/ldapux/config
   ./ldap_proxy_config -i uid=proxyagent,ou=profile,dc=example,dc=com redhat123

3. Run the setup program /opt/ldapux/config/setup. Detailed,
step-by-step instructions for this program are provided in the HP-UX
Client Configuration Guide on p. 30. The prompts are self-explanatory
and all schema elements should already have been installed on the
Directory Server. Note that the option to select SSL will only be
available if the cert7.db and key3.db files already exist on the system.
See the next section for details.

4. When prompted to accept the default options, select “no”. Then, enter
the options for the proxy user. Also, enter 0 for the ProfileTTL.
Profile refreshes will be configured manually as a cron job.

5. At the prompt, “Do you want to remap RFC2307 attributes?”, specify
“No.”

6. At the prompt, “Do you want to specify custom search descriptors?”,
specify “No.”

7. Select the option to restart the LDAPUX client services. The profile
is written to /etc/opt/ldadux/ldap_client_file, as well as to the
Directory Server. In addition, the proxy user's credentials are written
to the /etc/opt/ldapux/cred file.

8. Modify pam.conf and nsswitch.conf. Sample files are provided here.

9. Verify the configuration by running the following commands:

::

   pwget -n username
   grget -n groupname
   nsquery passwd username ldap

10. Set up a cron job to periodically refresh the profile. Instructions
on how to do this are in the HP-UX Client Configuration Guide on p.65.

**To configure further clients with the same profile:**

1. Copy the following files to the new server:

::

   /etc/opt/ldapux/pcred
   /etc/opt/ldapux/ldapux_client.conf
   /etc/pam.conf
   /etc/nsswitch.conf
   /etc/opt/ldapux/key3.db (if SSL is enabled)
   /etc/opt/ldapux/cert7.db (if SSL is enabled)

2. Make sure the file permissions are the same as the first server.

3. Download the profile:

::

   /opt/ldapux/config/get_profile_entry -s nss

4. Configure the proxy user:

::

   /opt/ldapux/config/ldap_proxy_config

5. Configure the cron job to refresh the profile.

.. _configuring_client_tls_2:

Configuring Client TLS
----------------------

The SSL/TLS settings are only required if you want to use SSL between
the clients and the server when performing operations such as account
lookups.

**To set up TLS on the clients:**

1. Copy the cert7.db and key3.db files into /etc/opt/ldapux. The
following instructions (steps 2 and 3) describe how to generate the
cert7.db files using certutil (which is actually delivered in
/opt/ldapux/contrib/bin/certutil). For subsequent boxes, it is necessary
only to copy the generated cert7.db and key3.db files to
/etc/opt/ldapux.

2. To create the cert, change directory to /etc/opt/ldapux. Create a new
database as follows:

::

   /opt/ldapux/contrib/bin/certutil -N -d /etc/opt/ldapux

This generates empty cert7.db and key3.db databases.

3. Install the cacert into the cert7.db and key3.db databases from the
cacert.asc file (previously exported from ldap01.example.com using
certutil according to the instructions in Appendix A).

::

   /opt/ldapux/contrib/bin/certutil -A -n ldap08-ca-cert -t "C,," -d /etc/opt/ldapux -a -i /etc/opt/ldapux/cacert.asc

4. Once the cert7.db and key3.db files are in place in /etc/opt/ldapux,
run the setup program “/opt/ldapux/config/setup” as described above, and
select SSL/Simple. This option will not show up if the db files are not
in /etc/opt/ldapux. This generates a new profile for TLS/Simple access
on the server.

::

   cn=hpux_11.0_tls,ou=profile,dc=example,dc=com

.. _hp_ux_11i_v.1_and_2:

HP-UX 11i v.1 and 2
===================

No special setup is required for HP-UX 11i. As with HP-UX11.0, the
client retrieves a profile from the Directory Server. The profile used
is almost the same as the HP-UX 11.0. The only difference is that the
automount services is LDAP-enabled in HP-UX-11i, and is therefore
configured in the HP-UX 11i client profile.

.. _configuring_client_authentication_3:

Configuring Client Authentication
---------------------------------

1. Download the LDAP-UX Integration software version B.04.00.03 from the
following URL:
https://h20293.www2.hp.com/portal/swdepot/try.do?productNumber=J4269AA

2. The configuration steps for HP-UX11i are identical to those for 11.0
except for step 6 (search descriptor configuration). For this step,
specify “yes” at the prompt, “Do you want to specify custom search
descriptors?” The only search descriptor that is required to be
specified is the one for automounts. Enter the following search
descriptor for automounts:

::

   searchBase: dc=example,dc=com
   searchScope: sub
   searchFilter: (objectclass=automount)

3. Create the /etc/ldap.conf file if it does not exist.

.. _configuring_client_tls_3:

Configuring Client TLS
----------------------

The SSL/TLS settings are only required if you want to use SSL between
the clients and the server when performing operations such as account
lookups.

**To set up TLS on the clients:**

1. Obtain the cert8.db and key.db for the CAcert. This can be obtained
from the CA by following the steps on p.42 of the HP-UX Client Setup
Guide, or using certutil as described on p.43. The cert8.db and key.db
tested at example were obtained from Mozilla as described on p.43.

2. Copy key.db and cert8.db into /etc/opt/ldapux

3. Run the client setup program /opt/ldapux/config/setup as described
above. This only needs to be done for the first client. Be sure to
specify SSL and SSL/simple authentication.

At example, running the setup program resulted in the following profile
being created:

::

   cn=ldapuxprofile-tls,ou=profile,dc=example,dc=com

4. For subsequent clients, the following files need to be copied into
/etc/opt/ldapux:

-  cert8.db
-  ldapux_client.conf
-  key3.db
-  pcred

The simple instructions on how to set up the machine using these files
are detailed on p.63 of the HP-UX Client Configuration Guide.

.. _aix_5.1:

AIX 5.1
=======

.. _building_nss_ldap:

Building nss_ldap
-----------------

AIX 5.1 does not deliver a native LDAP client. For these clients, the
nss_ldap module from padl.com must be compiled and installed.

**To build nss_ldap:**

1. Install the following install package: bos.adt.syscalls

2. Install the following rpms from the IBM toolkit:

-  libgcc-3.3.2-5
-  gcc-3.3.2-5
-  openldap-2.0.21-3
-  openldap-devel-2.0.21-3

3. Obtain and install the nss_ldap_226 source RPM from the Red Hat
source rpms. At the time this compilation was done, the latest version
available from padl.com was not stable.

4. Untar the nss_ldap source tarball and configure using the following
command:

::

   LDFLAGS="-L/opt/freeware/lib" LIBS=-lc \
           CPPFLAGS="-I/opt/freeware/include  -D_LINUX_SOURCE_COMPAT 
           -DPAM_EXTERN=" ./configure --with-ldap-lib=openldap 
           --with-ldap-conf-file=/etc/pam_ldap.conf –enable-ssl

5. Modify the following lines in the Makefile:

::

   nss_ldap_so_LDFLAGS = -bM:SRE -bnoentry -bE:$(srcdir)/exports.aix -brtl -lc
            -L/opt/freeware/lib
   NSS_LDAP_LDFLAGS = -bM:SRE -enss_ldap_initialize -brtl -lc -L/opt/freeware/lib

6. Run gmake

We have configured the compilation to use the /etc/pam_ldap.conf
configuration file, because IBM SecureWay uses /etc/pam.conf. The
compilation should have created the following files:

-  nss_ldap.so
-  NSS_LDAP
-  nsswitch.ldap
-  ldap.conf

7. Rename ldap.conf to pam_ldap.conf

These are the files that need to be installed on each client.

.. _installing_and_configuring_nss_ldap:

Installing and Configuring nss_ldap
-----------------------------------

1. The compiled NSS_LDAP module has been delivered as a tarball. Extract
the tarball.

2. Copy nss_ldap.so to /usr/lib/netsvc/dynload/nss_ldap.so. Create the
directory if it does not exist.

3. Copy NSS_LDAP to /usr/lib/security.

4. Make sure the two files have executable permissions.

5. If irs.conf exists, modify it to use nss_ldap (as appropriate).
Create this file if it does not exist. A sample file is given below:

::

   hosts dns continue
   hosts nis continue
   hosts local
   services nss_ldap continue
   services nis continue
   services local
   networks dns continue
   networks nss_ldap continue
   networks nis continue
   networks local
   netgroup nss_ldap continue
   netgroup nis continue
   netgroup local
   protocols nss_ldap continue
   protocols nis continue
   protocols local

6. Add the following stanza to /lib/security/methods.cfg:

::

   LDAP:
       program = /usr/lib/security/NSS_LDAP

7. Comment out any references to LDAP, because these are for the IBM
SecureWay product.

8. Edit /etc/security/user. Modify the "SYSTEM" attribute of the
"default" entry to "compat OR LDAP"

9. Modify ldap.conf to add the server name and other attributes. A
sample ldap.conf is provided in the Appendix. Copy this file to
/etc/pam_ldap.conf

10. Modify /etc/nsswitch.conf to add ldap as appropriate.

.. _configuring_client_tls_4:

Configuring Client TLS
----------------------

Because nss_ldap has been compiled with SSL and TLS support, it is
likely the following steps result in a working TLS enabled system. Due
to time constraints, these steps are untested.

**The following steps? Nothing provided here.**

.. _aix_5.2:

AIX 5.2
=======

.. _configuring_client_authentication_4:

Configuring Client Authentication
---------------------------------

1. Client authentication is performed using IBM's native LDAP client.
Client configuration is performed using the mksecldap command, as
follows:

::

   mksecldap -c -h ldap03.example.com -a "cn=Directory Manager” \ 
           -p mysecret -d "dc=example,dc=com" -u NONE

2. The mksecldap command modifies the /etc/security/ldap/ldap.cfg
configuration file based on the mappings it was able to find in the
Directory Server. These mappings are likely to be incorrect, however,
and should be modified to reflect the correct information as in the
sample below. Full configuration files are found in the Appendix.

::

   # Base DN where the user and group data are stored in the LDAP server.
   # e.g., if user foo's DN is: username=foo,ou=aixuser,cn=aixsecdb
   # then the user base DN is: ou=aixuser,cn=aixsecdb
   userbasedn:dc=example,dc=com
   groupbasedn:dc=example,dc=com 
   idbasedn:dc=example,dc=com 
   hostbasedn:dc=example,dc=com 
   servicebasedn:ou=services,dc=example,dc=com 
   protocolbasedn:ou=protocols,dc=example,dc=com
   networkbasedn:ou=networks,dc=example,dc=com
   netgroupbasedn:dc=example,dc=com 
   rpcbasedn:ou=rpc,dc=example,dc=com

3. If NIS maps are present, the mksecldap command also modifies the
/etc/irs.conf file. Make sure that the information in this file is
correct. An example is shown below.

::

   hosts dns continue
   hosts nis continue
   hosts local
   services nis_ldap continue
   services nis continue
   services local
   networks nis_ldap continue
   networks dns continue
   networks nis continue
   networks local
   netgroup nis_ldap continue
   netgroup nis continue
   netgroup local
   protocols nis_ldap continue
   protocols nis continue
   protocols local

4. If NIS maps are present, the mksecldap command modifies the
/etc/netsvc.conf file. Make sure these settings are correct, as follows:

::

   hosts = nis_ldap, bind, nis, local

.. _configuring_client_tls_5:

Configuring Client TLS
----------------------

The SSL/TLS settings are only required if you want to use SSL between
the clients and the server when performing operations such as account
lookups. It is possible to set up TLS for AIX 5.2. A full set of
instructions will be provided at a later date.

.. _aix_5.3:

AIX 5.3
=======

.. _configuring_client_authentication_5:

Configuring Client Authentication
---------------------------------

Use the same configuration as for AIX 5.2.

.. _configuring_client_tls_6:

Configuring Client TLS
----------------------

The SSL/TLS settings are only required if you want to use SSL between
the clients and the server when performing operations such as account
lookups.

**To set up TLS on the clients:**

1. Install the GSK kit. You should have the following filesets
installed:

-  gskak.rte 6.0.5.41 C F AIX Certificate and SSL Base
-  gsksa.rte 7.0.3.3 C F AIX Certificate and SSL Base
-  gskta.rte 7.0.3.3 C F AIX Certificate and SSL Base

2. Install the ldap.max_crypto_client filesets. These are found on the
expansion pack CD:

-  ldap.max_crypto_client.adt
-  ldap.max_crypto_client.rte

3.Create the key DB:

::

   gsk7cmd -keydb -create -db key.kdb -pw redhat123 -type cms

4. Copy over the DER binary CA Cert file. This is obtained by running
the following commands on the ldap01 server:

::

   cd /opt/redhat-ds/alias
    ../shared/bin/certutil -L -d . -P slapd-ldap01- -n "Certificate Manager" \
   -r > cacert.der

5. Install the CA cert and make it trusted:

::

   gsk7cmd -cert -add -db /usr/ldap/key.kdb -pw redhat123  -label "ldap08 CA Cert" -trust enable -format binary -file cacert.der

6. Now configure the ldap client:

::

   mksecldap -c -h ldap01.example.com -a "cn=Directory Manager" \
   -p redhat123 -k /usr/ldap/key.kdb -w redhat123 -d "dc=example,dc=com"

7. Modify the userbasedn, groupbasedn etc., in
/etc/security/ldap/ldap.conf as appropriate.

8. Restart the ldap client:

::

   stop-secldapclntd
   flush-secldapclntd
   start-secldapclntd

.. _aix_sudo:

AIX Sudo
========

Sudo on AIX can be configured to retrieve its rules from the IPA server
the same way Linux clients do, although it needs a little more effort to
get it working. For further details on how to setup this functionality
on AIX see: `SUDO Integration for AIX <SUDO_Integration_for_AIX>`__.

.. _aix_enabling_password_change_on_client:

AIX : Enabling password change on client
========================================

To enable users to change their password through the AIX Client, it's
very simple.

1. Edit the /etc/security/ldap/ldap.cfg

2. Change the value of authtype parameter from unix_auth to ldap_auth

Example:

::

   root@localhost - PROD# grep authtype /etc/security/ldap/ldap.cfg
   #authtype:unix_auth
   authtype:ldap_auth
   root@localhost- PROD#

3. Restart LDAP client, and try to change a user password.

::

   root@localhost - PROD# stop-secldapclntd && sleep 3 && start-secldapclntd
   The secldapclntd daemon terminated successfully.
   Starting the secldapclntd daemon.
   The secldapclntd daemon started successfully.
   root@localhost - PROD#

.. _client_configuration_files:

Client Configuration Files
==========================

-  `nsswitch.conf <SolarisNsswitchConf>`__
-  `pam.conf <SolarisPamConf89>`__ for Solaris 8/9
-  `pam.conf <SolarisPamConf10>`__ for Solaris 10
-  `ldap_client_file <SolarisLdapClientFile>`__
-  `ldap_client_cred <SolarisLdapClientCred>`__

.. _sample_profiles:

Sample Profiles
===============

Solaris
-------

-  `Sample Non-TLS Profile for Solaris <SolarisNonTlsProfile>`__
-  `Sample TLS Profile for Solaris <SolarisTlsProfile>`__

.. _hp_ux:

HP-UX
-----

-  `Sample Non-TLS Profile for HP-UX <HpuxNonTlsProfile>`__
