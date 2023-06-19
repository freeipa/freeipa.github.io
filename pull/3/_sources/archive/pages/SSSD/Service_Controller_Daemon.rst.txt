Overview
--------

The System Security Services Daemon (the SSSD) is actually a collection
of different services and daemons running in the system context. These
services handle a variety of authentication and authorization functions.
As a result, it is highly imperative that these services are fully
available and functional at all times. The continued outage of any
service provided by the SSSD may render the machine unusable. To avoid
this, the SSSD will provide a special Monitor daemon that will maintain
the lifecycle of all other SSSD services.

.. _main_functions:

Main Functions
--------------

The Service Controller will be in charge of:

-  Starting of the SSSD services at launch or the startup of the system
-  Stopping of the SSSD services when it shuts down or at the shutdown
   of the system
-  Monitor that all the SSSD processes that have to stay alive stay
   alive
-  Restart any SSSD processes that have exited or crashed

Additional goals of the Monitor daemon are as follows:

-  It must run on all supported client platforms.
-  It should not require the installation of any dependencies that are
   not installed by default on the supported client platforms.

.. _internal_logic:

Internal Logic
--------------

Configuration
~~~~~~~~~~~~~

The Monitor service for the SSSD will maintain its configuration within
an LDB database. Available configuration options for the Monitor will
be:

-  Enabled/Disabled services

   -  Arguments and options to be passed to the services

-  Number of retry attempts for failed service start
-  Service polling interval

   -  Service polling failure tolerance

-  D-BUS monitor socket

For further detail, see the Configuration Store section below.

.. _service_control:

Service Control
~~~~~~~~~~~~~~~

At startup, the Monitor service will fork off each of the enabled SSSD
services into their own, managed child process. Each child process will
connect via private D-BUS communication to the configured D-BUS monitor
socket. The monitor service will then challenge the client connection to
identify itself by service name and version.

If the child service goes down (see Service Monitoring), the Monitor
service will attempt to restart the service with the arguments and
options specified in the configuration database.

If the configuration of a client service changes, the Monitor will send
a private D-BUS message to the process, triggering a service update.
This may be an alteration of a particular option to the service, or it
may trigger the service to shut down completely.

.. _service_monitoring:

Service Monitoring
~~~~~~~~~~~~~~~~~~

The child services will be monitored in two ways.

#. At a configured interval, the client service will be challenged with
   a simple ping. If the service fails to respond to the configured
   number of pings (the failure tolerance), it will be marked as
   crashed/hung and restarted by the Service Control.
#. All SSSD services will maintain a connection to the Monitor over the
   private D-BUS socket. If the service crashes or is killed without the
   Service Manager's knowledge, this communication will be broken and
   the Monitor will immediately attempt to relaunch the affected
   service.

Appendix
--------

.. _configuration_store:

Configuration Store
~~~~~~~~~~~~~~~~~~~

The configuration store can will be represented by LDB which give a
flexibility of the memory mapped LDAP. The structure of the
configuration store will be represented thusly:

::

   cn=config
   |
   ----cn=services,cn=config
       |
       ----cn=monitor,cn=services,cn=config
       |   dbusSocket: /var/lib/sss/dbus/monitor
       |   ActiveServices: nss, pam
       |
       ----cn=nss,cn=services,cn=config
       |   unixSocket: /var/lib/sss/pipes/nss
       |
       ----cn=pam,cn=services,cn=config
           unixSocket: /var/lib/sss/pipes/pam

Further configuration options pending.

.. _d_bus_methods:

D-BUS Methods
~~~~~~~~~~~~~

Services
^^^^^^^^

The following D-BUS methods shall be exposed by all SSSD services
managed by the Monitor:

getIdentity
   INPUT:
   OUTPUT: STRING name, UINT16 version

ping
   INPUT:
   OUTPUT:
