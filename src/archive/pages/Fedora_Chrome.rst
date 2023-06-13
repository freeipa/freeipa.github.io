Overview
========

Chrome/Chromium is available from Fedora and Google. Kerberos support is
available in version 9.0 or later.

.. _adding_chrome_repository:

Adding Chrome Repository
========================

Fedora
------

::

   % cd /etc/yum.repos.d
   % wget http://repos.fedorapeople.org/repos/spot/chromium/fedora-chromium.repo

Google
------

Create /etc/yum.repos.d/google.repo:

::

   [google64]
   name=Google - x86_64
   baseurl=http://dl.google.com/linux/rpm/stable/x86_64
   enabled=1
   gpgcheck=1
   gpgkey=https://dl-ssl.google.com/linux/linux_signing_key.pub

.. _installing_chrome:

Installing Chrome
=================

.. _fedora_1:

Fedora
------

View available versions:

::

   % yum list | grep chromium
   chromium.x86_64                    10.0.634.0-1.fc14 fedora-chromium
   chromium-debuginfo.x86_64          10.0.634.0-1.fc14 fedora-chromium
   chromium-libs.x86_64               10.0.634.0-1.fc14 fedora-chromium

Install Chromium:

::

   % yum install chromium

.. _google_1:

Google
------

View available versions:

::

   % yum list | grep google-chrome
   google-chrome-beta.x86_64          8.0.552.200-65749 google64
   google-chrome-stable.x86_64        7.0.517.44-64615  google64
   google-chrome-unstable.x86_64      9.0.576.0-65344   google64

Install Chrome:

::

   % yum install google-chrome-unstable

.. _running_chrome:

Running Chrome
==============

.. _fedora_2:

Fedora
------

Run Chromium with Kerberos:

::

   % chromium-browser\
    --auth-server-whitelist="*.example.com"\
    --auth-negotiate-delegate-whitelist="*.example.com"

.. _google_2:

Google
------

Run Chrome with Kerberos:

::

   % google-chrome\
    --auth-server-whitelist="*.example.com"\
    --auth-negotiate-delegate-whitelist="*.example.com"

If the aforementioned approach doesn't work, you can override an
AuthScheme policy [4, 7]:

::

   { "AuthServerWhitelist": "*.example.com",
   "AuthNegotiateDelegateWhitelist": "*.example.com" }

.. _installing_ca_certificate:

Installing CA Certificate
=========================

This step is optional. To install CA certificate go to Preferences ->
Under the Hood -> Manage Certificates. Select the Authorities tab, then
click Import. Select the CA certificate, then select at least \*Trust
this certificate for identifying web sites.\* Finally, restart the
browser.

References
==========

#. `Issue 54694: Kerberos negotiate
   auth <http://code.google.com/p/chromium/issues/detail?id=54694#c13>`__
#. `Linux Software
   Repositories <http://www.google.com/linuxrepositories/>`__
#. `Install Google Chrome with YUM on Fedora 14/13, Red Hat (RHEL)
   6 <http://www.if-not-true-then-false.com/2010/install-google-chrome-with-yum-on-fedora-red-hat-rhel/>`__
#. `HTTP
   authentication <https://sites.google.com/a/chromium.org/dev/developers/design-documents/http-authentication>`__
#. `Fedora Chromium
   Repository <http://repos.fedorapeople.org/repos/spot/chromium/>`__
#. `Circumventing Chrome Access-control-allow-origin on the local file
   system? <http://pinoytech.org/question/4742467/circumventing-chrome-access-control-allow-origin-on-the-local-file-system>`__
#. `Google Chrome and Kerberos on
   Linux <https://kurt.seifried.org/2012/11/24/google-chrome-and-kerberos-on-linux/>`__
