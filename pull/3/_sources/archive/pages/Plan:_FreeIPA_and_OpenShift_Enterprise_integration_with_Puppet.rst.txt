NOTICE
------

This is a **draft** - the project is not yet complete - not even sure if
it will *work*.

.. _problem_definition:

Problem Definition
------------------

Currently, there is no easy way to implement an internal cloud with
OpenShift Enterprise on RHEL/Fedora systems that can seemlessly
integrate with IPA.

.. _hope_to_achieve:

Hope to Achieve
---------------

-  Provide a way for current or potential enterprise RHEL/Fedora + IPA
   users to easily implement an internal cloud using OpenShift
   technology.
-  To make a Puppet module to configure and manage the state of an
   OpenShift cloud implentation that integrates with an IPA instance.

Deliverables
------------

-  Puppet Module
-  Proof of Concept with mini-internal cloud

   -  5 Fedora 18 Machines

      -  IPA Server
      -  Puppet Master
      -  OpenShift Broker/Puppet Agent
      -  OpenShift Node/Puppet Agent
      -  User/Developer workstation

   -  Windows AD Trust with IPA Server
   -  Demo application running on internal cloud

.. _proposed_implementation:

Proposed Implementation
-----------------------

Assumptions
~~~~~~~~~~~

-  IPA Server already configured (with or without Windows AD trust)
-  Puppet Master & Agents are IPA Clients
-  OpenShift Brokers & Nodes are both IPA Clients and Puppet Agents
-  IPA manages Puppet Master as host/service
-  IPA manages Host SSH keys for both Broker + Node as host_keys
-  IPA manages SSH keys for User/Developer as authorized_keys
-  Puppet manages IPA Clients (Brokers/Nodes) through IPA delegated host
   management
-  Puppet Agents are configured to query Master every ${x} minutes for
   catalog

.. _initial_setup_of_ipa_and_puppet:

Initial Setup of IPA and Puppet
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Enroll Puppet Master as a host + service in IPA

   -  ``puppetmaster/puppet.${DOMAIN}``
   -  ``puppet/puppet.${DOMAIN}``

-  Enroll Puppet Agent as a service in IPA
-  Generate service certificate for puppetmaster
-  Generate service certificates for the puppet agents
-  Setup the puppetmaster to use Apache with Passenger and mod_nss (`see
   blogpost <http://jcape.name/2012/01/16/using-the-freeipa-pki-with-puppet/>`__)
-  Generate keytabs for both Puppet Master and Agent

.. _puppet_ipaopenshift_module_in_detail:

Puppet IPA/OpenShift Module in Detail
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Puppet's init script selects IPA setup = True
#. NTP is configured from Agent -> IPA Server
#. If operating system is Fedora, then packages = \`freeipa-*`,
   otherwise, \`ipa-*\`
#. Puppet ensures that both ipa-client and ipa-admintools are present on
   the system
#. Ensure that the appropriate ports are open for IPA
#. If ``/etc/ipa/default.conf`` does not exist, then execute
   ``ipa-client-install --ssh-trust-dns`` command with appropriate
   parameters as sudo/root.

   #. Configures Broker as a host in IPA
   #. Configures host SSH keys for IPA + SSSD

#. If ``/etc/krb5.keytab`` (and ``/etc/httpd/conf/krb5.keytab`` for a
   Broker machine) is not present, then Puppet Master, as a `delegated
   host <https://docs.fedoraproject.org/en-US/Fedora/17/html/FreeIPA_Guide/Extending_the_Permissions_of_IPA_Managed_Hosts.html#Delegating_Service_Management>`__,
   runs:

   ::

      $ kinit -kt /etc/krb5.keytab host/`hostname` 

   ::

      $ ipa-service-add HTTP/${broker/node}.example.com 

   ::

      $ ipa-getkeytab -s `hostname` -k /etc/httpd/conf/krb5.keytab -p HTTP/${broker/node}.example.com 

#. If the Puppet Agent is a Broker:

   #. Now that keytab definitely exists, ensure that apache is the owner
      of ``/etc/httpd/conf/krb5.keytab``.
   #. Ensure that the auth plugin for Apache configuration is present
      using a Puppet template, including ``mod_auth_kerb``.
   #. Ensure ``REMOTE_USER`` is used for the Apache instance on the
      Console (note: two HTTP instances, one 'Broker' one 'Console'.
   #. Have Broker's DHCP + BIND plays nice with IPA's DNS server using a
      Puppet template. An
      `overview <http://sosiouxme.wordpress.com/2012/12/31/openshift-with-dynamic-host-ips/>`__
      of how-to.

.. _open_questions:

Open Questions
--------------

-  Will the delegated host bit work? re: # 7 of the `Module in
   Detail <http://freeipa.org/page/Plan:_FreeIPA_and_OpenShift_Enterprise_integration_with_Puppet#Puppet_IPA.2FOpenShift_Module_in_Detail>`__
-  Is the ``/etc/ipa/default.conf`` file enough to determine if
   ipa-client is not installed? Should I catch the error of 'ipa-client
   is already installed'?

   -  if error of already installed, then \`ipa-client-install
      --uninstall\`
   -  remove all keytabs left behind
   -  run ``ipa-client-install --ssh-trust-dns``

-  Can databases, such as Mongo, MySQL, Postgres, be enrolled as a host
   and/or service w/ IPA?

.. _reference_documentation_research:

Reference Documentation & Research
----------------------------------

-  `Diagram of OpenShift + IPA
   Architecture <https://www.dropbox.com/s/qmsd3dulckn2nnh/IPA_OpenShift.pdf>`__
-  `Diagram of Puppet
   Architecture <https://www.dropbox.com/s/9ckp02q5jsy9cb3/Puppet.pdf>`__
-  `Diagram of Overall Design
   Architecture <https://www.dropbox.com/s/s6at6enzfh8lepn/IPA_OpenShift_Puppet.pdf>`__
-  `Blog Post re: IPA certs and
   Puppet <http://jcape.name/2012/01/16/using-the-freeipa-pki-with-puppet/>`__
-  `Skeleton Code of a Puppet Manifest for
   IPA <https://github.com/thias/puppet-modules/tree/master/modules/ipa>`__
-  `Dynamic host IPs with
   OpenShift <http://sosiouxme.wordpress.com/2012/12/31/openshift-with-dynamic-host-ips/>`__
