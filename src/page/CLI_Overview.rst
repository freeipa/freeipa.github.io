CLI_Overview
============

\__TOC_\_

Introduction
============

The IPA v1 toolset consisted of a slew of separate python programs which
called into an XML-RPC backend. This worked fine but was tedious to
maintain, often requiring the same fix to be applied to all of them.

IPA v2 uses a plugin system where the same plugin is used both for the
XML-RPC server-side interface and for the command-line interface. This
results in a consistent, unified interface that is easy to maintain.

From the user's perspective, plugins are more or less interchangeable
with commands. Most plugins implement commands used to manage IPA and
its data. With the exception of two build-ins (`help\` and \`console`)
all commands are introduced by plugins.

Commands are invoked like this:

``ipa [global-options] COMMAND [command-parameters-that-is-options-and-arguments]``

A list of global options can be displayed using:

``ipa --help``

The plugins are organized by type of objects they manage. This type can
also be used to get an overview of the available commands. To display
all commands in a specific module, use the \`help\` command as follows:

``ipa help TOPIC``

Parameters available for a specific command are displayed with:

``ipa COMMAND --help``

If a list of parameter isn't enough, more information about specific
commands is available through the \`help\` command:

``ipa help COMMAND``



Command conventions
===================



Command output
==============

All commands used to manipulate LDAP entries (which is currently a
majority of all plugins) automatically transform attributes as they are
stored in LDAP into a more user-friendly form and most of the time, some
less relevant attributes are hidden by default. Since this behaviour
isn't always desirable, all such commands accept the following options:

| ``--all     display all attributes``
| ``--raw     display attributes as they are stored in LDAP``

--all makes the command retrieve all attributes and display them, this
doesn't apply to explicitly hidden attributes.

--raw makes the command display all retrieved attributes as stored in
LDAP. Explicitly hidden attributes (if retrieved) are displayed as well.

Note: These options might make it into the global options in the future.



Plugin modules
==============

This section describes all plugin modules and use-cases for the commands
they implement.

aci
---

Used to create tasks for the `Delegation <Delegation>`__ system. To be
used with extreme caution as it can be used to grant write, add and
delete permissions.

automount
---------

The automount module implements commands used to manage automount
(autofs) maps and keys.

Recently, it was decided that we need to support different automount
maps per location. In other words, allow automount maps to be divided
into one or more groups, so that each client can use a different set.
EVERY AUTOMOUNT MAP AND KEY NEED TO BE PART OF AN AUTOMOUNT LOCATION.
After a clean IPA installation, there is no automount location defined.



Creating new locations
----------------------------------------------------------------------------------------------

``ipa automountlocation-add LOCATION_NAME``

LOCATION_NAME is a unique string used to identify location being
created.

If successful the new location is automatically populated with an
auto.master map.



Listing all locations
----------------------------------------------------------------------------------------------

``ipa automountlocation-find``



Searching for locations
----------------------------------------------------------------------------------------------

``ipa automountlocation-find SEARCH_STRING``

All locations with SEARCH_STRING in their name will be listed.



Exporting automount data for locations
----------------------------------------------------------------------------------------------

``ipa automountlocation-tofiles LOCATION_NAME``

This command might be useful, if (for whatever reason) some clients
can't connect to LDAP to retrieve the automount data.



Deleting locations
----------------------------------------------------------------------------------------------

``ipa automountlocation-del LOCATION_NAME``

This command deletes a location and EVERYTHING IN IT (ALL maps and
keys).



Creating maps
----------------------------------------------------------------------------------------------

``ipa automountmap-add LOCATION_NAME MAP_NAME``

LOCATION_NAME is the name of the location the map being created is to be
part of.

MAP_NAME is the name of the new indirect map.



Creating indirect maps
----------------------------------------------------------------------------------------------

An indirect map represents a common mountpoint for specific keys. An
indirect map is equivalent to an automount file listed in
/etc/auto.master.

There are 2 ways of creating them; the simple way:

``ipa automountmap-add-indirect LOCATION_NAME MAP_NAME --mount=MOUNT_POINT``

and the hard(er) way:

| `` ipa automountmap-add LOCATION_NAME MAP_NAME``
| `` ipa automountkey-add LOCATION_NAME auto.master MOUNT_POINT --info=MAP_NAME``

MOUNT_POINT is the mount point such as "/mnt".



Listing all maps in a location
----------------------------------------------------------------------------------------------

``ipa automountmap-find LOCATION_NAME``



Displaying maps
----------------------------------------------------------------------------------------------

``ipa automountmap-show LOCATION_NAME MAP_NAME``



Deleting maps
----------------------------------------------------------------------------------------------

``ipa automountmap-del LOCATION_NAME MAP_NAME``

This command deletes the map and EVERYTHING IN IT (ALL keys). Keys that
link to this map ARE NOT deleted, because they are stored in another map
(usually auto.master).



Creating keys
----------------------------------------------------------------------------------------------

Keys in automount have 2 roles:

-  In the auto.master map, they link other maps with mount points.
-  In any other map, they represent a specific device to be mounted and
   it's mount options.

``ipa automountkey-add LOCATION_NAME MAP_NAME KEY_NAME --info=MOUNT_INFO``

If addind a key to auto.master, KEY_NAME should be the mount point and
MOUNT_INFO the name of a map. Otherwise, KEY_NAME should be the
directory name we want the device to be mounted to and MOUNT_INFO should
contain a path to this device along with mount options.



Listing all keys in a specific map
----------------------------------------------------------------------------------------------

``ipa automountkey-find LOCATION_NAME MAP_NAME``



Deleting keys
----------------------------------------------------------------------------------------------

``ipa automountkey-del LOCATION_NAME MAP_NAME KEY_NAME``



Using automount information from LDAP on clients
----------------------------------------------------------------------------------------------

Assumptions:

-  IPA server is reachable at $SERVER_HOSTNAME, LDAP base DN is
   $LDAP_BASE_DN
-  Client being configured has autofs installed and its location of
   choise is $AUTOMOUNT_LOCATION.

In file /etc/nsswitch change this line:

``automount file``

to

``automount ldap``

In file /etc/sysconfig/autofs add the following lines at its end:

| ``LDAP_URI="``\ ```ldap://$SERVER_HOSTNAME`` <ldap://$SERVER_HOSTNAME>`__\ ``"``
| ``SEARCH_BASE="cn=$AUTOMOUNT_LOCATION,cn=automount,$LDAP_BASE_DN"``
| ``MAP_OBJECT_CLASS="automountMap"``
| ``ENTRY_OBJECT_CLASS="automount"``
| ``MAP_ATTRIBUTE="automountMapName"``
| ``ENTRY_ATTRIBUTE="automountKey"``
| ``VALUE_ATTRIBUTE="automountInformation"``

If unsure about what to put into the SEARCH_BASE line, issue this
command to retrieve the correct DN:

``ipa automountlocation-find --name=$AUTOMOUNT_LOCATION --raw``

Restart autofs and we're done.

cert
----

Manage certificates that are issued by the IPA CA server. This command
should not be confused with the certmonger ``ipa-getcert`` command.



cert-get
----------------------------------------------------------------------------------------------

Retrieve an issued certificate. This is not implemented in the selfsign
CA.



cert-remove-hold
----------------------------------------------------------------------------------------------

Removes a certificate hold put on hold using the ``cert-revoke``. This
is not implemented in the selfsign CA.



cert-request
----------------------------------------------------------------------------------------------

Provide a Certificate Signing Request (CSR) and receive back a server
certificate.

If the CA backend is a dogtag CA then the subject in the CSR will be
ignored except for the CN component (which should be the FQDN of the
server you are generating the certificate for).

If the CA backend is the selfsign CA then the subject needs to exactly
match the subject format IPA was configured with at install time (by
default CN=<fqdn, o=IPA). CSRs not matching this format will be
rejected.



cert-revoke
----------------------------------------------------------------------------------------------

Revoke a certificate and add it to the CRL. This is not implemented in
the selfsign CA. The revocation reasons, as defined by RFC 5280, are:

-  0 - unspecified
-  1 - keyCompromise
-  2 - cACompromise
-  3 - affiliationChanged
-  4 - superseded
-  5 - cessationOfOperation
-  6 - certificateHold
-  8 - removeFromCRL
-  9 - privilegeWithdrawn
-  10 - aACompromise



cert-status
----------------------------------------------------------------------------------------------

Return the status of a certificate request. The dogtag CA issues
certificates immediately so generally this will always be issued. This
is not implemented in the selfsign CA.

config
------

Manage IPA configuration information such as:

-  default login shell
-  default primary group
-  root of home directories
-  maximum username length
-  LDAP search limits
-  Attributes used in searches

dns
---

Domain Name System (DNS) management.

Implements a set of commands useful for manipulating DNS records used by
the BIND LDAP plugin.

EXAMPLES:

Add new zone;

``ipa dns-create example.com nameserver.example.com admin@example.com``

Add second nameserver for example.com:

``ipa dns-add-rr example.com @ NS nameserver2.example.com``

Delete previously added nameserver from example.com:

``ipa dns-del-rr example.com @ NS nameserver2.example.com``

Add new A record for www.example.com: (random IP)

``ipa dns-add-rr example.com www A 80.142.15.2``

Show zone example.com:

``ipa dns-show example.com``

Find zone with 'example' in it's domain name:

``ipa dns-find example``

Find records for resources with 'www' in their name in zone example.com:

``ipa dns-find-rr example.com www``

Find A records for resource www in zone example.com

``ipa dns-find-rr example.com --resource www --type A``

Show records for resource www in zone example.com

``ipa dns-show-rr example.com www``

Delete zone example.com with all resource records:

``ipa dns-delete example.com``

group
-----

Groups of users.

A notable change in v2 is that it allows non-Posix groups (the default).
To create a posix group add the --posix flag. If you forget to create
the group as posix at creation you can promote it with group-mod.

A posix group cannot be made into a non-posix group.

hbac
----

Managed Host-Based Access Control is used to control who can log into
what machines, when and from where.

There are 4 components to an HBAC:

-  host: hosts and hostgroups affected by HBAC rule (the target host)
-  sourcehost: hosts and hostgroups affected by HBAC rule (the source
   host)
-  user: users and groups affected by HBAC rule (the who)
-  accesstime: when the rule is active. There can be more than one.

A simple example: Allow the user admin to ssh into the host tiger from
the host lion.

| ``$ ipa hbac-add --type=allow --service=sshd tiger_sshd``
| ``$ ipa hbac-add-host tiger_sshd --hosts=tiger.example.com``
| ``$ ipa hbac-add-sourcehost tiger_sshd --hosts=lion.example.com``
| ``$ ipa hbac-add-user --users=admin tiger_sshd``

There is no access time associated with this rule so it is is always
available.

host
----

A host represents a computer. Certain information can be maintained with
the host but this isn't meant to serve as an inventory system. The
following pieces of information can be stored:

-  Description
-  Locality (broadly where it is, e.g. Baltimore)
-  Location (specifically where it is, Lab 2)
-  Platform
-  OS

A password can be set on the host to be used by the ipa-join command.
This allows the host to enroll into the IPA realm and obtain a keytab.
This password is a one-use password and is removed when a keytab is
retrieved.

hostgroup
---------

Groups of hosts.

misc
----

netgroup
--------

passwd
------

Used to set or reset a user's password.

Any password not set by the user will require a reset the first time the
user logs in.



Password Policy: pwpolicy
-------------------------

This plugin provides group-based password policy. A global policy is
defined but can be overridden with a group-based policy.

Each policy has a priority value set, an integer from 1-maxint. The
higher the value, the higher the priority. What this means is that if a
user is in multiple groups with password policies set then the one with
the highest priority wins.



Add new policy
----------------------------------------------------------------------------------------------

Not all aspects of the policy are required but policies are not
additive. So if your global policy sets maxlife to 99 days and you don't
set one in your group policy this does not default to 99 days, it
defaults to no policy set on max life.



Delete policy
----------------------------------------------------------------------------------------------

This allows you to remove the password policy from a given group. Any
users that were in that group will use the next highest policy or the
global policy. The global policy cannot be removed.



Modify policy
----------------------------------------------------------------------------------------------

This is used both for modifying per-group policy and the global policy.
If no --group option is passed in then the global policy is modified.



Show policy
----------------------------------------------------------------------------------------------

There are 3 options with this plugin:

-  Pass no arguments to see the global policy
-  Pass in --group to show the policy associated with that group
-  Pass in --user to show the policy that applies to the user

This last option is useful to see how the priorities impact the policy
that will apply to a given user.

rolegroup
---------

Used in the delegation system. A rolegroup is a high-level concept that
logically groups tasks together.

See `Delegation <Delegation>`__ for more information

service
-------

A service represents a kerberos or system service on a host. The format
is SERVICE/FQDN@REALM. The service string is generally a convention
appropriate to the service. Some examples are:

-  host - used for ssh
-  HTTP - used with mod_auth_kerb

Service names in kerberos are case-sensitive.

Services may also store certificates.

The ipa-getkeytab command can be used to obtain a keytab for a service.

taskgroup
---------

A group that is granted access by a specific ACI. The idea is that one
or more ACIs to complete a task grant permission to a taskgroup. A
rolegroup can then be made a member of a taskgroup. Members of a group
inherit ACI permissions.

So users of a rolegroup are granted permissions.

See `Delegation <Delegation>`__ for more information

user
----

Manage users.



Lock a user account
----------------------------------------------------------------------------------------------

user-lock will lock a user account, preventing them from logging in.



Unlock user account
----------------------------------------------------------------------------------------------

user-unlock will unlock a user account.