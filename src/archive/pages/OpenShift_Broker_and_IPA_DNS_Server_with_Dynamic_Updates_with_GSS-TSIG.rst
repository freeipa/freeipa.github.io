OpenShift_Broker_and_IPA_DNS_Server_with_Dynamic_Updates_with_GSS-TSIG
======================================================================

**Draft** Using IPA DNS for OpenShift - setup how-to



Install plugins on Broker

-  FYI: The default plugin for OpenShift Origin is its own BIND
   installation. OpenShift Enterprise already uses nsupdate by default.

   -  The config is found in
      ``/etc/openshift/plugins.d/openshift-origin-dns-bind.conf``
   -  The code is for the plugin in ``/usr/lib/ruby/gems/1.8/gems`` for
      RHEL, and ``/usr/share/gems/gems/`` in Fedora.

-  Must *remove* the old plugin first:
   ``gem uninstall openshift-origin-dns-bind``
-  Install the nsupdate plugin
   ``gem install openshift-origin-dns-nsupdate``
-  Move into the plugins directory, ``# cd /etc/openshift/plugins.d/``
   then move the old configuration:
   ``# mv openshift-origin-dns-bind.conf openshift-origin-dns-bind.conf.orig``
-  Configure the new conf file in
   ``/etc/openshift/plugins.d/openshift-origin-dns-nsupdate.conf`` to
   look like this (make the file if it doesn't exist):

::

   BIND_SERVER="IPA_SERVER_IP_ADDRESS"
   BIND_PORT=53
   BIND_ZONE="BROKER_DOMAIN"
   BIND_KRB_PRINCIPAL="DNS/NODE_HOSTNAME@IPA_REALM"
   BIND_KRB_KEYTAB="/etc/dns.keytab"

-  Mine looks like:

::

   BIND_SERVER="10.16.78.111"
   BIND_PORT=53
   BIND_ZONE="idm.lab.bos.redhat.com"
   BIND_KRB_PRINCIPAL="DNS/vm-040.idm.lab.bos.redhat.com@IDM.LAB.BOS.REDHAT.COM"
   BIND_KRB_KEYTAB="/etc/dns.keytab"



Get IPA Keytab for DNS

We need to give the Broker *another* keytab to update the IPA Server's
DNS. We will 1) enroll it as a service for DNS, 2) update the policy so
it is allowed to update CNAME records, and 3) get the keytab over to the
Broker machine and with correct ownership.

**If ipa-admintools** is installed on the Broker, then you can do the
following command on the Broker. Else, do it on the IPA Server machine:

::

   kinit admin
   ipa service-add DNS/$BROKER_FQDN
   ipa dnszone-mod $DNS_ZONE --dynamic-update=true\
       --update-policy="grant DNS\047$BROKER_FQDN@REALM wildcard * ANY;"

**IMPORTANT** the "\047" is important and must precede the $BROKER_FQDN
for correct ascii escaping.

**WARNING** the ``ipa dnszone-mod`` command rewrites the BIND policy; it
does not *append*. The easiest way to not overwrite the default setup
and just append the policy update is to navigate to the Web UI -> DNS ->
Settings and add ``grant DNS\047$BROKER_FQDN@REALM wildcard * ANY;`` to
the BIND policy.

**On the Broker host**:

::

   ipa-getkeytab -s $IPA_SERVER_HOSTNAME -p DNS/$BROKER_FQDN -k /etc/dns.keytab
   kinit -kt /etc/dns.keytab -p DNS/$BROKER_FQDN

-  Change the ownership: ``chown apache:apache /etc/dns.keytab``
-  Restart the broker service: ``service openshift-broker restart``



To test

I suggest testing three ways:

#. Manually with nsupdate tool. If this fails, you know it's a
   Kerberos/IPA/permissions issue, **NOT** an OpenShift or RHC issue.
#. Manually with rails console. If this fails but test #1 succeeds, then
   it is an OpenShift configuration issue with nsupdate.
#. With RHC command line tools. If this fails, then it is an OpenShift
   issue with either MongoDB or ActiveMQ/MCollective.



Manually

First we should manually see if nsupdate works on our **Broker machine**
to see if it can correctly update its zone:

::

   [root@broker]# kinit -kt /etc/dns.keytab DNS/$BROKER_FQDN
   [root@broker]# nsupdate -g
   &gt; server $IPA_SERVER_IP_ADDR
   &gt; zone $BROKER_ZONE
   &gt; update delete $YOUR_TEST_APP_FQDN
   &gt; update add $YOUR_TEST_APP_FQDN 180 CNAME $BROKER_FQDN
   &gt; send
   [root@broker]#

You should have no response after ``send`` - that means it went through.
Then try the following on the Broker, the IPA Server, and a third
machine in the realm (admin, client, anything)

::

   [root@broker]# dig @IPA_SERVER_IP_ADDR $YOUR_TEST_APP_FQDN
   [root@ipaserver]# dig @IPA_SERVER_IP_ADDR $YOUR_TEST_APP_FQDN
   [root@admin]# dig @IPA_SERVER_IP_ADDR $YOUR_TEST_APP_FQDN

Then on the IPA Server or Broker if the Broker has ``ipa-admintools``:

::

   [root@ipaserver]# ipa dns-resolve $YOUR_TEST_APP_FQDN



With OpenShift Enterprise

On the OSE Broker, move into the broker directory, remove the
Gemfile.lock (NOT Gemfile!), and locally install the :

::

   [root@broker]# cd /var/www/openshift/broker
   [root@broker]# rm -rf Gemfile.lock
   [root@broker]# bundle --local

Then we'll start the rails console that has access to all the Broker
gems, create a DNS instance, and create a CNAME record through the rails
console:

::

   [root@broker]# rails console
   irb(main):001:0&gt; d = OpenShift::DnsService.instance
   irb(main):002:0&gt; d.register_application "testapp1", "testns1", "node1.example.com
   =&gt; nil

If you are successful, the rails console will return with "nil".



Possible issue

The Broker will grab the TSIG Key/value, not the GSS-TSIG
keytab/principal. Within
``/usr/lib/rub/gems/1.8/gems/openshift-origin-dns-nsupdate-xxxx/config/initializers``,
edit the file: ``openshift-origin-dns-nsupdate.rb`` to look like
`this <https://github.com/openshift/origin-server/pull/2269/files#diff-1>`__
- this addresses an error that sets keyname/value incorrectly if using
IPA's keytab.

Once you edit this, restart the broker service:
``service openshift-broker restart``



With RHC command-line tools

Note that if you didn't update the manual test to delete the CNAME
record for the FQDN, you will need to choose another test FQDN.

On your client machine:

::

   [root@client]#  rhc app create APP_NAME APP_TYPE

It should not hang anywhere (well, creating name space and such can take
a minute or two, but not completely hang for longer than that).

If there are any issues on this side, check to see if ActiveMQ is
running – note that if you have to restart ActiveMQ, give it at least 5
minutes before testing again.

Check to see if MCollective is working with ``mco ping``.

Check broker development/production logs for any MongoDB issues.