On CentOS 7 (presumably on Fedora 19+ and RHEL 7) OpenLDAP is packaged
with it's own cert type, slapd_cert_t. This appears to be unrelated to
cert_t, as selinux forbids certmonger access. I submitted a bug report
to CentOS here: http://bugs.centos.org/view.php?id=7458 to allow
certmonger access. However, this does highlight the fact that there's
more than one selinux type for certificates. Should this be included
somehow? Is it relatively easy to see how many different cert types are
defined by the targeted policy that ships with Fedora/RHEL?

--`bnordgren <User:Bnordgren>`__ (`talk <User_talk:Bnordgren>`__) 17:04,
4 August 2014 (GMT)
