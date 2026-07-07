Package Management
==================

The OPM (OTOBO Package Manager) is a mechanism to distribute software packages for the OTOBO framework via HTTP, FTP or file upload.

For example, the OTOBO project offers OTOBO modules like a calendar, a file manager or web mail in OTOBO packages via online repositories on our FTP servers.
The packages can be managed (install, upgrade and uninstall) via the admin interface.


Package Distribution
--------------------

If you want to create an OPM online repository, just tell the OTOBO framework where the location is by activating the system configuration setting ``Package::RepositoryList`` and adding the new location there.
Then you will have a new select option in the package manager.

In your repository, create an index file for your OPM packages.
OTOBO just reads this index file and knows what packages are available.

::

   shell> bin/otobo.Console.pl Dev::Package::RepositoryIndex /path/to/repository/ > /path/to/repository/otobo.xml


Package Commands
----------------

You can use the following OPM commands over the admin interface or over ``bin/otobo.Console.pl`` to manage admin jobs for OPM packages.


Install
~~~~~~~

Install OPM packages.

-  Web: http://localhost/otobo/index.pl?Action=AdminPackageManager

-  CMD: ``bin/otobo.Console.pl Admin::Package::Install /path/to/package.opm``


Uninstall
~~~~~~~~~

Uninstall OPM packages.

-  Web: http://localhost/otobo/index.pl?Action=AdminPackageManager

-  CMD: ``bin/otobo.Console.pl Admin::Package::Uninstall /path/to/package.opm``


Upgrade
~~~~~~~

Upgrade OPM packages.

-  Web: http://localhost/otobo/index.pl?Action=AdminPackageManager

-  CMD: ``bin/otobo.Console.pl Admin::Package::Upgrade /path/to/package.opm``


List
~~~~

List all OPM packages.

-  Web: http://localhost/otobo/index.pl?Action=AdminPackageManager

-  CMD: ``bin/otobo.Console.pl Admin::Package::List``
