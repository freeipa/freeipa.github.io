Centralized_Logging
===================

Motivation
----------

When a system grows to multiple hosts, managing the logs and accessing
them can get complicated. Searching for a particular error across
hundreds of log files on hundreds of servers is difficult without good
tools. FreeIPA currently does not have documentation or even own tools
how to set up the infrastructure and it is being solved separately by
each FreeIPA user. This page summarizes the efforts and development of a
Centralized Logging solution in/for FreeIPA that would close that gap.

FreeIPA is a security related project, usually a backbone of the
infrastructure. Tracking, audit or analysis of the logs of the FreeIPA
server behavior and related FreeIPA user behavior is useful as it allows
detecting problems in time before they grow in even bigger problems
(e.g., replication errors) or helps detecting attacks, misbehavior or
simply configuration problems in the infrastructure and amending them.



FreeIPA logs
------------

FreeIPA server itself is not a monolithic application, but rather
compound of selected open source projects creating a solution for
identity management. This makes log collection, processing and
aggregation a challenge as each project have own way and technology how
to structure and produce logs. This applies both for FreeIPA server and
it's many services, but also for FreeIPA clients, where different
information lies in different logs (SSSD log, audit log).



Use Cases
---------

Before designing a particular solution, basic use cases and intents of
what logs will be used and how should be defined. Currently, following
set of use cases is tracked:

-  As Admin or Auditor, I want to see all calls to FreeIPA API so that I
   can audit administrative changes to FreeIPA server (for instance: who
   added user "foo"? Who revoked a certificate and when?)
-  As Auditor, I want to see all logins attempts for meeting security
   requirements and forensic analysis
-  As Security Administrator, I want to visualize failed login attempts
   to brute-force attack
-  As Admin, I want to see replication status of all my FreeIPA replicas
   so that I can amend the issue in a timely manner to avoid replication
   conflicts or out-of-sync data (I want my login to work on all
   servers)
-  As Admin, I want to identify AD interoperability problems on FreeIPA
   server (example: SSSD on the FreeIPA server cannot talk to the Active
   Directory server and thus resolve groups)
-  As Admin, I want to see load of Kerberos and LDAP server to be able
   to compare the load on the FreeIPA servers (and for example identify
   badly configured DNS SRV records or clients always calling the same
   server), network problems (some servers may not be reachable by most
   clients and have thus very small load) or simply by some of the
   services not running.



Proposed Solution
-----------------

The centralized log collection problem has 2 basic parts:

-  Selecting and configuring the right **centralized server** that will
   collect, process, annotate and visualize the logs
-  Configuring the actual machines (FreeIPA servers or clients) to
   **send the interesting logs** to the centralized server and selecting
   **proper and secure transport** for the logs.



Centralized Logging Server
----------------------------------------------------------------------------------------------

Currently, the most used and advanced open source solutions are the
centralized servers based on
`Elasticsearch <https://www.elastic.co/products/elasticsearch>`__ for
data storage and searches,
`rsyslog <http://www.rsyslog.com/>`__/`logstash <https://www.elastic.co/products/logstash>`__/`fluentd <http://www.fluentd.org/>`__
for log reception and processing and
`Kibana <https://www.elastic.co/products/kibana>`__ for data display and
dashboards (commonly referred to as REK, ELK servers based on the log
receiving part).

Currently proposed solution chosen ``rsyslog``. The server is packaged
as `Docker
image <https://registry.hub.docker.com/u/pschiffe/rsyslog-elasticsearch-kibana/>`__.
See the `Docker Hub
Registry <https://registry.hub.docker.com/u/pschiffe/rsyslog-elasticsearch-kibana/>`__
or respective `github
repo <https://github.com/pschiffe/rsyslog-elasticsearch-kibana>`__ for
details how to download and run the preconfigured REK server.



Log Gathering and Transport
----------------------------------------------------------------------------------------------

To meet the defined use cases, at least following logs have to be
gathered from the **FreeIPA server**:

-  ``/var/log/httpd/error_log``: FreeIPA API call logs (and Apache
   errors)
-  ``/var/log/krb5kdc.log``: FreeIPA `KDC <Kerberos>`__ utilization
-  ``/var/log/dirsrv/slapd-$REALM/access``: `Directory
   Server <Directory_Server>`__ utilization
-  ``/var/log/dirsrv/slapd-$REALM/errors``: `Directory
   Server <Directory_Server>`__ errors (including mentioned replication
   errors)
-  ``/var/log/pki/pki-tomcat/ca/transactions``: FreeIPA `PKI <PKI>`__
   transactions/logs

| 
| **FreeIPA client** logs:

-  ``/var/log/sssd/*.log``: SSSD logs (multiple, for all tracked logs)
-  ``/var/log/audit/audit.log``: user login attempts
-  ``/var/log/secure``: reasons why user login failed

| 
| For easier FreeIPA log forwarding configuration,
  `ipa-log-config <https://github.com/pschiffe/ipa-log-config>`__ tool
  can be used. Visit its `github
  page <https://github.com/pschiffe/ipa-log-config>`__ for more
  information and usage.

Results
----------------------------------------------------------------------------------------------

Current the proof of concept project contains:

-  Aforementioned CentOS-7 based REK server for log collection (`Docker
   image <https://registry.hub.docker.com/u/pschiffe/rsyslog-elasticsearch-kibana/>`__)
-  Set of preconfigured Kibana dashboards visualizing different logs in
   readable graphs. See the next subsection for details. (`Kibana
   dashboards and
   searches <https://github.com/pschiffe/rsyslog-elasticsearch-kibana/tree/master/kibana>`__)
-  Basic log message filtering rules for the ``rsyslog`` on the server,
   turning the log messages into structured logs that are searchable in
   Kibana (`rsyslog
   rules <https://github.com/pschiffe/rsyslog-elasticsearch-kibana/tree/master/rsyslog>`__)
-  Script to configure rsyslog-based log transmission on the FreeIPA
   servers and clients (`github
   page <https://github.com/pschiffe/ipa-log-config>`__)

Dashboards
^^^^^^^^^^

-  `User Logins <Media:Rek-user-logins.png>`__: User login attempts and
   their results across FreeIPA domain, most-logged-to hosts
-  `IPA Server
   Administration <Media:Rek-ipa-server-administration.png>`__:
   Administrative API usage on the FreeIPA servers
-  `IPA Server Utilization <Media:Rek-ipa-server-utilization.png>`__:
   Kerberos/LDAP protocol utilization across FreeIPA servers
-  `IPA Server Health <Media:Rek-ipa-server-health.png>`__: Replication
   errors, server SSSD errors
-  `SSSD <Media:Rek-sssd.png>`__: selected client SSSD errors (dashboard
   still in progress)



Demo of the Kibana Dashboards
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

{{#ev:youtube|7YjA6z5nE0I|640|center}}



Other Resources
---------------

-  `Howto/Centralised Logging with
   Logstash/ElasticSearch/Kibana <Howto/Centralised_Logging_with_Logstash/ElasticSearch/Kibana>`__
   - user provided HowTo on configuring the ELK