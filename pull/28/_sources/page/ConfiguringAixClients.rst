ConfiguringAixClients
=====================

Back to `FreeIPAv1:Client Configuration
Guide <FreeIPAv1:Client_Configuration_Guide>`__

Introduction
============

This document describes the procedures required to configure AIX 5.3 as
an IPA client, hints for later AIX versions are also included.

Prerequisites
-------------

Before you begin the configuration phase, you need to ensure that the
following software is installed and up to date on your system. This can
be installed from your AIX media:

-  v5.3 OS
-  v5.3 Updates
-  krb5 client packages
-  openssh
-  wget
-  bash
-  ldap.client (idsldap)
-  openssl
-  modcrypt.base (for gssd)



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

``   # mkkrb5clnt -r EXAMPLE.COM -d example.com -c ipaserver.example.com -s ipaserver.example.com``

2. Get a Kerberos ticket.

``       # kinit  admin``

3. Configure the LDAP client settings as follows:

``       # mksecldap -c -h ipaserver.example.com -d cn=accounts,dc=example,dc=com -a uid=nss,cn=sysaccounts,cn=etc,dc=example,dc=com -p secret``

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

| Change the /etc/security/ldap/ldap.cfg file and set the relevant
  options as follow.
| In this example the RELAM name is EXAMPLE.COM and the basedn is
  dc=example,dc=com
| Change all basedns values to conform to your installation realm name.

::

   userbasedn:cn=users,cn=accounts,dc=example,dc=com
   groupbasedn:cn=groups,cn=accounts,dc=example,dc=com

   userattrmappath:/etc/security/ldap/IPAuser.map
   groupattrmappath:/etc/security/ldap/IPAgroup.map

   userclasses:posixaccount

5. Start the ldap client daemon.

``       # start-secldapclntd``

6. Test the LDAP client connection to the IPA server.

``       # lsldap -a passwd``

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

| Edit the file /etc/security/user
| In the default section change the options 'SYSTEM' and 'registry' to
  look like this:

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