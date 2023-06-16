Back to `FreeIPAv1:Client Configuration
Guide <FreeIPAv1:Client_Configuration_Guide>`__

\__TOC_\_

.. _configuring_windows_as_an_ipa_client:

Configuring Windows as an IPA Client
====================================

This document describes the procedures required to configure both
Windows XP Pro and Windows 2000 Pro as IPA clients.

.. _configuring_client_authentication:

Configuring Client Authentication
---------------------------------

1. Download the `MIT Kerberos 3.x package for
Windows <http://web.mit.edu/kerberos/dist/index.html>`__ to a known
location, and then run the **kfw-3.x-exe** you downloaded to start the
*MIT Kerberos Installation Wizard*.

2. Read the license agreement and then click **I Agree** to accept the
agreement.

3. Ensure you choose to install **KfW Client**; the other components are
optional.

4. Accept the default destination path.

5. Select **Download from web path**, and enter the following URL:

::

   http://<your IPA server's fully-qualified domain name>/ipa/config/

6. Select **Autostart the Network Identity Manager each time you login
to Windows**.

7. Click **Install** to begin the installation. When the installation is
complete, click **Finish** to exit the Wizard.

8. Edit the hosts file and add the IPA server. For example:

::

   <numerical IP address>     ipaserver.example.com   ipaserver

Depending on the version of Windows, the HOSTS file could be located in
different directories. For example:

-  Windows 2000 Pro: ``C:\WINNT\system32\drivers\etc\``
-  Windows XP Pro: ``C:\WINDOWS\system32\drivers\etc\``
