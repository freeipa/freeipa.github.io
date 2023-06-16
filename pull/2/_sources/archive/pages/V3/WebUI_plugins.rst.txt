\__NOTOC_\_

Overview
========

This document tries to describe new plugin infrastructure of FreeIPA Web
UI.

Upstream ticket:
`#3235 <https://fedorahosted.org/freeipa/ticket/3235>`__



Use Cases
=========

Example plugins to be added.

Design
======

.. _plugin_location_and_structure:

Plugin location and structure
-----------------------------

Plugin is registered by creating a directory *$plugin_name* in
*/usr/share/ipa/ui/js/plugins/*. $plugin_name is name of the plugin, it
has to start with a letter and may contain only ASCII alphanumeric
character, underscore \_ and dash - .

Plugin folder must contain *$plugin_name.js* file. It's have to be
either AMD module or a layer built by Dojo builder.

.. _plugin_index___registration:

Plugin index - registration
---------------------------

Web UI consist only of static files. It can't examine plugins directory
in during. Therefore it needs a some configuration file with listed
plugins to load. We can call this file a plugin index. It's located in
*/usr/share/ipa/ui/js/freeipa/plugins.js*.

Structure
~~~~~~~~~

Plugin index is a simple AMD module with array *plugins* containing
plugin names.

::

   define([], function() {
       return [
           'module-1',
           'module-2'
       ];
   });

.. _index_regeneration:

Index regeneration
~~~~~~~~~~~~~~~~~~

Plugin index is dynamically created by wsgi script:
*/usr/share/ipa/wsgi/plugins.py*. Httpd rewrite rule hides this fact and
makes sure that it can be access as at it's location.

Load
----

Plugins have to be loaded before registration phase, that means right
after implicit resource load phase. First we must load index and then
plugins.

.. _loading_index:

Loading index
-------------

Plugin index is located in *freeipa* package of FreeIPA Web UI,
therefore the easiest would be to add *./plugins* as a dependency in
*./app* module. But that would be wrong. *./app* module is part of
freeipa layer build and it would be included - it couldn't be updated
later. Hence, it must be loaded dynamically.

.. _loading_plugins:

Loading plugins
---------------

From AMD's perspective plugin is a package. Loader can't load it because
it doesn't know where it is located, so each plugin needs to be
registered in a loader with:

::

   {
       name: '$plugin_name',
       path: 'plugins/$plugin_name'
   }

Then, plugin may be dynamically loaded. Application should proceed to
registration phase after all plugins are loaded.

.. _extending_ui:

Extending UI
------------

At this point plugins can do whatever they want to do. In future we
should provide more or less stable plugin API. Consumers of the API
would care only about the API and not the underlying functionality. It
would allow to create some internal changes without breaking simple
plugins.

There are 3 main extension use cases:

-  adding completely new page
-  adding new UI fields to existing pages

   -  by modifying page spec
   -  by extending already created pages

Both cases of adding new UI fields are not straightforward.

.. _modifying_page_spec:

Modifying page spec
~~~~~~~~~~~~~~~~~~~

Spec of most pages are enclosed in function calls which are usually
evaluated right before creating a page. All spec should be moved to
static property of corresponding module. Spec should be completely
declarative. It should no contain any calls dependent on already
initialized app. Extensions should extend the spec at *alternation*
phase.

.. _modifying_already_created_pages:

Modifying already created pages
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Facets don't support content recreation. They expects that
*create(container)* method is called only once. New rules for facets and
widgets have to be set to support extensibility:

-  Web UI components should not be directly depend on HTML
   representation of other components (widget/facets)
-  each widget should support destroying and recreating its HTML
   representation
-  widget should not modify it's container
-  communication should be done mostly by events and topics (global
   event)

Implementation
==============

N/A



Feature Management
==================

UI
~~

N/A

CLI
~~~

N/A



Major configuration options and enablement
==========================================

N/A

Replication
===========

N/A



Updates and Upgrades
====================

N/A

Dependencies
============

N/A



External Impact
===============

*extension.js* file is no longer available. All existing extension
should be changed to new format.
