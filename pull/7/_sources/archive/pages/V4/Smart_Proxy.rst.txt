Introduction
------------

A Smart Proxy is something that the `Foreman <http://theforeman.org>`__
project uses to add a layer of abstraction to certain operations. One of
these is `Realm
Enrollment <http://projects.theforeman.org/projects/foreman/wiki/RealmJoinIntegration>`__.

The IPA Smart Proxy isn't meant to be limited to Foreman, but will
provide the features that it needs in order to do provisioning using a
one-time password, with little or no user intervention.

The basic steps to do provisioning are:

#. Create a host
#. Set an OTP (can be done at host creation)
#. Pass this OTP to a new host via a kickstart snippet and execute
   ipa-client-install during the post process.

This can be coupled with more advanced IPA features to do automatic
hostgroup assignment based on automember rules. This in turn can let one
define sudo and HBAC security by hostgroup, without having to manually
assign enrolled hosts.

A Foreman smart proxy must implement a REST API, and that is fine. This
REST API is unauthenticated. It will use defined principal to make IPA
calls. It is expected that the smart proxy will be installed along side
Foreman, and listen only on the localhost port.

Data be returned as JSON.

API
---

Other Foreman Smart Proxy APIs can be seen
`here <http://projects.theforeman.org/projects/2/wiki/API>`__ for
reference.

Note I am proposing two namespaces here. One is to match the design
principal from Foreman, using /:realm:/:domain and one as a more generic
IPA Namespace. I'm open to discarding this second one in favor of one
from IPA, but given the nature of these requests I anticipate
domain-specific requirements that a generic IPA RESTful API can't
provide.

.. _foreman_specific_api:

Foreman-specific API
~~~~~~~~~~~~~~~~~~~~

+-----------------+-----------+-----------------+-----------------+
| Path            | REST Type | Description     | Input           |
+=================+===========+=================+=================+
| /real           | POST      | Create a host   | {"hostname":    |
| m/:domain/:fqdn |           |                 | string,         |
|                 |           |                 | "password":     |
|                 |           |                 | string          |
|                 |           |                 | (optional),     |
|                 |           |                 | "random":       |
|                 |           |                 | boolean,        |
|                 |           |                 | "force",        |
|                 |           |                 | boolean}        |
+-----------------+-----------+-----------------+-----------------+
| /real           | DELETE    | Remove a        |                 |
| m/:domain/:fqdn |           | specific        |                 |
|                 |           | hostgroup       |                 |
+-----------------+-----------+-----------------+-----------------+

.. _more_generic_api:

More Generic API
~~~~~~~~~~~~~~~~

+-----------------+-----------+-----------------+-----------------+
| Path            | REST Type | Description     | Input           |
+=================+===========+=================+=================+
| /ipa/           | GET       | Get all hosts   |                 |
| smartproxy/host |           |                 |                 |
+-----------------+-----------+-----------------+-----------------+
| /ipa/smartp     | GET       | Retrieve a host |                 |
| roxy/host/:fqdn |           | entry           |                 |
+-----------------+-----------+-----------------+-----------------+
| /ipa/smartp     | POST      | Create a host   | {"hostname":    |
| roxy/host/:fqdn |           | entry           | string,         |
|                 |           |                 | "password":     |
|                 |           |                 | string          |
|                 |           |                 | (optional),     |
|                 |           |                 | "random":       |
|                 |           |                 | boolean,        |
|                 |           |                 | "force",        |
|                 |           |                 | boolean}        |
+-----------------+-----------+-----------------+-----------------+
| /ipa/smartp     | DELETE    | Delete a host   |                 |
| roxy/host/:fqdn |           | entry           |                 |
+-----------------+-----------+-----------------+-----------------+
| /ipa/smart      | GET       | Get all host    |                 |
| proxy/hostgroup |           | groups          |                 |
+-----------------+-----------+-----------------+-----------------+
| /ipa/s          | GET       | Get a specific  |                 |
| martproxy/hostg |           | hostgroup       |                 |
| roup/:hostgroup |           |                 |                 |
+-----------------+-----------+-----------------+-----------------+
| /ipa/s          | POST      | Create a        | {"cn": string,  |
| martproxy/hostg |           | hostgroup       | "description":  |
| roup/:hostgroup |           |                 | string}         |
+-----------------+-----------+-----------------+-----------------+
| /ipa/s          | DELETE    | Delete a        |                 |
| martproxy/hostg |           | specific        |                 |
| roup/:hostgroup |           | hostgroup       |                 |
+-----------------+-----------+-----------------+-----------------+

The return value will be json for all requests.

When a host is created with a random password then the only time that
password will be available is in the return value from the creation. It
will look something like:

::

   $ curl http://localhost:8090/ipa/rest/host/test18.example.com -X POST --d random=true
   {
     "dn": "fqdn=test18.example.com,cn=computers,cn=accounts,dc=example,dc=com", 
     "fqdn": [
       "test18.example.com"
     ], 
     "has_keytab": false, 
     "has_password": true, 
     "ipauniqueid": [
       "cf46403c-7879-11e3-ba3f-52540042bb95"
     ], 
     "managedby_host": [
       "test18.example.com"
     ], 
     "objectclass": [
       "ipaobject", 
       "nshost", 
       "ipahost", 
       "pkiuser", 
       "ipaservice", 
       "ieee802device", 
       "ipasshhost", 
       "top", 
       "ipaSshGroupOfPubKeys"
     ], 
     "randompassword": "C5KK=pG6-QBW"
   }

Hosts
-----

A host in IPA needs to have a valid DNS entry somewhere. If the A record
lookup fails then the host will fail to be created unless force = True.

A host can be created with a specific password using the password
option, or a random password can be generated setting random to True. If
neither is set then a privileged account will be needed to enroll the
host.

Hostgroups
----------

The current API just manages the creation and deletion, not membership.
Membership can be handled using the IPA automember commands but is
outside of the scope of this initial implementation.
