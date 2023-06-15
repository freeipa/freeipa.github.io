Overview
========

The goal of this feature is to provide a consistent information about
the replication topology on each server. It is achieved by maintaining
the replication topology information in the shared LDAP tree. The new
feature allows to manage replication agreements via a set of standard
ipa commands and therefore deprecates a number of use-cases of
ipa-replica-manage command. Namely, it obsoletes 'connect' and
'disconnect' subcommands. Each replication agreement is referenced as a
Topology Segment. User can add and remove topology segments via ipa
topology\* subcommands and via web interface in the Topology section.
The topology graph is visualized in the aforementioned section of IPA
WebUI.



Test Plan
=========
