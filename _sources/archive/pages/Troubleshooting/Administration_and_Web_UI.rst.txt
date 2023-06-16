This page contains troubleshooting advice for the FreeIPA
**administration framework** and **Web UI**. For other issues, refer to
the index at `Troubleshooting <Troubleshooting>`__.

.. _administration_framework:

Administration Framework
========================

.. _privilege_separation:

Privilege Separation
--------------------

Starting with FreeIPA 4.5, management framework runs in separate
processes and uses GSS-Proxy to obtain Kerberos credentials. `Privilege
Separation <Troubleshooting/PrivilegeSeparation>`__ page describes this
setup in detail, including how to debug privilege separation related
issues.

.. _ipa_command_returns_internal_server_error:

ipa command returns Internal Server Error
-----------------------------------------

-  See ``/var/log/httpd/error_log`` for traceback and potentially for
   more related information

.. _ipa_command_crashes_or_returns_no_data:

ipa command crashes or returns no data
--------------------------------------

-  Try running the command with verbose output and see what exactly is
   being sent to the server:

      ``ipa -vv user-show admin``

-  Try enabling debug level on server and see if there is useful
   information:

   -  Add ``debug=True`` to ``[global]`` section of
      ``/etc/ipa/default.conf`` or ``/etc/ipa/server.conf`` and reload
      ``httpd`` service
   -  Run the command again

.. _web_ui:

Web UI
======

.. _cannot_authenticate_to_web_ui:

Cannot authenticate to Web UI
-----------------------------

-  Make sure that the user can authenticate in CLI, e.g. with
   ``kinit $USER``
-  Make sure that ``httpd``, ``dirsrv`` and ``ipa_memcached`` services
   on the affected FreeIPA server are running.
-  Make sure there are no related `SELinux
   AVCs <http://selinuxproject.org/page/NB_AL>`__
-  Make sure that cookies are enabled on the client browser
-  Make sure that the time on the FreeIPA server is up to date and there
   is no (significant) clock skew (`freeipa-users
   thread <https://www.redhat.com/archives/freeipa-users/2015-April/msg00605.html>`__)
-  Search for any related errors in ``/var/log/httpd/error_log``

.. _browser_shows_err_cert_common_name_invalid___missing_subject_alternative_name_extension_in_certificate:

Browser shows ERR_CERT_COMMON_NAME_INVALID - missing Subject Alternative Name extension in certificate
------------------------------------------------------------------------------------------------------

For more details see Fraser's blog post `Implications of Common Name
deprecation for Dogtag and
FreeIPA <https://blog-ftweedal.rhcloud.com/2017/07/implications-of-common-name-deprecation-for-dogtag-and-freeipa/>`__.

A certificate which is used for web needs to include Subject Alternative
Name extension. If cert was issued without this extension then it needs
to be renewed to include the extension in following way:

-  Use ``getcert list`` to find the REQUEST-ID to use; it will be the
   certificate in NSSDB ``/etc/httpd/alias`` with nickname
   ``Server-Cert``.
-  Use ``getcert resubmit -i REQUEST-ID -D DNS-NAME`` to request a new
   HTTP certificate with the appropriate DNS-NAME Subject Alt Name
   value(s).
