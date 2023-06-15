Gist
====

Have an inheritance hierarchy where each class extends the set of Knobs
(installer options) from the inherited class. Each class containing a
set of Knobs should also be able to process them.

Rationale
=========

In the current state of installers, there's a lot of copy-pasted code
and any change in one place needs to be propagated to all the places
where the same code resides. Also, same options are usually processed in
different ways (as different people wrote it) but the resulting state
should be the same.

This design tries to solve the current situation by consolidating the
options and their processing to one common spot to avoid duplicate code
and different approaches to working with the same options, which is
obviously more error-prone than this general approach.

Implementation
==============

There's going to be an inheritance hierarchy of classes from the most
general one containing all the common options for all installers to more
and more specific classes for each of the installers.

.. _class_hierarchy:

Class hierarchy
---------------

The below figure shows the implemented class hierarchy. Find the
description of the most important classes below:

- The ServiceInstallInterface class is the base class for all the
installers. It contains the most general options (knobs) that are
usually required for any service installer to have - domain, realm,
hostname, replica file etc. - The HostNameInstaller class contains
general hostname and IP related options - ip-addresses, no-host-dn,
no-wait-for-dns. - ServiceAdminInstallInterface serves as a class for
installers that have use for principal/admin-password options.
ConnCheckInterface extends this class with skip-conncheck option as
installers with principa/admin-password usually perform connection
checks. - DogtagInstallInterface serves as a base for installs based on
Dogtag which require ca-file option. This class is then extended by
CAInstallInterface and KRAInstallInterface for CA and KRA installation
specific options

.. figure:: Installers_class_hierarchy.svg
   :alt: Installers_class_hierarchy.svg

   Installers_class_hierarchy.svg

.. _limiting_options_scope:

Limiting options scope
----------------------

Some classes may be too general and contain options that may only appear
in certain branches of the inheritance (e.g. the '--external-ca' option
from the CAInstallInterface class should only be available to
master-server installers). For this purpose, there's a set of decorators
that allows some of the options to only appear in certain set of
installers, excluding them from the others.

We have a list (extensible) of option-scope limiting decorators:

| ``prepare_only``
| ``enroll_only``
| ``master_install_only``
| ``replica_install_only``

These decorators are applied on class variables to limit the scope of
where (in which classes) should these variables (containing knobs)
appear. For these restrictions to apply during inheritance, it's
required to use a set of functions that reflects these restrictions on a
given class. This set is (obviously):

| ``prepares``
| ``enrolls``
| ``installs_master``
| ``installs_replica``

It takes a given class as an argument and returns a subclass with any
attributes that should not appear in it removed.

Example usage: Suppose we want to have a class that has some options but
it makes sense to have '--external-ca' option there while classes that
implement replication installers interface also inherit from this class.
Defining such a class:

| ``class MyClass(ServiceInstallInterface):``
| ``    ...``
| ``    external_ca = knob(``\ ``)``
| ``    external_ca = master_install_only(external_ca)``
| ``    ...``

Then, to create a class that inherits from MyClass and should serve
later to install some service as a master:

| ``class MasterInstallClass(installs_master(MyClass), ``\ ``):``
| ``    pass``



Backward Compatibility
----------------------

Backward compatibility should be implemented on the very lowest level of
the class hierarchy. This means that backward-compatible options are
implemented in each special class for each installer (e.g. for replica
installation, this is done in CompatServerReplicaInstal class which
inherits from ServerReplicaInstall).

.. _future_plan:

Future plan
===========

For the future, the plan is for the master/replica installers to unify
the order of services installed (already done in replica-install domain
levels 0/1). This should eventually lead to independence of which
services are installed when as long as there's a minimal base.

After the above has been done, it should be possible to make installers
idempotent meaning the same installer can be run with different options
so that only the additional services get installed (e.g. running
ipa-replica-install --setup-ca the first time, then running
ipa-replica-install --setup-kra --setup-dns later to install missing
services).

Another aim depending on the above is to also be able to install
services on different hosts. That should be very benefitial with
containers in mind so that IPA can actually follow the idea that each
container should encapsulate only one process at a time.

The implemented class hierarchy should be a tool for all the above
goals. It should be further improved so that it not only performs as an
option-unifying store but also implements the behavior connected with
the options it encapsulate
