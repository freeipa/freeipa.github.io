Configuring AIX clients
=======================

Introduction
============

This document describes the procedures required to configure AIX 7.3 as
an IPA client, hints for earlier AIX versions are also included below.

Prerequisites
-------------

Before you begin the configuration phase, you need to ensure that the
following software is installed and up to date on your system. 

+--------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------+
| Component                                        | Where to find                                                                                                                       |
+==================================================+=====================================================================================================================================+
| AIX 7.3                                          | `IBM Entitled Systems Support site <https://www.ibm.com/servers/eserver/ess/index.wss>`__                                           |
+--------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------+
| IBM Network Authentication Service (Kerbreros 5) | AIX Expansion Pack DVD or `AIX Web Pack site <https://www.ibm.com/resources/mrs/assets?source=aixbp>`__                             |
+--------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------+
| IBM Security Verify Directory                    | AIX media or `AIX Web Pack site <https://www.ibm.com/resources/mrs/assets?source=aixbp>`__                                          |
+--------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------+
| GSKit                                            | AIX Expansion Pack DVD or `AIX Web Pack site <https://www.ibm.com/resources/mrs/assets?source=aixbp>`__                             |
+--------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------+
| sudo_ids RPM package                             | `AIX Toolbox for Open source applications <https://www.ibm.com/support/pages/aix-toolbox-open-source-software-downloads-alpha#S>`__ |
+--------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------------------+

You must also obtain several parameters from your FreeIPA installation before proceeding further.

* FreeIPA server names and their IP addresses
* FreeIPA DNS domain
* FreeIPA Kerberos realm
* FreeIPA LDAP base DN
* :abbr:`HBAC (Host-based access control)` rule ID for access to your AIX server (:command:`ipa hbacrule-show <name> --all | grep dn:`), in case you use `Host-based access control <https://www.freeipa.org/page/Howto/HBAC_and_allow_all>`__ in your enviornment.

Your AIX server must be registered in FreeIPA as a host and you must have Kerberos keytab generated for the host.

We use the following values as example:

::

    IPA server = ipaserver.example.com, 10.0.0.1
    IPA DNS domain = example.com
    IPA Kerberos realm = EXAMPLE.COM
    IPA LDAP base DN = dc=example,dc=com
    HBAC rule ID = ipaUniqueID=12345678-1234-1234-1234567890ab,cn=hbac,dc=example,dc=com
    AIX server = aix73.example.com, 10.0.0.73
    AIX Kerberos principal = host/aix73.example.com@EXAMPLE.COM


Installing prerequisites on AIX 7.3
===================================

Installing GSKit
----------------

Install both 32- and 64-bit packages for GSKit8. If there are any updates for GSKit, install the latest updates.

Installing Kerberos
-------------------

You need only client packages - :samp:`krb5.client.rte` fileset.

Installing LDAP packages
------------------------

Mount AIX DVD image to your AIX server and accept Security Verify Directory license. In the example below the image is mounted under :file:`/mnt`.

::

    /mnt/license/idsLicense

Press :kbd:`1` to accept the license.

Install the following filesets and their dependencies:

+------------------------------------+
+ Fileset                            +
+====================================+
| idsldap.clt64bit100.rte            |
+------------------------------------+
| idsldap.clt32bit100.rte            |
+------------------------------------+
| idsldap.clt_max_crypto32bit100.rte |
+------------------------------------+
| idsldap.clt_max_crypto64bit100.rte |
+------------------------------------+

After the installation create symlinks for the LDAP libraries and commands:

::

    /opt/IBM/ldap/V10.0/bin/idslink -i -l 64 -s base -f
    /opt/IBM/ldap/V10.0/bin/idslink -i -l 32 -s base -f


It is strongly recommended to install the latest fixpacks and updates for IBM Security Verify Directory.


Installing sudo
---------------

If you still don't have DNF on your AIX, install first DNF using `dnf_aixtoolbox.sh <https://public.dhe.ibm.com/aix/freeSoftware/aixtoolbox/ezinstall/ppc/dnf_aixtoolbox.sh>`__.

Use :command:`sudo_ids` package instead of :command:`sudo` because it seamlessly works with IBM LDAP libraries. If you already installed :command:`sudo` package, remove it first.

::

    /opt/freeware/bin/dnf -y install sudo_ids


If during the installation you get an error like :code:`Nothing provides libibmldap.a`, recreate virtual RPM package definitions first and then install :command:`sudo_ids`.

::

    updtvpkg
    /opt/freeware/bin/dnf -y install sudo_ids


Configuring AIX 7.3 as IPA client
=================================

DNS
---

Check that you correctly configured your DNS in :file:`/etc/resolv.conf`:

::

    options rotate,timeout:10
    domain example.com
    search example.com
    nameserver 10.0.0.1


You can also add your IPA server into :file:`/etc/hosts`:

::

    10.0.0.1 ipaserver.example.com ipaserver


NIS
---

Set up NIS domain name. You need it if you work with sudo and netgroups or user groups on IPA side:

::

    chypdom -B example.com


OpenSSL
-------

Download IPA server certificate and create a hash link to it:

::

    $ curl -o /var/ssl/certs/ipa.crt https://ipaserver.example.com/ipa/config/ca.crt
    $ openssl x509 -in /var/ssl/certs/ipa.crt -noout -hash
    01234567
    $ ln -s /var/ssl/certs/ipa.crt /var/ssl/certs/01234567.0


GSKit
-----

Create a new certificate store and add IPA server certificate into it. Note the password of your certificate store. You will need it later during LDAP client configuration.

::

    $ gsk8capicmd_64 -keydb -create -db /etc/security/ldap/key.kdb -pw MYPASSWORD -type cms -stashed
    $ gsk8capicmd_64 -cert -add -file /var/ssl/certs/ipa.crt -label ipa -db /etc/security/ldap/key.kdb -stashed


LDAP mapping files
------------------

These files define how to map LDAP information to AIX internal attributes.

Create :file:`/etc/security/ldap/ipagroup.map`:

::

    keyobjectclass  SEC_CHAR    posixgroup      s   na  yes
    groupname       SEC_CHAR    cn              s   na  yes
    id              SEC_INT     gidnumber       s   na  yes
    users           SEC_LIST    member          m   na  yes


Create :file:`/etc/security/ldap/ipauser.map`:

::

    keyobjectclass  SEC_CHAR    posixaccount        s   na      yes
    username        SEC_CHAR    uid                 s   na      yes
    id              SEC_INT     uidnumber           s   na      yes 
    pgrp            SEC_CHAR    gidnumber           s   na      yes
    home            SEC_CHAR    homedirectory       s   na      yes
    shell           SEC_CHAR    loginshell          s   na      yes
    gecos           SEC_CHAR    gecos               s   na      yes
    spassword       SEC_CHAR    userpassword        s   na      yes
    lastupdate      SEC_INT     nonexistingattr     s   days    yes
    account_locked  SEC_BOOL    nsaccountlock       s   na      yes
    auth_name       SEC_CHAR    krbprincipalname    s   na      yes


LDAP client configuration
-------------------------

Create :file:`/etc/security/ldap/ldap.cfg`, **note that you don't want to allow locked users**:

::

    ldapservers:ipaserver.example.com
    authtype:ldap_auth
    useSSL:yes
    verifyCertificate:yes
    ldapsslkeyf:/etc/security/ldap/key.kdb
    ldapsslkeypwd:MYPASSWORD
    userattrmappath:/etc/security/ldap/ipauser.map
    groupattrmappath:/etc/security/ldap/ipagroup.map
    useKRB5:yes
    krbcmddir:/usr/krb5/bin/
    krbkeypath:/etc/krb5/krb5.keytab
    krbprincipal:host/aix73.example.com@EXAMPLE.COM
    defaultentrylocation:local
    userbasedn:cn=users,cn=accounts,dc=example,dc=com??(!(nsaccountlocked=TRUE))
    groupbasedn:cn=groups,cn=accounts,dc=example,dc=com
    netgroupbasedn:cn=ng,cn=compat,dc=example,dc=com
    userclasses:posixaccount
    groupclasses:posixgroup
    searchmode:OS
    enableutf8_xlation:no
    serverschematype:rfc2307
    memberfulldn:yes
    resolveUserFromDN:yes


You need `netgroupbasedn` parameter only if you use sudo with netgroups. If you don't use sudo with netgroups, you can drop the parameter.

The password to the GSKit certificate store (`ldapsslkeypwd`) can be encrypted using :command:`secldapclntd -e` command. But the option is not documented.

HBAC (Host-based access control)
--------------------------------

You can have **true** :abbr:`HBAC (Host-based access control)` using `pam_ipahbac <https://github.com/rseabra/pam_ipahbac/>`, after installation you place a `/etc/ipahbac.conf` file with the pam module's configuration:

::

    -u YourSysAccount
    -b dc=your,dc=domain
    -P /etc/ldap.secret
    -l ldaps://ldap1/,ldaps://ldap2/..

And add the following to `/etc/pam.cfg`:

::

    sshd account    required     pam_ipahbac.so /etc/ipahbac.conf


**Alternatively**, if you don't mind using a limited version of HBAC support, you can change your *userbasedn* field in **ldap.cfg** to check the user properties for being a member of a particular HBAC rule:

::

    (...)
    userbasedn:cn=users,cn=accounts,dc=example,dc=com??(&(!(nsaccountlocked=TRUE))(memberOf=ipaUniqeID=12345678-1234-1234-1234567890ab,cn=hbac,dc=example,dc=com))
    (...)

If you don't want to use HBAC, or prevent locked users from logging in, your `userbasedn` parameter may be specified without any additional filters, like:

::

    userbasedn:cn=users,cn=accounts,dc=example,dc=com


Home directories
----------------

Unless you use NFS or something else to host your users' home directories, enable automatic creation of home directories:

::

    chsec -f /etc/security/login.cfg -s usw -a mkhomeatlogin=true


Domainless groups
-----------------

To enable LDAP users to be included into local AIX groups, enable domainless groups setting:

::

    chsec -f /etc/secvars.cfg -s groups -a domainlessgroups=true


Kerberos configuration
----------------------

Copy your AIX server's keytab into :file:`/etc/krb5/krb5.keytab` and create :file:`/etc/krb5/krb5.conf`:

::

    [libdefaults]
        default_realm = EXAMPLE.COM
        default_keytab_name = FILE:/etc/krb5/krb5.keytab
        permitted_enctypes = aes256-cts-hmac-sha384-192 aes128-cts-hmac-sha256-128 aes256-cts-hmac-sha1-96 aes128-cts-hmac-sha1-96
        dns_lookup_realm = false
        dns_lookup_kdc = false
        rdns = false
        forwardable = true
        renewable = true
        canonicalize = true

    [realms]
        EXAMPLE.COM = {
            kdc = ipaserver.example.com:88
            admin_server = ipaserver.example.com:749
            kpasswd_server = ipaserver.example.com:464
            default_domain = example.com
        }

    [domain_realm]
        .example.com = EXAMPLE.COM
        ipaserver.example.com = EXAMPLE.COM

    [logging]
        default = FILE:/var/krb5/log/krb5lib.log


sudo configuration
------------------

Create :file:`/etc/irs.conf` to enable searching through netgroups if you use netgroups:

::

    netgroup nis_ldap


Create :file:`/etc/sudo-ldap.conf`:

::

    uri ldaps://ipaserver.example.com
    ssl start_tls
    tls_key /etc/security/ldap/key.kdb
    tls_keypw MYPASSWORD
    use_sasl yes
    sasl_auth_id host/aix73.example.com@EXAMPLE.COM
    krb5_ccname /etc/security/ldap/krb5cc_secldapclntd
    sudoers_base ou=sudoers,dc=example,dc=com
    netgroup_base cn=ng,cn=compat,dc=example,dc=com
    netgroup_query yes


The file with Kerberos credentials cache is automatically created and updated by AIX LDAP client.

The password for GSKit certificate store (`tls_keypw`) must be in clear text.

If you don't use netgroups, you can remove the last two lines of the configuration file (`netgroup_base` and `netgroup_query`).


Authentication methods
----------------------

Add the following lines into :file:`/usr/lib/security/methods.cfg` to enable LDAP and Kerberos on AIX:

::

    LDAP:
        program = /usr/lib/security/LDAP
        program_64 = /usr/lib/security/LDAP64

    KRB5:
        program = /usr/lib/security/KRB5
        program_64 = /usr/lib/security/KRB5_64
        options = authonly

    KRB5LDAP:
        options = db=LDAP,auth=KRB5


Existing user migration
-----------------------

Set attributes `registry` and `SYSTEM` for local users, especially for system users like root, using :command:`chsec`:

::

    chsec -f /etc/security/user -s root -a registry=files -a SYSTEM=compat


.. warning::

    Don't use :command:`chuser` or :command:`smitty` to set the attributes in this case!


Enable user authentication through IPA server
---------------------------------------------

By default all users which are not defined in :file:`/etc/passwd` must be sought in LDAP and authenticated using Kerberos:

::

    chsec -f /etc/security/user -s default -a registry=KRB5LDAP -a SYSTEM=KRB5LDAP


Start LDAP client
-----------------

Start LDAP client:

::

    start-secldapclntd


Add the start of LDAP client into boot process:

::

    mkitab -i rctpip "ldapclntd:23456789:wait:/usr/sbin/start-secldapclntd 2>&1"


Troubleshooting
---------------

Use :command:`ls-secldapclntd` to check if LDAP client is working and connected to the IPA server.

Use :command:`lsldap` to check which objects can be found in LDAP using the configuration.

To check Kerberos authentication, use :command:`/usr/krb5/bin/kinit`. Check that you use :command:`kinit` from :command:`/usr/krb5/bin`, because sometimes Java's :command:`kinit` has precedence in `PATH` environment variable. Java's :command:`kinit` will not work.


Configuring the IPA Client on AIX 5.3
=====================================

The following instructions describe how to configure AIX 5.3 as an IPA
client. The following hostnames are used as examples only; you need to
replace these with the hostnames that apply to your deployment.

::

    REALM = EXAMPLE.COM
    IPA server = ipaserver.example.com
    IPA client = ipaclient.example.com



Configuring Kerberos and LDAP
-----------------------------

1. Configure the krb5 client settings as follows:

::

    # mkkrb5clnt -r EXAMPLE.COM -d example.com -c ipaserver.example.com -s ipaserver.example.com

2. Get a Kerberos ticket.

::

    # kinit  admin

3. Configure the LDAP client settings as follows:

::

    # mksecldap -c -h ipaserver.example.com -d cn=accounts,dc=example,dc=com -a uid=nss,cn=sysaccounts,cn=etc,dc=example,dc=com -p secret

4. Add custom settings for the LDAP client.

Under /etc/security/ldap create 2 new map files:

::

      #IPAuser.map file
      keyobjectclass  SEC_CHAR        posixaccount            s

      # The following attributes are required by AIX to be functional
      username        SEC_CHAR        uid                     s
      id              SEC_INT         uidnumber               s
      pgrp            SEC_CHAR        gidnumber               s
      home            SEC_CHAR        homedirectory           s
      shell           SEC_CHAR        loginshell              s
      gecos           SEC_CHAR        gecos                   s
      spassword       SEC_CHAR        userpassword            s
      lastupdate      SEC_INT         shadowlastchange        s

..

::

      #IPAgroup.map file
      groupname       SEC_CHAR    cn                    s
      id              SEC_INT     gidNumber             s
      users           SEC_LIST    member                m

..

    | Change the /etc/security/ldap/ldap.cfg file and set the relevant options as follow.
    | In this example the REALM name is EXAMPLE.COM and the basedn is dc=example,dc=com
    | Change all basedns values to conform to your installation realm name.

::

   userbasedn:cn=users,cn=accounts,dc=example,dc=com
   groupbasedn:cn=groups,cn=accounts,dc=example,dc=com

   userattrmappath:/etc/security/ldap/IPAuser.map
   groupattrmappath:/etc/security/ldap/IPAgroup.map

   userclasses:posixaccount

5. Start the ldap client daemon.

::

    # start-secldapclntd

6. Test the LDAP client connection to the IPA server.

::

    # lsldap -a passwd

7. Configure the system login to use Krberos and LDAP

Add the following sections to the file /usr/lib/security/methods.cfg

::

      KRB5A:
              program = /usr/lib/security/KRB5A
              program_64 = /usr/lib/security/KRB5A_64
              options = authonly

      KRB5ALDAP:
              options = auth=KRB5A,db=LDAP


For AIX 6.1 the line

::

              options = authonly

should be changed into

::

              options = authonly,kadmind=no

..

    | Edit the file /etc/security/user
    | In the default section change the options 'SYSTEM' and 'registry' to look like this:

::

           SYSTEM = "KRB5ALDAP"
           regisrty = LDAP


Please note: due to these changes to /etc/security/user LDAP is
configured, leading to local users with no individual entry not beeing
able to login. According to previous testing not setting registry to
LDAP is not preventing IPA users to login, but is preventing them to
change passwords.



Configuring Client SSH Access
-----------------------------

You can configure the IPA client to accept incoming SSH requests and
authenticate with the user's Kerberos credentials. After configuring the
IPA client, use the following procedure to configure the IPA client for
SSH connections. Remember to replace the example host and domain names
with your own host and domain names:

1. SSH syslog Configuration

::

           auth.info       /var/log/sshd.log
           auth.info       /var/log/sshd.log
           auth.crit       /var/log/sshd.log
           auth.warn       /var/log/sshd.log
           auth.notice     /var/log/sshd.log
           auth.err        /var/log/sshd.log

2. SSH Logging Configuration

::

           SyslogFacility AUTH
           LogLevel INFO

3. Configure sshd for GSSAPI (``/etc/ssh/sshd_config``)

::

           # GSSAPI options
           GSSAPIAuthentication yes
           #GSSAPICleanupCredentials yes

4. Restart sshd

::

           # stopsrc -s sshd
           # startsrc -s sshd

5. Restart syslogd

::

           # stopsrc -s syslogd
           # startsrc -s syslogd

..

   |Note.png| **Note:**

      The **ipa-admintools** package is not available for AIX.
      Consequently, you need to perform the following commands on the
      IPA server.

6. Add a host service principal on IPA v2:

::

           # ipa service-add host/ipaclient.example.com

Please note: adding the service principal should no longer be required,
but host-add and a host-add-managedby should be enough:

::

       # ipa host-add ipaclient.example.com
       # ipa host-add-managedby --hosts=ipaserver.example.com ipaclient.example.com

7. Retrieve the host keytab.

::

           # ipa-getkeytab -s ipaserver -p host/ipaclient.example.com -k /tmp/krb5.keytab

8. Copy the keytab from the server to the client.

::

           # scp /tmp/krb5.keytab root@ipaclient.example.com:/tmp/krb5.keytab

9. On the IPA client, use the **ktutil** command to import the keytab.

::

           # ktutil
           ktutil: read_kt /tmp/krb5.keytab
           ktutil: write_kt /etc/krb5/krb5.keytab
           ktutil: q

10. Add a user that is only used for authentication. (This can be
substituted with krb5 authentication if that works from the ldap
client). Otherwise go to the IPA server and use **ldapmodify**, bind as
**Directory Manager** and create this user.

::

           dn: uid=nss,cn=sysaccounts,cn=etc,dc=example,dc=com
           objectClass: account
           objectClass: simplesecurityobject
           objectClass: top
           uid: nss
           userPassword: Your own shared password here

11. On the IPA server, get a ticket for the **admin** user.

::

           # kinit admin

You should be able to log in as **admin** using ssh without providing a
password.

::

           # ssh admin@ipaclient.example.com



System Login
------------

On the AIX machine console, enter the admin username and password. You
should be able to log in.

Use the **id** command to verify user and group information.

   |Note.png|\ **Note:**

      By default, **admin** is given **/bin/bash** as the shell to use
      and ``/home/admin`` as the home directory. You may need to install
      **bash** (or link **sh** to **/bin/bash** or modify **admin** to
      use **/bin/sh** or a shell available in all of your systems) to be
      able to log in.

netgroup
--------

Some words of caution. The comment-line in most of these files in \* and
not # as is usual for \*nix configuration files. I found that deleting
values rather than trying to comment then out was more helpful.

The order in some files seems to be important for some reason. Simply
moving SYSTEM and registry in ``/etc/security/user`` once solved getting
netgroup working. My advice would be to maintain the existing formatting
and order as much as possible.

This example assumes we are going to be granting access to the machine
to users in the netgroup mygroup. To add this group to IPA and allow the
user admin do the following on the IPA server:

::

   $ kinit admin
   $ ipa netgroup-add --desc='AIX users' mygroup
   $ ipa netgroup-add-member --users=admin mygroup

1. On the AIX client, add the netgroup basedn to
``/etc/security/ldap/ldap.cfg``:

::

   netgroupbasedn: cn=ng,cn=compat,dc=example,dc=com

2. Restart the LDAP client:

::

   # /usr/sbin/restart-secldapclntd

3. Test that a netgroup is visible

::

   # lsldap -a netgroup mygroup

4. Add the netgroup option to LDAP and KRB5ALDAP in
/usr/lib/security/methods.cfg:

   ::

      LDAP: 
              program = /usr/lib/security/LDAP
              program_64 =/usr/lib/security/LDAP64
              options = netgroup
              ...
      KRB5ALDAP:
              options = auth=KRB5A,db=LDAP,netgroup

5. Configure ``/etc/irs.conf`` for netgroups:

::

   # cat /etc/irs.conf
   netgroup nis_ldap

6. Either modify the default user in ``/etc/security/user`` to use
``SYSTEM = compat`` and ``registry = compat`` or create a user ``admin``
entry and configure that for compat:

This is what it looks like if you create a separate user:

   ::

      default:
      ...
              SYSTEM = compat
              registry = compat
              ...

OR

   ::

      admin:
          SYSTEM = compat
          registry = compat   

7. Add our netgroup to ``/etc/passwd``:

::

   echo "+@mygroup" >> /etc/passwd

8. Configure ``/etc/group`` for netgroups:

::

   # echo "+:" >> /etc/group

9. Test the admin user:

   ::

      # lsuser -R compat admin
      admin id=155000000 pgrp=admins groups=admins home=/home/admin...

10. A full test is to su to the user or log in via ssh.

For users not found in the netgroup you'll get a log entry like this in
``/var/log/sshd.log``

``Aug 24 10:38:29 ipaaix auth|security:info sshd[348394]: Invalid user admin from x.x.x.x``
``Aug 24 10:38:30 ipaaix auth|security:info syslog: ssh: failed login attempt for UNKNOWN_USER from x.x.x.x``
``Aug 24 10:38:30 ipaaix auth|security:info sshd[348394]: Failed none for invalid user admin from x.x.x.x port 34085 ssh2``



Additional help
---------------

Some additional AIX configuration pages that may be relevant and
helpful.

-  http://www.ibm.com/developerworks/aix/library/au-aixadsupport.html
-  http://www.ibm.com/developerworks/aix/library/au-netgroup/
-  http://publib.boulder.ibm.com/infocenter/pseries/v5r3/index.jsp?topic=/com.ibm.aix.security/doc/security/krb_bind_ldap_client.htm

.. |Note.png| image:: Note.png
