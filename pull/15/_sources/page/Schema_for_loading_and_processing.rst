Schema_for_loading_and_processing
=================================

The following files are the files that contain just attributes and
object classes extracted from the pages `DS Design
Summary <FreeIPAv2:DS_Design_Summary>`__ and `DS Design Summary
2 <FreeIPAv2:DS_Design_Summary_2>`__ respectfully.

-  ExtractedBase.txt
-  ExtractedPolicy.txt

Take a note that there is a section commented out int the first files
and replaced with a temporary placeholder. This is dues to the fact that
currently the DS does not support the syntax we need for those
attributes. As soon as the syntax issue is addressed we would be able to
remove the placeholder and use the proper definition.

The following awk script does some formatting of the files above, uses
proper terms and also assigns OIDs to attributes and objects.

eobj.awk

These are the two files generated using the script above.

-  `60basev2.ldif <http://git.fedorahosted.org/cgit/freeipa.git/tree/install/share/60basev2.ldif?h=ipa-2-0>`__
-  `60policyv2.ldif <http://git.fedorahosted.org/cgit/freeipa.git/tree/install/share/60policyv2.ldif?h=ipa-2-0>`__

The commands that were used are:

| `` awk -f eobj.awk ebase1.txt > 60basev2.ldif``
| `` awk -v va=28 -v vo=12 -f eobj.awk ebase2.txt > 60policyv2.ldif``

the parameters indicate the starting number for attribute (va) and class
(vo) OIDs.

To take advantage of the v2 schema:

-  Drop the files into the /etc/dirsrv/slapd-/schema.
-  One of the attributes we plan to use contradicts with the attribute
   defined in 05rfc2247.ldif so move this ldif out of the directory - it
   is not used in IPA. We will address this collision later in the scope
   of IPA v2.
-  Restart 'dirsrv' service