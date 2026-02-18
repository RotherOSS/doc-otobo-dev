.. _events:

Events
======

OTOBO provides an event-driven extension mechanism. **Event modules** (also called *event handlers* or *listeners*) are executed when specific actions occur in the system—such as creating or updating a ticket, sending an article, installing a package, or changing configuration objects.

The event system allows you to add custom behavior without modifying core code: you register one or more modules that listen to certain events, and OTOBO calls those modules whenever the event is triggered.

Important Notes
---------------

* **Event availability depends on the installation.**
  The set of events can vary between systems depending on installed packages (e.g. ITSM, FAQ), custom development, and configuration.
* The lists in this document describe the **events available in the current OTOBO standard**.
  Your system may provide additional events.
* Event modules must be robust:
  the ``Data`` payload may differ by event type and may not contain the same keys for every event.

Event Modules
-------------

An event module typically implements two methods:

* ``new()`` (constructor)
* ``Run()`` (handler)

The method ``Run()`` receives at least these parameters:

* ``Event``: the event name (string)
* ``UserID``: the acting user (integer)
* ``Data``: a reference to a hash containing event-related data

Depending on the event, ``Data`` may contain ticket data, article data, object identifiers, configuration deltas, and more.

Where Event Modules Live
------------------------

Event modules are usually grouped by object type and package. A common convention is to place modules in directories named ``Event`` below the relevant subsystem.

For example, ticket event modules are commonly located in::

   Kernel/System/Ticket/Event

Other packages and object types may follow similar conventions depending on implementation.

How Event Modules Are Configured
--------------------------------

Event modules are activated and ordered through system configuration (SysConfig). The naming convention depends on the subsystem.

For tickets, a common convention is settings starting with::

   Ticket::EventModulePost###

See the ticket configuration file for examples::

   Kernel/Config/Files/XML/Ticket.xml

Other subsystems may use different SysConfig setting prefixes.

Ticket Event Modules
====================

Ticket event modules are executed right after a ticket or article action has taken place.
By convention, these modules are located in::

   Kernel/System/Ticket/Event

Ticket Event Module Code Example
--------------------------------

See the files in the ``Kernel/System/Ticket/Event`` folder of the source code.

Ticket Event Module Configuration Example
-----------------------------------------

See the settings in ``Kernel/Config/Files/XML/Ticket.xml`` that start with the name
``Ticket::EventModulePost###``.

Ticket Event Module Use Case Examples
-------------------------------------

Unlock a ticket after a move action
   This standard feature is implemented with the ticket event module
   ``Kernel::System::Ticket::Event::ForceUnlock``. When this feature is not wanted,
   it can be turned off by unsetting the system configuration entry
   ``Ticket::EventModulePost###910-ForceUnlockOnMove``.

Perform extra cleanup action when a ticket is deleted
   A customized OTOBO might store additional data in custom database tables.
   When a ticket is deleted, this additional data needs to be deleted as well.
   This functionality can be implemented with a ticket event module listening to
   ``TicketDelete`` events.

Send data to an external service when a ticket is created
   A ticket event module listening to ``TicketCreate`` can send notifications
   or payloads to external services (e.g. webhooks, chat systems, monitoring).

Events in the OTOBO Standard
============================

The following event names are available in the **current OTOBO standard**.
Depending on your installation, additional events may be available.

Ticket
------

* ``TicketAccountTime``
* ``TicketArchiveFlagUpdate``
* ``TicketCreate``
* ``TicketCustomerUpdate``
* ``TicketDelete``
* ``TicketDynamicFieldUpdate_``
* ``TicketFlagDelete``
* ``TicketFlagSet``
* ``TicketLockUpdate``
* ``TicketMerge``
* ``TicketOwnerUpdate``
* ``TicketPendingTimeUpdate``
* ``TicketPriorityUpdate``
* ``TicketQueueUpdate``
* ``TicketResponsibleUpdate``
* ``TicketServiceUpdate``
* ``TicketSLAUpdate``
* ``TicketStateUpdate``
* ``TicketSubscribe``
* ``TicketTitleUpdate``
* ``TicketTypeUpdate``
* ``TicketUnlockTimeoutUpdate``
* ``TicketUnsubscribe``
* ``Notification``
* ``NotificationAddNote``
* ``NotificationFollowUp``
* ``NotificationLockTimeout``
* ``NotificationMove``
* ``NotificationNewTicket``
* ``NotificationOwnerUpdate``
* ``NotificationPendingReminder``
* ``NotificationResponsibleUpdate``
* ``NotificationServiceUpdate``
* ``HistoryAdd``
* ``HistoryDelete``
* ``LinkAdd``
* ``LinkDelete``

Ticket Article
--------------

* ``ArticleAgentNotification``
* ``ArticleAutoResponse``
* ``ArticleBounce``
* ``ArticleCreate``
* ``ArticleCustomerNotification``
* ``ArticleDynamicFieldUpdate``
* ``ArticleEdit``
* ``ArticleEmailSending``
* ``ArticleFlagDelete``
* ``ArticleFlagSet``
* ``ArticleSend``
* ``ArticleTransmissionErrorCreate``
* ``ArticleTransmissionErrorUpdate``
* ``ArticleUpdate``
* ``AttachmentAddPost``
* ``AttachmentDeletePost``

Package Manager
---------------

* ``PackageInstall``
* ``PackageReinstall``
* ``PackageUninstall``
* ``PackageUpgrade``

Configuration
-------------

* ``QueueCreate``
* ``QueueUpdate``
* ``LinkObjectLinkAdd``
* ``LinkObjectLinkDelete``
* ``DynamicFieldAdd``
* ``DynamicFieldDelete``
* ``DynamicFieldUpdate``

Calendar
--------

* ``AppointmentCreate``
* ``AppointmentDelete``
* ``AppointmentDynamicFieldUpdate_``
* ``AppointmentNotification``
* ``AppointmentUpdate``
* ``ParticipationCreate``
* ``ParticipationDelete``
* ``ParticipationUpdate``

Customer Company
----------------

* ``CustomerCompanyAdd``
* ``CustomerCompanyDelete``
* ``CustomerCompanyUpdate``

CustomerUser
------------

* ``CustomerUserAdd``
* ``CustomerUserDelete``
* ``CustomerUserUpdate``

Examples from Other Packages
============================

The following event names are examples from commonly used add-on packages.
Whether they are available depends on your installation.

ITSM Configitem
---------------

* ``ConfigItemCreate``
* ``ConfigItemDelete``
* ``ConfigItemDynamicFieldUpdate_``
* ``ConfigItemUpdate``
* ``VersionCreate``
* ``VersionDelete``
* ``VersionUpdate``
* ``DefinitionCreate``

FAQ
---

* ``FAQAttachmentAddPost``
* ``FAQAttachmentDeletePost``
* ``FAQCreate``
* ``FAQDelete``
* ``FAQDynamicFieldUpdate_``
* ``FAQUpdate``

How to Find Events on Your System
=================================

Since installations can provide additional events, you should verify locally which events
exist and which are actually triggered.

Common approaches:

* **Search the source code** for event trigger calls (for example, calls into an event handler).
* **Review SysConfig** entries that register event modules (e.g. settings containing ``EventModule``).
* **Inspect installed packages** for additional event providers and modules.
* **Use logging** in your own module to confirm that a given event is fired in your environment.

  .. note::

        The following command is a starting point to find event names in the source code.
        It may need adjustments based on your specific installation and event naming conventions.
    
        .. code-block:: bash
            rg -n --hidden --follow "\bEvent\s*=>\s*'[^']+'" Kernel Custom 2>/dev/null | sed -E "s/.*\bEvent\s*=>\s*'([^']+)'.*/\1/" | sort -u

Best Practices for Event Modules
================================

* Be defensive when reading ``Data``: not every event supplies the same keys.
* Keep runtime short; avoid long-running work inside ``Run()``.
* Handle repeated execution gracefully; some events can occur multiple times.
* Log errors clearly and avoid breaking the main transaction whenever possible.

``Run()`` Skeleton (Conceptual)
-------------------------------

A minimal handler structure typically looks like this (conceptual overview):

* ``new()``: create the object
* ``Run(Event => ..., UserID => ..., Data => ...)``: execute when the event fires

Use existing event modules from your installation as templates, and register your module
through SysConfig for the relevant subsystem.
