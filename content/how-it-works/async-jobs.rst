Async Jobs
===========

OTOBO provides a system for executing asynchronous jobs in the background. The system is based on the idea of having small units of work that can be executed independently and asynchronously, without blocking the main application flow. This allows for better performance and scalability, as well as improved user experience.


Load parent module
------------------

To create an asynchronous job, you need to create a new class that inherits from ``Kernel::System::AsynchronousExecutor``. This class will define the logic for executing the job and handling any errors that may occur.

.. code-block:: Perl

    use parent qw(Kernel::System::AsynchronousExecutor);


Use functions from parent module
--------------------------------

Once you have created your asynchronous job class, you can execute it by calling the ``AsyncCall`` method on any object that inherits from ``Kernel::System::AsynchronousExecutor``. This method takes several parameters, including the name of the function to execute, the parameters for the function, and the number of attempts to lock the task by the scheduler.

.. code-block:: Perl

    my $Success = $Self->AsyncCall(
        ObjectName               => 'Kernel::System::OtoboAI',    # optional, if not given the object is used from where
                                                                  # this function was called
        FunctionName             => 'GetAnswer',                  # the name of the function to execute
        FunctionParams           => \%Param,                      # a ref with the required parameters for the function
        Attempts                 => 1,                            # optional, default: 1, number of tries to lock the
                                                                  #   task by the scheduler
        MaximumParallelInstances => 0,                            # optional, default: 0 (unlimited), number of same
                                                                  #   function calls from the same object that can be
                                                                  #   executed at the same time
    );

The ``AsyncCall`` method will return a boolean value indicating whether the job was successfully scheduled for execution. If the job was successfully scheduled, it will be executed in the background by the OTOBO scheduler, and any results or errors will be handled according to the logic defined in your asynchronous job class.
