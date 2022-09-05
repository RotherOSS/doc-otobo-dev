Database Mechanism
==================

OTOBO comes with a database layer that supports different databases.

The database layer ``Kernel::System::DB`` has two input options: *SQL* and *XML*.


SQL
---

The SQL interface should be used for normal database actions (``SELECT``, ``INSERT``, ``UPDATE``, etc.).
It can be used like a normal Perl DBI interface. The limitation is that only a single statement handle may
be active per database object.


INSERT/UPDATE/DELETE statements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: Perl

    my $Name        = 'Arne ßaknussemm';
    my $YearOfBirth = 1662;
    my $DBObject    = $Kernel::OM->Get('Kernel::System::DB');

    # created
    $DBObject->Do(
        SQL  => 'INSERT INTO alchemists (name, birth_year) VALUES ( ?, ? )',
        Bind => [ \$Name, \$YearOfBirth ],
    );

    # update
    $Name =~ s/ß/S/;
    $YearOfBirth++;
    $DBObject->Do(
        SQL  => 'UPDATE alchemists SET name = ?, birth_year = ?',
        Bind => [ \$Name, \$YearOfBirth ],
    );

    # delete
    $DBObject->Do(
       SQL=> 'DELETE FROM alchemists WHERE birth_year > 2022',
    );


SELECT statement
~~~~~~~~~~~~~~~~

.. code-block:: Perl

    my $DBObject = $Kernel::OM->Get('Kernel::System::DB');

    # get the last 15 steps for transaction 123
    my $TransactionNumber = 123;
    my $SQL = qq{SELECT step_id FROM transaction_steps WHERE tn = ? ORDER BY step_id DESC};
    $DBObject->Prepare(
        SQL   => $SQL,
        Bind  => [ \$TransactionNumber ],
        Limit => 15,
    );

    my @StepIds;
    while (my @Row = $DBObject->FetchrowArray()) {
        push @StepIds, $Row[0];
    }

    # get description for all valid steps
    $DBObject->Prepare(
        SQL   => q{SELECT step_id, description FROM transaction_steps WHERE valid = 1},
    );

    my %StepIdToDescription;
    while (my ($StepId, $Description) = $DBObject->FetchrowArray()) {
        $StepIdToDescription{$StepId} = $Description;
    }

    # get the number of valid steps
    $DBObject->Prepare(
        SQL   => q{SELECT COUNT(*) FROM transaction_steps WHERE valid = 1},
    );

    # when there is number of results is known, then no loop is required
    my ($NumValid) = $DBObject->FetchRowArray();

    # loops can alse be avoided with SelectAll()
    # SelectAll() returns a reference to an array of array references
    my @ValidIds = map { $_->[0] } $DBObject->SelectAll(
        SQL => q{SELECT DESTINCT valid_id FROM transaction_steps ORDER BY valid_id ASC},
    )->@*;

.. note::

   Take care to use ``Limit`` as param and not in the SQL string because not all databases support ``LIMIT`` in SQL strings.

.. note::

   Use the ``Bind`` attribute whereever you can, especially for long statements. If you use ``Bind`` you do not need the function ``Quote()``.

.. note::

   Beware that `SelectAll()` may not be used within a loop over the `FetchrowArray()` results.

QUOTE
~~~~~

String:

.. code-block:: Perl

   my $QuotedString = $Kernel::OM->Get('Kernel::System::DB')->Quote("It's a problem!");

Integer:

.. code-block:: Perl

   my $QuotedInteger = $Kernel::OM->Get('Kernel::System::DB')->Quote('123', 'Integer');

Number:

.. code-block:: Perl

   my $QuotedNumber = $Kernel::OM->Get('Kernel::System::DB')->Quote('21.35', 'Number');

.. note::

   Please use the ``Bind`` attribute instead of ``Quote()`` where ever you can.

XML
---

The XML interface should be used for ``INSERT``, ``CREATE TABLE``, ``DROP TABLE`` and ``ALTER TABLE``. As this syntax is different from database to database,
using it makes sure that you write applications that can be used in all of them.


INSERT
~~~~~~

.. code-block:: XML

   <Insert Table="some_table">
       <Data Key="id">1</Data>
       <Data Key="description" Type="Quote">exploit</Data>
   </Insert>


CREATE TABLE
~~~~~~~~~~~~

Possible data types are: ``BIGINT``, ``SMALLINT``, ``INTEGER``, ``VARCHAR`` (Size=1-1000000), ``DATE`` (format: yyyy-mm-dd hh:mm:ss) and ``LONGBLOB``.

.. code-block:: XML

   <TableCreate Name="calendar_event">
       <Column Name="id" Required="true" PrimaryKey="true" AutoIncrement="true" Type="BIGINT"/>
       <Column Name="title" Required="true" Size="250" Type="VARCHAR"/>
       <Column Name="content" Required="false" Size="250" Type="VARCHAR"/>
       <Column Name="start_time" Required="true" Type="DATE"/>
       <Column Name="end_time" Required="true" Type="DATE"/>
       <Column Name="owner_id" Required="true" Type="INTEGER"/>
       <Column Name="event_status" Required="true" Size="50" Type="VARCHAR"/>
       <Index Name="calendar_event_title">
           <IndexColumn Name="title"/>
       </Index>
       <Unique Name="calendar_event_title">
           <UniqueColumn Name="title"/>
       </Unique>
       <ForeignKey ForeignTable="users">
           <Reference Local="owner_id" Foreign="id"/>
       </ForeignKey>
   </TableCreate>

``LONGBLOB`` columns need special treatment. Their content needs to be base64 transcoded if the database driver does not support the feature ``DirectBlob``. Please see the following example:

.. code-block:: Perl

   my $Content = $StorableContent;
   if ( !$DBObject->GetDatabaseFunction('DirectBlob') ) {
       $Content = MIME::Base64::encode_base64($StorableContent);
   }

Similarly, when reading from such a column, the content must not automatically be decoded as UTF-8 by passing the ``Encode => 0`` flag to ``Prepare()``:

.. code-block:: Perl

   return if !$DBObject->Prepare(
       SQL => '
           SELECT content_type, content, content_id, content_alternative, disposition, filename
           FROM article_data_mime_attachment
           WHERE id = ?',
       Bind   => [ \$AttachmentID ],
       Encode => [ 1, 0, 0, 0, 1, 1 ],
   );

   while ( my @Row = $DBObject->FetchrowArray() ) {

       $Data{ContentType} = $Row[0];

       # Decode attachment if it's e. g. a postgresql backend.
       if ( !$DBObject->GetDatabaseFunction('DirectBlob') ) {
           $Data{Content} = decode_base64( $Row[1] );
       }
       else {
           $Data{Content} = $Row[1];
       }
       $Data{ContentID}          = $Row[2] || '';
       $Data{ContentAlternative} = $Row[3] || '';
       $Data{Disposition}        = $Row[4];
       $Data{Filename}           = $Row[5];
   }


DROP TABLE
~~~~~~~~~~

.. code-block:: XML

   <TableDrop Name="calendar_event"/>


ALTER TABLE
~~~~~~~~~~~

The following shows an example of add, change and drop columns.

.. code-block:: XML

   <TableAlter Name="calendar_event">
       <ColumnAdd Name="test_name" Type="varchar" Size="20" Required="true"/>

       <ColumnChange NameOld="test_name" NameNew="test_title" Type="varchar" Size="30" Required="true"/>

       <ColumnChange NameOld="test_title" NameNew="test_title" Type="varchar" Size="100" Required="false"/>

       <ColumnDrop Name="test_title"/>

       <IndexCreate Name="index_test3">
           <IndexColumn Name="test3"/>
       </IndexCreate>

       <IndexDrop Name="index_test3"/>

       <UniqueCreate Name="uniq_test3">
           <UniqueColumn Name="test3"/>
       </UniqueCreate>

       <UniqueDrop Name="uniq_test3"/>
   </TableAlter>

The next shows an example how to rename a table.

.. code-block:: XML

   <TableAlter NameOld="calendar_event" NameNew="calendar_event_new"/>


Code to Process XML
~~~~~~~~~~~~~~~~~~~

.. code-block:: Perl

   my @XMLARRAY = @{$Self->ParseXML(String => $XML)};

   my @SQL = $Kernel::OM->Get('Kernel::System::DB')->SQLProcessor(
       Database => \@XMLARRAY,
   );
   push @SQL, $Kernel::OM->Get('Kernel::System::DB')->SQLProcessorPost();

   for my $Statement (@SQL) {
       $Kernel::OM->Get('Kernel::System::DB')->Do(SQL => $Statement);
   }


Database Drivers
----------------

The database drivers are located under ``$OTOBO_HOME/Kernel/System/DB/*.pm``.


Supported Databases
-------------------

-  MySQL or MariaDB
-  PostgreSQL
-  Oracle
-  Microsoft SQL Server (only for external database connections, not as OTOBO database)
