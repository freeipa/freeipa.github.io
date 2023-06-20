Schema
======



OTP Schema
==========

::

   # FreeIPA tokens schema
   # BaseOID: TBD
   # We use ipatoken as "namespace"
   # See RFC 4517 for Syntax OID definitions
   dn: cn=schema

   #
   # Token related attributes
   #
   attributeTypes: (2.16.840.1.113730.3.8.16.1.1
       NAME 'ipatokenUniqueID'
       DESC 'Token Unique Identifier'
       EQUALITY caseIgnoreMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.15
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.2
       NAME 'ipatokenDisabled'
       DESC 'Optionally marks token as Disabled'
       EQUALITY booleanMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.7
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.3
       NAME 'ipatokenNotBefore'
       DESC 'Token validity date'
       EQUALITY generalizedTimeMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.24
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.4
       NAME 'ipatokenNotAfter'
       DESC 'Token expiration date'
       EQUALITY generalizedTimeMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.24
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.5
       NAME 'ipatokenVendor'
       DESC 'Optional Vendor identifier'
       EQUALITY caseIgnoreMatch
       SUBSTR caseIgnoreSubstringsMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.15
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.6
       NAME 'ipatokenModel'
       DESC 'Optional Model identifier'
       EQUALITY caseIgnoreMatch
       SUBSTR caseIgnoreSubstringsMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.15
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.7
       NAME 'ipatokenSerial'
       DESC 'OTP Token Serial number'
       EQUALITY caseIgnoreMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.15
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.8
       NAME 'ipatokenOTPkey'
       DESC 'OTP Token Key'
       EQUALITY octetStringMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.40
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.9
       NAME 'ipatokenOTPalgorithm'
       DESC 'OTP Token Algorithm'
       EQUALITY caseIgnoreMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.15
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.10
       NAME 'ipatokenOTPdigits'
       DESC 'OTP Token Number of digits'
       EQUALITY integerMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.27
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.11
       NAME 'ipatokenTOTPclockOffset'
       DESC 'TOTP clock offset'
       EQUALITY integerMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.27
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.12
       NAME 'ipatokenTOTPtimeStep'
       DESC 'TOTP time-step'
       EQUALITY integerMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.27
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.13
       NAME 'ipatokenOwner'
       DESC 'User entry that owns this token'
       SUP distinguishedName
       EQUALITY distinguishedNameMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.12
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.21
       NAME 'ipatokenHOTPcounter'
       DESC 'HOTP counter'
       EQUALITY integerMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.27
       SINGLE-VALUE X-ORIGIN 'IPA OTP')


   #
   # Token related objectclasses
   #
   objectClasses:  (2.16.840.1.113730.3.8.16.2.1
       NAME 'ipaToken'
       SUP top ABSTRACT
       DESC 'Abstract token class for tokens'
       MUST (ipatokenUniqueID)
       MAY (description $ managedBy $ ipatokenOwner $ ipatokenDisabled $ ipatokenNotBefore $
            ipatokenNotAfter $ ipatokenVendor $ ipatokenModel $ ipatokenSerial)
       X-ORIGIN 'IPA OTP')

   objectClasses:  (2.16.840.1.113730.3.8.16.2.2
       NAME 'ipatokenTOTP'
       SUP ipaToken STRUCTURAL
       DESC 'TOTP Token Type'
       MUST (ipatokenOTPkey $ ipatokenOTPalgorithm $ ipatokenOTPdigits $
             ipatokenTOTPclockOffset $ ipatokenTOTPtimeStep)
       X-ORIGIN 'IPA OTP')

   objectClasses:  (2.16.840.1.113730.3.8.16.2.5
       NAME 'ipatokenHOTP'
       SUP ipaToken STRUCTURAL
       DESC 'HOTP Token Type'
       MUST (ipatokenOTPkey $ ipatokenOTPalgorithm $ ipatokenOTPdigits $ ipatokenHOTPcounter)
       X-ORIGIN 'IPA OTP')

   #
   # RADIUS related attributes
   #
   attributeTypes: (2.16.840.1.113730.3.8.16.1.14
       NAME 'ipatokenRadiusUserName'
       DESC 'Corresponding Radius username'
       EQUALITY caseIgnoreMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.15
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.15
       NAME 'ipatokenRadiusConfigLink'
       DESC 'Corresponding Radius Configuration link'
       SUP distinguishedName
       EQUALITY distinguishedNameMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.12
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.16
       NAME 'ipatokenRadiusServer'
       DESC 'Server String Configuration'
       EQUALITY caseIgnoreIA5Match
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.26
       X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.17
       NAME 'ipatokenRadiusSecret'
       DESC 'Server Secret'
       EQUALITY octetStringMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.40
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.18
       NAME 'ipatokenRadiusTimeout'
       DESC 'Server Timeout'
       EQUALITY integerMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.27
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.19
       NAME 'ipatokenRadiusRetries'
       DESC 'Number of allowed Retries'
       EQUALITY integerMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.27
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   attributeTypes: (2.16.840.1.113730.3.8.16.1.20
       NAME 'ipatokenUserMapAttribute'
       DESC 'Attribute to map from the user entry for RADIUS server authentication'
       EQUALITY caseIgnoreMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.15
       SINGLE-VALUE X-ORIGIN 'IPA OTP')

   #
   # RADIUS related objectClasses
   #

   objectClasses:  (2.16.840.1.113730.3.8.16.2.3
       NAME 'ipatokenRadiusProxyUser'
       SUP top AUXILIARY
       DESC 'Radius Proxy User'
       MAY (ipatokenRadiusConfigLink $ ipatokenRadiusUserName)
       X-ORIGIN 'IPA OTP')

   objectClasses:  (2.16.840.1.113730.3.8.16.2.4
       NAME 'ipatokenRadiusConfiguration'
       SUP top STRUCTURAL
       DESC 'Proxy Radius Configuration'
       MUST (cn $ ipatokenRadiusServer $ ipatokenRadiusSecret)
       MAY (description $ ipatokenRadiusTimeout $ ipatokenRadiusRetries $
            ipatokenUserMapAttribute)
       X-ORIGIN 'IPA OTP')

   # Class for authentication method definition

   attributetypes: ( 2.16.840.1.113730.3.8.11.40
       NAME 'ipaUserAuthType'
       DESC 'Allowed authentication methods'
       EQUALITY caseIgnoreMatch
       SYNTAX 1.3.6.1.4.1.1466.115.121.1.15
       X-ORIGIN 'FreeIPA' )

   objectclasses: ( 2.16.840.1.113730.3.8.12.19
       NAME 'ipaUserAuthTypeClass'
       SUP top AUXILIARY
       DESC 'Class for authentication methods definition'
       MAY ipaUserAuthType
       X-ORIGIN 'FreeIPA' )



OTP ACIs
========

::

   dn: $SUFFIX
   changetype: modify
   add: aci

   aci: (targetfilter = "(objectClass=ipaToken)")(targetattrs = "objectclass || description || managedBy || ipatokenUniqueID || ipatokenDisabled || ipatokenNotBefore || ipatokenNotAfter || ipatokenVendor || ipatokenModel || ipatokenSerial || ipatokenOwner")(version 3.0; acl "Users/managers can read basic token info"; allow (read, search, compare) userattr = "ipatokenOwner#USERDN" or userattr = "managedBy#USERDN";)

   aci: (targetfilter = "(objectClass=ipatokenTOTP)")(targetattrs = "ipatokenOTPalgorithm || ipatokenOTPdigits || ipatokenTOTPtimeStep")(version 3.0; acl "Users/managers can see TOTP details"; allow (read, search, compare) userattr = "ipatokenOwner#USERDN" or userattr = "managedBy#USERDN";)

   aci: (targetfilter = "(objectClass=ipatokenHOTP)")(targetattrs = "ipatokenOTPalgorithm || ipatokenOTPdigits")(version 3.0; acl "Users/managers can see HOTP details"; allow (read, search, compare) userattr = "ipatokenOwner#USERDN" or userattr = "managedBy#USERDN";)

   aci: (targetfilter = "(objectClass=ipaToken)")(targetattrs = "description || ipatokenDisabled || ipatokenNotBefore || ipatokenNotAfter || ipatokenVendor || ipatokenModel || ipatokenSerial")(version 3.0; acl "Managers can write basic token info"; allow (write) userattr = "managedBy#USERDN";)

   aci: (targetfilter = "(objectClass=ipaToken)")(version 3.0; acl "Managers can delete tokens"; allow (delete) userattr = "managedBy#USERDN";)

   aci: (target = "ldap:///ipatokenuniqueid=*,cn=otp,$SUFFIX")(targetfilter = "(objectClass=ipaToken)")(version 3.0; acl "Users can create self-managed tokens"; allow (add) userattr = "ipatokenOwner#SELFDN" and userattr = "managedBy#SELFDN";)