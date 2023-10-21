DNA_plugin_default_configuration
================================



The user/Group ID assignment problem
------------------------------------

ID assignment is extremely delicate. Usually once an UID or GID is
assigned on a UNIX machine it is very difficult and expensive to change
it, because these numbers are stored on the filesystem to identify which
resources a user have access to. It is also extremely important that 2
users never happen to have the same ID or they will be considered
affectively the same user for authorization purposes.

The problem is even more critical in a networked environment, because
posix does not define per machine uid sets. UID numbers are global and
yet, at the same time are extremely limited in number. Only 4B (32 bit)
are available in most recent platforms. And some very old ones are still
limited to 16bit UIDs, although so old platforms can be ignored for our
purposes because they are way too limited to be taken into
consideration.

Another argument to take into consideration is that in future we may
want to allow to trust of federate multiple IPA domains together.
Company acquisitions, consolidations, resource separation may drive the
adoption of multiple separated directories that need to be integrated
later on. It is therefore important that we make it improbable to have
ID overlaps between 2 different IPA domains.

Because the ID space is 32 bit we can do this by splitting the ID space
into 2 halves. And use the first 10 bit as a "domain selector", and the
remaining 22 bits as the domain assigned range. 22bits can sill
represent 4M different users and should be enough for any installation
in the close future. Of course nothing prevent an administrator to add
new ranges later on. The 10 bit "domain selector" will be randomly
determined at installation time (optionally set by an install option).

NOTE: to cope with the recent OpenSolairs concept of ephemeral IDs we
might want to limit the domain selector to 9 bit and set the higher bit
always to 0.

Another issue with ID management is that the UID and GID ranges are
separate and autonomous. this means that a user and a totally unrelated
group can have the same numeric ID. While this is not technically a
problem there are cases where a unique space might be useful (esp
interoperability with windows client where SIDs share a unique number
space). Also by default in IPA we do not create a group for each user as
it is done on some operating systems. To allow admins to easily
associate a group with each user at a later date, it might make sense to
allocate groups so that the groups numeric ID space used by default does
not numerically overlap with that of the users. This can be accomplished
by setting the 22th bit of the ID spaces by default to generate the
group range base. This effectively reduces the default number of Ids
available to only 2M with the default range. Again we anticipate that 2M
groups is an acceptable number for a default setup, the range is easily
extensible with admin intervention so it is not binding.



Implementation and default configuration
----------------------------------------

The Directory Server DNA plugin is the tool we use to automatically
assign user and group IDs. In its latest revised design it is fully MMR
aware and can also transfer ID ranges between masters automatically.

On the first installation we need to create configuration entries that
are used by the plugin to perform ID range transfers and also define
what are the ranges in use.

The plugin configuration is divided in 2 parts. Local configuration
under the cn=config tree, and global configuration in the public
instance.

Given an IPA domain called example.com (REALM: EXAMPLE.COM), the
following example the configuration for the first IPA Server apply. The
server is called first.example.com We assume our RNG choose 123
(decimal) as the "domain selector" which means the base range for user
would be: 515899392-517996543 and for groups would be:
517996544-520093695

| ``123 is 01111011 binary 8bit, and 0001111011 binary (10bit)``
| ``if we append 22 bit set to 0 we get 0x1EC00000 hex``
| ``the uidNumber range is 0x1EC00000-0x1EEFFFFF in hex``
| ``the gidNumber range is 0x1EE00000-0x1EFFFFFF in hex``



First server initial local configuration
----------------------------------------------------------------------------------------------

   ::

      dn: cn=Posix Account UID Numbers,
       cn=Distributed Numeric Assignment Plugin,cn=plugins,cn=config
      objectClass: top
      objectClass: extensibleObject
      cn: Posix Account UID Numbers
      dnatype: uidNumber
      dnainterval: 1
      dnamaxvalue: 517996543
      dnamagicregen: 0
      dnathreshold: 100
      dnafilter: (objectclass=posixAccount)
      dnascope: cn=accounts,dc=example,dc=com
      dnasharedcfgdn: cn=Posix Account UID Numbers,
       cn=DNAranges,cn=ipa,cn=etc,dc=example,dc=com
      dnanextvalue: 515899392

..

   ::

      dn: cn=Posix Group GID Numbers,
       cn=Distributed Numeric Assignment Plugin,cn=plugins,cn=config
      objectClass: top
      objectClass: extensibleObject
      cn: Posix Group GID Numbers
      dnatype: gidNumber
      dnainterval: 1
      dnamaxvalue: 520093695
      dnamagicregen: 0
      dnathreshold: 100
      dnafilter: (objectclass=posixGroup)
      dnascope: cn=accounts,dc=example,dc=com
      dnasharedcfgdn: cn=Posix Group GID Numbers,
       cn=DNAranges,cn=ipa,cn=etc,dc=example,dc=com
      dnanextvalue: 517996544



Server global configuration
----------------------------------------------------------------------------------------------

Three entries are pre-created by the configuration scripts.

   ::

      dn: cn=DNAranges,cn=ipa,cn=etc,dc=example,dc=com
      objectClass: top
      objectClass: nsContainer
      cn: DNAranges

      dn: cn=Posix Account UID Numbers,
       cn=DNAranges,cn=ipa,cn=etc,dc=example,dc=com
      objectClass: top
      objectClass: nsContainer
      cn: Posix Account UID Numbers

      dn: cn=Posix Group GID Numbers,
       cn=DNAranges,cn=ipa,cn=etc,dc=example,dc=com
      objectClass: top
      objectClass: nsContainer
      cn: Posix Group GID Numbers

Given the example above, the following are the 2 entries that will be
generated by the DNA plugin.

   ::

      dn: dnaHostname=first.example.com+dnaPortNum=389, cn=Posix Account UID Numbers,
       cn=DNAranges,cn=ipa,cn=etc,dc=example,dc=com
      objectClass: extensibleObject
      objectClass: top
      dnahostname: first.example.com
      dnaPortNum: 389
      dnaSecurePortNum: 636
      dnaRemainingValues: 2097151

      dn: dnaHostname=first.example.com+dnaPortNum=389, cn=Posix Group GID Numbers,
       cn=DNAranges,cn=ipa,cn=etc,dc=example,dc=com
      objectClass: extensibleObject
      objectClass: top
      dnahostname: first.example.com
      dnaPortNum: 389
      dnaSecurePortNum: 636
      dnaRemainingValues: 2097151



Replicas configurations
-----------------------

All servers in the same realm share the same range of uid and gid
numbers. When a new replica is created we do not need to select a new
"domain selector", nor assign arbitrary range values. For a replica the
local configuration will have the dnamaxvalue=0 and the dnanextvalue=0
for all range configuration entries. This will cause the replica to
request part of the range from one of the existing available master and
all replicas will use a part of the original range interval reandomly
selected at installation.



replicas initial local configuration
----------------------------------------------------------------------------------------------

   ::

      dn: cn=Posix Account UID Numbers,
       cn=Distributed Numeric Assignment Plugin,cn=plugins,cn=config
      objectClass: top
      objectClass: extensibleObject
      cn: Posix Account UID Numbers
      dnatype: uidNumber
      dnainterval: 1
      dnamaxvalue: 0
      dnamagicregen: 0
      dnathreshold: 100
      dnafilter: (objectclass=posixAccount)
      dnascope: cn=accounts,dc=example,dc=com
      dnasharedcfgdn: cn=Posix Account UID Numbers,
       cn=DNAranges,cn=ipa,cn=etc,dc=example,dc=com
      dnanextvalue: 0

..

   ::

      dn: cn=Posix Group GID Numbers,
       cn=Distributed Numeric Assignment Plugin,cn=plugins,cn=config
      objectClass: top
      objectClass: extensibleObject
      cn: Posix Group GID Numbers
      dnatype: gidNumber
      dnainterval: 1
      dnamaxvalue: 0
      dnamagicregen: 0
      dnathreshold: 100
      dnafilter: (objectclass=posixGroup)
      dnascope: cn=accounts,dc=example,dc=com
      dnasharedcfgdn: cn=Posix Group GID Numbers,
       cn=DNAranges,cn=ipa,cn=etc,dc=example,dc=com
      dnanextvalue: 0