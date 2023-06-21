Tool_to_Check_Status_of_All_Replicas
====================================

Overview
========

Related tickets `#3068 <https://fedorahosted.org/freeipa/ticket/3068>`__

Tool for FreeIPA which will check status of every replica in FreeIPA
infrastructure. We can check status of FreeIPA related services via
ipactl status command on each replica but we do not have an information
about services running on other replicas. I chose an approach to have a
running net-snmp agent on each replica. Every agent has a set of
services in its configuration file which statuses will be check.



Use Cases
=========

On each replica will be a client which sends snmp request message to
agents on replicas. Agents send back a message or messages with
information about number of running services of each name. This
communication is asynchronous therefore timeouts for waiting for replies
will be needed.

Design
======

When FreeIPA server starts then ipactl gets a set of services for each
replica from directory server and saves them to the net-snmp agent’s
config file. Net-snmp agent cannot check status of services because it
can only check the number of running processes of each service name
(names which could be visible with "/bin/ps -e") in config file. NTP
service could be configured as FreeIPA part but this information is not
stored in Directory Server as for other services, for example like DNS.
I choose solution when I "grep" /var/lib/ipa/sysrestore/sysrestore.state
file for presence '[ntpd]' option in it.

Client will be authorized to agent via SNMPv3 security possibilities.
SNMPv3 is used for authentication and for privacy. Each replica should
have own account for access to SNMP MIB. Replicas could be created or
removed dynamically, but net-snmp loads account informations during
start from file. Solution is to restart net-snmp daemons on all replicas
after every infrastructure change. This will be the first solution.

Better soulution is implement LDAP backend support for net-snmp
(credentials for client authentication and privacy should be stored
here). New LDAP schema for this support should be defined. Net-snmp
agent then could send query to local 389 DS instance and look up account
information in it. Agent can verify client’s identity after he gets hash
of client passphrase. This extension will be implement later.

.. figure:: Tool_to_check_statuses_of_services.png
   :alt: 650|x

   650|x



Example agent's configuration
-----------------------------

| ``   /etc/snmp/snmpd.conf``
| ``   ### Added by IPA Installer ###``
| ``   proc krb5kdc``
| ``   proc kadmind``
| ``   proc named``
| ``   proc memcached``
| ``   proc httpd``
| ``   proc java``
| ``   proc ntpd``
| ``   proc ns-slapd``
| ``   ``
| ``   createUser  myuser SHA authpassphrase AES privpassphrase``
| ``   rouser -s usm myuser authpriv .1.3.6.1.4.1.2021.2``

proc option checks number of running processes "name".

createUser option creates user "myuser" with SHA authentication password
"authpassphrase" and with AES "privpassphrase" password for privacy.

rouser specifies read-only access for user "myuser" to
".1.3.6.1.4.1.2021.2 " OID tree. This OID is subtree which stores
information for proc options. authpriv specifies minimum security level
which enforces use of authentication and encryption.

Proc java is in example because pki-tomcatd is visible only as "java"
with /bin/ps -e.



Example how to get data from agent
----------------------------------

``   snmpwalk -v 3 -u myuser -l authPriv -a SHA -A authpassphrase -x AES -X privpassphrase localhost .1.3.6.1.4.1.2021.2 ``

Implementation
==============

Any additional requirements or changes discovered during the
implementation phase.



Feature Management
==================

Installation
------------

Tool is optional component of FreeIPA and it could be enabled with
--setup-snmp parameter in install script (ipa-server-install). If master
server is installed with --setup-snmp parameter then tool will be
enabled on all replicas. Enabled services in Directory Server on FreeIPA
master server will be checked during replica installation. If entry for
SNMP service will be present, then tool will be enabled.

UI

All command line possibilities can be accesible via web browser FreeIPA
interface.

CLI

Command to gain information could be something like ipa replica-status .
Command will use a net-snmp client, which sends a request to all agents
and client waits for replies. If filter is specified then only status of
specified services or status of specified replicas will be printed.

Client needs to know names of all replicas in FreeIPA infrastructure.
This names should be get from directory server. Entries of connected
replicas are stored in cn=master,cn=ipa,cn=etc,dc= subtree, for example
cn=master,cn=ipa,cn=etc,dc=.

Command should accept timeout argument like --timeout=10, by default
should be used some value, for example 10 seconds.

Filter for replicas should be --replica=name1 --replica=name2. Filter
for services should be --service=name1 --service=name2.

Client will use net-snmp-utils commands, especially snmpwalk to get data
from SNMP agent.



Major configuration options and enablement
==========================================

N/A

Replication
===========

N/A



Updates and Upgrades
====================

N/A

Dependencies
============

net-snmp and net-snmp-utils packages will be needed



External Impact
===============

N/A



Design page authors
===================

`Dspurek <User:Dspurek>`__