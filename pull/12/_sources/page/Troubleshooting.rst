Troubleshooting
===============

This document should help FreeIPA users who are trying to troubleshoot
why their setup is not working as expected. After following the steps
and advises described in this article, users should be able to either
**fix the configuration themselves** or **provide the right
information** for developers/support to investigate and advise or to fix
the issue.

Feedback is expected to be sent either to:

-  `freeipa-users mailing
   list <https://lists.fedorahosted.org/archives/list/freeipa-users@lists.fedorahosted.org/>`__
-  or by filing an issue to `FreeIPA Pagure ticketing
   system <https://pagure.io/freeipa/new_issue>`__ or `Red Hat
   Bugzilla <https://bugzilla.redhat.com/enter_bug.cgi?bug_status=NEW&component=freeipa&form_name=enter_bug&product=Fedora>`__.

Before we dive into particular scenarios we offer you presentation about
`FreeIPA troubleshooting
principles <media:FreeIPA_Architecture_and_Troubleshooting.odp>`__.



Reporting bugs
--------------

Providing the right information in a report, regardless whether it is
filed by mail or in bug tracking system will help FreeIPA developers
quickly identify the root cause of an issue and provide help.

The following information is expected to be passed in the report:

-  **Version and distribution**: we need to know the version (and
   distribution) of the FreeIPA you are running on. If other components
   (like NFS, sudo) are involved, their versions may be useful as well.
-  **Problem description**: What happened, in what part/component of
   FreeIPA it happened, how severe is the issue. The problem description
   should help us to either *reproduce* the problem or *identify* the
   faulty component.
-  **What have you tried**: transcript of what you already did to
   investigate the issue
-  **Logs**: Related (and sanitized) logs are essential source of
   information. At least the logs mentioned in the different scenarios
   below or the ones mentioned on `Files to be attached to bug
   report <Files_to_be_attached_to_bug_report>`__ page should be
   included.

If you want to read some theory about bug reporting we recommend you the
famous article `How to Report Bugs
Effectively <http://www.chiark.greenend.org.uk/~sgtatham/bugs.html>`__.



Troubleshooting scenarios
-------------------------

FreeIPA consists of many integrated technologies and components.
Therefore, investigation of issues occurring in one part of FreeIPA will
take different path and steps from investigation of issues in other
part. The same applies for the list of information expected to be
provided. Please consult the dedicated troubleshooting page for the
relevant aspect or component of FreeIPA:

-  `Installation <Troubleshooting/Installation>`__
-  `Directory Server <Troubleshooting/Directory_Server>`__
-  `Kerberos authentication, including
   trusts <Troubleshooting/Kerberos>`__
-  `DNS and DNSSEC <Troubleshooting/DNS>`__
-  `PKI (CA and KRA) <Troubleshooting/PKI>`__
-  `Administration framework and Web
   UI <Troubleshooting/Administration_and_Web_UI>`__
-  `Integration with Active Directory <Active_Directory_trust_setup>`__

   -  `Privilege separation <Troubleshooting/PrivilegeSeparation>`__

-  `Integration with other software <Troubleshooting/Integration>`__
-  `Troubleshooting client-side issues with
   SSSD <https://sssd.io/troubleshooting/basics.html>`__