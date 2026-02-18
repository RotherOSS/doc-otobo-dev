Using the power of the OTOBO module layers
==========================================

OTOBO has a large number of so-called *module layers* which make it very easy to extend the system without patching existing code. One example is the number generation mechanism for tickets. It is a *module layer* with pluggable modules, and you can add your own custom number generator modules if you wish to do so. Let's look at the different layers in detail!

.. toctree::
   :maxdepth: 3
   :caption: Authentication and User Management

   otobo-module-layers/agent-auth
   otobo-module-layers/auth-sync
   otobo-module-layers/customer-auth

.. toctree::
   :maxdepth: 3
   :caption: Preferences

   otobo-module-layers/customer-user-preferences
   otobo-module-layers/queue-preferences
   otobo-module-layers/service-preferences
   otobo-module-layers/sla-preferences

.. toctree::
   :maxdepth: 3
   :caption: Other Core Functions

   otobo-module-layers/log
   otobo-module-layers/outputfilter
   otobo-module-layers/stats
   otobo-module-layers/ticketnumber-generator
   otobo-module-layers/events

.. toctree::
   :maxdepth: 3
   :caption: Front End Modules

   otobo-module-layers/dashboard
   otobo-module-layers/notify
   otobo-module-layers/ticket-menu
   otobo-module-layers/rootapplication

.. toctree::
   :maxdepth: 3
   :caption: Generic Interface Modules

   otobo-module-layers/gi-transport
   otobo-module-layers/gi-mapping
   otobo-module-layers/gi-invoker
   otobo-module-layers/gi-operation

.. toctree::
   :maxdepth: 3
   :caption: Daemon And Scheduler

   otobo-module-layers/daemon/daemon-modules
   otobo-module-layers/daemon/scheduler-task-worker-modules

.. toctree::
   :maxdepth: 3
   :caption: Dynamic Fields

   otobo-module-layers/dynamic-fields/dynamic-fields-overview
   otobo-module-layers/dynamic-fields/dynamic-fields-framework
   otobo-module-layers/dynamic-fields/dynamic-fields-interaction
   otobo-module-layers/dynamic-fields/dynamic-fields-extend-options
   otobo-module-layers/dynamic-fields/dynamic-fields-new-field
   otobo-module-layers/dynamic-fields/dynamic-fields-extend

.. toctree::
   :maxdepth: 3
   :caption: Email Handling

   otobo-module-layers/ticket-postmaster-modules

.. toctree::
   :maxdepth: 3
   :caption: Process Management

   otobo-module-layers/process-management-modules

.. toctree::
   :maxdepth: 3
   :caption: Overriding Core Modules

   otobo-module-layers/autoload-modules
