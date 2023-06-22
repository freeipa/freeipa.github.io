Dogtag9ToDogtag10Migration
==========================

FreeIPA version 3.1 and newer introduced support for Dogtag/PKI version
10 containing features making FreeIPA and CA integration more seamless.
However, due to changes in architecture of the Dogtag 10 software, there
is no automatic procedure to simply upgrade FreeIPA Dogtag 9-based
instances to Dogtag 10-based instances and a manual procedure is
required (`related Dogtag
article <http://pki.fedoraproject.org/wiki/Migrating_a_Dogtag_9_Instance_to_a_Dogtag_10_Instance>`__).



Affected systems
----------------

All FreeIPA servers which have CA configured and which were installed
prior to version `3.1 <IPAv3_310>`__. In Fedora, this is relevant for
all FreeIPA servers with CA installed in *Fedora 17* or older.



Upgrade procedure
-----------------

There are 2 options for the procedure:

#. *Manual procedure*: requires manual installation steps as described
   in the article in `PKI
   wikipedia <http://pki.fedoraproject.org/wiki/Migrating_a_Dogtag_9_Instance_to_a_Dogtag_10_Instance>`__
#. *FreeIPA migration procedure*: migrating FreeIPA servers with CA by
   creating replicas with CA capability. See details in the next
   section.



Migrating FreeIPA servers with CA
----------------------------------------------------------------------------------------------

Baseline of the procedure:

#. Prepare replica file (``ipa-replica-prepare``) and install a new
   FreeIPA replica with CA capability (``--setup-ca`` option) version
   higher or equal to 3.1
#. Shut down the old affected FreeIPA server with CA
#. Verify that CA and other identity management are OK
#. Remove the affected FreeIPA system from the infrastructure
#. Apply for all affected systems

Detailed instructions can be found in `article dedicated to
migration <Howto/Migration#Migrating_to_different_platform_or_OS>`__.