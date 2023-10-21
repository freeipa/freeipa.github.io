IpaConcepts
===========

-  http://www.freeipa.org/page/IpaConcepts#What_is_389_Directory_Server.3F

   -  Is the 389 Directory Server's web management interface used at all
      in/by IPA?
      http://www.freeipa.org/page/IpaConcepts#How_IPA_and_389_DS_Work_Together
      says that IPA uses 389 as datastore and access control mechanism
      but maybe it would be good to specifically say what the IPA and
      389 WebUI is -- can both be used, is 389's WebUI disabled when IPA
      is installed, etc.

-  http://www.freeipa.org/page/IpaConcepts#How_IPA_and_Kerberos_Work_Together

   -  The KDC abbreviation is used without being defined / explained.

-  The word "principal" is not defined nor explained really. The page
   could use some simplistic definition like "Identities in Kerberos are
   called principals and they can exist both for users
   ("user@REALM.NET") and for machines and services
   ("host/server.example.com@EXAMPLE.COM",
   "HTTP/www.example.com@EXAMPLE.COM")."

-  http://www.freeipa.org/page/IpaConcepts#What_is_NTP.3F and
   http://www.freeipa.org/page/IpaConcepts#How_IPA_and_NTP_Work_Together
   kinda talk about the same thing twice. My proposal is to just keep
   http://www.freeipa.org/page/IpaConcepts#IPA_and_NTP and put something
   like the following there:

   -  Many computer services require that the time differential between
      different hosts on a network be kept to a minimum for correct or
      accurate operation. The same is true for IPA which combines a
      number of different technologies that might constantly communicate
      with each other over the network. For example, Kerberos only
      tolerates a five minute time difference between the KDC and a
      client requesting authentication. A time difference greater than
      five minutes will result in a failed authentication.
   -  The Network Time Protocol (NTP) is a protocol used to synchronize
      computer clocks over the network. Most operating systems can be
      configured to synchronize their clocks with any of a number of
      time servers.
   -  System Administrators also rely on accurate time keeping for
      correlation of system logs across machines on the network. In the
      event of problems on the network or other aspects of a deployment,
      it may be necessary to inspect the log files of various machines
      to determine if specific problems occur at the same time. Without
      NTP or another time synchronization system, such troubleshooting
      would be all but impossible.
   -  The IPA startup process ensures that the NTP service is started
      and that the time and date is synchronized before any other
      IPA-related processes are started. This is to avoid problems with
      certificates, LDAP entry creation dates, password expiration
      dates, account expiration dates, and any other date-related
      issues.