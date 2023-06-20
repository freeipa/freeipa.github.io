Promote_CA_to_Renewal_and_CRL_Master
====================================

Introduction
------------

A CA replica is called a clone in dogtag parlance, and it is a very apt
description. Several of the CA certificates are duplicated on each
clone. This makes the process of renewing those certificates complicated
because they need to remain the same across the infrastructure.

Another area where clones acts differently is a generation of CRL. The
list needs to be generated from a single server to avoid inconsistent
revocation lists across replicas, for example when a replication is
temporarily broken between such replicas.

And finally, a single server is responsible to execute a task to update
the state of the certificates issued (VALID, EXPIRED, etc). This is run
on a single server to avoid replication issues due to updating the same
values on multiple servers at once. For more details see
https://github.com/dogtagpki/pki/wiki/Configuring-Certificate-Status-Update-Task

To achieve this, FreeIPA marks the first installed master with a CA, as
the "first master." It is configured to renew the certificates and make
them available to the other clones and to listen to and generate the
CRL.

Two important things to note:

#. There should only one master at a time, otherwise the renewed
   certificates will step all over each other.
#. Any CA can be the master. There is nothing magical about it, this is
   just configuration.



Procedure in FreeIPA 4.0 or later
---------------------------------



Identifying current first master
----------------------------------------------------------------------------------------------

The hostname of the renewal master can be determined from LDAP:

::

   $ ldapsearch -H ldap://$HOSTNAME -D 'cn=Directory Manager' -W -b 'cn=masters,cn=ipa,cn=etc,dc=example,dc=com' '(&(cn=CA)(ipaConfigString=caRenewalMaster))' dn
   Enter LDAP Password: 
   # extended LDIF
   #
   # LDAPv3
   # base <cn=masters,cn=ipa,cn=etc,dc=example,dc=com> with scope subtree
   # filter: (&(cn=CA)(ipaConfigString=caRenewalMaster))
   # requesting: dn 
   #

   # CA, ipa1.example.com, masters, ipa, etc, example.com
   dn: cn=CA,cn=ipa1.example.com,cn=masters,cn=ipa,cn=etc,dc=example,dc=com

   # search result
   search: 2
   result: 0 Success

   # numResponses: 2
   # numEntries: 1

Here it is ``ipa1.example.com``.

The CRL generation master can be determined by looking at CS.cfg on each
CA:

::

   # grep ca.crl.MasterCRL.enableCRLUpdates /etc/pki/pki-tomcat/ca/CS.cfg
   ca.crl.MasterCRL.enableCRLUpdates=true

If the value is ``true``, then it is the CRL generation master,
otherwise it is a clone.



Reconfiguring a CA as a clone
----------------------------------------------------------------------------------------------



Configure clone renewal
^^^^^^^^^^^^^^^^^^^^^^^

This is done automatically when you configure some other CA as renewal
master.



Stop CRL generation
^^^^^^^^^^^^^^^^^^^

Stop CA service:

::

   # systemctl stop pki-tomcatd@pki-tomcat

Set the value of ``ca.crl.MasterCRL.enableCRLCache`` and
``ca.crl.MasterCRL.enableCRLUpdates`` in
``/etc/pki/pki-tomcat/ca/CS.cfg`` to ``false``:

::

   ca.crl.MasterCRL.enableCRLCache=false
   ca.crl.MasterCRL.enableCRLUpdates=false



Stop Certificate Status Update Task
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Remove the values of ``ca.transitRecordPageSize`` and
``ca.transitMaxRecords`` in ``/etc/pki/pki-tomcat/ca/CS.cfg``.

Set the value of ``ca.certStatusUpdateInterval`` in
``/etc/pki/pki-tomcat/ca/CS.cfg`` to:

::

   ca.certStatusUpdateInterval=0

Start CA service:

::

   # systemctl start pki-tomcatd@pki-tomcat

Configure Apache to redirect CRL requests to the new master in
``/etc/httpd/conf.d/ipa-pki-proxy.conf`` by uncommenting the RewriteRule
on the last line:

::

   # Only enable this on servers that are not generating a CRL
   RewriteRule ^/ipa/crl/MasterCRL.bin https://<hostname>/ca/ee/ca/getCRL?op=getCRL&crlIssuingPoint=MasterCRL [L,R=301,NC]

Restart Apache:

::

   # systemctl restart httpd



Reconfigure a CA as the new master
----------------------------------------------------------------------------------------------



Configure CA renewal
^^^^^^^^^^^^^^^^^^^^

Run the following command:

::

   # ipa-csreplica-manage set-renewal-master



Start CRL generation
^^^^^^^^^^^^^^^^^^^^

Stop CA service:

::

   # systemctl stop pki-tomcatd@pki-tomcat

Set the value of ``ca.crl.MasterCRL.enableCRLCache`` and
``ca.crl.MasterCRL.enableCRLUpdates`` in
``/etc/pki/pki-tomcat/ca/CS.cfg`` to ``true``:

::

   ca.crl.MasterCRL.enableCRLCache=true
   ca.crl.MasterCRL.enableCRLUpdates=true



Start Certificate Status Update Task
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Set the values ``ca.transitRecordPageSize`` and ``ca.transitMaxRecords``
in ``/etc/pki/pki-tomcat/ca/CS.cfg`` ca.transitRecordPageSize=200
ca.transitMaxRecords=1000000

Either remove ``ca.certStatusUpdateInterval=0`` or set the value to 600
(the default).

Start CA service:

::

   # systemctl start pki-tomcatd@pki-tomcat

Configure Apache to handle CRL requests in
``/etc/httpd/conf.d/ipa-pki-proxy.conf`` by commenting out the
RewriteRule on the last line:

::

   # Only enable this on servers that are not generating a CRL
   #RewriteRule ^/ipa/crl/MasterCRL.bin https://<hostname>/ca/ee/ca/getCRL?op=getCRL&crlIssuingPoint=MasterCRL [L,R=301,NC]

Restart Apache:

::

   # systemctl restart httpd

https://github.com/dogtagpki/pki/wiki/Configuring-Certificate-Status-Update-Task



Procedure in FreeIPA < 4.0
--------------------------



Identifying current first master
----------------------------------------------------------------------------------------------

This can be determined by looking at the certificates managed by
certmonger on each CA

::

   # getcert list -d /var/lib/pki-ca/alias -n "subsystemCert cert-pki-ca" | grep post-save
           post-save command: /usr/lib64/ipa/certmonger/renew_ca_cert "subsystemCert cert-pki-ca"

If it contains ``renew_ca_cert`` then it is the CA renewal master.

If it contains ``restart_pkicad`` then it is a CA renewal clone.



Reconfiguring a CA as a clone
----------------------------------------------------------------------------------------------

This step changes current *first master* into a standard clone.



Unconfigure the master renewal
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

   # getcert stop-tracking -d /var/lib/pki-ca/alias -n "auditSigningCert cert-pki-ca"
   # getcert stop-tracking -d /var/lib/pki-ca/alias -n "ocspSigningCert cert-pki-ca"
   # getcert stop-tracking -d /var/lib/pki-ca/alias -n "subsystemCert cert-pki-ca"
   # getcert stop-tracking -d /etc/httpd/alias -n ipaCert

You should see output like:

::

   Request "20131127184547" removed.
   Request "20131127184548" removed.
   Request "20131127184549" removed.
   Request "20131127184550" removed.



Configure clone renewal
^^^^^^^^^^^^^^^^^^^^^^^

First see if the renewal CA is available:

``# getcert list-cas``

Look for a /var/lib/certmonger/cas/ca_renewal

If it does not exist:

::

   # cp /usr/share/ipa/ca_renewal /var/lib/certmonger/cas/ca_renewal
   # chmod 0600 /var/lib/certmonger/cas/ca_renewal
   # /sbin/restorecon  /var/lib/certmonger/cas/ca_renewal
   # service certmonger restart
   # getcert list-cas

Verify that the new CA is available in the ``list-cas`` output:

::

   CA 'dogtag-ipa-retrieve-agent-submit':
           is-default: no
           ca-type: EXTERNAL
           helper-location: /usr/libexec/certmonger/dogtag-ipa-retrieve-agent-submit

Get the CA certificate database pin:

``# grep internal= /var/lib/pki-ca/conf/password.conf``

Configure renewal

::

   # getcert start-tracking -c dogtag-ipa-retrieve-agent-submit -d /var/lib/pki-ca/alias -n "auditSigningCert cert-pki-ca" -B /usr/lib64/ipa/certmonger/stop_pkicad -C '/usr/lib64/ipa/certmonger/restart_pkicad "auditSigningCert cert-pki-ca"' -T "auditSigningCert cert-pki-ca" -P <pin>
   # getcert start-tracking -c dogtag-ipa-retrieve-agent-submit -d /var/lib/pki-ca/alias -n "ocspSigningCert cert-pki-ca" -B /usr/lib64/ipa/certmonger/stop_pkicad -C '/usr/lib64/ipa/certmonger/restart_pkicad "ocspSigningCert cert-pki-ca"' -T "ocspSigningCert cert-pki-ca" -P <pin>
   # getcert start-tracking -c dogtag-ipa-retrieve-agent-submit -d /var/lib/pki-ca/alias -n "subsystemCert cert-pki-ca" -B /usr/lib64/ipa/certmonger/stop_pkicad -C '/usr/lib64/ipa/certmonger/restart_pkicad "subsystemCert cert-pki-ca"' -T "subsystemCert cert-pki-ca" -P <pin>
   # getcert start-tracking -c dogtag-ipa-retrieve-agent-submit -d /etc/httpd/alias -n ipaCert -C /usr/lib64/ipa/certmonger/restart_httpd -T ipaCert -p /etc/httpd/alias/pwdfile.txt

You should see output like:

::

   New tracking request "20131127184743" added.
   New tracking request "20131127184744" added.
   New tracking request "20131127184745" added.
   New tracking request "20131127184746" added.



Stop CRL generation
^^^^^^^^^^^^^^^^^^^

Stop CA service:

::

   # service pki-cad stop

Set the value of ``ca.crl.MasterCRL.enableCRLCache`` and
``ca.crl.MasterCRL.enableCRLUpdates`` in ``/etc/pki-ca/CS.cfg`` to
``false``:

::

   ca.crl.MasterCRL.enableCRLCache=false
   ca.crl.MasterCRL.enableCRLUpdates=false

Start CA service:

::

   # service pki-cad start

Configure Apache to redirect CRL requests to the new master in
``/etc/httpd/conf.d/ipa-pki-proxy.conf`` by uncommenting the RewriteRule
on the last line:

::

   # Only enable this on servers that are not generating a CRL
   RewriteRule ^/ipa/crl/MasterCRL.bin https://<hostname>/ca/ee/ca/getCRL?op=getCRL&crlIssuingPoint=MasterCRL [L,R=301,NC]

Restart Apache:

::

   # service httpd restart



Reconfigure a CA as the new master
----------------------------------------------------------------------------------------------



Unconfigure the clone renewal
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

   # getcert stop-tracking -d /var/lib/pki-ca/alias -n "auditSigningCert cert-pki-ca"
   # getcert stop-tracking -d /var/lib/pki-ca/alias -n "ocspSigningCert cert-pki-ca"
   # getcert stop-tracking -d /var/lib/pki-ca/alias -n "subsystemCert cert-pki-ca"
   # getcert stop-tracking -d /etc/httpd/alias -n ipaCert

You should see output like:

::

   Request "20131127163822" removed.
   Request "20131127163823" removed.
   Request "20131127163824" removed.
   Request "20131127164042" removed.



Configure CA renewal
^^^^^^^^^^^^^^^^^^^^

Get the CA certificate database pin:

``# grep internal= /var/lib/pki-ca/conf/password.conf``

Configure renewal

::

   # getcert start-tracking -c dogtag-ipa-renew-agent -d /var/lib/pki-ca/alias -n "auditSigningCert cert-pki-ca" -B /usr/lib64/ipa/certmonger/stop_pkicad -C '/usr/lib64/ipa/certmonger/renew_ca_cert "auditSigningCert cert-pki-ca"' -P <pin>
   # getcert start-tracking -c dogtag-ipa-renew-agent -d /var/lib/pki-ca/alias -n "ocspSigningCert cert-pki-ca" -B /usr/lib64/ipa/certmonger/stop_pkicad -C '/usr/lib64/ipa/certmonger/renew_ca_cert "ocspSigningCert cert-pki-ca"' -P <pin>
   # getcert start-tracking -c dogtag-ipa-renew-agent -d /var/lib/pki-ca/alias -n "subsystemCert cert-pki-ca" -B /usr/lib64/ipa/certmonger/stop_pkicad -C '/usr/lib64/ipa/certmonger/renew_ca_cert "subsystemCert cert-pki-ca"' -P <pin>
   # getcert start-tracking -c dogtag-ipa-renew-agent -d /etc/httpd/alias -n ipaCert -C /usr/lib64/ipa/certmonger/renew_ra_cert -p /etc/httpd/alias/pwdfile.txt

You should see output like:

::

   New tracking request "20131127185430" added.
   New tracking request "20131127185431" added.
   New tracking request "20131127185432" added.
   New tracking request "20131127185433" added.



Start CRL generation
^^^^^^^^^^^^^^^^^^^^

Stop CA service:

::

   # service pki-cad stop

Set the value of ``ca.crl.MasterCRL.enableCRLCache`` and
``ca.crl.MasterCRL.enableCRLUpdates`` in ``/etc/pki-ca/CS.cfg`` to
``true``:

::

   ca.crl.MasterCRL.enableCRLCache=true
   ca.crl.MasterCRL.enableCRLUpdates=true

Start CA service:

::

   # service pki-cad start

Configure Apache to handle CRL requests in
``/etc/httpd/conf.d/ipa-pki-proxy.conf`` by commenting out the
RewriteRule on the last line:

::

   # Only enable this on servers that are not generating a CRL
   #RewriteRule ^/ipa/crl/MasterCRL.bin https://<hostname>/ca/ee/ca/getCRL?op=getCRL&crlIssuingPoint=MasterCRL [L,R=301,NC]

Restart Apache:

::

   # service httpd restart