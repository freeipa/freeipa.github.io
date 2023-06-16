.. _signing_keys:

Signing keys
------------

Release tarballs are signed by our FreeIPA Master Signing Key.

| `` pub   rsa4096/F40800B6298EB963 2017-11-28 [SC]``
| ``       0E63D716D76AC080A4A33513F40800B6298EB963``
| `` uid                 [ unknown] FreeIPA Releases <releases@mg.freeipa.org>``

Note: releases prior to 2017-11-29 were signed by following key:
4A8BA48C2AED933BD495C509A1FBA5F7EF8C4869.

.. _verifying_signature:

Verifying signature
-------------------

Make sure you have the key above in your keyring. In the example below,
a temporary keyring is used.

``$ gpg2 --no-default-keyring --keyring tmp.gpg --keyserver keys.openpgp.org --recv-keys 0E63D716D76AC080A4A33513F40800B6298EB963``

Also, make sure your keys are refreshed:

``$ gpg2 --keyserver keys.openpgp.org --refresh-keys``

Download the release tarball and its signature file, for example:

| ``$ wget -O /tmp/src ``\ ```https://releases.pagure.org/freeipa/freeipa-4.9.6.tar.gz`` <https://releases.pagure.org/freeipa/freeipa-4.9.6.tar.gz>`__
| ``$ wget -O /tmp/src.asc ``\ ```https://releases.pagure.org/freeipa/freeipa-4.9.6.tar.gz.asc`` <https://releases.pagure.org/freeipa/freeipa-4.9.6.tar.gz.asc>`__

You can then verify the tarball with the following command:

| ``$ gpg2 --no-default-keyring --keyring tmp.gpg --verify /tmp/src.asc /tmp/src``
| ``gpg: Signature made Tue 29 Jun 2021 05:32:36 PM CEST``
| ``gpg:                using RSA key 840A1D1C7F3EC4B2FE5304354719E2B8ABBF621A``
| ``gpg: Good signature from "FreeIPA Releases <releases@mg.freeipa.org>" [unknown]``
| ``gpg: WARNING: This key is not certified with a trusted signature!``
| ``gpg:          There is no indication that the signature belongs to the owner.``
| ``Primary key fingerprint: 0E63 D716 D76A C080 A4A3  3513 F408 00B6 298E B963``
| ``    Subkey fingerprint: 840A 1D1C 7F3E C4B2 FE53  0435 4719 E2B8 ABBF 621A``
