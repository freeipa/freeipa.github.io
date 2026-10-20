Reworking_wiki_pages_procedure
==============================

This is the procedure I'm using to work through wiki pages to make them
easier to index and find. This also creates a series of categories and
sub-categories, which is an essential tool in MediaWiki.

Concepts
--------

-  Each page should be in at least one category.
-  Each category should be a sub-category to another, with only a few
   master categories.
-  Page names need to including spaces, following MediaWiki/Wikipedia
   best practice (search doesn't like CamelCase; better for l10n;
   natural, descriptive language titles are the best practice from
   Mw/Wikipedia; easier to remember; less exclusive knowledge required
   to understand title. Must manually re-link, redirect is automatic.)

Procedure
---------

#. Take the next page in
   `Special:Uncategorizedpages <Special:Uncategorizedpages>`__.
#. If the page name is CamelCase or is very undescriptive, use the
   `#Page renaming procedure <#Page_renaming_procedure>`__.
#. Once the name is established/confirmed, add the page to one or more
   categories

   -  Use the `:Category:How to <:Category:How_to>`__ category to good
      effect.
   -  If it is in any way useful documentation, add it to
      `:Category:Documentation <:Category:Documentation>`__ or as part
      of a sub-category collection that is gathered in the docs
      category.

#. Create description text for all new categories
#. Add new categories to super-categories by putting [[Category:Super
   category name]] in the content for the sub-category.
#. Keep creating categories and descriptions until you get to the master
   category, creating and adding a description when necessary.
#. Go back to the start.



Page renaming procedure
----------------------------------------------------------------------------------------------

#. Use the link for `What links here <Special:Whatlinkshere>`__ to
   establish a list of pages or redirects that need to be fixed with the
   new name.

   -  Yes, this is manual, sorry.
   -  However, when you move the page, there is an automatic creation of
      a redirect. You must fix any incoming redirects to not be double
      redirects, as per MW best practice (cf.
      `Special:DoubleRedirects <Special:DoubleRedirects>`__.)

#. Use the link to **Move** the page, filling in the new name and a
   reason for the renaming

   -  Seek natural language, descriptive titles that are easy to l10n.
   -  Pages should not be in Articial/Nested/Directories. These are not
      search or information friendly; they require understanding of an
      informational hierarchy to find information. MediaWiki prefers a
      flat series of individually named articles. Categories are the
      tool for creating an integrated info hierarchy.