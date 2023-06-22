.. _enhancements_addressed_in_the_current_release:

Enhancements Addressed in the Current Release
---------------------------------------------

+----------------------+----------------------+----------------------+
| Enhancement          | Comment              | Design document      |
+======================+======================+======================+
| http                 | Added crond as a     | ht                   |
| s://fedorahosted.org | default HBAC service | tp://www.freeipa.org |
| /freeipa/ticket/3215 |                      | /page/V2/PAMServices |
|                      |                      | was updated          |
+----------------------+----------------------+----------------------+
| http                 | When deleting a      | N/A                  |
| s://fedorahosted.org | replica from the     |                      |
| /freeipa/ticket/2879 | replication          |                      |
|                      | topology, check to   |                      |
|                      | see if this replica  |                      |
|                      | contains the only    |                      |
|                      | CA/DNS. If so,       |                      |
|                      | refuse to delete the |                      |
|                      | last CA, and warn if |                      |
|                      | trying to delete the |                      |
|                      | last DNS.            |                      |
+----------------------+----------------------+----------------------+
| http                 | Refuse to install    | N/A                  |
| s://fedorahosted.org | server with SELinux  |                      |
| /freeipa/ticket/3359 | disabled unless      |                      |
|                      | --al                 |                      |
|                      | low-selinux-disabled |                      |
|                      | option was used.     |                      |
|                      | Client installs      |                      |
|                      | display warning      |                      |
|                      | message if SELinux   |                      |
|                      | is installed but     |                      |
|                      | disabled.            |                      |
+----------------------+----------------------+----------------------+
| http                 | Add a --no-lookup    | N/A                  |
| s://fedorahosted.org | flag to              |                      |
| /freeipa/ticket/3524 | ipa-replica-manage   |                      |
|                      | to disable host      |                      |
|                      | existence checks.    |                      |
+----------------------+----------------------+----------------------+
| http                 | Add a --force flag   | N/A                  |
| s://fedorahosted.org | to force removal of  |                      |
| /freeipa/ticket/3787 | an empty ID range    |                      |
|                      | belonging to an      |                      |
|                      | active trust.        |                      |
+----------------------+----------------------+----------------------+
| http                 | Add 'debug' option   | N/A                  |
| s://fedorahosted.org | to nsupdate call in  |                      |
| /freeipa/ticket/3629 | ipa-client-install.  |                      |
|                      | This makes debugging |                      |
|                      | easier.              |                      |
+----------------------+----------------------+----------------------+
| http                 | Add                  | N/A                  |
| s://fedorahosted.org | '-                   |                      |
| /freeipa/ticket/3740 | -automount-location' |                      |
|                      | option to            |                      |
|                      | ipa-client-install.  |                      |
|                      | Using this option    |                      |
|                      | invokes              |                      |
|                      | ipa-client-automount |                      |
|                      | at the end of        |                      |
|                      | ipa-client-install.  |                      |
+----------------------+----------------------+----------------------+
|                      |                      |                      |
+----------------------+----------------------+----------------------+
