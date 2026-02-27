IPAv2_UI_Framework\_-_Tutorial
==============================



Defining an Entity
==================

An entity can be defined as follows:

::

   function ipa_service() {

       // inherit from ipa_entity
       var that = ipa_entity({
           'name': 'service'
       });

       that.superior_init = that.superior('init');

       that.init = function() {

           // create associations
           var association = ...
           that.add_association(association);

           // create dialog boxes
           var dialog = ...
           that.add_dialog(dialog);

           // create facets
           var facet = ...
           that.add_facet(facet);


           // invoke entity's init()
           that.superior_init();
       };

       return that;
   }



Defining an Add Dialog Box
==========================

An add dialog box can be defined as follows:

::

   function ipa_service_add_dialog(spec) {

       spec = spec || {};

       // inherit from ipa_add_dialog
       var that = ipa_add_dialog(spec);

       that.superior_init = that.superior('init');

       that.init = function() {

           var field = ...
           that.add_field(field);

           // invoke add dialog's init()
           this.superior_init();
       };

       return that;
   }



Defining a Search Facet
=======================

A search facet can be defined as follows:

::

   function ipa_service_search_facet(spec) {

       spec = spec || {};

       // inherit from ipa_search_facet
       var that = ipa_search_facet(spec);

       that.superior_init = that.superior('init');

       that.init = function() {

           // create columns
           var column = ...
           that.add_column(column);

           // invoke search facet's init()
           that.superior_init();
       };

       return that;
   }



Defining a Details Facet
========================

A details facet can be defined as follows:

::

   function ipa_service_details_facet(spec) {

       spec = spec || {};

       // inherit from ipa_details_facet
       var that = ipa_details_facet(spec);

       that.superior_init = that.superior('init');

       that.init = function() {

           // create section
           var section = ...
           that.add_section(section);

           // create field
           var field = ...
           section.add_field(field);

           // invoke details facet's init()
           that.superior_init();
       };

       return that;
   }



Registering Entity
==================

The entity should be registered as follows:

::

   var entity = ipa_service();
   IPA.add_entity(entity);



Defining Navigation
===================

The navigation can be defined as follows:

::

   var admin_tab_set = [
       {name:'identity', children:[
           {name:'service', label:'Services', setup: ipa_entity_setup}
       ]}
   ];