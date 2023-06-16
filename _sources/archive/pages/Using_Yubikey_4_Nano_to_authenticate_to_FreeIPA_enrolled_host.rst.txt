Goal of this page is to show how to:

-  issue certificate and store it on Yubikey 4 Nano,
-  add the certificate to FreeIPA User,
-  configure FreeIPA client to authenticate user with certificate stored
   on Yubikey 4 Nano.

Sources
-------

-  https://blog-ftweedal.rhcloud.com/2016/08/smart-card-login-with-yubikey-neo/
-  https://developers.yubico.com/yubico-piv-tool/YubiKey_PIV_introduction.html

.. _prepare_yubikey:

Prepare yubikey
===============

.. _install_packages:

Install packages
----------------

``# dnf install -y ykpers yubico-piv-tool pcsc-lite opensc``

.. _start_pcscd:

Start pcscd
-----------

``# systemctl start systemctl start pcscd``

.. _verify_readers:

Verify readers
--------------

| ``# opensc-tool --list-readers``
| ``Detected readers (pcsc)``
| ``Nr.  Card  Features  Name``
| ``0    Yes             Yubico Yubikey 4 OTP+U2F+CCID 00 00``

| ``# yubico-piv-tool -a status -v``
| ``trying to connect to reader 'Yubico Yubikey 4 OTP+U2F+CCID 00 00'.``
| ``Action 'status' does not need authentication.``
| ``Now processing for action 'status'.``
| ``CHUID:  No data available``
| ``CCC:    No data available``
| ``PIN tries left: 3``

.. _reset_yubikey_optional:

Reset Yubikey (optional)
------------------------

Useful if you've lost PIN/PUK.

.. _first_we_need_to_get_the_yubikey_blocked:

First we need to get the Yubikey blocked
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Blocking the yubikey is straighforward. First enter wrong PIN tree times
then enter wrong PUK tree times.

| ``# yubico-piv-tool -a verify -P 000000``
| ``# yubico-piv-tool -a verify -P 000000``
| ``# yubico-piv-tool -a verify -P 000000``
| ``# yubico-piv-tool -a unblock-pin -P 000000 -N 000000``
| ``# yubico-piv-tool -a unblock-pin -P 000000 -N 000000``
| ``# yubico-piv-tool -a unblock-pin -P 000000 -N 000000``

.. _now_we_can_reset_it:

Now we can reset it
~~~~~~~~~~~~~~~~~~~

``# yubico-piv-tool -a reset``

.. _set_new_management_key_pin_and_puk:

Set new Management Key, PIN and PUK
-----------------------------------

| ``# KEY=$(hexdump -v -e '/1 "%x"' /dev/urandom | head -c 48)``
| ``# PIN=$(hexdump -v -e '/1 "%u"' /dev/urandom | head -c 6)``
| ``# PUK=$(hexdump -v -e '/1 "%u"' /dev/urandom | head -c 8)``

| ``# yubico-piv-tool -a set-mgm-key -n $KEY``
| ``# yubico-piv-tool -a change-pin -P 123456 -N $PIN``
| ``# yubico-piv-tool -a change-puk -P 12345678 -N $PUK``

.. _create_certificate:

Create certificate
------------------

.. _generate_private_key:

Generate Private Key
~~~~~~~~~~~~~~~~~~~~

``# yubico-piv-tool --key=$KEY -a generate -s 9a -A RSA2048 -o pub.pem``

.. _generate_certificate_signing_request:

Generate Certificate Signing Request
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``# yubico-piv-tool -a verify -a request -s 9a -P $PIN -S '/CN=test/O=EXAMPLE.ORG/' -i pub.pem -o req.pem``

.. _sign_certificate_signing_request:

Sign Certificate Signing Request
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In production environment the CSR is sent to Security Administrator (or
similar role) and he will provide the certificate. But for testing it's
OK to generate self-sign CA certificate and use it to sign the request.

| ``# head -c 40 /dev/urandom > noise``
| ``# base64 /dev/urandom | head -c 20 > pwfile``
| ``# mkdir /tmp/nssdb``
| ``# certutil -d /tmp/nssdb/ -f pwfile -N``
| ``# echo -e "y\n\ny\n" | certutil -d /tmp/nssdb/ -f pwfile -S -x -n ca -t T,, -m 1 -s "CN=Smart Card CA,O=EXAMPLE.ORG" -z noise -2``
| ``# certutil -d /tmp/nssdb/ -f pwfile -C -c ca -m $RANDOM -a -i req.pem -o cert.pem --keyUsage digitalSignature,nonRepudiation,keyEncipherment,dataEncipherment``

The CA certificate (will be needed later) can be exported to file:

``# certutil -d /tmp/nssdb/ -f pwfile -L -n ca -a -o ca.pem``

.. _import_the_certificate_into_yubikey:

Import the certificate into Yubikey
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``# yubico-piv-tool --key=$KEY -a import-certificate -i cert.pem -s 9a``

.. _assign_the_certificate_to_freeipa_user:

Assign the certificate to FreeIPA user
======================================

user-add-cert command expects only the base64 encoded blob:

| :literal:`- `tail -n +2` omits the -----BEGIN CERTIFICATE---- line`
| :literal:`- `head -n -1` omits the -----END CERTIFICATE---- line`
| :literal:`- `tr -d '\n\r'` joins all lines into one`

| ``# kinit test``
| ``# cat cert.pem | tail -n +2 | head -n -1 | tr -d '\n\r' | ipa user-add-cert test``

.. _enable_support_on_freeipa_client:

Enable support on FreeIPA client
================================

.. _install_packages_1:

Install packages
----------------

``# dnf install -y opensc python{2,3}-sssdconfig``

.. _add_smart_card_to_etcpkinssdb:

Add Smart Card to /etc/pki/nssdb
--------------------------------

``# modutil -dbdir /etc/pki/nssdb -add "OpenSC" -libfile opensc-pkcs11.so``

.. _start_and_enable_pc_smart_card_daemon:

Start and enable PC Smart Card Daemon
-------------------------------------

| ``# systemctl start pcscd.service pcscd.socket``
| ``# systemctl enable pcscd.service pcscd.socket``

.. _enable_authentication_using_certificates_in_sssd:

Enable authentication using certificates in SSSD
------------------------------------------------

| ``# python << EOF``
| ``from SSSDConfig import SSSDConfig``
| ``c = SSSDConfig()``
| ``c.import_config()``
| ``c.set('pam', 'pam_cert_auth', 'True')``
| ``c.write()``
| ``EOF``

.. _disable_ocsp_if_oscp_unreachable:

Disable OCSP (if oscp unreachable)
----------------------------------

| ``# python << EOF``
| ``from SSSDConfig import SSSDConfig``
| ``c = SSSDConfig()``
| ``c.import_config()``
| ``c.set('sssd', 'certificate_verification', 'no_ocsp')``
| ``c.write()``
| ``EOF``

.. _import_ca_certificates_for_smart_cards:

Import CA certificates for Smart Cards
--------------------------------------

``# certutil -d /etc/pki/nssdb -A -i ca.pem -n "Smart Card CA ($RANDOM)" -t T,,``

.. _restart_sssd:

Restart SSSD
------------

``# systemctl restart sssd.service``
