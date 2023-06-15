\__NOTOC_\_

Overview
========

It's expected that Web UI plugins want to add new pages. They need a way
how to modify application navigation, for user to be able to navigate to
new pages. It's not an easy task to do in FreeIPA up to version 3.0.
This enhancement tries to make the task easy.



Use Cases
=========

Server plugin defines a new object type. Web UI needs to be extended to
support modification of objects of the new type.

Design
======

Web UI internals and mostly navigation code undertook a major
refactoring which is temporarily `documented
outside <http://pvoborni.fedorapeople.org/doc/navigation.html>`__ of
this wiki.

This refactoring introduces a menu object which contains an menu item
object store. Menu object is observed by menu widget. Changes in menu
object are therefore reflected in the UI.

Plugins should use new menu proxy located in *freeipa/menu* module for
performing modifications of menu. The main benefit of the proxy is that
plugins don't have to care about knowing how to get an instance of the
menu object.

This proxy has following methods:

-  ::

      add_item(item, parent, options)

   Add new menu item. Look into
   */install/ui/src/freeipa/navigation/Menu.js* for all available
   properties of item spec.

-  ::

      remove_item(item)

   Remove menu item, *item* is menu item object or menu item name.

-  ::

      query(query, search_options)

   Query menu store. Uses Dojo SimpleQueryEngine.

-  ::

      get()

   Get instance of menu object for advanced use cases.

Examples
--------

.. _add_menu_item:

Add menu item
~~~~~~~~~~~~~

Example with specified dependencies and recommended phase.

::

   define([
           'freeipa/menu',
           'freeipa/phases'
          ],
          function(menu, phases) {

       var add_item = function() {
           // check for module enablement 

           // add new item into identity section
           menu.add_item({ entity: 'dhcp' }, 'identity');
           // Note: each menu item has a name, if not specified in spec it is 
           // based on entity name or facet name. Each name contains its parent's
           // name as a prefix. The added item's name will be: 'identity/dhcp'
       };

       // add menu item after default menu items are added
       phase.on('profile', add_item);
   });

.. _remove_menu_item:

Remove menu item
~~~~~~~~~~~~~~~~

::

       // remove previously added item
       menu.remove_item('identity/dhcp');

Query
~~~~~

::

       // get added item
       var item = menu.get({name: 'identity/dhcp'})[0];

.. _observing_menu_store:

Observing menu store
~~~~~~~~~~~~~~~~~~~~

::

       var m = menu.get(); // get current menu instance

       // observes modifications of internal menu store
       var handler = function(object, removedFrom, insertedInto) {};
       var q = m.items.query();
       q.observe(handler, true);

       // using dojo/on
       // invokes select handler when menu item is selected (set by application controller)
       var select_hanler = function(select_state) {};
       on(m, 'selected', select_hanler);

Implementation
==============

N/A



Feature Management
==================

UI
~~

No visible UI changes.

CLI
~~~

No CLI changes.



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

N/A



RFE Author
==========

pvoborni
