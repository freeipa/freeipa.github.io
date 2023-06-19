Following/updating this (a bit outdated) `blog
post <http://jcape.name/2012/01/16/using-the-freeipa-pki-with-puppet/>`__
along with Puppet Labs'
`instructions <http://docs.puppetlabs.com/puppet/3/reference/config_ssl_external_ca.html#option-1-single-ca>`__
on setting up Puppet with an external Certificate Authority.

.. _initial_setup:

Initial Setup
-------------

#. Need three Fedora 19 machines: IPA Server
   (``ipaserver.example.com``), Puppet Master
   (``puppetmaster.example.com``), Puppet Agent
   (``puppet.example.com``).

#. Set up the IPA Server on one machine
   (``yum install freeipa-server; ipa-server-install``). Be sure that
   the firewall is setup appropriately.

#. Enrolled the other two machines as IPA
   clients(``yum install freeipa-client; ipa-client-install``). Be sure
   that the firewall is setup appropriately.

#. On the IPA Server, or if one of the clients has
   ``freeipa-admintools`` installed:

   .. code:: bash

      # create the puppet master service
      $ ipa service-add puppetmaster/puppetmaster.example.com
      # create the puppet agent service
      $ ipa service-add puppet/puppet.example.com

.. _puppet_master_setup:

Puppet Master setup
-------------------

On the machine for the Puppet Master:

#. Installation:

   .. code:: bash

      # install latest puppet-server
      # (yum install puppet-server is a couple minor versions behind)
      # version 3.2 fixes a CA bug that isn't in the yum repo
      $ rpm -ivh http://yum.puppetlabs.com/fedora/f19/products/i386/puppetlabs-release-19-2.noarch.rpm
      $ yum install -y http://yum.puppetlabs.com/fedora/f19/products/x86_64/puppet-server-3.2.4-1.fc19.noarch.rpm
      # stop the puppetmaster service since we'll be using apache
      $ service puppetmaster stop
      # install additional requirements
      $ yum install -y mod_nss mod_passenger

#. Setting up the certificates:

   .. code:: bash

      # now grab the certs for the master
      $ ipa-getcert request -K puppetmaster/puppetmaster.example.com 
                             -d /etc/httpd/alias 
                             -n puppetmaster/puppetmaster.example.com
      # identify where to put the public, private, and CA pem files for host:
      $ puppet master --configprint hostcert
      /var/lib/puppet/ssl/certs/puppetmaster.example.com.pem
      $ puppet master --configprint hostprivkey
      /var/lib/puppet/ssl/private_keys/puppetmaster.example.com.pem
      $ puppet master --configprint localcacert
      /var/lib/puppet/ssl/certs/ca.pem            
      # you may need to create the above directories      
      # grab the public key for host and place it in the appropriate directory
      $ certutil -L -d /etc/pki/nssdb 
                 -a -n "IPA Machine Certificate - puppetmaster.example.com" > 
                 /var/lib/puppet/ssl/certs/puppetmaster.example.com.pem
      # if there's an error about the directory, set SELinux to permissive for the
      # certutil commands, then you can return it to enforce.
      # grab the private key for host and place it in the appropriate directory
      # if/when prompted for a password, this is the same admin password that you used to setup IPA
      $ certutil -K -d /etc/pki/nssdb -a
      $ pk12util -o keys.p12 
                 -n "IPA Machine Certificate - puppetmaster.example.com"
                 -d /etc/pki/nssdb
      $ openssl pkcs12 -in keys.p12 
                         -out /var/lib/puppet/ssl/private_keys/puppetmaster.example.com.pem 
                         -nodes
      # export IPA's CA in the localcacert directory
      $ certutil -L -d /etc/pki/nssdb 
                 -a -n "IPA CA" > /var/lib/puppet/ssl/certs/ca.pem

#. Setup rack/passenger

   .. code:: bash

      $ mkdir -p /var/www/puppet/public
      $ cp /usr/share/puppet/ext/rack/files/config.ru /var/www/puppet

#. Setup the master configuration in ``/etc/puppet/puppet.conf`` by
   adding:

   ::

      [master]
          ca = false
          certificate_revocation = false
          certname = 'puppetmaster.example.com'

#. Setup NSS in ``/etc/httpd/conf.d/nss.conf``:

   ::

      LoadModule          nss_module modules/libmodnss.so
      AddType             application/x-x509-ca-cert .crt
      AddType             application/x-pkcs7-crl    .crl
      NSSPassPhraseDialog     builtin
      NSSPassPhraseHelper     /usr/sbin/nss_pcache
      NSSSessionCacheSize     10000
      NSSSessionCacheTimeout      100
      NSSSession3CacheTimeout     86400
      NSSRandomSeed           startup builtin
      NSSRenegotiation        off
      NSSRequireSafeNegotiation   off

      Listen 8140
      &lt;VirtualHost _default_:8140&gt;
          ServerName  puppetmaster.example.com
          ServerAdmin puppetmaster@example.com

          NSSEngine           on
          NSSCertificateDatabase  /etc/httpd/alias
          NSSNickname         "puppetmaster/puppetmaster.example.com"
          NSSOptions          +StdEnvVars
          NSSEnforceValidCerts        on
          NSSVerifyClient         require
          NSSProtocol         SSLv3,TLSv1
          NSSCipherSuite          +rsa_rc4_128_md5,+rsa_rc4_128_sha,+rsa_3des_sha,-rsa_des_sha,-rsa_rc4_40_md5,-rsa_rc2_40_md5,-rsa_null_md5,-rsa_null_sha,+fips_3des_sha,-fips_des_sha,-fortezza,-fortezza_rc4_128_sha,-fortezza_null,-rsa_des_56_sha,-rsa_rc4_56_sha,+rsa_aes_128_sha,+rsa_aes_256_sha

          RequestHeader set X-SSL-Subject %{SSL_CLIENT_S_DN}e
          RequestHeader set X-Client-DN &quot;/CN=%{SSL_CLIENT_S_DN_CN}e&quot;
          RequestHeader set X-Client-Verify %{SSL_CLIENT_VERIFY}e

          PassengerHighPerformance    on
          PassengerStatThrottleRate   120
          PassengerUseGlobalQueue     on

          RackAutoDetect  off
          RailsAutoDetect off
          RackBaseURI /

          DocumentRoot    /var/www/puppet/public
          &lt;Directory /var/www/puppet&gt;
              Options     None
              AllowOverride   None
              Order       allow,deny
              Allow       from all
          &lt;/Directory&gt;
      &lt;/VirtualHost&gt;

#. Open up ports for puppet and restart Apache:

   .. code:: bash

      $ firewall-cmd --add-port=8140/tcp
      $ service httpd restart

.. _puppet_agent_setup:

Puppet Agent setup
------------------

On the Puppet Agent:

#. Installation:

   .. code:: bash

      # install latest puppet (agent)
      # (yum install puppet-server is a couple minor versions behind)
      # version 3.2 fixes a CA bug that isn't in the yum repo
      $ rpm -ivh http://yum.puppetlabs.com/fedora/f19/products/i386/puppetlabs-release-19-2.noarch.rpm
      $ yum install -y http://yum.puppetlabs.com/fedora/f19/products/x86_64/puppet-3.2.4-1.fc19.noarch.rpm

#. Setup certificates for the agent

   .. code:: bash

      $ ipa-getcert request -K puppet/puppet.example.com
                             -D puppet.example.com
                             -k /var/lib/puppet/ssl/private_keys/puppet.example.com.pem
                             -f /var/lib/puppet/ssl/certs/puppet.example.com.pem

#. Setup the agent configuration in ``/etc/puppet/puppet.conf``, by
   editing/adding the ``[agent]`` & ``[main]`` block:

   ::

      [main]
         # &lt;--snip--&gt;
         server = 'puppetmaster.example.com'
         certname = 'puppetmaster.example.com'
         # &lt;--snip--&gt;
      [agent]
         # &lt;--snip--&gt;
         certificate_revocation = false
         certname = 'puppet.example.com'
         # &lt;--snip--&gt;

#. Test the entire setup in puppet agent:

   .. code:: bash

      # open up port for Puppet
      $ firewall-cmd --add-port=8140/tcp
      # test to see if the setup works
      $ puppet agent --test
      # you'll probably get a catalog error if you have no catalogs
      # setup with your puppet master
