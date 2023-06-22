Replica_Conncheck
=================

Overview
--------

If the firewall isn't properly configured and required ports aren't
reachable, the replica installation would fail unexpectedly with
inconvenient error messages. Replica conncheck addresses this issue by
testing whether the required ports are open.



Use Cases
---------

-  Before replica installation is started, the script is automatically
   run to verify whether required ports are open. The installation won't
   proceed if the ports are unreachable.
-  As a system administrator, I want to verify that all required ports
   are reachable without actually installing the replica.

Design
------

The following ports are required by FreeIPA:

-  80 tcp (http)
-  443 tcp (https)
-  389 tcp (ldap)
-  636 tcp (ldaps)
-  88 tcp+udp (kerberos)
-  464 tcp+udp (kpasswd)
-  7389 tcp (separate Dogtag instance - used on RHEL 6)

These ports have to be reachable on both master and replica. Therefore,
two checks are performed.

First, the replica checks that the ports are open on master. The check
skips udp ports, because KDC is already running on master and the check
would have to be performed protocol-wise.

Second, the conncheck running on replica starts to listen on these ports
and the master can check the ports. The check on master can be performed
automatically by an RPC call or an SSH connection. In this case the exit
code is checked to verify the check has passed. The command can also be
issued manually on master.

IPv4/IPv6
----------------------------------------------------------------------------------------------

The --master and --replica parameters support both hostnames and IP
addresses. If an IP address is provided, then it will check that
particular IP address. Conncheck works the same way if hostname was
provided and it resolves to a single IP address.

If hostname is provided which resolves to multiple IP addresses (either
IPv4, IPv6 or both), then every port has to be reachable on all of the
IP addresses.

Implementation
--------------

Class PortResponder handles binding and listening on ports. The socket
handling code is started in a separate thread. This is necessary because
the automated check performed afterwards is a blocking call. All the
sockets are handled in a single thread. Since they can be handled very
quickly, so there is no need to spawn additional threads.

Once the ports are bound, the next step is to perform a check from
master. This can be handled automatically. There are two ways to perform
this check. Either via an RPC call or through as SSH connection. During
an automated check, an RPC call is attempted at first. This requires
Kerberos credentials. The SSH connection is used as a fallback for older
versions.