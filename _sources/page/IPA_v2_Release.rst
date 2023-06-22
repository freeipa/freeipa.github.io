IPA_v2_Release
==============



IPA v2.0 Release
----------------------------------------------------------------------------------------------

**Release Date**: Mar 2011

**Overview**: Machine and Service Identity. Pluggable management, HBAC.

-  `Requirements Doc <http://www.freeipa.org/page/V2BPRD>`__
-  `Detailed outline of the IPA v2
   Features <http://www.freeipa.org/page/V2Outline>`__

**Components**: The release will include:

-  Linux Distribution (Fedora / Red Hat Enterprise Linux / CentOS)
-  389 Directory Server
-  MIT Kerberos
-  NTP
-  Tools for installation
-  Pluggable and extensible UI/CLI tools
-  CA & RA (Dogtag Certificate Server)
-  DNS (Bind)

**Main Use Cases for IPAv2**

-  User Identity Management (based on functionality implemented in v1)
-  Machine identity

   -  Enrollment of the new machines

      -  As a result of the enrollment machine principal must be created
         and machine credentials provisioned to the machine
      -  Machine credentials can be keytab and/or machine certificate.

   -  Machine authentication

      -  Machines coming on the network and requesting services within
         the IPA realm shall be authenticated against that realm
      -  Machine authentication credentials shall be used to provide
         mutual authentication/trust, encryption, and SSO capabilities
         for the services and applications requesting resources and
         accessing other services within the same IPA realm

-  Machine Management

   -  Allow management of individual machines, groups of machines and
      virtual instances
   -  Allow centralized management of different kinds of machine
      policies

-  Access Control

   -  Enable central management of PAM login access controls (HBAC -
      host based access control)

**Compelling Reason to Use**

-  Compliance and efficiency are forcing organizations to move off NIS
   and pushing them to use a better identity management and access
   control solution for the Linux/Unix world
-  Efficiency is forcing organizations to use a better identity
   management solution
-  Too expensive to maintain own custom LDAP/Kerberos implementation
-  Have been using services that "assume a security mechanism" and wish
   to secure connections with kerberos or PKI
-  Compliance and efficiency motivate to centrally manage administrator
   delegation