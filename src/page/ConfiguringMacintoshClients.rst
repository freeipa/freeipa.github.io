ConfiguringMacintoshClients
===========================

Back to `FreeIPAv1:Client Configuration
Guide <FreeIPAv1:Client_Configuration_Guide>`__

\__TOC_\_



Configuring Macintosh OS X 10.4 as an IPA Client
================================================

This document describes the procedures required to configure a Macintosh
OS as an IPA client. These instructions are specific to Mac OS X 10.4
(Tiger). This version of the OS includes a partial install of the
Kerberos tools you need by default, especially if you perform an upgrade
from 10.1 or 10.2.

   |Note.png|\ **Note:**

      Before starting the freeIPA installation, ensure that you update
      your system with all the latest packages.



Configuring Kerberos Authentication
-----------------------------------

The current version of IPA does not provide for automatic configuration
of Macintosh clients. Configuring authentication is a manual process and
is described below.



Configuring Kerberos
----------------------------------------------------------------------------------------------

1. Ensure that **/System/Library/CFMSupport/Kerberos** is version 4.2 or
higher. If that directory does not exist or is the wrong version,
install the `Kerberos Extras
support <http://web.mit.edu/macdev/www/osx-kerberos-extras.html>`__.

2. Launch **/System/Library/Coreservices/Kerberos**.

3. From the **Edit** menu, choose **Edit Realms**.

4. On the **Settings** tab, enter your IPA server's Kerberos realm (for
example, EXAMPLE.COM).

5. On the **Servers** tab, leave two lines, whose hostnames you then
need to replace with your IPA server's hostname (for example,
*ipaserver.example.com*):

::

   kdc  ipaserver.example.com 88
   admin ipaserver.example.com 749

6. On the **Domains** tab, replace the existing domains with your IPA
server's actual domain (such as example.com):

::

   .example.com
   example.com

7. Click **Make default**, and then close the Kerberos tool. This
creates the files you need, but as they may not be 100% correct, it is
recommended that you verify them manually.

The ``/Library/Preferences/edu.mit.kerberos`` file should look similar
to the following. Remember to replace the example.com settings with your
own IPA server name, Kerberos realm and domain details.

::

   [domain_realm]
       example.com = EXAMPLE.COM
       .example.com = .EXAMPLE.COM
   [libdefaults]
       default_realm = EXAMPLE.COM
       dns_lookup_realm = true
       dns_lookup_kdc = true
       ticket_lifetime = 24h
       forwardable = yes
   [realms]
       EXAMPLE.COM = {
       admin_server = ipaserver.example.com:749
       default_domain = example.com
       kdc = ipaserver.example.com:88
       }



Enabling Kerberos Authentication
----------------------------------------------------------------------------------------------

You now need to modify the ``/private/etc/authorization`` file to allow
Kerberos authentication.

1. Log in as the **admin** user and launch the
**/Applications/Utilities/Terminal** application.

2. Change to the ``/private/etc`` directory and make a backup of the
existing authorization file.

::

   # cd /private/etc
   # cp -p authorization authorization_bak

3. Open the authorization file, and locate the string
"system.login.console".

4. Locate the entry below this string, and then locate the mechanisms
entry.

5. Make the following changes:

   Change ``authinternal``
   To ``builtin:krb5authnoverify,privileged``

..

   |Caution.png| **Caution:**

      **authinternal** may occur more than once. Ensure that you change
      the correct one.

6. Save and close the file.

7. Restart the machine to enable Kerberos authentication.



Configuring LDAP Authorization
------------------------------

These instructions are specific to Mac OS X 10.4 (Tiger).



Creating the LDAP Configuration
----------------------------------------------------------------------------------------------

1. Launch **/Applications/Utilities/Directory Access**.

2. On the **Services** tab, clear all checkboxes except LDAPv3 and
Bonjour.

3. Select the LDAPv3 entry and click **Configure**.

4. Ensure the **Add DHCP-supplied LDAP servers** checkbox is not
selected.

5. Click the arrow next to the **Show Options** label, and then click
**New**.

6. Enter the Server Name (for example, *ipaserver.example.com*).

7. Clear the **Encrypt using SSL** checkbox, and then click **Manual**.

8. Enter the Configuration Name (for example, "IPA LDAP").

9. Ensure that the **Enable** checkbox is selected, and that the **SSL**
checkbox is cleared.



Setting up the LDAP Service Configuration Options
----------------------------------------------------------------------------------------------

1. Select the newly-created LDAP configuration and then click **Edit**.

2. On the **Connection** tab, specify the following:

   2.1. Open/close times out in: 10 seconds
   2.2. Query times out in: 10 seconds
   2.3. Re-bind attempted in: 10 seconds
   2.4. Connection idles out in: 1 minute
   2.5. Clear all checkboxes

3. On the **Search & Mappings** tab, specify the following:

   3.1. Access this LDAP server using: CUSTOM
   3.2. In the **Record Types and Attributes** panel, select **Default
   Attribute Types**, and then click **Add**.
   3.3. Select the **Attribute Types** option, select **RecordName**
   from the list, and then click **OK**.
   3.4. Select the newly-added **RecordName** attribute, and then click
   **Add** under the **Map to any items in list** panel.
   3.5. Type "uid" (without the quotes) in the text box. Click outside
   of the text box to set the value.

4. Add a **Users** record, as follows:

   4.1. Under the **Record Types and Attributes** panel, click **Add**.
   4.2. Select the **Record Types** option, select **Users** from the
   list, and then click **OK**.
   4.3. Select the newly-added **Users** record type, and then click
   **Add** under the **Map to any items in list** panel.
   4.4. Type "inetOrgPerson" (without the quotes) in the text box. Click
   outside of the text box to set the value.
   4.5. In the **Search base** field, type "dc=example,dc=com" (without
   the quotes), and select the **Search in all subtrees** option.

5. Add attributes to the **Users** record as appropriate for your
deployment. The following is an example of the required procedure.

   5.1. Under the **Record Types and Attributes** panel, click **Add**.
   5.2. Select the **Attribute Types** option, and then use
   **Command+Click** to select the attributes that you want to add. For
   example, a typical deployment might include the following attributes:

   -  AuthenticationAuthority
   -  PrimaryGroupID
   -  RealName
   -  RecordName
   -  UniqueID
   -  UserShell

   5.3. Click **OK** to add the selected attributes to the **Users**
   record.

6. Specify appropriate mappings for the attributes that you just added.
For example:

   6.1. Select the **AuthenticationAuthority** record type, and then
   click **Add** under the **Map to any items in list** panel.
   6.2. Type "#;Kerberosv5;;$uid$;EXAMPLE.COM" (without the quotes) in
   the text box. Click outside of the text box to set the value.
   6.3. Use the same procedure to map **PrimaryGroupID** to
   **gidNumber**.
   6.4. Use the same procedure to map **UniqueID** to **uidNumber**.
   6.5. Continue until all required entries have been mapped, and then
   click **OK**.

7. Click **OK** finish setting up the LDAP service configuration
options.



Configuring the LDAP Authorization Options
----------------------------------------------------------------------------------------------

You now need to add the LDAP service to the list of locations used to
search for user authentication information.

1. On the **Authentication** tab, change the **Search** value to
**Custom path**, and then click **Add**.

2. Select the configuration that you added in the `Creating the LDAP
Configuration <FreeIPAv1:ConfiguringMacintoshClients#Creating_the_LDAP_Configuration>`__
step, and then click **Add**.

3. Click **Apply** to update the LDAP configuration, and then exit the
Directory Access application.



Configuring NTP
---------------

-  Open the **Date&Time** utility and point it to
   *ipaserver.example.com* to automatically set the date and time.



Accessing the IPA Server via SSH
--------------------------------

After configuring client authentication, you should be able to use SSH
to connect to the IPA server without be prompted for a password.

1. Get a Kerberos ticket for the **admin** user.

::

   # kinit admin
   # klist (to verify that you successfully retrieved a ticket)

2. If you have a valid Kerberos ticket, ssh should proceed with GSSAPI
authentication without asking for a password:

::

   # ssh admin@ipaserver.example.com



Configuring Client SSH Access
-----------------------------



Configuring System Login
------------------------

1. On the Mac login window, log in as an IPA user.

2. After you have logged in, open a terminal and try the following:

::

   $ id (ensure that the userid and groupid are correct)
   $ klist (ensure that you have a valid Kerberos ticket)

..

   |Note.png| **Note:**

      To open the **Terminal** application, navigate to
      **Applications/Utilities/Terminal.app** or use the keyboard
      shortcut *Command-Shift-U*. You can also drag the Terminal icon to
      the Dock to make it permanently available on your Desktop.

.. |Note.png| image:: Note.png
.. |Caution.png| image:: Caution.png