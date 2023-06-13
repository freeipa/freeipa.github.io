Introduction
------------

The certmonger daemon monitors certificates for impending expiration,
and can optionally refresh soon-to-be-expired certificates with the help
of a CA. If told to, it can drive the entire enrollment process from key
generation through enrollment and refresh.

It can work with either flat files, like those used by OpenSSL, or with
NSS databases.

Commands
--------

The certmonger command-line tool, ``getcert`` is a very generic tool
that can manage the certificates you are tracking. A superset of this
tool, ``ipa-getcert`` works specifically with an IPA CA. ipa-getcert is
equivalent to ``getcert -c IPA``

.. _common_usage:

Common Usage
------------

For the NSS cases this assumes there is no password associated with the
NSS database. If there is then the key must be in a file readable by
certmonger and passed in using the -p or -P options.

.. _get_a_list_of_currently_tracked_certificates:

Get a list of currently tracked certificates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The difference between "all" and "IPA-issued" is subtle. An IPA-issued
certificate means it was issued using the API provided by IPA. It does
not refer to any certificate issued by the IPA CA. There are some
certificates, such as some subsystem certificates of the CA itself, than
can be tracked by certmonger but are not issued by the API.

.. _certmonger_cas:

Certmonger "CAs"
~~~~~~~~~~~~~~~~

Certmonger uses helpers to communicate with CAs. These are listed and
defined using the command ``getcert-list-cas`` and ``getcert-add-ca``
commands. Each defined CA in certmonger has a name. It's perfectly legal
to have multiple certmonger CA entries pointing to the same helper but
defined differently.

For the case of an IPA installation there are two relevant certmonger
CAs:

-  IPA
-  dogtag-ipa-ca-renew-agent

IPA uses a CA helper that uses the host keytab entry to authenticate to
the IPA API in order to obtain certificates.

The dogtag-ipa-ca-renew-agent is used to directly authenticate to the CA
(dogtag) backend. This is used to manage the certificates that the CA
itself requires.

And this is why ``getcert list`` returns more certificates that
``ipa-getcert list``

.. _anatomy_of_a_tracked_certificate:

Anatomy of a tracked certificate
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A typical OpenSSL certificate looks like this:

| ``Request ID '20120912211542':``
| ``       status: MONITORING``
| ``       stuck: no``
| ``       key pair storage: type=FILE,location='/etc/ssl/private.key'``
| ``       certificate: type=FILE,location='/etc/ssl/server.crt'``
| ``       CA: IPA``
| ``       issuer: CN=Certificate Authority,O=EXAMPLE.COM``
| ``       subject: CN=edsel.example.com,O=EXAMPLE.COM``
| ``       expires: 2014-09-13 21:15:44 UTC``
| ``       eku: id-kp-serverAuth,id-kp-clientAuth``
| ``       pre-save command: ``
| ``       post-save command: ``
| ``       track: yes``
| ``       auto-renew: yes``

You'll need the Request ID when running other certmonger commands. This
will be a number if an ID is not provided when the request is originally
made.

The status MONITORING means that the certificate is valid and being
tracked by certmonger.

The key and certificate values indicate where the OpenSSL private key
and certificate are stored in the filesystem.

The CA type is IPA. This was passed in as the -c option of getcert, or
automatically set to IPA by ipa-getcert.

The issuer is the subject of the CA that issued the certificate.

The subject is the subject of the certificate in the certificate file.

The expiration date is UTC. By default certmonger will start trying to
renew the certificate 28 days before it expires by default.

eku lists the Enhanced Key Usage (EKU) extensions of the certificate. By
default IPA certificates are usable both as server and client
certificates.

The pre and post-save commands define commands that are executed before
and after the renewal process. This can be used, for example, to restart
a service when a certificate is renewed.

.. _all_tracked_certificates:

All tracked certificates
^^^^^^^^^^^^^^^^^^^^^^^^

``# getcert list``

.. _all_ipa_issued_certificate:

All IPA-issued certificate
^^^^^^^^^^^^^^^^^^^^^^^^^^

``# ipa-getcert list``

This is the equivalent of:

`` # getcert list -c IPA``

The difference is that ipa-getcert sets the value of the CA (-c). It's
just a shortcut.

.. _request_a_new_certificate:

Request a new certificate
~~~~~~~~~~~~~~~~~~~~~~~~~

This will generate a new key pair, create a CSR and request a
certificate from the IPA server configured in ``/etc/ipa/default.conf``,
authenticating using the machine's host service principal in
``/etc/krb5.keytab``.

A certificate needs to be associated with an object in IPA so the -K
option is required in order to set that. Other useful options are:

-A to add an IPaddress Subject Alternative name (SAN) to the request -D
to add DNS Subject Alternative Name (SAN) to the request -g to set the
RSA key size -I to set the ID of the certmonger request

By default all certmonger requests are set to auto-renew. The -R option
will disable this.

OpenSSL
^^^^^^^

``#  ipa-getcert request -f /path/to/server.crt -k /path/to/private.key -K ``

NSS
^^^

``# ipa-getcert request -d /path/to/database -n 'Test' -K ``

.. _manually_renew_a_certificate:

Manually renew a certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you want to manually renew a certificate prior to its expiration
date, run:

``# ipa-getcert resubmit -i REQUEST_ID``

.. _stop_tracking_a_certificate:

Stop tracking a certificate
~~~~~~~~~~~~~~~~~~~~~~~~~~~

To tell certmonger to forget about a certificate and stop tracking it
run:

``# ipa-getcert stop-tracking -i REQUEST_ID``

This does **not** touch the certificate or keys, it merely tells
certmonger to not track it for for rewnewals.

.. _issue_a_certificate_with_specific_properties:

Issue a certificate with specific properties
--------------------------------------------

To issue a certificate with specific CN or other properties, specify
additional options to the utility. ``getcert`` has a lot of flexibility
with options described in its manual page. For example, to issue a
certificate for Nginx to use a specific fully qualified hostname on a
host without it, use following sequence:

| ``# cd /etc/nginx/ssl``
| ``# fqdn=$(hostname -f); REALM=(hostname -d|tr '[:lower:]' '[:upper:]'); ``
| ``# ipa-getcert request -f $fqdn.crt -k $fqdn.key -r -K HTTP/$fqdn@$REALM -N $fqdn``

The CA has the final say on what the subject will be in the certificate
it issues.

.. _external_documentation:

External Documentation
----------------------

-  `Certmonger user guide in RHEL
   documentation <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/System-Level_Authentication_Guide/certmongerX.html>`__

.. _how_certmonger_finds_an_ipa_ca:

How Certmonger finds an IPA CA
------------------------------

Certmonger will first look in ``/etc/ipa/default.conf`` for the value of
``xmlrpc_uri`` and use that to make the certificate request for IPA.

Any IPA Master, even those that do not have a CA locally installed, can
handle a certificate request by proxying the request to a master that
does have a CA.

If the request fails due to a connect error Certmonger will next look
for a value of ``server`` in ``/etc/ipa/default.conf``. If one is found
then a similar request is made to the value of ``server`` plus
``/ipa/xm``.

If there is no ``server`` defined, and there likely will not be given
this directive is deprecated, then an LDAP search will be done using the
default LDAP search values in ``/etc/openldap/ldap.conf`` for the list
of IPA servers that have a CA. One of these servers, if any, will be
picked by certmonger and the request will be made again. If this request
fails then certmonger will give up.
