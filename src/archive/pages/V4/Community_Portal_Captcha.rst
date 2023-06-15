.. _the_captcha:

The CAPTCHA
===========

Overview
--------

Several forms on the Community Portal site have the potential for abuse.
A warped-characters captcha forms the first line of defense against
automated spam. While probably not sufficient to ward off a determined
and sophisticated adversary, the captcha will deter untargeted, drive-by
spam and casual targetted attacks.



Use Cases
---------

A captcha is present on each form with the potential for abuse, namely
the self-service user registration form and the password reset
initiation form. The password reset completion form is probably not
susceptible to abuse and does not need a captcha.

Design
------

1.) The user requests a form protected with a captcha.

2.) A random string of 4 characters is generated. The string is fed to
the python captcha library, which returns a BytesIO object representing
the captcha image, which is then turned straightaway into a bytes
object. The string is also mixed with a secret key to form an HMAC
string. The HMAC and a timestamp are stored in a database.

3.) The image, as a data-uri, and the HMAC string, as a hexidecimal text
string, are sent to the client as part of the web form being protected.
The HMAC is placed into a hidden form field to be submitted with the
form. The user solves the captcha and enters the solution into a field
in the protected form. The form is submitted by the user.

4.) Before any processing is done on form data, the captcha is verified
by mixing the user's solution with the secret key from step 2 and
compared securely to the HMAC. If the two match, a correct solution has
been found. The HMAC is then looked up in the database and, if found, is
deleted, regardless of a correct solution or not, to prevent multiple
attempts at the same captcha.

5.) If the solution does not match the HMAC or the HMAC is not found in
the database, the user is returned to the form and informed that their
captcha solution is incorrect. If the captcha is correct, processing of
form data begins and the rest of the form's workflow is carried out.

Implementation
--------------

The captcha system depends on a python library called captcha, which
does not current appear to be available in the Fedora package repos, but
is available through pip.

Old captcha data will be periodically expired and deleted from the
database with a script run regularly by cron.

.. _feature_management23:

Feature Management
------------------

The secret key for the HMAC is currently hard-coded into the program,
but will be read from a configuration file in the future.

.. _how_to_test23:

How to Test
-----------

Automated unit tests will be written.

`Category:FreeIPA Community
Portal <Category:FreeIPA_Community_Portal>`__
