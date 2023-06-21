CA-less_to_CA-full_conversion
=============================

Overview
--------

Allow converting an existing CA-less IPA deployment to CA-full.



Use Cases
---------



IPA CA install
----------------------------------------------------------------------------------------------

Allow installing the IPA CA on top of an existing CA-less deployment.

Design
------

The ``ipa-ca-install`` command will be extended to allow installing IPA
CA master in existing CA-less environment. Currently it can install IPA
CA replicas from provided replica info file. Make the replica info file
optional - when not provided, new IPA CA master will be installed. To
allow installing externally signed CA, add the ``--external-ca``,
``--external_ca_file`` and ``--external_cert_file`` options from
``ipa-server-install`` to ``ipa-ca-install``.

Installing the CA will not cause existing service certificates to be
replaced with new certificates issued by the CA. If necessary, this will
have to be done manually, using ``ipa cert-request``.

This requires IPA to support having multiple CA certificates (the old
CA-less CA certificate and the new IPA CA certificate), which is
implemented in the `CA certificate
renewal <V4/CA_certificate_renewal>`__ feature.

Implementation
--------------

In order to properly install the CA on an existing deployment, the
following has to be done:

-  configure CA master instance, like in ``ipa-server-install``,
-  add CA related DNS records if DNS server is enabled,
-  set ``enable_ra=True``, ``ra_plugin=dogtag`` and
   ``dogtag_version=10`` in ``/etc/ipa/default.conf``,
-  put the new IPA CA certificate chain into the
   ``/etc/dirsrv/slapd-``\ **``REALM``** NSS database,
-  put the old CA-less CA certificate into the
   ``/var/lib/pki/pki-tomcat/alias`` NSS database,
-  restart DS, httpd and Dogtag.

IPA assumes the following nicknames (and subject names) of CA-related
certificates in the ``/etc/httpd/alias`` and
``/etc/dirsrv/slapd-``\ **``REALM``** NSS databases:

-  **``REALM``**\ ``IPA CA``
   (``CN=Certificate Authority,``\ **``SUBJECT_BASE``**),
-  ``ipaCert`` (``CN=IPA RA,``\ **``SUBJECT_BASE``**),
-  ``Signing-Cert`` (``CN=Object Signing Cert,``\ **``SUBJECT_BASE``**).

If any matching certificate is present prior to installing the CA,
``ipa-ca-install`` will refuse to continue.



Feature Management
------------------

UI

N/A

CLI

N/A

Installers
----------------------------------------------------------------------------------------------

See `the design <#Design>`__.

Upgrade
-------

N/A



How to Test
-----------

TODO



Test Plan
---------

TODO



RFE Author
----------

`Jan Cholasta <User:Jcholast>`__