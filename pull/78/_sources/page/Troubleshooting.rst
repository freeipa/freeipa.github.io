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
-  or by filing an issue to `FreeIPA Codeberg ticketing
   system <https://codeberg.org/freeipa/freeipa>`__ or `Red Hat
   Bugzilla <https://bugzilla.redhat.com/enter_bug.cgi?bug_status=NEW&component=freeipa&form_name=enter_bug&product=Fedora>`__.

Before we dive into particular scenarios we offer you a presentation about
:download:`FreeIPA troubleshooting principles <FreeIPA_Architecture_and_Troubleshooting.odp>`.



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
   below or the ones mentioned on the
   :doc:`Files to be attached to bug report </page/Files_to_be_attached_to_bug_report>` page should be
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

.. toctree::
   :maxdepth: 1
   :titlesonly:

   Installation </page/Troubleshooting/Installation>
   Directory Server </page/Troubleshooting/Directory_Server>
   Kerberos authentication, including trusts </page/Troubleshooting/Kerberos>
   DNS and DNSSEC </page/Troubleshooting/DNS>
   PKI (CA and KRA) </page/Troubleshooting/PKI>
   Administration framework and Web UI </page/Troubleshooting/Administration_and_Web_UI>
   Integration with Active Directory </page/Active_Directory_trust_setup>
   Privilege separation </page/Troubleshooting/PrivilegeSeparation>
   Integration with other software </page/Troubleshooting/Integration>

-  `Troubleshooting client-side issues with
   SSSD <https://sssd.io/troubleshooting/basics.html>`__
