The FreeIPA team sometimes gets asked about wildcard certificate
support. A wildcard certificate is an X.509 certificate where the DNS-ID
has a wildcard in it (typically as the most specific domain component,
e.g. ``*.cloudapps.example.com``). Most TLS libraries match wildcard
domains in the obvious way.

This article discusses the state of wildcard certificates in FreeIPA and
how to issue them, but before proceeding it is fitting to point out that
`wildcard certificates are
deprecated <https://tools.ietf.org/html/rfc6125#section-7.2>`__, and for
good reason. While the compromise of any TLS private key is a serious
matter, the attacker can only impersonate the entities whose names
appear on the certificate (typically one or a handful of DNS addresses).
But a wildcard certificate can impersonate *any* host whose name happens
to match the wildcard value.

In time, validation of wildcard domains will be disabled by default and
(hopefully) eventually removed from TLS libraries. The emergence of
protocols like ACME that allow automated domain validation and
certificate issuance mean that there is no real need for wildcard
certificates anymore, but a lot of programs are yet to implement ACME or
similar; therefore there is still a perceived need for wildcard
certificates. In my opinion some of this boils down to lack of awareness
of novel solutions like ACME, but there can also be a lack of
willingness to spend the time and money to implement them, or a desire
to avoid changing deployed systems, or taking a "wait and see" approach
when it comes to new, security-related protocols or technologies. So for
the time being, some organisations have good reasons to want wildcard
certificates.

FreeIPA currently has no special support for wildcard certificates, but
with support for custom certificate profiles, we can create and use a
profile for issuing wildcard certificates.

.. _creating_a_wildcard_certificate_profile_in_freeipa:

Creating a wildcard certificate profile in FreeIPA
--------------------------------------------------

This procedure works on FreeIPA 4.2 (RHEL 7.2) and later.

First, ``kinit admin`` and export an existing service certificate
profile configuration to a file:

::

   ftweedal% ipa certprofile-show caIPAserviceCert --out wildcard.cfg
   ---------------------------------------------------
   Profile configuration stored in file 'wildcard.cfg'
   ---------------------------------------------------
     Profile ID: caIPAserviceCert
     Profile description: Standard profile for network services
     Store issued certificates: TRUE

Modify the profile; the minimal diff is:

::

   --- wildcard.cfg.bak
   +++ wildcard.cfg
   @@ -19 +19 @@
   -policyset.serverCertSet.1.default.params.name=CN=$request.req_subject_name.cn$, o=EXAMPLE.COM
   +policyset.serverCertSet.1.default.params.name=CN=*.$request.req_subject_name.cn$, o=EXAMPLE.COM
   @@ -108 +108 @@
   -profileId=caIPAserviceCert
   +profileId=wildcard

Now import the modified configuration as a new profile called
``wildcard``:

::

   ftweedal% ipa certprofile-import wildcard \
       --file wildcard.cfg \
       --desc 'Wildcard certificates' \
       --store 1
   ---------------------------
   Imported profile &quot;wildcard&quot;
   ---------------------------
     Profile ID: wildcard
     Profile description: Wildcard certificates
     Store issued certificates: TRUE

Next, set up a CA ACL to allow the ``wildcard`` profile to be used with
the ``cloudapps.example.com`` host:

::

   ftweedal% ipa caacl-add wildcard-hosts
   -----------------------------
   Added CA ACL &quot;wildcard-hosts&quot;
   -----------------------------
     ACL name: wildcard-hosts
     Enabled: TRUE

   ftweedal% ipa caacl-add-profile wildcard-hosts --certprofiles wildcard
     ACL name: wildcard-hosts
     Enabled: TRUE
     CAs: ipa
     Profiles: wildcard
   -------------------------
   Number of members added 1
   -------------------------

   ftweedal% ipa caacl-add-host wildcard-hosts --hosts cloudapps.example.com
     ACL name: wildcard-hosts
     Enabled: TRUE
     CAs: ipa
     Profiles: wildcard
     Hosts: cloudapps.example.com
   -------------------------
   Number of members added 1
   -------------------------

An additional step is required in FreeIPA 4.4 (RHEL 7.3) and later (it
does not apply to FreeIPA < 4.4):

::

   ftweedal% ipa caacl-add-ca wildcard-hosts --cas ipa
     ACL name: wildcard-hosts
     Enabled: TRUE
     CAs: ipa
   -------------------------
   Number of members added 1
   -------------------------

Then create a CSR with subject ``CN=cloudapps.example.com`` (details
omitted), and issue the certificate:

::

   ftweedal% ipa cert-request my.csr \
       --principal host/cloudapps.example.com \
       --profile wildcard
     Issuing CA: ipa
     Certificate: MIIEJzCCAw+gAwIBAgIBCzANBgkqhkiG9w0BAQsFADBBMR8...
     Subject: CN=*.cloudapps.example.com,O=EXAMPLE.COM
     Issuer: CN=Certificate Authority,O=EXAMPLE.COM
     Not Before: Mon Feb 20 04:21:41 2017 UTC
     Not After: Thu Feb 21 04:21:41 2019 UTC
     Serial number: 11
     Serial number (hex): 0xB

Alternatively, you can use Certmonger to request the certificate:

::

   ftweedal% ipa-getcert request \
     -d /etc/httpd/alias -p /etc/httpd/alias/pwdfile.txt \
     -n wildcardCert \
     -T wildcard

This will request a certificate for the current host. The ``-T`` option
specifies the profile to use.

Discussion
----------

Observe that the subject common name (CN) in the CSR *does not contain
the wildcard*. FreeIPA requires naming information in the CSR to
perfectly match the subject principal. As mentioned in the introduction,
FreeIPA has no specific support for wildcard certificates, so if a
wildcard were included in the CSR, it would not match the subject
principal and the request would be rejected.

When constructing the certificate, Dogtag performs a variable
substitution into a subject name string. That string contains the
literal wildcard and the period to its right, and the common name (CN)
from the CSR gets substituted in after that. The relevant line in the
profile configuration is:

::

   policyset.serverCertSet.1.default.params.name=CN=*.$request.req_subject_name.cn$, o=EXAMPLE.COM

When it comes to wildcards in *Subject Alternative Name* DNS-IDs, it
might be possible to configure a Dogtag profile to add this in a similar
way to the above, but I do not recommend it, nor am I motivated to work
out a reliable way to do this, given that wildcard certificates are
deprecated. (By the time TLS libraries eventually remove support for
treating the subject CN as a DNS-ID, I will have little sympathy for
organisations that still haven't moved away from wildcard certs).

In conclusion: you shouldn't use wildcard certificates, and FreeIPA has
no special support for them, but if you really need to, you can do it
with a custom certificate profile.

Reference
---------

This article was created based on an `original discussion on
freeipa-devel <http://www.redhat.com/archives/freeipa-devel/2017-February/msg00160.html>`__
and subsequent `blog by Fraser
Tweedale <http://blog-ftweedal.rhcloud.com/2017/02/wildcard-certificates-in-freeipa/>`__.
