Development Environment
=======================

To facilitate the writing of OTOBO expansion modules, the creation of a development environment is necessary. The source code of OTOBO and additional public modules can be found on `GitHub <http://github.com/RotherOSS>`__.


Obtain the Source Code
----------------------

First of all a directory must be created in which the modules can be stored. Then switch to the new directory using the command line and clone the Git repository by using the following command:

.. code-block:: bash

   shell> git clone git@github.com:RotherOSS/otobo.git -b rel-10_1

For other versions like OTOBO 11.0.x:

.. code-block:: bash

   shell> git clone git@github.com:RotherOSS/otobo.git -b rel-11_0

Please configure the OTOBO system according to the `installation instructions`_.

.. _`installation instructions`: https://doc.otobo.de/


Useful Tools
------------

`OTOBOCodePolicy <https://github.com/RotherOSS/CodePolicy>`__ is a code quality checker that enforces the use of common coding standards also for the OTOBO development team. It is highly recommended to use it if you plan to make contributions. You can use it as a standalone test script or even register it as a git commit hook that runs every time that you create a commit. Please see `the module documentation <https://github.com/RotherOSS/CodePolicy/blob/master/doc/en/CodePolicy.xml>`__ for details.

.. code-block:: bash

   shell> git clone git@github.com:RotherOSS/CodePolicy.git

Tips and tricks
-------------------------

Debug syntax errors in OTOBO Perl files
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Change to the OTOBO Homedirectory:

.. code-block:: bash

   shell> docker exec -it otobo_web_1 bash
   # or for non docker
   shell> cd /opt/otobo

After that execute the syntax check:

.. code-block:: bash

   shell> perl -cw -I Custom/ -I Custom/Kernel/cpan-lib/ -I . -I Kernel/cpan-lib/ Path/To/The/OTOBO/perlfile.pm
