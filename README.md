# NAME

Exporter::ConditionalSubs

# VERSION

version v1.11.0

# SYNOPSIS

Allows subroutines to be conditionally exported.  If the condition
is satisfied, the subroutine will be exported as usual.  But if not,
the subroutine will be replaced with a stub that gets optimized away
by the compiler.  When stubbed out, not even the arguments to the
subroutine will get evaluated.

This allows for e.g. assertion-like behavior, where subroutine calls
can be left in the code but effectively ignored under certain conditions.

First create a module that `ISA` [Exporter::ConditionalSubs](https://metacpan.org/pod/Exporter%3A%3AConditionalSubs):

    package My::Assertions;

    require Exporter::ConditionalSubs;
    our @ISA = qw( Exporter::ConditionalSubs );

    our @EXPORT    = ();
    our @EXPORT_OK = qw( _assert_non_empty );

    sub _assert_non_empty
    {
        carp "Found empty value" unless length(shift // '') > 0;
    }

Then, specify an `-if` or `-unless` condition when `use`ing that module:

    package My::App;

    use My::Assertions qw( _assert_non_empty ), -if => $ENV{DEBUG};

    use My::MoreAssertions -unless => $ENV{RUNTIME} eq 'prod';

    # Coderefs work too:
    use My::OtherAssertions -if => sub { ... some logic ... };

    _assert_non_empty($foo);    # this subroutine call might be a no-op

This is a subclass of [Exporter](https://metacpan.org/pod/Exporter) and works just like it, with the
addition of support for the `-if` and `-unless` import arguments.

# NAME

Exporter::ConditionalSubs - Conditionally export subroutines

# VERSION

Version 1.01

# SUBROUTINES

## import

Works like the [Exporter](https://metacpan.org/pod/Exporter) `import()` function, except that it checks
for an optional `-if` or `-unless` import arg, followed by either
a boolean, or a coderef that returns true/false.

If the condition evaluates to true for `-if`, or false for `-unless`,
then any subs are exported as-is.  Otherwise, any subs in `@EXPORT_OK`
are replaced with stubs that get optimized away by the compiler (with
one exception - see ["CAVEATS"](#caveats) below).

You can specify either `-if` or `-unless`, but not both.  Croaks if
both are specified, or if you specify the same option more than once.

# CAVEATS

This module uses [B::CallChecker](https://metacpan.org/pod/B%3A%3ACallChecker) and [B::Generate](https://metacpan.org/pod/B%3A%3AGenerate) under the covers
to optimize away the exported subroutines.  Loading one or the other
of those modules can potentially break test coverage metrics generated
by [Devel::Cover](https://metacpan.org/pod/Devel%3A%3ACover) in mysterious ways.

To avoid this problem, subroutines are never optimized away
if [Devel::Cover](https://metacpan.org/pod/Devel%3A%3ACover) is in use, and are always exported as-is
regardless of any `-if` or `-unless` conditions.  (You probably
want [Devel::Cover](https://metacpan.org/pod/Devel%3A%3ACover) to assess the coverage of your real exported
subroutines in any case.)

# SEE ALSO

[Exporter](https://metacpan.org/pod/Exporter)

[B::CallChecker](https://metacpan.org/pod/B%3A%3ACallChecker)

[B::Generate](https://metacpan.org/pod/B%3A%3AGenerate)

# AUTHOR

Larry Leszczynski, `<larryl at cpan.org>`

# BUGS

Please report any bugs or feature requests at:
[https://github.com/GrantStreetGroup/Exporter-ConditionalSubs](https://github.com/GrantStreetGroup/Exporter-ConditionalSubs)

# SUPPORT

You can find documentation for this module with the perldoc command.

    perldoc Exporter::ConditionalSubs

You can also look for information at:

- GitHub

    [https://github.com/GrantStreetGroup/Exporter-ConditionalSubs](https://github.com/GrantStreetGroup/Exporter-ConditionalSubs)

- MetaCPAN

    [https://metacpan.org/pod/Exporter::ConditionalSubs](https://metacpan.org/pod/Exporter::ConditionalSubs)

- AnnoCPAN, Annotated CPAN documentation

    [http://annocpan.org/dist/Exporter-ConditionalSubs](http://annocpan.org/dist/Exporter-ConditionalSubs)

- CPAN Ratings

    [http://cpanratings.perl.org/d/Exporter-ConditionalSubs](http://cpanratings.perl.org/d/Exporter-ConditionalSubs)

# AUTHOR

Larry Leszczynski, `<larryl at cpan.org>`

# ACKNOWLEDGEMENTS

Thanks to Grant Street Group [http://www.grantstreet.com](http://www.grantstreet.com) for funding
development of this code.

Thanks to Tom Christiansen (`<tchrist@perl.com>`) for help with the
symbol table hackery.

Thanks to Zefram (`<zefram@fysh.org>`) for the pointer to his
[Debug::Show](https://metacpan.org/pod/Debug%3A%3AShow) hackery.

# LICENSE AND COPYRIGHT

Copyright 2015 Grant Street Group

This program is free software; you can redistribute it and/or modify it
under the terms of the the Artistic License (2.0). You may obtain a
copy of the full license at:

[http://www.perlfoundation.org/artistic\_license\_2\_0](http://www.perlfoundation.org/artistic_license_2_0)

Any use, modification, and distribution of the Standard or Modified
Versions is governed by this Artistic License. By using, modifying or
distributing the Package, you accept this license. Do not use, modify,
or distribute the Package, if you do not accept this license.

If your Modified Version has been derived from a Modified Version made
by someone other than you, you are nevertheless required to ensure that
your Modified Version complies with the requirements of this license.

This license does not grant you the right to use any trademark, service
mark, tradename, or logo of the Copyright Holder.

This license includes the non-exclusive, worldwide, free-of-charge
patent license to make, have made, use, offer to sell, sell, import and
otherwise transfer the Package with respect to any patent claims
licensable by the Copyright Holder that are necessarily infringed by the
Package. If you institute patent litigation (including a cross-claim or
counterclaim) against any party alleging that the Package constitutes
direct or contributory patent infringement, then this Artistic License
to you shall terminate on the date that such litigation is filed.

Disclaimer of Warranty: THE PACKAGE IS PROVIDED BY THE COPYRIGHT HOLDER
AND CONTRIBUTORS "AS IS' AND WITHOUT ANY EXPRESS OR IMPLIED WARRANTIES.
THE IMPLIED WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR
PURPOSE, OR NON-INFRINGEMENT ARE DISCLAIMED TO THE EXTENT PERMITTED BY
YOUR LOCAL LAW. UNLESS REQUIRED BY LAW, NO COPYRIGHT HOLDER OR
CONTRIBUTOR WILL BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, OR
CONSEQUENTIAL DAMAGES ARISING IN ANY WAY OUT OF THE USE OF THE PACKAGE,
EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

# AUTHOR

Grant Street Group <developers@grantstreet.com>

# COPYRIGHT AND LICENSE

This software is Copyright (c) 2015 - 2020 by Grant Street Group.

This is free software, licensed under:

    The Artistic License 2.0 (GPL Compatible)
