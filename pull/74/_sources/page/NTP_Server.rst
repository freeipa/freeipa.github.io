NTP_Server
==========

`Kerberos <Kerberos>`__ protocol is very
`sensitive <Troubleshooting#Authentication.2FKerberos>`__ to proper time
synchronization between KDC and authentication nodes. With time
difference more than several minutes, authentication will not work. For
this reason, the server installers configure NTP server (ntpd at the
moment) on every FreeIPA server.



Known NTP services
------------------

Client and server installers detect 2 time synchronization services -
`ntpd <http://www.ntp.org/>`__ and
`chrony <http://chrony.tuxfamily.org/>`__. While on the FreeIPA server
ntpd is required as chrony only provides NTP client services, the
`Client <Client>`__ accepts already configured chrony. In `future
versions <https://fedorahosted.org/freeipa/ticket/4669>`__, chronyd
configuration should be
`preferred <http://fedoraproject.org/wiki/Features/ChronyDefaultNTP>`__.