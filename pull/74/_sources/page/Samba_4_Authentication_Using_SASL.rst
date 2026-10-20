Samba_4_Authentication_Using_SASL
=================================

Overview
========

Currently when used with Fedora DS, Samba communicates with the backend
anonymously. This needs to be fixed such that Samba uses SASL
authentication.

Patches
=======



DS Patches
----------

-  `Config schema not included in core
   schema <https://bugzilla.redhat.com/show_bug.cgi?id=520921>`__
-  `SASL IO sometimes loops with "error: would
   block" <https://bugzilla.redhat.com/show_bug.cgi?id=526319>`__



Samba Patches
-------------

The patch has been committed in these revisions:

-  `s4: Use SASL authentication against Fedora
   DS <http://gitweb.samba.org/?p=samba.git;a=commit;h=b1dabb11333a715b0e23e91eecaf29933ea383a7>`__
-  `s4:provision Don't reference provision_backend when using
   LDB <http://gitweb.samba.org/?p=samba.git;a=commit;h=22c4ffa398a4c4855f79c36e75fdf467cdd47184>`__
-  `s4:auth - fixed problem reading bind DN from secrets
   database <http://gitweb.samba.org/?p=samba.git;a=commit;h=180ca8ed881593e08c291b504e26ea7b8adf7705>`__

`Category:Obsolete <Category:Obsolete>`__