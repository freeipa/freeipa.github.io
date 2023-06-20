LdapUpdate
==========



Apache Configuration Changes
----------------------------

If the templates for ``/etc/httpd/conf.d/ipa.conf`` or
``/etc/httpd/conf.d/ipa-rewrite.conf`` need to be updated and bump the
VERSION field. These files will be automatically updated by a program we
provide the next time the RPM is installed.
``/usr/sbin/ipa-upgradeconfig`` is executed during the ipa-server rpm
%post.

This program will be installed in /usr/sbin/ipa-upgrade-template



LDIF Changes
------------

Changes to the LDIF files will only affect new installations.



ACI Changes
----------------------------------------------------------------------------------------------

ACI changes will need to be documented and done manually by IPA
administrators.



Other Changes
----------------------------------------------------------------------------------------------

This applies to Plugins, indices and other IPA core configuration. This
may have applicability during a new installation to replace the current
ldapmodify < template.ldif that we currently use.

A "script" will be supplied to describe what the default values are (in
case a record doesn't exist) and a series of changes that are applied in
a given order to a single distinguished name.

The syntax of each line in the file is: keyword:attribute:value

No syntax checking of attribute will be done so typos will attempt to
add non-existent attributes which will fail in LDAP.

Blank lines and lines beginning with # are ignored

There are 4 keywords:

-  default: the starting value
-  add: add a value (or values) to an attribute
-  remove: remove a value (or values) from an attribute
-  only: set an attribute to this

The difference between default and add is if the DN of the entry exists
default is ignored. So for updating something like schema, which will be
under cn=schema, you must always use add. It will not re-add the same
information again and again.

We will also provide some things that can be templated such as
architecture (for plugin paths), realm and domain name.

The values are:

-  $REALM - the kerberos realm (EXAMPLE.COM)
-  $FQDN - the fully-qualified domain name of the IPA server being
   updated (ipa.example.com)
-  $DOMAIN - the domain name (example.com)
-  $SUFFIX - the IPA LDAP suffix (dc=example,dc=com)
-  $LIBARCH - set to 64 on x86_64 systems to be used for plugin paths
-  $TIME - an integer representation of current time

Example:

| ``dn: cn=IPA,ou=profile,$SUFFIX``
| ``default:defaultServerList: $FQDN``
| ``default:defaultSearchBase: $SUFFIX``
| ``default:nsslapd-pluginpath: /usr/lib$LIBARCH/dirsrv/plugins/schemacompat-plugin.so``

The values are quoted, comma-separated fields. Quotes are only required
if the value contains a comma.

For example, these convert into the following python lists:

-  sub -> ['sub']
-  "sub" -> ['sub']
-  "sub, eq" -. ['sub, eq']
-  sub,eq -> ['sub', 'eq']
-  sub, eq, "Hi there, Joe" -> ['sub', 'eq', 'Hi there, Joe']

Examples
^^^^^^^^

| ``dn: cn=memberof,cn=index,cn=userRoot,cn=ldbm database,cn=plugins,cn=config``
| ``default: objectClass:top``
| ``default: objectClass:nsIndex``
| ``default: cn:memberof``
| ``default: nsSystemIndex:false``
| ``default: nsIndexType:eq``
| ``# Oops, nsIndexType was wrong, fix it``
| ``only: nsIndexType: sub``
| ``# Ack, should be eq too``
| ``add: nsIndexType: sub, eq``
| ``# Nope, was right the first time, no more sub``
| ``remove: nsIndexType: sub``

A few rules:

#. Only one rule per line
#. Each line stands alone (e.g. an only followed by an only results in
   the last only being used)
#. adding a value that exists is ok. The request is ignored, duplicate
   values are not added
#. removing a value that doesn't exist is ok. It is simply ignored.
#. If a DN doesn't exist it is created from the 'default' entry and all
   updates are applied
#. If a DN does exist the default values are skipped
#. Only the first rule on a line is respected



Special Considerations
----------------------------------------------------------------------------------------------

Plugins
^^^^^^^

plugin configuration is stored in cn=config which is not replicated.
This means that plugin configurations need to be backwards compatible
for the case where IPA servers are running at a lower functional level.

Indexes
^^^^^^^

If an index changes we need to run a task to build the index.



Use Cases
----------------------------------------------------------------------------------------------

Here are some concrete use cases:



Adding a new plugin
^^^^^^^^^^^^^^^^^^^

We are going to switch to a new mechanism for integrating with Solaris.
The current solution requires the customer install nss_ldap on their
machine because the native version doesn't understand the memberOf
attribute. It wants memberUid.

Using the DS Schema Compatibilty plugin provided by the slapi-nis
package we can generate memberUid value from the memberOf entries.

In order to load this into a running IPA server we will provide the
plugin configuration as an update:

| ``dn: cn=Schema Compatibility, cn=plugins, cn=config``
| ``default:objectclass: top``
| ``default:objectclass: nsSlapdPlugin``
| ``default:objectclass: extensibleObject``
| ``default:cn: Schema Compatibility``
| ``default:nsslapd-pluginpath: /usr/lib/dirsrv/plugins/schemacompat-plugin.so``
| ``default:nsslapd-plugininitfunc: schema_compat_plugin_init``
| ``default:nsslapd-plugintype: object``
| ``default:nsslapd-pluginenabled: on``
| ``default:nsslapd-pluginid: schema-compat-plugin``
| ``default:nsslapd-pluginversion: 0.8``
| ``default:nsslapd-pluginvendor: redhat.com``
| ``default:nsslapd-plugindescription: Schema Compatibility Plugin``
| ``[ snip ]``

This will add the entry to a running IPA server if it doesn't already
exist. The library itself will have been added by the RPM installer.



Adding a new index
^^^^^^^^^^^^^^^^^^

During IPA development we realized that we had forgotten to add an index
for the memberOf attribute. This would have had a negative performance
impact so we added it to our default index template. There was no way,
other than manually, to add this index to a running IPA server.

This updater can add a new index or modify existing indices (for
example, if we want to modify the type of index to maintain).

If an index is modified then a task will be created to regenerate the
index for the affected attribute.