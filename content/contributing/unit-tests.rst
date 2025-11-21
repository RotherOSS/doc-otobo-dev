Unit Tests
==========

OTOBO provides a test suite which can be used to develop and run unit tests for all system related code.


Creating a Test File
--------------------

The test files are stored in ``.t`` files under ``scripts/test``. For example, let's take a look at the file ``scripts/test/Calendar.t`` for the *Calendar* class.

Every test file should ideally instantiate unit test helper object at the start, so it can benefit from some built-in methods provided by it:

.. code-block:: Perl

   # --
   # Copyright (C) 2001-2024 Rother OSS GmbH, https://otobo.de/
   # --
   # This software comes with ABSOLUTELY NO WARRANTY. For details, see
   # the enclosed file COPYING for license information (GPL). If you
   # did not receive this file, see https://www.gnu.org/licenses/gpl-3.0.txt.
   # --

   use strict;
   use warnings;
   use utf8;

   use vars (qw($Self));

   $Kernel::OM->ObjectParamAdd(
       'Kernel::System::UnitTest::Helper' => {
           RestoreDatabase => 1,
       },
   );
   my $Helper = $Kernel::OM->Get('Kernel::System::UnitTest::Helper');

By providing ``RestoreDatabase`` parameter to helper constructor, any database statement executed during the unit test will be rolled back at the end, making sure no permanent change has been done.

Like any other test suite, OTOBO provides assertion methods which can be used to test conditions. For example, this is how we create a test user and test that it has been indeed created:

.. code-block:: Perl

   my $UserLogin = $Helper->TestUserCreate();
   my $UserID = $UserObject->UserLookup( UserLogin => $UserLogin );

   $Self->True(
       $UserID,
       "Test user $UserID created"
   );

Please consult API section below for complete list of assertion methods.

It's always good practice to create random data in unit tests, which can help distinguish it from previously added data. Use random methods from API to get the strings and include them in your parameters:

.. code-block:: Perl

   my $RandomID = $Helper->GetRandomID();

   # create test group
   my $GroupName = 'test-calendar-group-' . $RandomID;
   my $GroupID   = $GroupObject->GroupAdd(
       Name    => $GroupName,
       ValidID => 1,
       UserID  => 1,
   );

   $Self->True(
       $GroupID,
       "Test group $GroupID created"
   );

Good developers make their unit test easy to maintain. Consider putting all test cases in an array and then iterate over them with some code. This will provide an easy way to extend the test later:

.. code-block:: Perl

   #
   # Tests for CalendarCreate()
   #
   my @Tests = (
       {
           Name    => 'CalendarCreate - No params',
           Config  => {},
           Success => 0,
       },
       {
           Name   => 'CalendarCreate - All required parameters',
           Config => {
               CalendarName => "Calendar-$RandomID",
               Color        => '#3A87AD',
               GroupID      => $GroupID,
               UserID       => $UserID,
           },
           Success => 1,
       },
       {
           Name   => 'CalendarCreate - Same name',
           Config => {
               CalendarName => "Calendar-$RandomID",
               Color        => '#3A87AD',
               GroupID      => $GroupID,
               UserID       => $UserID,
           },
           Success => 0,
       },
   );

   for my $Test (@Tests) {

       # make the call
       my %Calendar = $CalendarObject->CalendarCreate(
           %{ $Test->{Config} },
       );

       # check data
       if ( $Test->{Success} ) {
           for my $Key (qw(CalendarID GroupID CalendarName Color CreateTime CreateBy ChangeTime ChangeBy ValidID)) {
               $Self->True(
                   $Calendar{$Key},
                   "$Test->{Name} - $Key exists",
               );
           }

           KEY:
           for my $Key ( sort keys %{ $Test->{Config} } ) {
               next KEY if $Key eq 'UserID';

               $Self->IsDeeply(
                   $Test->{Config}->{$Key},
                   $Calendar{$Key},
                   "$Test->{Name} - Data for $Key",
               );
           }
       }
       else {
           $Self->False(
               $Calendar{CalendarID},
               "$Test->{Name} - No success",
           );
       }
   }

Selenium Tests
--------------

Selenium test scripts need a running Selenium server. This server is configured in the SysConfig using the key ``SeleniumTestsConfig``. It that setting
does not exist, then the tests should not be executed. In OTOBO 11.1.x this check can be achieved by using the module ``Test2::Require::OTOBO::Selenium``.
In older versions the method ``Kernel::System::UnitTest::Selenium::RunTest()`` does the checking.

Prerequisites for Testing
-------------------------

To be able to run the unit tests, you need to have all optional environment dependencies (Perl modules and other modules) installed, except those for different database back ends than what you are using. Run ``bin/otobo.CheckEnvironment.pl`` to verify your module installation.

You also need to have an instance of the OTOBO web front end running on the FQDN that is configured in your local OTOBO's ``Config.pm`` file. This OTOBO instance must use the same database that is configured for the unit tests.


Testing
-------

To run your tests, just use ``bin/otobo.Console.pl Dev::UnitTest::Run --test Calendar`` to use ``scripts/test/Calendar.t``.

.. code-block:: bash

   shell:/opt/otobo> bin/otobo.Console.pl Dev::UnitTest::Run --test Calendar
   +-------------------------------------------------------------------+
   /opt/otobo/scripts/test/Calendar.t:
   +-------------------------------------------------------------------+
   .................................................................................................
   =====================================================================
   yourhost ran tests in 2s for OTOBO 10.0.x git
   All 97 tests passed.
   shell:/opt/otobo>

You can even run several tests at once, just supply additional ``--test`` arguments to the command:

.. code-block:: bash

   shell:/opt/otobo> bin/otobo.Console.pl Dev::UnitTest::Run --test Calendar --test Appointment
   +-------------------------------------------------------------------+
   /opt/otobo/scripts/test/Calendar.t:
   +-------------------------------------------------------------------+
   .................................................................................................
   +-------------------------------------------------------------------+
   /opt/otobo/scripts/test/Calendar/Appointment.t:
   +-------------------------------------------------------------------+
   ..................................................................................................................
   =====================================================================
   yourhost ran tests in 5s for OTOBO 10.0.x git
   All 212 tests passed.
   shell:/opt/otobo>

If you execute ``bin/otobo.Console.pl Dev::UnitTest::Run`` without any argument, it will run all tests found in the system. Please note that this can take some time to finish.

Provide ``--verbose`` argument in order to see messages about successful tests too. Any errors encountered during testing will be displayed regardless of this switch, provided they are actually raised in the test.


Unit Test API
-------------

OTOBO provides API for unit testing that was used in the previous example. Here we'll list the most important functions, please also see the online API reference of ```Kernel::System::UnitTest`` <https://otobo.github.io/doc/api/otobo/8.0/Perl/Kernel/System/UnitTest.pm.html>`__.

``True()``
   This function tests whether given scalar value is a true value in Perl.

   .. code-block:: Perl

      $Self->True(
          1,
          'Scalar 1 is always evaluated as true'
      );

``False()``
   This function tests whether given scalar value is a false value in Perl.

   .. code-block:: Perl

      $Self->False(
          0,
          'Scalar 0 is always evaluated as false'
      );

``Is()``
   This function tests whether the given scalar variables are equal.

   .. code-block:: Perl

      $Self->Is(
          $A,
          $B,
          'Test Name',
      );

``IsNot()``
   This function tests whether the given scalar variables are unequal.

   .. code-block:: Perl

      $Self->IsNot(
          $A,
          $B,
          'Test Name'
      );

``IsDeeply()``
   This function compares complex data structures for equality. ``$A`` and ``$B`` have to be references.

   .. code-block:: Perl

      $Self->IsDeeply(
          $A,
          $B,
          'Test Name'
      );

``IsNotDeeply()``
   This function compares complex data structures for inequality. ``$A`` and ``$B`` have to be references.

   .. code-block:: Perl

      $Self->IsNotDeeply(
          $A,
          $B,
          'Test Name'
      );

``GetRandomID()``
   This function creates a random ID that can be used in tests as a unique identifier. It is guaranteed that within a test this function will never return a duplicate.

   .. note::

      Please note that these numbers are not really random and should only be used to create test data.

   .. code-block:: Perl

      my $RandomID = $Helper->GetRandomID();
      # $RandomID = 'test6326004144100003';

``TestUserCreate()``
   This function creates a test user that can be used in tests. It will be set to invalid automatically during the destructor. It returns the login name of the new user, the password is the same.

   .. code-block:: Perl

      my $TestUserLogin = $Helper->TestUserCreate(
          Groups   => ['admin', 'users'],          # optional, list of groups to add this user to (rw rights)
          Language => 'de',                        # optional, defaults to 'en' if not set
      );

``FixedTimeSet()``
   This functions makes it possible to override the system time as long as this object lives. You can pass an optional time parameter that should be used, if not, the current system time will be used.

   .. note::

      All calls to methods of ``Kernel::System::Time`` and ``Kernel::System::DateTime`` will use the given time afterwards.

   .. code-block:: Perl

      $HelperObject->FixedTimeSet(366475757);         # with Timestamp
      $HelperObject->FixedTimeSet($DateTimeObject);   # with previously created DateTime object
      $HelperObject->FixedTimeSet();                  # set to current date and time

``FixedTimeUnset()``
   This functions restores the regular system time behavior.

``FixedTimeAddSeconds()``
   This functions adds a number of seconds to the fixed system time which was previously set by ``FixedTimeSet()``. You can pass a negative value to go back in time.

``ConfigSettingChange()``
   This functions temporarily changes a configuration setting system wide to another value, both in the current instance of the ``ConfigObject`` and also in the system configuration on disk. This will be reset when the ``Helper`` object is destroyed.

   .. note::

      Please note that this will not work correctly in clustered environments.

   .. code-block:: Perl

      $Helper->ConfigSettingChange(
          Valid => 1,            # (optional) enable or disable setting
          Key   => 'MySetting',  # setting name
          Value => { ... } ,     # setting value
      );

``CustomCodeActivate()``
   This function will temporarily include custom code in the system. For example, you may use this to redefine a subroutine from another class. This change will persist for remainder of the test. All code will be removed when the ``Helper`` object is destroyed.

   .. note::

      Please note that this will not work correctly in clustered environments.

   .. code-block:: Perl

      $Helper->CustomCodeActivate(
          Code => q^
      use Kernel::System::WebUserAgent;
      package Kernel::System::WebUserAgent;
      use strict;
      use warnings;
      {
          no warnings 'redefine';
          sub Request {
              my $JSONString = '{"Results":{},"ErrorMessage":"","Success":1}';
              return (
                  Content => \$JSONString,
                  Status  => '200 OK',
              );
          }
      }
      1;^,
          Identifier => 'News',   # (optional) Code identifier to include in file name
      );

``ProvideTestDatabase()``
   This function will provide a temporary database for the test. Please first define test database settings in ``Kernel/Config.pm``, i.e:

   .. code-block:: Perl

      $Self->{TestDatabase} = {
          DatabaseDSN  => 'DBI:mysql:database=otobo_test;host=127.0.0.1;',
          DatabaseUser => 'otobo_test',
          DatabasePw   => 'otobo_test',
      };

   The method call will override global database configuration for duration of the test, i.e. temporary database will receive all calls sent over system ``DBObject``.

   All database contents will be automatically dropped when the ``Helper`` object is destroyed.

   This method returns ``undef`` in case the test database is not configured. If it is configured, but the supplied XML cannot be read or executed, this method will ``die()`` to interrupt the test with an error.

   .. code-block:: Perl

      $Helper->ProvideTestDatabase(
          DatabaseXMLString => $XML,      # (optional) OTOBO database XML schema to execute
                                          # or
          DatabaseXMLFiles => [           # (optional) List of XML files to load and execute
              '/opt/otobo/scripts/database/otobo-schema.xml',
              '/opt/otobo/scripts/database/otobo-initial_insert.xml',
          ],
      );
