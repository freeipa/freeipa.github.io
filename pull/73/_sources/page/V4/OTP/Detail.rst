Detail
======

*This page is a work in progress.*



Requirements, Workflows and Use Cases
=====================================



RADIUS Proxy
----------------------------------------------------------------------------------------------

*Prerequisite:* Company has a 3rd party 2FA authentication server
provided by a vendor. All known commonly used authentication servers
support authentication via RADIUS protocol.

-  IPA users should be able to authenticate using passwords (as it is
   now) until as specific administrative action is taken against their
   account.
-  Administrator should be able to define a subset of user accounts that
   must authenticate using external RADIUS server. That means that after
   this administrative action is taken user can’t authenticate using his
   Kerberos password. He can only use his 2FA credential (usually PIN
   and token code) as instructed by the OTP solution vendor. It is
   implied that the environment would use kerberos libraries via sssd or
   kinit coming from the Kerberos version not earlier than 1.12. There
   is no intent to support OTP based 2FA for the direct LDAP binds to
   the IPA directory. Selected users would not be able to bind directly
   to LDAP using OTP credentials from the external vendor. However they
   should be able to authenticate over the Kerberos protocol, acquire
   TGT and then bind to LDAP using GSSAPI.
-  As a result of the kerberos authentication using 2FA credential IPA
   KDC must issue a TGT that can then be used for access of the
   resources in IPA domain as well as negotiation of the SSO access to
   the resources in the trusted domains.
-  Administrator should be able to define this set of users using UI and
   CLI
-  It would be preferable if administrator can select a group of users
   and apply the setting. If the association is removed the user can
   again use his Kerberos password to authenticate, however it might be
   prudent to require user to change the password on his first
   authentication after the association with the remote OTP solution was
   removed.
-  There can be only one RADIUS server configuration per user.
   Configuring more then one should be prohibited.
-  IPA system should have no knowledge about the type of credential the
   user used. The only pieces of information that it needs to know are:

   -  whether the user account needs to proxy what user credential or
      not
   -  what name user should be mapped to

-  The map that the user can be mapped to can be defined in different
   ways. By default with no special configuration user ‘uid’ should be
   used. Administrator should be able to define an alternative attribute
   for example user Kerberos principal or some other attribute for
   example employee ID or badge number created and populated by the
   customer for alternative user identification.
-  IPA should be capable of storing configuration for different groups
   of RADIUS servers so that a subset of users is authenticated against
   one set of RADIUS servers represented by one vendor while other
   subset of users can authenticate against another set of RADIUS
   servers represented by another vendor.
-  Administrative interface UI/CLI should allow to define radius server
   configuration. This configuration should include: list of RADIUS
   server hosts with optional ports, RADIUS secret,number of retries,
   timeout of each retry. The secret, timeout and retries are expected
   to be the same across all RADIUS servers of the same set.



Native OTP
----------------------------------------------------------------------------------------------

-  IPA users should be able to authenticate using passwords as it is now
   until as specific administrative action is taken against their
   account.
-  Administrator should be able to define a subset of user accounts that
   must authenticate using OTP tokens stored inside IPA. That means that
   after this administrative action is taken user can’t authenticate
   using his Kerberos password. He can only use his 2FA credential which
   consists of his Kerberos password as PIN and the OTP from one of the
   tokens that was assigned to him. It is implied that the environment
   would use Kerberos libraries via sssd or kinit coming from the
   Kerberos version not earlier than 1.12.
-  If both RADIUS config and local tokens are assigned to user the
   authentication should be proxied to the external RADIUS server.
-  Selected users would be able to bind directly to LDAP using OTP
   credentials using locally defined tokens
-  There can be more than one token assigned to the user.
-  System should support software tokens and hardware tokens
-  It should be possible to create and provision software tokens as
   needed via UI and/or CLI
-  It should be possible to import external XML file (RFC 6030) with the
   external tokens provided by a 3rd party vendor (mostly HW tokens) and
   manage them in the system.
-  The device information that comes from 3rd party vendor XML should be
   loaded into the system and preserved in the system as is (probably
   protected from modification by ACIs by default).
-  There should be a CLI/UI to manage tokens.

   -  It should implement:

      -  Token Creation
      -  Token Editing
      -  Token Deletion (except the last active token)
      -  Token Synchronization
      -  Token Assignment (manager / owner)

   -  Admins should be able to perform all the above operations.
   -  Users should have permission to synchronize tokens owned by them.
      Additionally, users should have access to create, edit, and delete
      all tokens managed by them.
   -  The user should be able to access the UI before tokens are
      provisioned using the first factor.

-  There should be a self service portal, available outside the
   firewall, implementing:

   -  Token Synchronization

-  System should support basic “Lost token” use case. This means that if
   the user looses his token he should call a help desk and a temporary
   authentication solution should be given to him. The solution should
   have a validity time and expire after. User should be able to use
   this solution until he can use a token again or it expires.
-  Once the user was capable of using any of his lost token the
   capability of using temporary solution should be removed from him
   automatically without administrator’s involvement.
-  Temporary solution should be at least as strong as user’s original
   Kerberos password
-  It is not required so far to support more complex lost password
   scenarios however system should assume that more complex lost
   password scenarios will be required in future
-  Tokens can be disabled by administrator. Once disabled the token
   can’t be used until it is enabled back by administrator.
-  Administrator should be able to define a start date and end date of
   the validity of the token. This is the range that constraints the
   token authentication. The device data (start and end dates for the
   device) loaded from the XML file should not be considered
   authoritative.
-  Until a true offline authentication is supported there should be a
   way to authenticate offline with just password but not acquire
   Kerberos ticket. If the system was offline but got online and there
   is no Kerberos ticket, the system should prompt user for
   re-authentication by locking his screen as soon as he gets online.



Common use cases
----------------------------------------------------------------------------------------------

-  Requiring all users to authenticate two factor all the time might be
   a challenge. There should be a way to accept either just password or
   OTP code.
-  The system should allow defining what authentication methods are
   allowed/required for the account.
-  The system should detect which which authentication method was used
   and have policies that define which services would be accessible
   using which kind of the authentication.
-  There should be a way to define and centrally manage which services
   would require which kind of the authentication.



Future use cases (current non goals)
----------------------------------------------------------------------------------------------

-  Administrative interface should allow creating different reports
   about tokens and user assignments.
-  More complex lost token work flows (I just left my token home I have
   not lost it completely)
-  Ability to transform the attribute using regex before it is sent to
   the RADIUS server. For example translate uid into uid@domain using
   regex
-  Ability to define tokens that replace other tokens
-  Ability to bulk assign multiple tokens
-  Support of the offline authentication



Non goals
----------------------------------------------------------------------------------------------

-  Replay attack prevention

Design
======

Architecture
----------------------------------------------------------------------------------------------

::

   +--------+     +-------+     +----------+     +------+
   | client | <=> |  KDC  | <=> | ipa-otpd | <=> | LDAP | 
   +--------+     +-------+     +----------+     +------+
                                         \\    +--------+
                                          \\>  | RADIUS |
                                               +--------+

The krb5 client and KDC code is implemented according to the designs
here:
` <http://k5wiki.kerberos.org/wiki/Projects/Responder>`__\ http://k5wiki.kerberos.org/wiki/Projects/Responder
(see also krb5.h)
` <http://k5wiki.kerberos.org/wiki/Projects/OTPOverRADIUS>`__\ http://k5wiki.kerberos.org/wiki/Projects/OTPOverRADIUS
The KDC OTP plugin does not permit dynamic configuration. For this
reason, we will provide a companion daemon which is an intelligent
proxy/multiplexer. This daemon listens for RADIUS packets on a UNIX
socket and will forward the authentication request, based on the user's
configuration in LDAP, to either LDAP (via an LDAP bind) or a 3rd party
RADIUS server. The daemon will be socket activated by systemd when the
KDC attempts to connect. The companion daemon will re-use krb5 RADIUS
code for simplicity. This implies that the companion daemon will have
two dependencies:

-  libverto - for main loop
-  libkrb5/libk5crypto - for randomization, MD5 hashing and krb5 data
   types

No other dependencies are required. The 3rd party RADIUS daemon is not
owned by us, so no specification is necessary. However, the LDAP daemon
(389) will gain an implementation of RFC 6238.



Token Types
----------------------------------------------------------------------------------------------

-  TOTP (time-based) tokens are fully supported.
-  HOTP (counter-based) tokens are somewhat functional, but not
   supported. This is due to the fact that counter replication is
   extremely costly in a replicated environement.

Logic
----------------------------------------------------------------------------------------------

-  There will be a global policy in the ipaConfig (so it is replicated)
   to control authentication methods across the deployment. There also
   be the same attribute in the user entry. If the attribute in the user
   entry is not present then the global setting will be respected. If
   the global setting is also not present the current existing logic
   would be assumed. This will be a multi-valued string attribute called
   ipaUserAuthType. Possible values for that attribute are:

   -  "password" - password authentication is allowed
   -  "otp" - 2FA (password + OTP) authentication is allowed
   -  "pkinit" - pkinit is allowed
   -  "radius" - RADIUS proxying is allowed
   -  "disabled" - all extended authentication methods (local OTP
      authentication or remote OTP proxy over RADIUS) are disabled
      regardless of the data in the user entry. This value is applicable
      only to the global setting and will be ignored in the local
      setting.

   If global setting is set to "disabled", only the password
   authentication is allowed. In all other cases a combination of global
   flags defines the default allowed methods. The value in user entry,
   if explicitly set, overwrites the global value in this case. For
   example if the global says "otp" and a user account has the value of
   "password" than every account must use OTP and only this single
   account is allowed to use password. In other words the global setting
   provides the default that can be overwritten on the per entry basis.
   Since the attribute is multivalued, a combination of multiple
   attributes is possible. For example "password" + "pkinit" is
   equivalent to the current default behavior. Configuring RADIUS proxy
   for an account or assigning a token to that would not be enough for
   the system to start using RADIUS or 2FA (respectively) for the
   account. The attribute must be set to "radius" / "otp" either
   globally or per-user. Invalid (misspelled) values would be ignored.

-  KDB will use this attribute in user entry and in the global
   configuration and merge it when it presents data to the KDC. The KDC
   would make a decision what preauth methods to send to client based on
   that value. If only "otp" is present only "otp" preauth will be
   possible, i.e. only OTP preauth mechanism will be present in the
   preauth request. This means that it would work only on the clients
   that support "OTP" preauth method and not on the older clients that
   do not support OTP. This is an expected behavior. Old client would
   automatically fail. This means that if "otp" is required for the user
   the deployment would need to make sure that the user would log only
   via clients that fully support OTP preauth method
-  To allow either 2FA or password authentication both "otp" and
   "password" should be specified. KDC then will send both OTP and
   encrypted timestamp methods. On the older clients the otp would be
   ignored and user input would be sent to KDC using encrypted timestamp
   algorithm. This will allow users to authenticate with passwords from
   the older systems. NOTE: Current limitations of MIT's kinit causes
   the encrypted timestamp method to always be selected even when OTP is
   offered. To work around this, FreeIPA currently disables enctyped
   timestamp when OTP is enabled.
-  Mixed mode "otp" + "password" should not be used until we implement
   the ability to record which authentication method was used inside the
   ticket. Once this is done we would be able to use discretion and
   create service tickets for specific services only if the policy
   allows user to access that service. This functionality is planned for
   later so in the initial implementation mixing "otp" and "password"
   methods would not be possible.
-  If "otp" method is enabled for the account the client will receive
   the request, prompt the user and put the user input into the OTP
   authentication data and ship to the server within the Kerberos FAST
   tunnel protected by Kerberos identity of the client. The use of the
   FAST tunnel requires the use of the -T option with kinit.
-  KDC will receive the response from the client and if the response
   contains OTP auth data the KDC will create a RADIUS request and
   forward it to the companion daemon.
-  Companion daemon at the start will read all possible RADIUS
   configurations from the LDAP server and cache them. It will refresh
   them if it sees that any of them change.
-  Upon receiving of the RADIUS request from the KDC companion daemon
   will look-up user account by principal. If the user account contains
   a link to the RADIUS configuration the radius request will be sent to
   that RADIUS server using already cached configuration.
-  Companion daemon might perform remapping of the user name depending
   on the configuration.

   -  If a special overriding attribute 'ipatokenRadiusUserName' is
      present in the user entry the value of this attribute will be used
      as a user name in the proxied RADIUS request.
   -  If the attribute is not present but RADIUS server configuration
      contains an attribute 'ipatokenUserMapAttribute' then the
      attribute that it points to will be used as a source of the user
      name.
   -  If neither specified (default) 'uid' attribute will be used.

-  If there is no link to the RADIUS server in the user entry companion
   daemon will perform a local LDAP bind using a DN of the looked up
   user entry as a bind DN and provided user input as a credential
   assuming that user authenticates with a locally defined OTP token.
-  There will be a new DS bind plugin that would perform the
   authentication. It will be used by the companion daemon but would
   allow any client to authenticate using OTP code. This will be
   equivalent to simple bind so non local LDAP clients would have to
   bind over SSL/TLS to not expose the user provided passcode in clear.
-  In case of successful authentication the reply will be sent to the
   caller. If the caller is companion daemon it would in turn reply to
   KDC and KDC will reply to the client with a TGT.



Lost Token Processing
----------------------------------------------------------------------------------------------

-  In the case of the external RADIUS server is configured for user
   account the lost token processing is completely opaque and delegated
   to the external RADIUS server to handle.
-  If the user with OTP token assigned to him inside IPA losses his
   token there are two ways to handle this situation.

   -  If token is permanently lost (together with device it is on) it
      should be disabled and a new token should be issued to the user.
   -  If token is not permanently lost but rather left home or in some
      place where user would be able to get a hold of it later without
      jeopardizing the system security then help desk admin should set
      the expiration on that status. Following administrator's actions a
      special attribute will be added to that token. This attribute
      would specify a moment when the misplaced token must be found.

      -  If the attribute is present and time has not elapsed yet the
         user would be able to use just his password because the
         presence of the valid attribute would force the server to treat
         the length of the OTP token code to be 0 (temporarily).
      -  If user authenticates with the full OTP code while the
         attribute is valid that would indicate the the token is found
         and attribute will be removed.
      -  If time passes and token is not found it would require a manual
         intervention to revive/remove this token. Any attempts to
         authenticate with such token would automatically fail.

-  This logic will be adjusted later if/when other more sophisticated
   lost token methods are implemented.



DS Bind Plugin Logic
----------------------------------------------------------------------------------------------

-  User will be looked up by his DN.
-  Tokens assigned to this user will be looked up by user DN they belong
   to. The searches will be indexed.
-  If no active or enabled tokens are found the provided credential is
   assumed to be user password.
-  If there are any valid active tokens these tokens will be used to
   authenticate the user.
-  The standard TOTP/HOTP algorithm will be used. Several codes will be
   generated and matched. If the tokens is way off sync it might require
   a synchronization procedure (TBD).
-  If any of the token codes matched the authentication is successful
   and the reply will be sent to the caller.



Top Token Container
----------------------------------------------------------------------------------------------

::

   cn=otp,$SUFFIX