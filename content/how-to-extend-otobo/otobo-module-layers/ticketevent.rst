Ticket Event Module
===================

Ticket event modules are running right after a ticket or an article action took place.
Per convention these modules are located in the directory ``Kernel/System/Ticket/Event``. A ticket event module needs only two functions: ``new()`` and ``Run()``. The method ``Run()`` receives at least the parameters ``Event``, ``UserID`` and ``Data``. The parameter ``Data`` is a referecne to a hash.
It contains data of the ticket and, in the case of article related events, also data of the article.


Ticket Event Module Code Example
--------------------------------

See the files in ``Kernel/System/Ticket/Event`` folder of the source code.


Ticket Event Module Configuration Example
-----------------------------------------

See the settings in ``Kernel/Config/Files/XML/Ticket.xml`` that are starting with the name ``Ticket::EventModulePost###``.


Ticket Event Module Use Case Example
------------------------------------

A ticket should be unlocked after a move action
   This standard feature has been implemented with the ticket event module ``Kernel::System::Ticket::Event::ForceUnlock``. When this feature is not wanted, then it can be turned off by unsetting the system configuration entry ``Ticket::EventModulePost###910-ForceUnlockOnMove``.

Perform extra cleanup action when a ticket is deleted
   A customized OTOBO might hold non-standard data in additional database tables. When a ticket is deleted then this additional data needs to be deleted. This functionality can be achieved with a ticket event module listening to ``TicketDelete`` events.

New tickets should be twittered
   A ticket event module listening to ``TicketCreate`` can send out tweets.


Ticket and Article Events
-------------------------

Available ticket events:

- ``TicketCreate``
- ``TicketDelete``
- ``TicketTitleUpdate``
- ``TicketUnlockTimeoutUpdate``
- ``TicketQueueUpdate``
- ``TicketTypeUpdate``
- ``TicketServiceUpdate``
- ``TicketSLAUpdate``
- ``TicketCustomerUpdate``
- ``TicketPendingTimeUpdate``
- ``TicketLockUpdate``
- ``TicketArchiveFlagUpdate``
- ``TicketStateUpdate``
- ``TicketOwnerUpdate``
- ``TicketResponsibleUpdate``
- ``TicketPriorityUpdate``
- ``HistoryAdd``
- ``HistoryDelete``
- ``TicketAccountTime``
- ``TicketMerge``
- ``TicketSubscribe``
- ``TicketUnsubscribe``
- ``TicketFlagSet``
- ``TicketFlagDelete``
- ``EscalationResponseTimeNotifyBefore``
- ``EscalationUpdateTimeNotifyBefore``
- ``EscalationSolutionTimeNotifyBefore``
- ``EscalationResponseTimeStart``
- ``EscalationUpdateTimeStart``
- ``EscalationSolutionTimeStart``
- ``EscalationResponseTimeStop``
- ``EscalationUpdateTimeStop``
- ``EscalationSolutionTimeStop``
- ``NotificationNewTicket``
- ``NotificationFollowUp``
- ``NotificationLockTimeout``
- ``NotificationOwnerUpdate``
- ``NotificationResponsibleUpdate``
- ``NotificationAddNote``
- ``NotificationMove``
- ``NotificationPendingReminder``
- ``NotificationEscalation``
- ``NotificationEscalationNotifyBefore``
- ``NotificationServiceUpdate``

Available article events:

- ``ArticleCreate``
- ``ArticleUpdate``
- ``ArticleSend``
- ``ArticleBounce``
- ``ArticleAgentNotification``
- ``ArticleCustomerNotification``
- ``ArticleAutoResponse``
- ``ArticleFlagSet``
- ``ArticleFlagDelete``
- ``ArticleAgentNotification``
- ``ArticleCustomerNotification``
