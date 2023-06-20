IPAv2_UI_Framework\_-_Opening_a_Page
====================================



Opening a Page
==============

When a page is being opened, the following operations are executed:

-  Updating navigation
-  Setting up entity
-  Setting up action panel
-  Creating client areas's layout
-  Setting up client area's look and feel
-  Loading data



Updating Navigation
===================

The hashchange handler will invoke nav_update_tabs() to update the
navigation.



Setting up Entity
=================

The nav_update_tabs() will invoke entity.setup() to display an entity.
The entity will display the selected facet.



Setting up Action Panel
=======================

The entity.setup() will invoke facet.setup_views() which will create the
action panel.



Creating Client Area's Layout
=============================

The entity.setup() will invoke facet.create() to create the page layout
or load it from a template.



Setting up Client Area's Look and Feel
======================================

The entity.setup() will invoke facet.setup() to setup the page's look
and feel which may include:

-  removing template sections of the HTML page
-  replacing HTML components
-  setting event handlers



Loading Data
============

The entity.setup() will invoke facet.load() to load the initial data.