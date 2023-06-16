.. _configuration_files:

Configuration files
-------------------

The server and client share a common core configuration file,
``/etc/ipa/default.conf``. This file is read by the server at startup
and the ``ipa`` command every time it is executed.

The section ``[global]`` is mandatory and the only section used. A
sample default configuration file is:

| ``[global]``
| ``basedn=dc=example,dc=com``
| ``realm=EXAMPLE.COM``
| ``domain=example.com``
| ``xmlrpc_uri=``\ ```https://ipa.example.com/ipa/xml`` <https://ipa.example.com/ipa/xml>`__
| ``ldap_uri=ldapi://%2fvar%2frun%2fslapd-EXAMPLE-COM.socket``
| ``enable_ra=True``

These values are shared between the client and server (on the same
machine). If you need to make client or server-specific configuration
changes you can create a configuration file for it. For the server used
``/etc/ipa/server.conf`` and for client configuration used
``/etc/ipa/cli.conf``

For example, if you want to enable debug logging for the server only,
create ``/etc/ipa/server.conf`` containing:

| ``[global]``
| ``debug=True``

To always have verbose mode set on the client, create
``/etc/ipa/cli.conf`` that contains:

| ``[global]``
| ``verbose=True``

Merging
~~~~~~~

The order that file configuration files are read is:

#. context file (``/etc/ipa/server.conf`` or ``/etc/ipa/cli.conf``)
#. default file (``/etc/ipa/default.conf``)

Once an attribute is set it is not overridden if it appears in another
configuration file.

For example, if you set ``debug=True`` in ``server.conf`` and
``debug=False`` in ``default.conf`` then it will be ``True`` because the
value in ``default.conf`` will be ignored.
