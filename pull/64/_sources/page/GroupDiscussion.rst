GroupDiscussion
===============



Primary Groups Proposals
========================

In UNIX permissions on files are always represented (at minimum) by the
triplet: user, group, others, and by default the user umask that control
default permissions on new file creation is set to 002. This means that,
usually, when a user creates a new file it is writable by the user's
primary group. This design led many people to face a "default privacy"
problem, where in systems with many users sharing the same primary group
documents are accessible by default by all members of such group.

There are 2 ways to change configurations so that by default only the
user owner can access his files. One is to change the default umask to
022, the other one, is to create a group for each user and make it the
user's primary group, so that in effect only the user can access the
file.

This second method has the advantage that even if one of many systems
has a umask not configured properly files are still created with a
reasonable access mode. Actually what this scheme reveals is a deep flaw
in the concept of security principals in UNIX. Unix still separate, at
the kernel level, the concept of users and groups using two different
name and number spaces, and forces each resource to have both a user and
a group owner. By creating "Personal Groups" (a group for each user with
the same user name and often also with a gid the matches the numeric
value of the user's uid) what people actually unconsciously do is to
unify the group and user spaces into one space (the group space).

Since I think this is a beneficial side effect I think this scheme is
not actually bad, but it creates a problem, as it multiply the number
of, almost useless, groups on a system. When we start putting
users/groups on LDAP this is translated in higher latency when
enumerating groups. While we discourage enumeration of both users and
groups, unfortunately current clients like nss_ldap still implements it
and some OSs may even require it to function properly.

But our GUI/CLI tools does not need to necessarily pay this fee when
used, if we decide to make Personal Groups a very limited kind of group
we can think of a possibly better way to handle the problem and actually
really merge users and Personal Groups concepts in a single "Security
Principal" so to say.



The Virtual Personal Groups Proposal
------------------------------------

The proposal is to actually write a plugin that intercept searches for
posixGroup objectclasses and actually perform an additional search on
posixAccount Objectclasses, the search would ask only for the uid and
the gidNumber and would return a virtual object where the uid is the
group name, the gidNumber is unchanged and objectclass is transformed
into a posixGroup class, and the unique member of this group would be
the user itself (uid). This kind of personal groups would be denied any
other membership.

Our GUI/CLI would not list these Personal Groups by default unless
explicitly commanded to do so by not searching for posixGroup when
searching groups (This will require an additional auxiliary objectclass
to be installed on normal groups).

A very nice side effect of this scheme is that if the uid/gid number
space is unified by default, the "personal groups" plugin can be
activated/deactivated at will without causing too much trouble, letting
admins experiment with both schemes.

Problems/Concerns with this approach: 1. Concern is that this plugin may
slow down searches on groups. I think that if done carefully this may
not be a real problem, the translation required does not involve many
attributes, but tests are required to asses it's impact. 2. The group
names name-space needs to be mergegd with the user names name-space
(this is a general problem with personal groups independent from the
implementation). A plugin that controls users creation need to assess
users and group names do not clash and deny the operation otherwise, and
viceversa. 3. The group gid number-space needs to be merged with the
user uid number-space. This will make it much simpler to assigned ids to
either users or groups, a single id counter will be used for both users
and groups uid/gid so that each users uid is guaranteed to also be the
personal group gid. This will guarantee that uid==gid always and if a
gid is free the corresponding uid is free as well. So when we need to
create users we can even decide to reuse the uid/gid of a deleted
user/group without problems.

One issue with older UNIX boxes is that on some OS the uid/gid number
space is limited to 16 bits, or a total maximum amount of ids around
65k. The nice side effect of this plugin is that it can be deactivated
(eventually based on which client connects) at will so that for this
older system an easy backup mode can be conceived. On these system
user's are assigned a default primary group ("users" is generally a good
default), and personal groups are not created. At the same time the id
allocation algorithm is built so that id for real groups are allocated
starting for example at 2G so that for these older system real groups
are remapped using newgid=gid-2G This would keep compatibility with
these legacy systems without compromising the flexibility for newer and
more capable systems. The only limit would be the 65K users or groups
per installation which is a hard limit in these systems.



Real Groups Proposal
--------------------

Another approach to providing a group per user is to store a group per
user. In this scheme whenever a user is created a group is automatically
created at the same time and the group gid is used as the gid for the
user. Also when the user is deleted the group is also automatically
deleted. This group is "special" in that it is more restrictive than
other groups - it will allow only one member which must be a
posixAccount, deny the addition of any member that does not share the
same gid, and it will not allow itself to be nested in other groups.

Merging of the user and group numeric identifier space is still possible
with this scheme.

The advantages to such a scheme

-  The groups are real and so scale with the directory just like any
   other entry - no on the fly translation.

   -  This has implications for other directory features, such as access
      control, just working with the scheme
   -  The real problem with a group per user scheme is not that the
      server must store the groups, but that the clients must sift
      through them.

-  Clients that do things that are not allowed are given reasonable
   responses and can act appropriately (when virtualizing group entries
   from user entries any subsequent group operation on a user may, for
   example, fall foul of schema violations which would likely be
   confusing to clients)
-  it is definitely possible to implement without DS modifications
   beyond a plugin (it is not so clear that is possible for virtualized
   search results)

Disadvantages:

-  The DIT becomes littered with groups. Though this might be mitigated
   somewhat by segregating the automatically created groups so that at
   least casual browsing is not encumbered by them.