About
=====

What is FreeIPA?
----------------

FreeIPA is an integrated security information management solution
combining Linux (Fedora), 389 Directory Server, MIT Kerberos, NTP, DNS,
Dogtag (Certificate System). It consists of a web interface and
command-line administration tools.

FreeIPA is an integrated Identity and Authentication solution for
Linux/UNIX networked environments. A FreeIPA server provides centralized
authentication, authorization and account information by storing data
about user, groups, hosts and other objects necessary to manage the
security aspects of a network of computers.

FreeIPA is built on top of well known Open Source components and
standard protocols with a very strong focus on ease of management and
automation of installation and configuration tasks.

Multiple FreeIPA servers can easily be configured in a FreeIPA Domain in
order to provide redundancy and scalability. The `389 Directory
Server <http://directory.fedoraproject.org/wiki/Main_Page>`__ is the
main data store and provides a full multi-master
`LDAPv3 <http://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol>`__
directory infrastructure. Single-Sign-on authentication is provided via
the `MIT <http://web.mit.edu/kerberos/>`__
`Kerberos <http://en.wikipedia.org/wiki/Kerberos_%28protocol%29>`__ KDC.
Authentication capabilities are augmented by an integrated `Certificate
Authority <http://en.wikipedia.org/wiki/Certificate_authority>`__ based
on the `Dogtag <http://pki.fedoraproject.org/wiki/PKI_Main_Page>`__
project. Optionally `Domain
Names <http://en.wikipedia.org/wiki/Domain_Name_System>`__ can be
managed using the integrated `ISC
Bind <https://www.isc.org/software/bind>`__ server.

Security aspects related to `access
control <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Linux_Domain_Identity_Authentication_and_Policy_Guide/configuring-host-access.html>`__,
`delegation <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Linux_Domain_Identity_Authentication_and_Policy_Guide/sudo.html>`__
of administration tasks and other
`network <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Linux_Domain_Identity_Authentication_and_Policy_Guide/automount.html>`__
`administration <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Linux_Domain_Identity_Authentication_and_Policy_Guide/user-keys.html>`__
`tasks <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Linux_Domain_Identity_Authentication_and_Policy_Guide/selinux-mapping.html>`__
can be fully centralized and managed via the `Web
UI <https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Linux_Domain_Identity_Authentication_and_Policy_Guide/using-the-ui.html>`__
or the *ipa* Command Line tool.

Resources
~~~~~~~~~

-  Most of our activity happens on the
   `freeipa-devel <https://lists.fedoraproject.org/archives/list/freeipa-devel@lists.fedorahosted.org/>`__
   and
   `freeipa-user <https://lists.fedoraproject.org/archives/list/freeipa-users@lists.fedorahosted.org/>`__
   mailing lists as well as on the #freeipa IRC channel on the
   irc://irc.libera.chat.

References
~~~~~~~~~~

FreeIPA takes advantage of different technologies:

-  `MIT KDC <http://k5wiki.kerberos.org/wiki/Main_Page>`__ - core of the
   FreeIPA's authentication.
-  `389 Directory Server <http://directory.fedoraproject.org/>`__ - back
   end where FreeIPA keeps all data.
-  `Dogtag Certificate
   System <http://pki.fedoraproject.org/wiki/PKI_Main_Page>`__ - FreeIPA
   includes CA & RA for certificate management functions.
-  `SSSD <https://sssd.io>`__ - client side component that integrates
   FreeIPA as a authentication and identity provider in a better way
   than traditional NSS & PAM.
