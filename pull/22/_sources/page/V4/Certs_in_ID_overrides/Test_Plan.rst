Test_Plan
=========

Overview
========

Idviews are placeholders for storing external user identities (coming
from the Active Directory domains with which a trust is established). By
default, when a Trust is established, no user records are created for AD
users in the IPA. If an admin wants to setup per-user attributes for AD
users such as public ssh keys or ssl certificates, he needs to
explicitly create an idoverrideuser entity for the external user in
either the default idview for trust, "Default Trust View", or some
custom idview. These entities can then be used to store keys and certs.



Test Plan
=========

\|setup=

#. Setup ipa master and create a trust with existing AD.

#. Create an ID view in IPA and add an AD user. Make sure the id view is
   applied to ipa master host

#. Create a new certificate profile for users:

   ::

      ipa certprofile-show caIPAserviceCert --out=caIPAuserCert.txt

   ::

      sed -i "s/profileId=caIPAserviceCert/profileId=caIPAuserCert/" caIPAuserCert.txt

   ::

      ipa certprofile-import caIPAuserCert --file=caIPAuserCert.txt --store=True

#. Create a certificate database folder and a password file:

   ::

      mkdir certs

   ::

      touch certs/pwd

#. Generate a new certificate for the AD user

   ::

      certutil -d certs -N -f

   ::

      certutil -S -s "cn=testuser,dc=ad,dc=test" -n MyCert -x -t "CT,C,C" -v 120 -m 1234 -d certs -f certs/pwd

   ::

      certutil -L -d certs -n MyCert -a > mycert.crt

#. Repeat previous step to generate one more certificate for the same
   user

\|actions=

#. Create an idoverrideuser for AD user:

   ::

      ipa idoverrideuser-add "Default Trust View" testuser@%ad.domain_name%

#. Add a certificate you created during step 5 of the Setup to this
   idoverrideuser:

   ::

      ipa idoverrideuser-add-cert 'Default Trust View' testuser@%ad.domain_name% --certificate="$(openssl x509 -outform der -in mycert.crt | base64 -w 0)"

#. Try to add the same cert again to the same user

#. Add second certificate to the same idoverrideuser.

#. Remove this cert from the user

   ::

      ipa idoverrideuser-remove-cert %username% --certificate="$saved_certificate_text"

#. Remove the first certificate as well

\|results=

#. The step should succeed

#. The step should succeed

#. The step should fail

   ::

      ipa: ERROR: 'usercertificate;binary' already contains one or more values

#. The step should succeed

#. The step should succeed

#. The step should succeed

}}

`Category:FreeIPA V4 Test Plan <Category:FreeIPA_V4_Test_Plan>`__
`Category:FreeIPA Test Plan <Category:FreeIPA_Test_Plan>`__