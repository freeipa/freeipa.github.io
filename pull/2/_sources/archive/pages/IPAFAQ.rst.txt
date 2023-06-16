This discussion page serves as the development page for the IPA FAQ.
Questions and answers are first posted here, finalized, and then moved
to the content page.

The following is a template for each section:

Heading
-------

#:**Q:** Can I do this?

#:**A:** No, this is not supported.

#. 

      **Q:** Can I do that?
      **A:** Yes. Use the blek tool to do that.

.. _general_queries:

General Queries
---------------

.. _migration_issues:

Migration Issues
----------------

#:**Q:** I have been running a pilot program with freeIPA with a number
of AD users. Is it possible to "link" these user accounts from freeIPA
to the same people's AD accounts, without deleting and re-creating all
the accounts? How are these accounts matched?

#:**A:** Yes, this is possible. The user entries coming from AD are
matched by name with the existing entries in IPA, and will "take over"
control of those entries. There is no need to delete and re-create them.
