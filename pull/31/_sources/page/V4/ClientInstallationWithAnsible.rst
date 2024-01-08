ClientInstallationWithAnsible
=============================

Overview
--------

Automation of installation and configuration tasks is an important part
of the sysadmin's responsibilities.
`Ansible <http://docs.ansible.com/ansible/latest/index.html>`__ is a
powerful engine providing automation, that is becoming widely used, and
can allow to avoid the repetition of manual tasks.

In this context, IPA client deployment is a perfect candidate for
automation through Ansible: when the sysadmin needs to configure
additional IdM clients, Ansible can be used to install the IdM packages
and configure IdM client on the nodes with the same set of options.

The goal of this project is to provide Ansible
`modules <http://docs.ansible.com/ansible/latest/modules.html>`__ and
`roles <http://docs.ansible.com/ansible/latest/playbooks_roles.html>`__
allowing the automation of IdM client configuration.



Use Cases
---------



IPA client configuration
----------------------------------------------------------------------------------------------

A sysadmin has already deployed IdM on one or more servers, and wants to
configure one or more nodes as IdM clients using Ansible. The sysadmin
needs to define an `inventory
file <http://docs.ansible.com/ansible/latest/intro_inventory.html>`__
describing the IdM clients and the IdM server (hostname, domain name,
realm name...) and is able to call ansible either through an `Ansible
playbook <http://docs.ansible.com/ansible/latest/playbooks.html>`__ or
`Ansible ad-hoc
commands <http://docs.ansible.com/ansible/latest/intro_adhoc.html>`__ in
order to configure the IdM clients.

If a host defined as IdM client in the inventory is already configured,
Ansible will detect this and check that the domain and realm are
matching the expectations of the inventory file.

-  If it is the case, Ansible will skip the configuration of IPA client.
   Note: if the configuration options (except domain and realm) differ,
   the IPA client will not be reconfigured. This means that the solution
   will not be able to repair broken installations or modify
   installation options.
-  If the domain or realm is different, Ansible will exit on error for
   the specified host.

If the host defined as IdM client in the inventory is not configured yet
as IPA client, Ansible will configure IPA client.

This use case can be enhanced in the future, in order to allow:

-  repair of broken installations (by comparing what should be
   configured with the actual settings)
-  modification of current installation (for instance in order to re-run
   ipa-client-install but with a different set of options)

but the scope for this first release is limited to the configuration and
check of domain and realm consistency.

As an authorized user is required to join a client machine to IPA,
Ansible needs to be provided either with the keytab of a user allowed to
join hosts (which will be used to call ipa host-add and create a OTP for
the machine), or a kerberos principal + password.

The re-enrollment of a host is also supported by providing Ansible with
a backed up host keytab from previous enrollment.



IPA client unconfiguration
----------------------------------------------------------------------------------------------

A sysadmin has already deployed IPA client on a host (either using
Ansible or with the ipa-client-install command line), and wants to
un-enroll the host from IdM. The sysadmin needs to define an inventory
file containing the host to be unenrolled and is able to call ansible
either through an Ansible playbook or Ansible ad-hoc commands in order
to unconfigure the IdM client.

Design
------

The solution is implemented as Ansible modules and role:

-  a module "ipahost" allowing to perform the equivalent to ipa
   host-add/mod/del, supporting the --random option in order to create a
   One-Time Password used for host enrollment
-  a module "ipaclient" checking if the host is already configured for
   IPA and wrapping ipa-client-install. This module accepts a "state"
   parameter which can take "present" or "absent" values and will either
   configure or unconfigure IPA client depending on this value.
-  a role "ipaclient" combining the above modules and existing ansible
   modules (for instance the
   `package <http://docs.ansible.com/ansible/latest/package_module.html>`__
   module to install ipa-client package), accepting a "state" parameter
   with value either "present" or "absent".

Implementation
--------------



Ansible ipaclient module
----------------------------------------------------------------------------------------------

The ansible ipaclient module is a module performing the equivalent of
ipa-client-install. In this first version, it is written as a wrapper
calling ipa-client-install. It requires ipa-client packages to be
installed on the managed node.

Potential issues:

-  Compatibility of the Ansible modules with various versions of
   ipa-client: what is the requirement? What is the minimum supported
   version of ipa-client?

The module supports a "state" parameter allowing to either configure
(state=present) or unconfigure (state=absent) IPA client.

The module supports the Ansible check_mode, i.e. if called with -C or
--check, it will not perform any modification but simply return if
changes would need to be made.

==== state=present ====

In order to guarantee idempotency, the module first checks if ipa client
is already installed, by checking for the presence of
/etc/ipa/default.conf and
/var/lib/ipa-client/sysrestore/sysrestore.state.

-  If IPA client is already installed, it checks that the required
   domain and realm are matching the configured domain and realm (as
   read from the configuration in /etc/ipa/default.conf).

   -  If domain and realm are matching, the module exits successfully
      and reports that it did not perform any modification
      (changed=False).
   -  If domain or realm is inconsistent with the module parameters, the
      module exits on error

-  If IPA client is not installed, it calls ipa-client-install

The module doesn't support exactly the same set of parameters as those
supported by ipa-client-install. This allows to handle some parts of the
configuration with other Ansible modules. For instance, the ipaclient
module always calls ipa-client-install with the --no-ntp option and the
NTP configuration is done in the ipaclient role by calling existing
Ansible module for NTP.

This strategy will allow to gradually transfer the configuration of IPA
components to Ansible modules and benefit from their features (repair
config, update config...)

==== state=absent ====

The module first checks if ipa client is already installed.

-  If IPA client is installed, the module wraps a call to
   ipa-client-install --uninstall -U in order to unconfigure IPA client.
-  If IPA client is not installed, the module simply returns
   successfully and reports that it did not perform any modification
   (changed=False).

The uninstallation is possible for IPA clients installed with the
command line ipa-client-install as well as clients installed with
Ansible.



Ansible ipahost module
----------------------------------------------------------------------------------------------

Note: Ansible already provides `Identity
Modules <http://docs.ansible.com/ansible/latest/list_of_identity_modules.html>`__
for IPA, especially one for
`ipa_host <http://docs.ansible.com/ansible/latest/ipa_host_module.html>`__,
but these modules currently lack some features:

-  ipa_host module does not allow to create a random One-Time Password
-  all the IPA modules are authenticating to IPA server using principal
   + password and do not support keytabs
-  all the IPA modules are communicating with the IPA server using the
   remote JSON API instead of the Python API

These limitations argue in favor of a new ipahost module.

The ansible ipahost module is a module performing the equivalent of ipa
host-add/mod/del. It must be executed on an IPA host. It is needed
especially when the goal is to install a client using a One-Time
Password and allows to obtain the OTP.

It is written with an ansible Action Plugin which allows to prepare the
authentication, and a module executed on the managed node.

The action plugin is executed on the control node and:

-  creates a kerberos client configuration file using IPA server, stored
   in a temp directory
-  sets the environment variable KRB5_CONFIG on the control node to the
   temp kerberos config file
-  performs kinit on the control node, using a specific credential cache
   file in the temp directory, and specifying a limited lifetime
-  copies the credential cache file from the control node to the managed
   node in the Ansible temp directory used by the module
-  calls the module on the managed node by providing the path to the
   credential cache file

The module is executed on the managed node and:

-  sets the environment variable KRB5CCNAME to the credential cache file
-  uses IPA client API to add/mod/del the host.

When the module has finished its execution, the credential cache file is
automatically deleted on the managed node as Ansible removes the temp
directory used by the module.

The kinit can be done either with principal/password or with a keytab.
They are provided as arguments to the ansible module. The password or
the keytab are never transfered to the managed node, only the cache file
is.

The module can be called with the argument state=present or state=absent
to add or remove a host.

The module supports the Ansible check_mode, i.e. if called with -C or
--check, it will not perform any modification but simply return if
changes would need to be made.

==== state=present ====

When called with state=present (or state not defined), the module
ensures that the host is defined in IPA configuration: it performs the
equivalent of ipa host-show to check if the host is already defined.

-  if the host is already defined, it compares the host attributes with
   the ones specified by the module. If needed, it calls the equivalent
   of ipa host-mod.
-  if the host is not defined, it calls the equivalent of ipa host-add.

==== state=absent ====

When called with state=absent, the module ensures that the host is not
defined in IPA configuration: it performs the equivalent of ipa
host-show to check if the host is already defined.

-  if the host is already defined, it calls the equivalent of ipa
   host-del.
-  if the host is not defined, it returns successfully and reports that
   it did not perform any modification (changed=false).

It is desirable to be able to set an OTP on a host prior to calling the
client enrollment module so the client host entry will be in a state
ready for enrollment, even if this means re-enrollment. It cannot be
assumed that an enrolled host can unenroll itself to handle cases of a
broken or missing keytab.

IPA currently (as of IPA 4.6) prevents setting an OTP on an enrolled
host by a check in the host_mod pre callback. I'd propose a new option
to allow overriding this for backwards compatibility in case existing
integration relies on the ValidationError raised when setting an OTP on
an enrolled client to know the state. I'd propose --force.

Similarly the 389-ds ipa_enrollment plugin prevents enrolling an already
enrolled host in ipa_join(). This will be needed to be updated to handle
re-enrollment as well, particularly in the assumptions about what
objectclasses and attributes already exist in the host entry.



Ansible ipaclient role
----------------------------------------------------------------------------------------------

The ipaclient role takes a "state" parameter allowing to either
configure or unconfigure IPA client. It combines the installation of ipa
client packages (using the pre-existing Ansible module "package") and
the configuration of IPA client using the Ansible module ipaclient.



state: present
^^^^^^^^^^^^^^

When the role is called with the parameter "state: present" (or the
parameter state is not defined), the role performs IPA client
configuration:

-  install the packages required by IPA client
-  if needed obtain a One-Time Password for enrolling the host using
   Ansible module "ipahost"
-  configure IPA client using the Ansible module "ipaclient"

The package names depend on the OS of the managed node: in RHEL the
package is named ipa-client while in Fedora freeipa-client. In order to
work with both OSes, it is possible to import variables specific to the
distribution. The role defines the ipaclient_package variable which will
have a default value of freeipa-client, or a value ipa-client for RHEL.

Question: which OS should be supported for the IPA client? fedora, rhel,
other distros?

IPA client enrollment can be performed using one of the 3 following
methods:

-  supply a principal and a password
-  supply a principal and a One-Time Password
-  supply a host keytab from previous enrollment



state: absent
^^^^^^^^^^^^^

When the role is called with the parameter "state: absent", the role
performs IPA client unconfiguration. The role is using the Ansible
module "ipaclient".

Question: should the role with state=absent also remove ipa-client
packages?



How to Use
----------

Example of inventory file:

| ``$ cat inventory/hosts``
| ``[ipaclients]``
| ``ipaclient1.example.com``
| ``ipaclient2.example.com``
| ``[ipaservers]``
| ``ipaserver.example.com``
| ``[ipaclients:vars]``
| ``ipaclient_domain=example.com``
| ``ipaclient_realm=EXAMPLE.COM``
| ``ipaclient_extraargs=[ '--kinit-attempts=3', '--mkhomedir']``
| ``# To enroll the IPAclient, the module needs either a host keytab from a previous enrollment,``
| ``# or a principal + password, or a One-Time Password``
| ``# If you wish to use principal + password, you need to provide ipaclient_principal and ipaclient_password:``
| ``ipaclient_principal=admin``
| ``ipaclient_password=MySecretPassword123``
| ``# If you wish to use a host keytab from a previous enrollment, you need to provide ipaclient_keytab:``
| ``#ipaclient_keytab=``
| ``# If you wish to use a One-Time Password, you need to select an auth method to IPA server``
| ``# (either using password or keytab) and need to provide``
| ``# - for password auth: ipaserver_principal and ipaserver_password:``
| ``#ipaserver_principal=``
| ``#ipaserver_password=``
| ``# - for keytab auth: ipaserver_principal and ipaserver_keytab:``
| ``#ipaserver_principal=``
| ``#ipaserver_keytab=``



Using the Ansible ipaclient module to unconfigure IPA
----------------------------------------------------------------------------------------------

Create a playbook calling the ipaclient module, with state=absent

| ``$ cat uninstall.yml``
| ``---``
| ``- name: Playbook to unconfigure IPA clients``
| ``  hosts: ipaclients``
| ``  become: true``
| ``  tasks:``
| ``  - name: Unconfigure IPA client``
| ``    ipaclient:``
| ``      state: absent``

Call the playbook

``$ ansible-playbook -i inventory/hosts uninstall.yml``



Using the Ansible role ipaclient to unconfigure IPA
----------------------------------------------------------------------------------------------

Create a playbook calling the ipaclient role, with state=absent

| ``$ cat uninstall.yml``
| ``---``
| ``- name: Playbook to unconfigure IPA clients``
| ``  hosts: ipaclients``
| ``  become: true``
| ``  roles:``
| ``  - role: ipaclient``
| ``    state: absent``

Call the playbook

``$ ansible-playbook -i inventory/hosts uninstall.yml``



Use the ipaclient module to configure IPA client by providing username and password
----------------------------------------------------------------------------------------------

Create a playbook calling the ipaclient module, with state=present. The
module takes principal and password as arguments.

| ``$ cat install.yml``
| ``---``
| ``- name: Playbook to configure IPA clients with username/password``
| ``  hosts: ipaclients``
| ``  become: true``
| ``  tasks:``
| ``  - name: Install IPA client package``
| ``    package:``
| ``      name: ipa-client``
| ``      state: present``
| ``  - name: Configure IPA client``
| ``    ipaclient:``
| ``      state: present``
| ``      domain: "{{ ipaclient_domain }}"``
| ``      realm: "{{ ipaclient_realm }}"``
| ``      principal: "{{ ipaclient_principal }}"``
| ``      password: "{{ ipaclient_password }}"``
| ``      extra_args: "{{ipaclient_extraargs }}"``

Call the playbook

``$ ansible-playbook -i inventory/hosts install.yml``

Note: Ansible provides a feature named `Ansible
Vault <http://docs.ansible.com/ansible/latest/playbooks_vault.html>`__
to avoid writing clear-text sensible data in playbooks. The sysadmin can
create a file containing the password:

| ``$ cat playbook_sensitive_data.yml``
| ``---``
| ``ipaclient_password: MySecretPassword123``

Then encrypt this file using ansible-vault command:

::

   | ``$ ansible-vault encrypt playbook_sensitive_data.yml``
   | ``New Vault password: ``
   | ``Confirm New Vault password: ``
   | ``Encryption successful``
   | ``$``

At this point, the file is encrypted and the variables it contains can
be used by the playbook:

::

   | ``[...]``
   | ``  hosts: ipaclients``
   | ``  become: true``
   | ``  ``\ **``vars_files:``**
   | ``  ``\ **``-``\ ````\ ``playbook_sensitive_data.yml``**
   | ``  - name: Configure IPA client``
   | ``    ipaclient:``
   | ``      state: present``
   | ``      domain: "{{ ipaclient_domain }}"``
   | ``      realm: "{{ ipaclient_realm }}"``
   | ``      principal: "{{ ipaclient_principal }}"``
   | ``      ``\ **``password:``\ ````\ ``"{{``\ ````\ ``ipaclient_password``\ ````\ ``}}"``**
   | ``      extra_args: "{{ipaclient_extraargs }}"``
   | ``[...]``

When the playbook is called, the password used to protect the file is
either supplied interactively:

::

   ``$ ansible-playbook -i inventory/hosts ``\ **``--ask-vault-pass``**\ `` install.yml``

or provided in a file:

::

   ``$ ansible-playbook -in inventory/hosts ``\ **``--vault-password-file``\ ````\ ``~/.vault_pass.txt``**\ `` install.yml``



Use the ipahost and ipaclient modules to configure IPA client by providing an OTP password
----------------------------------------------------------------------------------------------

Create a playbook calling ipahost module in order to create an OTP
password for the managed node, then calling ipaclient module to
configure IPA with the OTP password just obtained.

| ``$ cat install_otp.yml``
| ``---``
| ``- name: Playbook to configure IPA clients with an OTP password``
| ``  hosts: ipaclients``
| ``  become: true``
| ``  tasks:``
| ``  - name: For OTP client registration, add client and get OTP``
| ``    ipahost:``
| ``      state: present``
| ``      principal: "{{ ipaserver_principal }}"``
| ``      ``\ **``keytab:``**\ `` "{{ ipaserver_keytab }}"``
| ``      fqdn: "{{ ansible_fqdn }}"``
| ``      random: True``
| ``    register: ipahost_output``
| ``    delegate_to: "{{ groups.ipaservers[0] }}"``
| ``  - name: Configure ipaclient``
| ``    ipaclient:``
| ``      state: present``
| ``      domain: "{{ ipaclient_domain }}"``
| ``      realm: "{{ ipaclient_realm }}"``
| ``      ``\ **``otp:``**\ `` "{{ ipahost_output.host.randompassword }}"``
| ``      extra_args: "{{ ipaclient_extraargs }}"``

Call the playbook

``$ ansible-playbook -i inventory/hosts install_otp.yml``

Note: the ipahost module can also be called with a principal and
password instead of the admin keytab:

| ``  - name: For OTP client registration, add client and get OTP``
| ``    ipahost:``
| ``      state: present``
| ``      principal: "{{ ipaserver_principal }}"``
| ``      ``\ **``password:``**\ `` "{{ ipaserver_password }}"``
| ``      fqdn: "{{ ansible_fqdn }}"``
| ``      random: True``
| ``    register: ipahost_output``
| ``    delegate_to: "{{ groups.ipaservers[0] }}"``



Use the ipaclient role to configure IPA client
----------------------------------------------------------------------------------------------

Create a playbook calling ipaclient role.

| ``$ cat install_with_role.yml``
| ``---``
| ``- name: Playbook to install IPA clients``
| ``  hosts: ipaclients``
| ``  become: true``
| ``  roles:``
| ``  - role: ipaclient``
| ``    state: present``

Call the playbook

``$ ansible-playbook -i inventory/hosts install_with_role.yml``

The role is written to handle the various cases based on the content of
the inventory:

-  enrollment options:

   -  if ipaclient_principal and ipaclient_password are specified, the
      IPA client will be configured with the specified credentials
      (equivalent to ipa-client-install --principal ... --password ...)
   -  if ipaclient_keytab is specified, the IPA client will be
      configured with the specified keytab (equivalent to
      ipa-client-install --keytab ...)
   -  if neither ipaclient_password nor ipaclient_keytab is specified,
      the role will assume that OTP enrollment is desired and will
      register the host as client using ipahost module. The ipahost
      module requires authentication to the IPA server as described
      below.

-  if the ipahost module is needed to create a OTP, it will be delegated
   to the server and will use one of the following auth methods:

   -  use ipaserver_principal and ipaserver_password, or
   -  use ipaserver_principal and ipaserver_keytab



Test Plan
---------