Overview
--------

`Puppet <http://puppetlabs.com/puppet/what-is-puppet>`__ is IT
automation software that helps system administrators manage
infrastructure throughout its life-cycle, from provisioning and
configuration to orchestration and reporting. Using Puppet, you can
easily automate repetitive tasks, quickly deploy critical applications,
and proactively manage change, scaling from 10s of servers to 1000s,
on-premise or in the  cloud.

Puppet provides an option allowing for support of `external
CAs <http://docs.puppetlabs.com/puppet/3/reference/config_ssl_external_ca.html>`__.
Internal Puppet CA must be disabled, if external CA is configured.
External CA must provide its own system for issuing and distributing
certificates as internal Puppet public key infrastructure is disabled.

`IPA <http://www.freeipa.org/page/Main_Page>`__ with its clients and
`certmonger <certmonger>`__ can provide issuance and distribution of
certificates for Puppet.

The main requirement is to avoid modifications of Puppet code base,
which limits options for IPA-Puppet integration methods. A simple
replacement of Puppet CA with IPA CA was
`done <http://jcape.name/2012/01/16/using-the-freeipa-pki-with-puppet/>`__
in the past, but it failed to distinguish Puppet certificates from other
certificates issued by IPA.

.. _use_cases106:

Use Cases
---------

A service like Puppet wants to use public key infrastructure provided by
IPA CA.

Design
------

The internal Puppet CA can be replaced with the IPA CA. Currently,
Puppet is using
`mod_ssl <http://httpd.apache.org/docs/current/mod/mod_ssl.html>`__ to
provide
`SSL/TLS <http://httpd.apache.org/docs/current/ssl/ssl_howto.html>`__
connection between `Puppet master and its
clients <http://projects.puppetlabs.com/projects/1/wiki/certificates_and_security>`__.
Puppet's
`mod_ssl <http://httpd.apache.org/docs/current/mod/mod_ssl.html>`__
could be configured to use
`SSLRequire <http://httpd.apache.org/docs/2.2/mod/mod_ssl.html#sslrequire>`__
or
`Require <http://httpd.apache.org/docs/current/mod/mod_authz_core.html#reqexpr>`__
`expr <http://httpd.apache.org/docs/current/expr.html>`__ directives to
identify its certificates based on the presence of additional subject
name components. This is relatively weak method of authentication but it
eliminates requirement for Puppet modifications.

IPA needs to provide support for multiple certificate profiles in order
to issue certificates with different sets of subject name components.

Subject names in certificates issued currently by IPA CA are based on
the following pattern:

-  CN=, O=

Assuming multiple Puppet deployments within IPA, Puppet certificates can
be distinguished by adding OU= as a component of their subject names. In
such case subject names of Puppet certificates issued by IPA-CA would
look as follows:

-  CN=, OU=, O=

.. _certmonger_considerations_for_issuing_puppet_certificates_by_ipa_ca:

Certmonger considerations for issuing Puppet certificates by IPA CA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  `certmonger <certmonger>`__ provides -T option allowing to specify
   certificate profile name, which could be used to include Puppet's
   certificate profile name in requests for Puppet entities.
-  `certmonger <certmonger>`__ provides -N option allowing to specify
   subject name, which could be used in case of multiple Puppet
   deployments within IPA supporting only two profiles.

   -  -N option will allow to include OU component in the following way:

      -  -N 'CN=`hostname --fqdn`, OU=' -T

.. _considerations_for_mod_ssl_sslrequire_and_require_expr_directives:

Considerations for `mod_ssl <http://httpd.apache.org/docs/current/mod/mod_ssl.html>`__ `SSLRequire <http://httpd.apache.org/docs/2.2/mod/mod_ssl.html#sslrequire>`__ and `Require <http://httpd.apache.org/docs/current/mod/mod_authz_core.html#reqexpr>`__ `expr <http://httpd.apache.org/docs/current/expr.html>`__ directives
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sample configuration of mod_ssl including SSLRequire directive is
included in:

-  http://httpd.apache.org/docs/current/ssl/ssl_howto.html

Description of SSLRequire directive is provided in:

-  http://httpd.apache.org/docs/2.2/mod/mod_ssl.html#sslrequire

The latest description is provided in:

-  http://httpd.apache.org/docs/current/mod/mod_ssl.html#sslrequire

In Puppet case, SSLRequire directive can be used as follows:

-  SSLRequire %{SSL_CLIENT_S_DN_OU} eq ""
-  SSLRequire %{SSL_CLIENT_S_DN_OU} in {""}

In the latest version of mod_ssl, SSLRequire is deprecated and should in
general be replaced by Require expr. The so called ap_expr syntax of
Require expr is a superset of the syntax of SSLRequire, with few
exceptions:

-  Require expr %{SSL_CLIENT_S_DN_OU} eq ""
-  Require expr %{SSL_CLIENT_S_DN_OU} in {""}

.. _considerations_for_certificate_request_handling:

Considerations for certificate request handling
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

IPA is currently using combination of *virtual operations*, *ACI*,
*managedby*, and *Get Effective Rights* to determine whether entity
certificate requests can be approved and certificates issued. Adding
support for multiple certificate profiles will extend request approval
process

IPA may need to manage default and/or allowed profile information either
per each entity or entity pointed by *managedby* reference.

Here are some sample suggestions for profile processing:

-  IPA receives certificate request from registered entity. It performs
   usual certificate request verifications. If request does not include
   certificate profile name, then IPA verifies if requesting entity has
   information about *default profile*.

   -  if yes, then IPA adds default profile to request passed to CA
   -  If not, IPA checks if *default profiles* is inherited from entity
      pointed by *managedby* or *memberof* reference

      -  if yes, then IPA adds default profile to request passed to CA
         otherwise IPA's default profile is passed to CA

-  IPA receives certificate request from registered entity. It performs
   usual certificates request verifications. If request includes
   certificate profile name, then IPA verifies if requesting entity has
   information about *allowed profiles*

   -  if yes, then it verifies if requested profile is allowed
   -  If not, IPA verifies if *allowed profiles* are inherited from
      entity pointed by *managedby* or *memberof* reference

      -  if yes, then it verifies if requested profile is allowed
         otherwise IPA assumes that only IPA's default profile is
         allowed

.. _certificate_profile_considerations_for_issuing_puppet_certificates_by_ipa_ca:

Certificate profile considerations for issuing Puppet certificates by IPA CA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

IPA is required to support at least two certificate profiles in case of
multiple Puppet deployments within IPA, There are two possible profile
solutions:

-  IPA supports separate profiles for each Puppet deployment and one
   extra profile for all other services.

   -  Pros:

      -  No need for IPA to inject OU component into request passed to
         CA

   -  Cons:

      -  CA does not replicate profile operations at this moment
      -  CA may need to extend handling for name components

-  IPA supports two profiles, one profile for all Puppet deployments and
   second profile for all other services. This solution requires IPA CA
   to accept OU name component provided by either IPA or certificate
   requesting entity .

   -  Pros:

      -  No need to replicate profile operations

   -  Cons:

      -  IPA needs to inject OU component into request passed to CA
      -  CA needs to handle OU component injected into request by IPA
      -  Such handling of OU subject name components is currently
         unavailable in both IPA and CA

In case of supporting two profiles, OU name component might be derived
from *fqdn* name part of *managedby* reference attribute of requesting
entity.

Implementation
--------------

The implementation details will be defined base on elected solution.



Feature Management
------------------

UI
~~

TBD

CLI
~~~

TBD



Major configuration options and enablement
------------------------------------------

There might be quite a bit of manual configuration required related to
Puppet's pem-files locations, mod_ssl configuration modifications,
extending the list of IPA profiles, and replication of profile list
extension.

Replication
-----------

Replication of profile configuration updates and profile operations
might be required In case of supporting separate profiles for each
Puppet deployment, Initially this could be provided as a manual
procedure.



Updates and Upgrades
--------------------

There might be schema extensions required adding attributes storing
*default profile* and *allowed profiles*. This extension may have impact
on updates and upgrades.

Dependencies
------------

IPA as external Puppet CA does not require any new packages or
libraries.



External Impact
---------------

Development of this functionality will require working closely with the
Dogtag development team. There might be new handling of certificate
requests on IPA and CA sides, which may require extension of
corresponding interfaces.



Backup and Restore
------------------

IPA as external Puppet CA does not require any additional backup or
restore procedures. Regular IPA backup or restore procedure should also
cover this new feature.



Test Plan
---------

TBD



RFE Author
----------

Andrew Wnuk
