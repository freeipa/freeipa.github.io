.. _patch_format:

Patch format
============

We have accepted several conventions and techniques that helps keeping
track of patches, ACKs, and commits.

.. _patch_format_1:

Patch format
------------

All patches should be in a format to apply cleanly with

``git am``

This is produced in two steps:

1. in a git repository with your changes create a *commit*:

| ``git add .``
| ``git commit``

Appropriate commit message should be provided, e.g.:

::

   Add new command compat-is-enabled

   Add a new API command 'compat-is-enabled' which can be used to determine
   whether Schema Compatibility plugin is configured to serve trusted domain
   users and groups. The new command is not visible in IPA CLI.

   https://fedorahosted.org/freeipa/ticket/3671
   https://fedorahosted.org/freeipa/ticket/3672

Good and concise commit description is an integral part of the patch
itself. Make sure that the commit message contains enough information so
that the reader can understand why was the patch done and what it fixes
or enhances. If the patch adresses a ticket in Trac, the last line of
the commit should be the URL to that ticket.

2. create the patch using the command

``git format-patch -M -C --patience --full-index -1``

Naming
------

Format: ``project-username-seq[-update]-description.patch``

-  *project*: always *freeipa*
-  *username*: your Fedora account name
-  *seq*: sequence number. Please, try to not skip numbers, as we will
   use this number to ensure all patches from a given contributor get
   reviewed. The sequence is per developer, so if you are submitting
   your first patch, it should be 0001.
-  *update*: if a patch requires modification and additional changes
   prior to submission, append a number starting at 2 and increasing by
   one for each update. Thus, if the above patch required additional
   changes, the first would be:

      freeipa-edewata-0019-2-Certificate-management-for-services.patch
      and then
      freeipa-edewata-0019-3-Certificate-management-for-services.patch

-  *description*: this is the first line of the git commit, and should
   be less than six words long (ideally two or three). git format-patch
   command will translate this line into the subject of the patch, with
   hyphens replacing the white space.

Example patch name:
``freeipa-edewata-0019-Certificate-management-for-services.patch``

.. _ensuring_correct_transmission_of_patches:

Ensuring correct transmission of patches
----------------------------------------

Some MTAs (perhaps mailman?) have a nasty habit of prepending a ``'>'``
character to the patch (quoting the ``"From <SHA>"`` line), messing up
the format. This occurs when the attachment is 7bit clear; the sender
can avoid this by sending the patch with an alternative
Content-Transfer-Encoding.

To configure Thundebird:

-  In Thunderbird Preferences,
-  Go to the Advanced->General tab.
-  Select "Config Editor"
-  Search for ``mail.file_attach_binary`` and
-  Set this value to ``true``

To configure Mutt:

-  Add ``set encode_from=yes`` to your ``muttrc``.
