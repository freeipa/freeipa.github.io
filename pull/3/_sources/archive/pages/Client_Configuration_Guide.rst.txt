\__TOC_\_

Introduction
============

FreeIPA supports a range of clients, all of which can be configured to
work with an IPA server. In freeIPA version 1.0, the client installation
script is only available for a limited range of clients.

.. _purpose_of_this_guide:

Purpose of this Guide
---------------------

This guide provides instructions on how to configure all of the
supported clients to connect to an IPA server. This includes:

-  System login (for accounts that exist in the IPA server)
-  NFS v4 with Kerberos (for mounting remote filesystems)
-  SSH access (secure client system access with Kerberos)
-  Using Firefox to access the IPA WebUI (for administrative operations)

Audience
--------

This guide is aimed at IPA administrators and those who are responsible
for the installation and day to day operation and maintenance of an IPA
deployment.

.. _configuring_ipa_clients:

Configuring IPA Clients
=======================

This guide covers the following topics:

-  `Configuring Red Hat Enterprise Linux
   Clients <FreeIPAv1:ConfiguringRhelClients>`__
-  `Configuring Fedora Clients <FreeIPAv1:ConfiguringFedoraClients>`__
-  `Configuring Solaris Clients <FreeIPAv1:ConfiguringSolarisClients>`__
-  `Configuring AIX Clients <FreeIPAv1:ConfiguringAixClients>`__
-  `Configuring HP-UX Clients <FreeIPAv1:ConfiguringHpuxClients>`__
-  `Configuring Mac OS X
   Clients <FreeIPAv1:ConfiguringMacintoshClients>`__
-  `Configuring Windows Clients <FreeIPAv1:ConfiguringWindowsClients>`__

.. _configuring_your_browser:

Configuring Your Browser
========================

Firefox can use your Kerberos credentials for authentication, but you
need to specify which domains you want to communicate with, and using
which attributes.

1. Open Firefox, and type "about:config" in the Address Bar.

2. In the Search field, type "negotiate".

3. Ensure the following lines reflect your setup. Replace ".example.com"
with your own IPA server's domain, including the preceding period (.):

::

    network.negotiate-auth.trusted-uris  .example.com
    network.negotiate-auth.delegation-uris  .example.com
    network.negotiate-auth.using-native-gsslib true

For firefox on Windows , do these:

::

    network.negotiate-auth.trusted-uris .example.com
    network.auth.use-sspi false 
    network.negotiate-auth.gsslib: C:\Program Files\MIT\Kerberos\bin\gssapi32.dll

On Some installs this last value may need to be

::

    network.negotiate-auth.gsslib: C:\Program Files(x86)\MIT\Kerberos\bin\gssapi32.dll

`Category:NoLink <Category:NoLink>`__
