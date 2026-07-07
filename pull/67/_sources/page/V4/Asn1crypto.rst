Asn1crypto
==========

Overview
--------

Move to `python-asn1crypto <https://github.com/wbond/asn1crypto>`__ as
ASN.1 parsing library. *asn1crypto* is a modern library to parse and
create ASN.1 data. It has been designed to be faster and easier to use
than pyasn1.



Use Cases
---------

`PyCA cryptography <https://github.com/pyca/cryptography/>`__ has
replaced *pyasn1* and *pyasn1_modules* with *asn1crypto* in version
`1.8 <https://cryptography.io/en/latest/changelog/#v1-8>`__. On Fedora
26 and newer, FreeIPA depends on two different ASN.1 libraries. It
directly depends on *pyasn1* to parse X.509 certs in *ipalib.x509*, to
create CSRs in *ipaclient.csrgen*, and in *ipaserver.rpcserver* to parse
OTP sync requests. There is an indirect dependency on *asn1crypto*
because latest Fedora comes with *python-cryptography 2.x*.

In general we prefer to avoid dependencies on two libraries with same
purpose. Moving to *asn1crypto* allows FreeIPA to drop two packages from
its dependency list.



Feature Management
------------------

The change does not introduce new features.

Implementation
--------------

asn1crypto
----------------------------------------------------------------------------------------------

FreeIPA can be ported to asn1crypto now. X.509 parsing is stable.
asn1crypto 0.22 has `known
issues <https://github.com/wbond/asn1crypto/issues/63>`__ to handle
Kerberos' ASN.1 data that contains explicit application tags. It only
affects *python-kdcproxy*. The issue will be solved soonish.

FreeIPA
----------------------------------------------------------------------------------------------

Four modules must be ported:

-  ipaclient/csrgen.py
-  ipalib/x509.py
-  ipaserver/rpcserver.py
-  ipatests/test_integration/create_caless_pki.py

and some additional places that import PyASN1Error. asn1crypto simply
raises ValueError in case of an error.

kdcproxy
----------------------------------------------------------------------------------------------

The *python-kdcproxy* package also depends on pyasn1. The next release
will have support for *asn1crypto*, too. Progress is track
`upstream <https://github.com/latchset/kdcproxy/issues/33>`__.

Dependencies
----------------------------------------------------------------------------------------------

-  Add *python[23]-asn1crypto*
-  Remove *python[23]-pyasn1* and *python[23]-pyasn1_modules*
-  Depend on new version of *python-kdcproxy*
-  Update *setup.py*



How to Test
-----------

Existing unit and integration tests should be sufficient to cover the
migration. In case some aspects of *pyasn* are not covered yet, tests
should be written first.