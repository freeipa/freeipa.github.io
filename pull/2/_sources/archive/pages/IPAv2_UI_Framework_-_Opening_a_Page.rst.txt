.. _opening_a_page:

Opening a Page
==============

When a page is being opened, the following operations are executed:

-  Updating navigation
-  Setting up entity
-  Setting up action panel
-  Creating client areas's layout
-  Setting up client area's look and feel
-  Loading data

.. _updating_navigation:

Updating Navigation
===================

The hashchange handler will invoke nav_update_tabs() to update the
navigation.

.. _setting_up_entity:

Setting up Entity
=================

The nav_update_tabs() will invoke entity.setup() to display an entity.
The entity will display the selected facet.

.. _setting_up_action_panel:

Setting up Action Panel
=======================

The entity.setup() will invoke facet.setup_views() which will create the
action panel.

.. _creating_client_areas_layout:

Creating Client Area's Layout
=============================

The entity.setup() will invoke facet.create() to create the page layout
or load it from a template.

.. _setting_up_client_areas_look_and_feel:

Setting up Client Area's Look and Feel
======================================

The entity.setup() will invoke facet.setup() to setup the page's look
and feel which may include:

-  removing template sections of the HTML page
-  replacing HTML components
-  setting event handlers

.. _loading_data:

Loading Data
============

The entity.setup() will invoke facet.load() to load the initial data.
