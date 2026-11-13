FreeIPA - Identity, Policy, Audit
=================================

Identity
--------

Manage Linux users and :doc:`client hosts </page/Client>` in your :term:`realm` from
:doc:`one central location </page/Directory_Server>` with CLI, :doc:`Web UI </page/Web_UI>` or
RPC access. Enable :doc:`Single Sign On </page/Kerberos>` authentication for all
your systems, services and applications.

Policy
------

Define :doc:`Kerberos </page/Kerberos>` authentication and authorization policies
for your identities. Control services like :doc:`DNS </page/DNS>`, SUDO, SELinux
or autofs.

Trusts
------

Create mutual :doc:`trust </page/Trusts>` with other Identity Management systems
like :doc:`Microsoft Active Directory </page/Active_Directory_trust_setup>`.

:doc:`About FreeIPA </About>` • :doc:`Roadmap </page/Roadmap>` •
:doc:`FreeIPA Leaflet </page/Leaflet>` • :doc:`FreeIPA public demo </page/Demo>` •
`Blogs/RSS <http://planet.freeipa.org>`__


Main features
-------------

-  Integrated security information management solution combining Linux
   (Fedora), `389 Directory
   Server <http://directory.fedoraproject.org/>`__, `MIT
   Kerberos <http://k5wiki.kerberos.org/wiki/Main_Page>`__, NTP,
   `DNS <https://codeberg.org/freeipa/bind-dyndb-ldap>`__, `Dogtag certificate
   system <http://pki.fedoraproject.org>`__,
   `SSSD <https://sssd.io/>`__ and others.
-  Built on top of well known Open Source components and standard
   protocols
-  Strong focus on ease of management and automation of installation and
   configuration tasks.
-  Full multi master replication for higher redundancy and scalability
-  Extensible management interfaces (CLI, Web UI, XMLRPC and JSONRPC
   API) and Python SDK


Releases
--------

- :doc:`/release-notes/4-13-5`
- :doc:`/release-notes/4-13-4`
- :doc:`/release-notes/4-13-3`
- :doc:`/release-notes/4-13-2`
- :doc:`/release-notes/4-13-1`
- :doc:`/release-notes/4-13-0`
- :doc:`/release-notes/4-12-5`
- :doc:`/release-notes/4-12-4`
- :doc:`/release-notes/4-12-3`
- :doc:`/release-notes/4-12-2`

See :doc:`/release-notes/index` for all release announcements.

Getting involved
----------------

Whether you'd like to contribute to discussion, to code, or simply test
it out, FreeIPA needs your help!

-  To contribute to the development of FreeIPA go to
   :doc:`Contribute </Contribute>` and subscribe to
   `freeipa-devel <https://lists.fedoraproject.org/archives/list/freeipa-devel@lists.fedorahosted.org/>`__
-  To share deployment experience with FreeIPA and ask "how to"
   questions subscribe to
   `freeipa-users <https://lists.fedoraproject.org/archives/list/freeipa-users@lists.fedorahosted.org/>`__
-  To file a bug, RFE or to see where you can help, please see :doc:`/Contribute`.
-  For security-related communication, please use the security contact in
   :doc:`/Contribute`.
-  Contributions are always welcome!

:doc:`Learn more </Contribute>`

Public Demo
-----------

People eager to try the looks and feel of the most recent FreeIPA, can
visit our :doc:`public FreeIPA instance </page/Demo>`! It is great for

-  Testing changes in the most recent CLI/Web UI/API
-  Testing :doc:`client </page/Client>` enrollment
-  Testing :doc:`web applications </page/Web_App_Authentication>` with
   :doc:`LDAP </page/Directory_Server>` / :doc:`Kerberos </page/Kerberos>` authentication

Read more on the page :doc:`Demo </page/Demo>`.

.. toctree::
   :hidden:
   :maxdepth: 2

   /About
   /page/Documentation
   /page/Troubleshooting
   /glossary
   /Downloads
   /Contribute
   /release-notes/index
