Autoload Modules
================

Autoload modules are a measure of last resort when the regular module layers do not suffice. These modules live in the namespace ``Kernel::Autoload``.
They are required whenever a new config is loaded. This assures that they are available in the web interface and in just about all scripts.
The primary use of autoload modules is to add, or to override, core functionality. Please handle with care!

Autoload Module Configuration Example
-------------------------------------------------

In order to activate an autoload module it must be declared in the system configuration. See for example
the activation of our sample module ``Kernel::Autoload::Test`` in ``Kernel/Config/Files/XML/Framework.xml``.

.. code-block:: XML

    <Setting Name="AutoloadPerlPackages###1000-Test" Required="0" Valid="0">
        <Description Translatable="1">Example package autoload configuration.</Description>
        <Navigation>Core::Autoload</Navigation>
        <Value>
            <Array>
                <Item ValueType="String">Kernel::Autoload::Test</Item>
            </Array>
        </Value>
    </Setting>

Adding Methods to Core Modules
------------------------------

Methods are added simple by declaring a subroutine in the target namespace.

.. code-block:: Perl

    use Kernel::System::Valid;    ## no critic (Modules::RequireExplicitPackage)

    package Kernel::System::Valid;    ## no critic (Modules::RequireFilenameMatchesPackage)

    use strict;
    use warnings;
    use v5.24;
    use utf8;

    sub AutoloadTest {
        return 1;
    }

See ``scripts/test/Config/Autoload.t`` for an example of how the new subroutine can be used.

Because of the line ``use Kernel::System::Valid`` we could also override existing methods. But that is not recommended.

Modifying Existing Methods of Core Modules
----------------------------~~~~~~~~~~~~--

Often it suffices to modify existing code. This can by overriding the relevant subroutine.

.. code-block:: Perl

    package Kernel::Autoload::Test;    ## no critic qw(Modules::ProhibitMultiplePackages)

    use strict;
    use warnings;
    use v5.24;
    use utf8;

    use Kernel::System::State;

    our @ObjectDependencies = (
        'Kernel::System::Log'
    );

    {
        no warnings 'redefine';    ## no critic qw(TestingAndDebugging::ProhibitNoWarnings)

        # keep reference to the original StateLookup()
        my $Orig = \&Kernel::System::State::StateLookup;

        # redefine StateLookup
        *Kernel::System::State::StateLookup = sub {
            my $Self = shift;

            $Kernel::OM->Get('Kernel::System::Log')->Log(
                Priority => 'info',
                Message  => 'Calling the modified method Kernel::System::State::StateLookup',
            );

            my $Result = $Orig->( $Self, @_ );

            # return a default value
            return $Result // 'unknown state';
        };
    }
