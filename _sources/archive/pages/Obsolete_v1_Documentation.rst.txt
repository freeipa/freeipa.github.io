.. _document_conventions:

Document Conventions
--------------------

This is perhaps going to be part of the main documentation, and explains
various conventions used throughout the doc to make it easier for
readers to understand.

Examples
--------

Throughout this documentation, examples are used to illustrate various
procedures, commands, and configurations. A typical IPA deployment will
include a server, one or more replicas, a domain, realm, IPA clients,
etc. These are named as follows:

-  Domain: *example.com*
-  Realm: *EXAMPLE.COM*
-  IPA Server: *ipaserver.example.com*
-  IPA Replica: *ipareplica.example.com*
-  IPA Clients:

   -  Generic client: *ipaclient.example.com*
   -  HP-UX client: *hpuxipaclient.example.com*
   -  RHEL 4 client: *rhel4ipaclient.example.com*

Where multiple IPA replicas or clients are used, these will be
identified in the examples where they are used.

Styles
------

The following is an attempt to organize some of the styles used
throughout the doc on this wiki. There's not a whole lot you can do with
styles in wikis (remember, I'm a writer, I'm fussy) so this could change
and at times not make much sense.

Headings
~~~~~~~~

Just use the normal wiki headings and use common sense. Use the TOC as a
guideline to determine if the result is sensible or not.

Filenames
~~~~~~~~~

Filenames and directories should be tagged <tt> inline. If they're part
of a <pre> set it doesn't matter.

Commands
~~~~~~~~

Use **bold** ('''bold'''). Don't include the path unless it seems
absolutely necessary. Distinguish between user and superuser commands
with the appropriate prompt:

   **# ipa-server-install**

   **$ ipa-finduser**

The thing you need to be aware of here is that (currently) you need to
add ``/usr/sbin`` to your path as a regular user, or change to that
directory in order to run the commands as described above. This is not
the case for the superuser, who has ``/usr/sbin`` in the path by
default.

.. _rpms_packages_etc.:

RPMs, Packages, etc.
~~~~~~~~~~~~~~~~~~~~

Use **``bold true-type``** ('''<tt>bold true-type</tt>''').

GUI
~~~

Most elements on a GUI will be in bold as well, such as field labels,
button names, etc.

Links
^^^^^

When referring to a link (e.g., on the WebUI), enclose it in double
quotes.

.. _attributes_parameters_etc.:

Attributes, Parameters, etc.
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Probably bold, unless I or somebody else comes up with a better idea.
They're mostly part of commands, which are bold, so it seems logical.
Some things are italic, but I don't have a definite system yet. I miss
docbook...

.. _notes_and_stuff:

Notes and stuff
~~~~~~~~~~~~~~~

I've started doing this. Time will tell...

   |Note.png| **Note:**

      The note text. The image is called Note.png.

..

   |Caution.png| **Caution:**

      The caution text. The image is called Caution.png.

If we decide it's necessary, I can add *Warning* and *Tip* as well.
These are all standards used throughout RH documentation.

To be continued...

.. |Note.png| image:: Note.png
.. |Caution.png| image:: Caution.png
