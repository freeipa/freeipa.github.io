Overview
--------

Currently on FreeIPA `clients <Client>`__ the full list of
group-memberships for users from trusted domains is only available for
authenticated users. Authentication is needed because the
group-memberships are extracted from the PAC which is a part of the
`Kerberos <Kerberos>`__ ticket of the user. For un-authenticated users
only the primary POSIX group is available.

With the introduction of SSSD's ipa-server-mode IPA servers can resolve
the group memberships for user from trusted domains even for
un-authenticated users by reading them from the AD DCs of the trusted
domains directly. This data should be made available to the IPA clients
as well.



Use Cases
---------

-  Calling the 'id' command on a IPA client or calling getgrouplist() in
   a process should return the full list of secondary groups for any
   user from a trusted domain without requiring that the user has to log
   in first

Design
------

The IPA extdom plugin is already called by SSSD running on the IPA
clients to translate SID to names or POSIX IDs. It can be enhanced to
return the secondary groups as well.

To make this change backward compatible in the sense that newer client
can still work with older servers

-  a new OID can be added to the extdom plugin, currently it uses
   2.16.840.1.113730.3.8.10.4 , if we agree on this approach it might be
   possible to just add a ".1" to the OID. With this the client can
   determine from reading the rootDSE if the server supports this
   operation or not.
-  the client just sends the new request and assumes in the case of a
   specific error the request is not supported. A suitable error code
   might be LDAP_UNWILLING_TO_PERFORM which is currently only returned
   by the extdom plugin if either there is no request data or if there
   are issues reading and understanding the first enumeration in the
   request data (see Implementation section for more details).

If the request was parsed successful the extdom plugin will return the
secondary groups for the given user as a list of fully qualified group
names. Additionally the POSIX user data (UID, GID, gecos, home-directory
and shell) can be send. This would reduce the number of requests to the
extdom plugin, because refreshing user and group-membership data often
happen together. Please note that UID and GID may change due to
user-views. But as long as this feature is implemented after user-views
everything what is needed will be available.

Implementation
--------------

Request
~~~~~~~

The current request looks like this:

| ``/* We expect the following request:``
| `` * ExtdomRequestValue ::= SEQUENCE {``
| `` *    inputType ENUMERATED {``
| `` *        sid (1),``
| `` *        name (2),``
| `` *        posix uid (3),``
| `` *        posix gid (3)``
| `` *    },``
| `` *    requestType ENUMERATED {``
| `` *        simple (1),``
| `` *        full (2)``
| `` *    },``
| `` *    data InputData``
| `` * }``
| `` *``
| `` * InputData ::= CHOICE {``
| `` *    sid OCTET STRING,``
| `` *    name NameDomainData``
| `` *    uid PosixUid,``
| `` *    gid PosixGid``
| `` * }``
| `` *``
| `` * NameDomainData ::= SEQUENCE {``
| `` *    domain_name OCTET STRING,``
| `` *    object_name OCTET STRING``
| `` * }``
| `` *``
| `` * PosixUid ::= SEQUENCE {``
| `` *    domain_name OCTET STRING,``
| `` *    uid INTEGER``
| `` * }``
| `` *``
| `` * PosixGid ::= SEQUENCE {``
| `` *    domain_name OCTET STRING,``
| `` *    gid INTEGER``
| `` * }``
| `` */``

Basically only a new request type must be added, e.g.

| `` *    requestType ENUMERATED {``
| `` *        simple (1),``
| `` *        full (2)``
| `` *        full_with_groups (3)``
| `` *    },``

But as mentioned before the LDAP_UNWILLING_TO_PERFORM error code is only
send if there are issues with the inputType. The requestType is
evaluated later in the existing code and the quite generic
LDAP_OPERATIONS_ERROR error code is returned if an unexpected value is
found. This would make the detection based on error code much more
fragile. I would suggest if we do not want to use a new OID to indicate
the feature, that the inputType is extended as well, e.g. by adding 128
or 256 to the values. Nevertheless I think it is less error prone to use
a new OID.

.. _processing_the_request:

Processing the request
~~~~~~~~~~~~~~~~~~~~~~

The extdom plugin will call getgrouplist() and the resolve the fully
qualified names by calling getgrgid(). By default groups from the local
IPA domain are returned unqualified and the local domain name should be
added in this case so that the response only contains fully qualified
group names.

There are two things to note here. First getgrgid() returns the full
list of group members which might cause some unneeded overhead e.g. with
respect to memory allocation. Second easiest way to determine if a
domain name is fully qualified is to look for a '@' character. But this
will only work if the full_name_format option is not changed from the
default. Both can be fixed by adding a call to libsss_nss_idmap to map a
GID to user and domain name. If it turns out the such a call is needed
it can be added later and the extdom plugin can be updated accordingly.

Additionally the extdom plugin call getpwnam() with user-view code to
get the data of the POSIX user entry.

Response
~~~~~~~~

Currently the response looks like:

| ``/* We send to follwing response:``
| `` * ExtdomResponseValue ::= SEQUENCE {``
| `` *    responseType ENUMERATED {``
| `` *        sid (1),``
| `` *        name (2),``
| `` *        posix_user (3),``
| `` *        posix_group (4)``
| `` *    },``
| `` *    data OutputData``
| `` * }``
| `` *``
| `` * OutputData ::= CHOICE {``
| `` *    sid OCTET STRING,``
| `` *    name NameDomainData,``
| `` *    user PosixUser,``
| `` *    group PosixGroup``
| `` * }``
| `` *``
| `` * NameDomainData ::= SEQUENCE {``
| `` *    domain_name OCTET STRING,``
| `` *    object_name OCTET STRING``
| `` * }``
| `` *``
| `` * PosixUser ::= SEQUENCE {``
| `` *    domain_name OCTET STRING,``
| `` *    user_name OCTET STRING,``
| `` *    uid INTEGER``
| `` *    gid INTEGER``
| `` * }``
| `` *``
| `` * PosixGroup ::= SEQUENCE {``
| `` *    domain_name OCTET STRING,``
| `` *    group_name OCTET STRING,``
| `` *    gid INTEGER``
| `` * }``
| `` */``

Here a new responds type e.g.

``posix_user_grouplist (5)``

is needed which returns OutputData

``user_grouplist PosixUserGrouplist``

as

| ``PosixUser ::= SEQUENCE {``
| ``   domain_name OCTET STRING,``
| ``   user_name OCTET STRING,``
| ``   uid INTEGER``
| ``   gid INTEGER``
| ``   gecos OCTET STRING,``
| ``   home_directory OCTET STRING,``
| ``   shell OCTET STRING,``
| ``   grouplist GroupNameList``
| ``}``

``GroupNameList ::= SEQUENCE OF groupname OCTET STRING``

Since the new response type will only be returned if requested by the
client there are no compatibility concerns because older clients cannot
request it.



Feature Management
------------------

The extdom plugin is automatically configured during
ipa-adtrust-install. No additional configuration is needed.

Configuration
~~~~~~~~~~~~~

No additional configuration is needed. If chosen a new OID can indicate
that the feature is available.

.. _how_to_test25:

How to Test
-----------

It is possible to test the new feature using an IPA client or directly
call into the plugin.

.. _integration_tests_with_sssd:

Integration tests with SSSD
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The user-visible effect of this feature is that group members, POSIX
attributes and user's group memberships can all be resolved with the
help of the extdom plugin.

Please make sure to run these tests on an IPA client as the IPA server
doesn't use the plugin but connects to the server directly!

#. Prepare a user who is a member of at least one non-primary group in
   Active Directory
#. Make sure the that hasn't logged in previously. Clearing the cache
   ensures a clean state.
#. Run "id user". The output would show all groups the user is a member
   of
#. Run "getent group $groupname" where $groupname is an AD group that
   contains at least one user. All the member users need to be displayed
   on the command line.

.. _manual_tests_of_the_plugin:

Manual tests of the plugin
~~~~~~~~~~~~~~~~~~~~~~~~~~

Besides running integration tests with a separate IPA client the plugin
can be exercised manually on the server as well and since the extdom
plugin uses standard libc and SSS interfaces IPA users can be requested
via the extdom plugin as well. This mean the plugin can be tested on the
server without established trust which I think would make it possible to
include it in the CI tests as well.

The current version of the extdom plugin can be manually tested in the
following way:

| ``$ cat extdom_req_user_admin.asc``
| ``Example Example.Sid2NameRequestValue``
| ``inputType 2``
| ``requestType 1``
| ``data name``
| ``data.name.domain_name ipa20.devel``
| ``data.name.object_name admin``
| ``$ asn1Coding extdom_req.asn extdom_req_user_admin.asc ``
| ``Parse: done.``
| ``var=Example, value=Example.Sid2NameRequestValue``
| ``var=inputType, value=2``
| ``var=requestType, value=1``
| ``var=data, value=name``
| ``var=data.name.domain_name, value=ipa20.devel``
| ``var=data.name.object_name, value=admin``
| ``name:NULL  type:SEQUENCE``
| ``  name:inputType  type:ENUMERATED  value:0x02``
| ``  name:requestType  type:ENUMERATED  value:0x01``
| ``  name:data  type:CHOICE``
| ``    name:name  type:SEQUENCE``
| ``      name:domain_name  type:OCT_STR  value:69706132302e646576656c``
| ``      name:object_name  type:OCT_STR  value:61646d696e``
| ``Coding: SUCCESS``
| ``-----------------``
| ``Number of bytes=30``
| ``30 1c 0a 01 02 0a 01 01 30 14 04 0b 69 70 61 32 30 2e 64 65 76 65 6c 04 05 61 64 6d 69 6e ``
| ``-----------------``
| ``OutputFile=extdom_req_user_admin.out``
| ``Writing: done.``
| ``$ cat extdom_req_user_admin.out | base64 ``
| ``MBwKAQIKAQEwFAQLaXBhMjAuZGV2ZWwEBWFkbWlu``
| ``$ ldapexop -Y GSSAPI 2.16.840.1.113730.3.8.10.4::MBwKAQIKAQEwFAQLaXBhMjAuZGV2ZWwEBWFkbWlu``
| ``SASL/GSSAPI authentication started``
| ``SASL username: admin@IPA20.DEVEL``
| ``SASL SSF: 56``
| ``SASL data security layer installed.``
| ``# extended operation response``
| ``oid: 2.16.840.1.113730.3.8.10.4``
| ``data:: MDIKAQEELVMtMS01LTIxLTEyMjMyODkxODgtMzE5ODQ0MDM1My0zMzAwMjExMDMyLTUwMA=``
| `` =``
| ``$ echo -n MDIKAQEELVMtMS01LTIxLTEyMjMyODkxODgtMzE5ODQ0MDM1My0zMzAwMjExMDMyLTUwMA== |base64 -d > extdom_resp_user_admin.bin``
| ``$ asn1Decoding extdom_resp.asn extdom_resp_user_admin.bin Example.Sid2NameResponseValue``
| ``Parse: done.``
| ``Decoding: SUCCESS``
| ``DECODING RESULT:``
| ``name:NULL  type:SEQUENCE``
| ``  name:responseType  type:ENUMERATED  value:0x01``
| ``  name:data  type:CHOICE``
| ``    name:sid  type:OCT_STR  value:532d312d352d32312d313232333238393138382d333139383434303335332d333330303231313033322d353030``
| ``$ echo  532d312d352d32312d313232333238393138382d333139383434303335332d333330303231313033322d353030 | xxd -r -p ``
| ``S-1-5-21-1223289188-3198440353-3300211032-500``



RFE Author
----------

`Sumit Bose <User:Sbose>`__
