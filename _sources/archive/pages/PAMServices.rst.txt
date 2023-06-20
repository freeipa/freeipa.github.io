PAMServices
===========

During initial testing of Host-Based Access Control (HBAC) rules it
became apparent that a way to manage the services in the HBAC rules was
needed. The service name in an HBAC rule in general is the name of the
program used to access the service, not the program used to provide the
service (e.g. for wu-ftpd the service name is ftp), as documented at
http://www.redhat.com/docs/manuals/linux/RHL-8.0-Manual/ref-guide/s1-pam-config-files.html

Knowing the services is the first step, being able to put them into HBAC
rules is the next. The problems with the current design are:

-  A rule has only one service (or all services)
-  There is no grouping mechanism, so even if we allowed multiple
   services in a rule one would have to manually manage this.

I'm proposing a PAM service object and groups of these objects as a
solution.

These objects will be stored in ``cn=hbacservices,cn=accounts`` and
``cn=hbacservicegroups,cn=accounts``

A PAM Service object is defined as:

| ``objectClasses: (2.16.840.1.113730.3.8.4.XX NAME 'ipaHBACService' AUXILIARY``
| ``MUST ( cn ) MAY ( description ) X-ORIGIN 'IPA v2' )``

An example of a an entry is:

| ``dn: cn=sshd,cn=hbacservices,cn=accounts,dc=example,dc=com``
| ``objectClass: ipahbacservice``
| ``cn: sshd``
| ``description: SSH``

A PAM Service group is defined as:

::

   | ``objectClasses: (2.16.840.1.113730.3.8.4.XX NAME 'ipaHBACServiceGroup' ``
   | ``DESC 'IPA HBAC service group object class' SUP nestedGroup STRUCTURAL``
   | ``X-ORIGIN 'IPA v2' )``

A sample group entry looks like:

| ``dn: cn=logins,cn=hbacservices,cn=accounts,dc=example,dc=com``
| ``objectClass: ipahbacservicegroup``
| ``objectClass: nestedGroup``
| ``objectClass: groupOfNames``
| ``objectClass: top``
| ``cn: logins``
| ``description: services that allow logins``
| ``member: cn=sshd,cn=hbacservices,cn=accounts,dc=example,dc=com``
| ``member: cn=login,cn=hbacservices,cn=accounts,dc=example,dc=com``

The all rule for services will be handled by a new attribute,
serviceCategory, in the ipaHBACRule objectclass.

::

| ``attributeTypes: (2.16.840.1.113730.3.8.3.XX NAME 'serviceCategory' DESC``
| ``'Additional classification for services' EQUALITY caseIgnoreMatch ORDERING ``
| ``caseIgnoreMatch SUBSTR caseIgnoreSubstringsMatch SYNTAX ``
| ``1.3.6.1.4.1.1466.115.121.1.15 X-ORIGIN 'IPA v2' )``

IPA will ship with a set of preconfigured individual services including
an ``all`` service that means any service will match. The default
services will include:

| ``* sshd``
| ``* ftp``
| ``* su``
| ``* sudo``
| ``* su-l``
| ``* login``
| ``* sudo-i (Added in version 2.0)``
| ``* gdm (Added in version 2.0)``
| ``* gdm-password (Added in version 2.0)``
| ``* kdm (Added in version 2.0)``
| ``* gssftp (Added in version 2.1)``
| ``* proftpd (Added in version 2.1)``
| ``* pure-ftpd (Added in version 2.1)``
| ``* vsftpd (Added in version 2.1)``
| ``* crond (Added in version 3.1)``

Two new IPA plugins will be required to manage these entries:

| ``* hbacsvc``
| ``* hbacsvcgroup``

I might be convinced to not shorten service to svc but it seems like
quite a lot for the command-line.

The hbac plugin will drop the serviceName attribute and add a
memberService attribute to store the DNs of either individual services
or groups of services.