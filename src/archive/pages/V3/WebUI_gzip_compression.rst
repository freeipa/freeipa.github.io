\__NOTOC_\_

.. _enable_compression_on_httpd:

Enable compression on httpd
===========================

Overview
--------

Text based resources like JavaScript, HTML, CSS files are currently
transmitted in uncompressed form. Output compression is one of most
recommended and widely use optimization techniques for web pages. Rough
test shows that it can reduce the amount of data transferred from 2MB to
360KB. The compression has the biggest impact on JSON communication,
mainly downloading of metadata (reduction from 1MB to 50KB).

Design
------

Enable gzip encoding using mod_deflate for following mime types:

-  text/html (HTML files)
-  text/plain (for future use)
-  text/css (CSS files)
-  text/xml (XML RPC)
-  application/javascript (JavaScript files)
-  application/json (JSON RPC)
-  application/x-font-woff (woff fonts)

.. _use_cases:

Use Cases
---------

Every usage of Web UI. Should not have any impact on automated tests.

Implementation
--------------

Change of ``/etc/httdp/conf.d/ipa.conf`` by change in its template:
freeipa/install/conf/ipa.conf

.. _impact_to_webui_and_other_components:

Impact to WebUI and other components
------------------------------------

Speeds up communication for IPA clients - Web UI, CLI.

.. _major_configuration_options_and_enablement:

Major configuration options and enablement
------------------------------------------

N/A

Replication
-----------

N/A

.. _updates_and_upgrades:

Updates and Upgrades
--------------------

N/A

Dependencies
------------

N/A, mod_deflate is part of httpd package.

.. _external_impact:

External Impact
---------------

N/A

.. _rfe_author:

RFE author
----------

`Pvoborni <User:Pvoborni>`__
