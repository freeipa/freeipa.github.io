Test_Plan
=========



Test Plan - Certificate Profiles
================================



Test Plan - CA ACLs
===================



Complex test cases - notes
==========================

1. test the store option. To do this, issue a certificate.

| ``   1.1 store=true``
| ``   1.2 store=false``

how does this work?

2. Disable/enable of an ACL

| ``   2.1 Enabled ACL for a particular profile``
| ``   2.2 Disabled ACL``

3. Issue certificate with a custom profile (s-mime)

| ``   3.1 Make sure the certificate is issued``
| ``   3.2 Check the certificate extensions against the CSR``
| ``   ``

4. Update an existing profile

| ``   4.1 Update a profile with new data``
| ``   4.2 Sign a certificate with the profile. Make sure it doesn't fail.``
| ``   4.3 Check if the generated certificate matches constraints set by the updated profile.``

`Category:FreeIPA V4 Test Plan <Category:FreeIPA_V4_Test_Plan>`__
`Category:FreeIPA Test Plan <Category:FreeIPA_Test_Plan>`__