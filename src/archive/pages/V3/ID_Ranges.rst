ID_Ranges
=========



What are RIDs and RID ranges
----------------------------------------------------------------------------------------------

RIDs are unique 32-bit identifiers of the user/group objects in the
scope one Active Directory domain. The RID is the last part of the SID,
e.g. for SID S-1-5-21-123-456-789-1010, the last segment, that is 1010
is the RID. RID range is the analog of the ID range.



The purpose of primary and secondary RID range
----------------------------------------------------------------------------------------------

If we want to export user / group objects from IPA to the Active
Directory, we need to generate SIDs for them. The whole SID but the last
segment (which is above mentioned RID) is an unique string to the
domain. To keep SIDs of the objects unique, we need to generate an
unique RID for each object in the IPA domain. RID range (or an RID
space) is then an interval of natural numbers, where we expect our RIDs
to be generated. RIDs are generated from the object IDs (UID for users,
GID for groups). Their generation from the IDs is completely
deterministic, and in the simple terms it's just a shift of the
interval:

| ``   ID       |   RID``
| ``   10000   ->    0``
| ``   10001   ->    1``
| ``   10002   ->    2``
| ``   10003   ->    3``
| ``   10004   ->    4``
| ``   ....``

However, the UID and GID spaces in UNIX are independent. It's completely
okay to have an user with UID 10002 and a group with GID 10002.
Following the example above, we would end up with RID 2 for both these
objects. To avoid that, we define secondary RID range for local ranges.
This new interval serves as a backup interval where we shift the object
if corresponding RID in the primary range is already taken:

| ``     UID    |    GID      |     RID``
| ``   10000                 ->      0``
| ``   10001                 ->      1``
| ``                10001    ->      10001``
| ``                10002    ->      2``
| ``   10003                 ->      3``

(secondary RID range starts at 10000 in this example)

Please note that objects created by IPA by default have non-conflicting
UIDs and GIDs, since they are assigned by DNA Directory Server plugin,
which assigns to a newly created user/group a new ID which is unique
across UID/GID space. However, admin can manually create specify UID or
GID at the object creation and create conflicts, or any data imported
from other LDAP/NIS systems might.



Constraints on ID ranges
----------------------------------------------------------------------------------------------



Any domain
^^^^^^^^^^

-  Base range space should not overlap for any range combination (of any
   type)

   -  Reasoning: Base ID range space is the ID range space as seen in
      the IPA world. Overlapping base ID range space could cause
      conflicts on UIDs/GIDs.
   -  Exception: Base ranges with type ipa-ad-trust-posix may overlap if
      the domains are coming from the same forest, they might even be
      the same.

      -  Reasoning: with type ipa-ad-trust-posix we do not have to do
         any mapping at all but all data (Posix IDs and SIDs/RIDs) are
         coming from AD and we have to trust AD here that the values are
         choosen wisely and do not overlap. By default in this case only
         one idrange will be created for the forest root and SSSD
         automatically takes the same ID range for any member domain in
         the forest which does not have an specific idrange set.

``   Q: So why would anyone set up a new ID range of ipa-ad-trust-posix type for subdomain? Is it the case that user/group entries from subdomain are not replicated to the forest root?``

``   A: E.g. if the POSIX IDs are not managed centrally for the whole forest, but domain admins do it on a per domain basis, seperate ipa-ad-trust-posix ranges with disjunct base idrange would help to avoid collisions. This happens by assigining each domain range a specific reserved ID range space, which is non-overlapping with any other ID range space, thus if any two objects from two different domains collide on UID/GID, only one of them can be resolved in IPA since only one of them would be in ID range space of its own particular domain.``



Ranges from the same domain
^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  If there are multiple ranges for a single domain, all must have the
   same type.



ipa-ad-trust ranges
'''''''''''''''''''

-  Primary RID ranges should not overlap

   -  Reasoning: RIDs need to be unique inside one AD domain.



ipa-ad-trust-posix ranges:
''''''''''''''''''''''''''

-  since we do not do any RID -> UID/GID mapping for this ranges the RID
   range is of no use here, hence no checking needed, attribute should
   not be set at all.
-  https://fedorahosted.org/freeipa/ticket/4221



Local ranges
''''''''''''

-  RID ranges of any type (primary or secondary) should not overlap

   -  Reasoning: RIDs need to be unique inside one AD domain.