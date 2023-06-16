Overview
========

Some new features were introduced in Managed Topology functionality of
IPA with IPA version 4.4.

#. 'ipa-server-install --uninstall' executed on replica now triggers the
   remote 'server_del' API call on master causing removal of RUV's which
   eliminates the necessity to run 'ipa-replica-manage del' on master to
   correctly clean replication agreements.
#. A replica can be now removed on the server with the command 'ipa
   server-del' - it serves as a combination of 'ipa-csreplica-manage
   del' and 'ipa-replica-manage del'
#. ipa-replica-manage list-ruv now lists not only domain-related Replica
   Update Vectors (RUVs), but also ca-specific
#. 'ipa-replica-manage del' now correctly cleans all RUV's even if the
   replica is offline (before that the RUV's were cleaned only if the
   replica could be contacted during the process)



Test Plan
=========

`Category:FreeIPA V4 Test Plan <Category:FreeIPA_V4_Test_Plan>`__
`Category:FreeIPA Test Plan <Category:FreeIPA_Test_Plan>`__
