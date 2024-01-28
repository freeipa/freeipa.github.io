Files_to_be_attached_to_bug_report
==================================



Server instalation failed
-------------------------

Please be aware that some logs may contain sensitive information and
should be sanitized or transported over a secure channel.



ipa-server-install
----------------------------------------------------------------------------------------------



Generic failure
^^^^^^^^^^^^^^^

.. code-block:: text

    date -R
    /var/log/ipaserver-install.log
    ausearch -m AVC > avc.log



Directory server failed
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: text

    date -R
    /var/log/ipaserver-install.log
    ausearch -m AVC > avc.log
    /var/log/dirsrv/slapd-*/errors
    /var/log/dirsrv/slapd-*/access
    journalctl -xe



Dogtag CA failed
^^^^^^^^^^^^^^^^

.. code-block:: text

    date -R
    /var/log/ipaserver-install.log
    ausearch -m AVC > avc.log
    journalctl -u pki-tomcatd@pki-tomcat.service
    /var/log/pki/pki-tomcat/ca/debug
    /var/log/pki/pki-ca-spawn.``\ ``.log



Dogtag KRA failed
^^^^^^^^^^^^^^^^^

.. code-block:: text

    date -R
    /var/log/ipaserver-install.log
    ausearch -m AVC > avc.log
    journalctl -u pki-tomcatd@pki-tomcat.service
    /var/log/pki/pki-tomcat/kra/debug
    /var/log/pki/pki-kra-spawn.``\ ``.log



Kerberos (KDC, kadmin) failed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: text

    date -R
    /var/log/ipaserver-install.log
    ausearch -m AVC > avc.log
    /var/log/kadmind.log
    /var/log/krb5kdc.log



Apache (httpd) failed
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: text

    date -R
    /var/log/ipaserver-install.log
    ausearch -m AVC > avc.log
    journalctl -u httpd
    /var/log/httpd/error_log



Custodia failed
^^^^^^^^^^^^^^^

.. code-block:: text

    date -R
    /var/log/ipaserver-install.log
    ausearch -m AVC > avc.log
    journalctl -u ipa-custodia
    less /var/log/ipa-custodia.audit.log  # from both master and replica



DNS part failed
^^^^^^^^^^^^^^^

.. code-block:: text

    date -R
    /var/log/ipaserver-install.log
    ausearch -m AVC > avc.log
    journalctl -u named-pkcs11
    journalctl -u ipa-dnskeysyncd



AD Trust installation failed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: text

    date -R
    /var/log/ipaserver-install.log
    ausearch -m AVC > avc.log
    /var/log/httpd/error_log
    /var/log/dirsrv/slapd-*/errors
    /var/log/dirsrv/slapd-*/access
    journalctl -u smb
    journalctl -u winbind



Installation of updates failed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: text

    date -R
    /var/log/ipaserver-install.log
    ausearch -m AVC > avc.log
    /var/log/dirsrv/slapd-*/errors



Client part failed
^^^^^^^^^^^^^^^^^^

.. code-block:: text

    date -R
    /var/log/ipaserver-install.log
    ausearch -m AVC > avc.log
    /var/log/ipaclient-install.log
    /var/log/httpd/error_log



ipa-replica-install
----------------------------------------------------------------------------------------------



Generic failure
^^^^^^^^^^^^^^^

.. code-block:: text

    date -R
    /var/log/ipareplica-install.log
    ausearch -m AVC > avc.log

In case of failure of any specific component follow `list of services
from installation
section <https://www.freeipa.org/page/Files_to_be_attached_to_bug_report#ipa-server-install>`__
and provide those logs too.



Connection check failed
^^^^^^^^^^^^^^^^^^^^^^^

Please make sure that firewall and network are correctly set (servers
can see each other) before you report issue against replica connection
check.

From both *master* and *replica*

.. code-block:: text

    date -R
    /var/log/ipareplica-conncheck.log



ipa-dns-install
----------------------------------------------------------------------------------------------

See `ipa-server-install DNS
part <https://www.freeipa.org/page/Files_to_be_attached_to_bug_report#DNS_part_failed>`__



ipa-ca-install
----------------------------------------------------------------------------------------------

.. code-block:: text

    date -R
    /var/log/ipareplica-ca-install.log

And see `ipa-server-install CA
part <https://www.freeipa.org/page/Files_to_be_attached_to_bug_report#Dogtag_CA_failed>`__.



ipa-kra-install
----------------------------------------------------------------------------------------------

.. code-block:: text

    date -R
    /var/log/ipaserver-kra-install.log

And see `ipa-server-install KRA
part <https://www.freeipa.org/page/Files_to_be_attached_to_bug_report#Dogtag_KRA_failed>`__.



ipa-adtrust-install
----------------------------------------------------------------------------------------------

See `ipa-server-install AD Trust
part <https://www.freeipa.org/page/Files_to_be_attached_to_bug_report#AD_Trust_installation_failed>`__.



I HAVE NO IDEA
----------------------------------------------------------------------------------------------

Then provide everything you can ;-)

.. code-block:: text

    date -R
    /var/log/ipa*.log
    ausearch -m AVC > avc.log
    /var/log/dirsrv/slapd-*/errors
    /var/log/dirsrv/slapd-*/access
    journalctl -xe
    journalctl -u named-pkcs11
    journalctl -u ipa-dnskeysyncd
    journalctl -u httpd
    journalctl -u pki-tomcatd@pki-tomcat.service
    /var/log/pki/pki-tomcat/ca/debug
    /var/log/pki/pki-ca-spawn.``\ ``.log
    /var/log/pki/pki-tomcat/kra/debug
    /var/log/pki/pki-kra-spawn.``\ ``.log
    /var/log/httpd/error_log
    /var/log/kadmind.log
    /var/log/krb5kdc.log



Client installation failed
--------------------------

.. code-block:: text

    date -R
    /var/log/ipaclient-install.log
    ausearch -m AVC > avc.log



Upgrade failed
--------------

.. code-block:: text

    date -R
    /var/log/ipaupgrade.log
    ausearch -m AVC > avc.log
    /var/log/dirsrv/slapd-*/errors

In case of upgrade failure of any specific components follow `list of
services from installation
section <https://www.freeipa.org/page/Files_to_be_attached_to_bug_report#ipa-server-install>`__
and provide those logs too.



FreeIPA CLI failed
------------------



Kerberos related errors
----------------------------------------------------------------------------------------------

``KRB5_TRACE=/dev/stderr ipa --debug ping``

From the server:

.. code-block:: text

    date -R
    /var/log/httpd/error_log
    /var/log/krb5kdc.log



Internal server error
----------------------------------------------------------------------------------------------

Please execute steps **on the server** which is returning an internal
error.

Prologue:

.. code-block:: text

    set ``\ *``debug=true``*\ `` in ``\ *``/etc/ipa/default.conf``*

    apachectl graceful

Run broken command:

.. code-block:: text

    ``ipa ``

Provide logs from the server:

.. code-block:: text

    | ``date -R``
    | ``/var/log/httpd/error_log``
    | ``/var/log/dirsrv/slapd-*/access``
    | ``/var/log/dirsrv/slapd-*/errors``

Epilogue:

.. code-block:: text

    remove ``\ *``debug=true``*\ `` from ``\ *``/etc/ipa/default.conf``*

    apachectl graceful



FreeIPA WebUI failed
--------------------



Login failed
----------------------------------------------------------------------------------------------

Please execute steps **on the server** with FreeIPA server installed.

Prologue:

.. code-block:: text

    change ``\ *``LogLevel``*\ `` to ``\ *``info``*\ `` in ``\ *``/etc/httpd/conf.d/nss.conf``*

    apachectl graceful

Try to log in again.

Provide logs from the server:

.. code-block:: text

    date -R
    /var/log/httpd/error_log
    /var/log/httpd/access_log
    /var/log/krb5kdc.log

Epilogue:

.. code-block:: text

    set back ``\ *``LogLevel``*\ `` to ``\ *``warn``*\ `` in ``\ *``/etc/httpd/conf.d/nss.conf``*

    apachectl graceful



Other failures
----------------------------------------------------------------------------------------------

Usually seen as 50x HTTP error in WebUI.

.. code-block:: text

    date -R
    /var/log/httpd/error_log
    /var/log/httpd/access_log
    journalctl -u httpd



Internal server error
----------------------------------------------------------------------------------------------

Please follow `FreeIPA CLI failed: Internal server
error <https://www.freeipa.org/page/Files_to_be_attached_to_bug_report#Internal_server_error>`__
and execute action in WebUI instead of running an *ipa* .