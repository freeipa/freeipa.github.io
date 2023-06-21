Certificate_Request_Queues
==========================

Overview
========

With the arrival of `V4/Certificate
Profiles <V4/Certificate_Profiles>`__ and `V4/Sub-CAs <V4/Sub-CAs>`__,
we will initially be issuing certificates automatically provided the
certificate request is permitted by the ACIs. This design proposal adds
the ability to work with certificate profiles that do not automatically
issue certificates, instead enqueuing requests for manual processing. It
adds commands to list, review, approve or reject certificate requests,
and is primarily an interface to underlying Dogtag capabilities.



Use Cases
=========



Review and approval of certificate requests
-------------------------------------------

Once user certificates are allowed via profiles, the next RFE is likely
to be a queue management system so that certificates are not
automatically issued.

Design
======

Dogtag already has the capability to enqueue, review, approve, deny or
delete certificate requests, in cases where certificates are not
automatically issued. FreeIPA will expose a subset of Dogtag's
capabilities for managing these queues.

Existing ACIs will be used to control which entities can issue requests
to which CAs, using which profiles.

Implementation
==============

The ``ipa cert-request`` command currently assumes that it will get a
certificate back. It will need to be updated to handle the case where a
request gets enqueued and provide useful feedback to the user. It should
also prominently display the request identifier that can be used with
the new ``ipa certrequest-*`` commands (see below).



Feature Management
==================

UI

**TODO**

CLI



``ipa certrequest-find <cahandle>``
----------------------------------------------------------------------------------------------

Search for or list pending certificate requests for the given CA.



``ipa certrequest-show <cahandle> <requestId>``
----------------------------------------------------------------------------------------------

Show detail of the given certificate request.



``ipa certrequest-approve <cahandle> <requestId>``
----------------------------------------------------------------------------------------------

Approve the certificate request, resulting in certificate issuance.



``ipa certrequest-reject <profileId>``
----------------------------------------------------------------------------------------------

Reject the certificate request.

Upgrade
=======

No upgrade procedures are required.



How to Test
===========



Test Plan
=========

Dependencies
============

-  `V4/Sub-CAs <V4/Sub-CAs>`__

Author
======

Fraser Tweedale

Email
   ftweedal@redhat.com
IRC
   ftweedal