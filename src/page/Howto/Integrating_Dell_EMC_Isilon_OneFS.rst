Integrating_Dell_EMC_Isilon_OneFS
=================================



Create a System Account
-----------------------

First, create a `system
account <https://www.freeipa.org/page/HowTo/LDAP#System_Accounts>`__.

An example of a system account for Foreman is available at
`Creating_a_binddn_for_Foreman <Creating_a_binddn_for_Foreman>`__ as
well.

Note the complete DN of your system account. The foreman example uses
"uid=foreman,cn=sysaccounts,cn=etc,dc=example,dc=com" (without quotes).



Connect Isilon OneFS to FreeIPA
-------------------------------

Isilon OneFS can be configured to connect to LDAP using one of the two
methods:



Using the web UI
----------------------------------------------------------------------------------------------

| Access --> Authentication Providers --> LDAP
| + Add an LDAP provider
| Enter the LDAP provider name of choice.
| ``Server URI:``\ ```ldaps://fqdn`` <ldaps://fqdn>`__\ ``of FreeIPA server``
| *Make sure the fqdn is resolvable from Isilon!*
| ``Base Distinguished Name:``\ *``enter``\ ````\ ``your``\ ````\ ``BaseDN``*
| ``Bind to:``\ *``enter``\ ````\ ``the``\ ````\ ``DN``\ ````\ ``created``\ ````\ ``above``*
| Then enter the password for the DN and Isilon OneFS should be
  connected to FreeIPA via LDAP.



Using the command-line
----------------------------------------------------------------------------------------------

| Get the status of authentication providers before beginning the
  configuration: ``isi auth status``
| Create a new LDAP provider using the command (replace BaseDN, DN,
  DNpassword as necessary):
| ``isi auth ldap create test-ldap \``
| ``--base-dn="BaseDN" \``
| ``--bind-dn="DN" \``
| ``--bind-password="DNpassword" \``
| ``--server-uris="``\ ```ldaps://`` <ldaps://>`__\ ``" \``
| ``--groupnet=``



Double-checking the LDAP configuration
--------------------------------------

| Run the ldap search from the Isilon node to test whether the LDAP
  connection works fine: ``ldapsearch -x uid=admin``
| The `Isilon+LDAP
  Howto <https://www.dellemc.com/en-us/collaterals/unauth/technical-guides-support-information/products/storage-5/docu51637.pdf>`__,
  `"isi auth ldap create"
  documentation <http://doc.isilon.com/onefs/7.0.1/help/en-us/GUID-82489406-9D48-4FE1-AF23-3913444E3AA4.html>`__
  and `LDAP troubleshooting
  guide <https://www.emc.com/collateral/TechnicalDocument/docu63147.pdf>`__
  were used to create this howto.

Notes
-----

| The HowTo above was adapted from a community thread that mentioned
  using the admin user.
| The steps above are not verified as correct but are provided as a base
  to work from.
| The FreeIPA user and development community would appreciate feedback.

References
----------

The `original thread on the FreeIPA mailing
list <https://lists.fedorahosted.org/archives/list/freeipa-users@lists.fedorahosted.org/thread/6RKT5WSBOA54CUYERLL6G6ZGKVSQJTY2/>`__.