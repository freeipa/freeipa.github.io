**This page has been deprecated.**

.. _building_candlepin:

Building Candlepin
==================

Follow `this
instruction <https://fedorahosted.org/candlepin/wiki/DeveloperDocs>`__
to build Candlepin.

If there's a problem installing buildr, try the following commands:

::

   % gem install rdoc
   % gem install rdoc-data
   % rdoc-data --install
   % gem rdoc --all --overwrite

Build works on F14 with the following gem combination:

::

   % gem list

   *** LOCAL GEMS ***

   Antwrap (0.7.0)
   archive-tar-minitar (0.5.2)
   atoulme-Antwrap (0.7.1)
   builder (2.1.2)
   buildr (1.4.3)
   diff-lcs (1.1.2)
   highline (1.5.1)
   hoe (2.3.3)
   json_pure (1.4.3)
   mime-types (1.16)
   minitar (0.5.3)
   net-sftp (2.0.4)
   net-ssh (2.0.23)
   oauth (0.4.4)
   rake (0.8.7)
   rdoc-data (2.5.3)
   rest-client (1.6.1)
   rjb (1.2.5)
   rspec (1.3.1)
   rspec-core (2.5.1)
   rspec-expectations (2.5.0)
   rspec-mocks (2.5.0)
   rubyforge (2.0.3)
   rubyzip (0.9.1)
   term-ansicolor (1.0.5)
   xml-simple (1.0.12)

Deployment
==========

Deploy Candlepin to Tomcat 6 with test data:

::

   % TESTDATA=1 buildconf/scripts/deploy
   % ln -s /etc/candlepin/certs/keystore /usr/share/tomcat6/conf/keystore
   % ln -s /etc/candlepin/certs/candlepin-ca.crt /etc/candlepin/certs/candlepin-ca.pem
   % wget -qO- http://localhost:8080/candlepin/admin/init

.. _registering_ipa:

Registering IPA
===============

::

   % ipa entitle-register admin
   Password: admin
   Enter Password again to verify: admin

   % ipa entitle-consume 25

   % ipa entitle-status

   % ipa-compliance

References
==========

-  `Candlepin <https://fedorahosted.org/candlepin/>`__
-  `Apache Buildr <http://buildr.apache.org/>`__
-  `RubyGems <http://rubyforge.org/projects/rubygems/>`__
-  `RSpec-1 <http://rspec.info/>`__
-  `RSpec-2 <http://relishapp.com/rspec>`__
-  `PostgreSQL
   Tips <https://fedorahosted.org/candlepin/wiki/Deployment#PostgreSQLTips>`__

`Category:Improve <Category:Improve>`__
`Category:NoLink <Category:NoLink>`__
