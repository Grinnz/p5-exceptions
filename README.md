# NAME

Exceptions - Documentation for exception handling in Perl.

# DESCRIPTION

This module doesn't do anything, it exists solely to document how to handle
exceptions in Perl.

# WHY?

This module was originally released in 1996, but it hasn't been installable or
usable in any fashion since then. Many other alternatives have cropped up over
the years to make exception handling much easier. If you want to skip the
explanations below, then you should look directly at some of the modules that
make exception handling dead-simple:

[Try::Tiny](https://metacpan.org/pod/Try::Tiny) - Catch exceptions in a familiar `try` and `catch` way. If you
look no further, make use of this module!

With a good way to catch exceptions, now you need exception types so you can
re-throw exceptions when they're something that should be handled elsewhere.

- [Throwable](https://metacpan.org/pod/Throwable) and [Throwable::SugarFactory](https://metacpan.org/pod/Throwable::SugarFactory)
- [Exception::Class](https://metacpan.org/pod/Exception::Class)
- [Mojo::Exception](https://metacpan.org/pod/Mojo::Exception)

# AN EXCEPTION

An exception is what happens anytime your program's execution exits
unexpectedly. Let's start with a simple example.

    #!/usr/bin/env perl
    use strict;
    use warnings;

    print "0 plus 1 = ", increment(0), "\n"; # 1
    print "zero plus 1 = ", increment('zero'), "\n"; # Exception
    print "0 plus 1 = ", increment(0), "\n"; # never executes

    sub increment {
        my $int = shift;
        die "That's not an int!" unless defined $int && $int =~ /^[0-9]+\z/;
        return $int+1;
    }

The first line prints `0 plus 1 = 1\n` as expected. The second line, however,
dies in a way that we can't recover from which prevents the rest of our program
from doing any further execution. So, we must handle our exceptions!

# A HANDLED EXCEPTION

The only way you can handle an exception is to wrap the code that could
die in an `eval` block. This sounds simple enough, but there are some gotchas
that lead many developers to do this incorrectly.

The correct way to handle an exception requires that you understand how to
preserve the global `$@` variable. Please see ["BACKGROUND" in Try::Tiny](https://metacpan.org/pod/Try::Tiny#BACKGROUND) for a
great explanation of this problem.

Let's look at our previous simple application with error handling using `eval`.

    #!/usr/bin/env perl
    use strict;
    use warnings;

    my $value;
    my $error = do { # catch block
        local $@;
        eval { $value = increment(0) }; # try
        $@;
    };

    print "0 plus 1 = ";
    if ($error) {
        print "error";
    }
    else {
        print $value;
    }
    print "\n"; # 1

    $value = undef;
    $error = undef;
    $error = do { # catch block
        local $@;
        eval { $value = increment('zero') }; # try
        $@;
    };

    print "zero plus 1 = ";
    if ($error) {
        print "error";
    }
    else {
        print $value;
    }
    print "\n"; # error

    $value = undef;
    $error = undef;
    $error = do { # catch block
        local $@;
        eval { $value = increment(0) }; # try
        $@;
    };

    print "0 plus 1 = ";
    if ($error) {
        print "error";
    }
    else {
        print $value;
    }
    print "\n"; # 1

    sub increment {
        my $int = shift;
        die "That's not an int!" unless defined $int && $int =~ /^[0-9]+\z/;
        return $int+1;
    }

As you can see, it gets quite ugly and cumbersome to handle exceptions this way.
Don't let that scare you away from Perl, though, keep reading and be happy!

# THE SOLUTION

Lucky for us, Perl is an awesome language where the community provides many
solutions to common tasks for us. One such solution is [Try::Tiny](https://metacpan.org/pod/Try::Tiny).

If you get nothing else out of this document, let it be that using [Try::Tiny](https://metacpan.org/pod/Try::Tiny)
will save you time and heartache.

    #!/usr/bin/env perl
    use strict;
    use warnings;
    use Try::Tiny qw(try catch);

    print "0 plus 1 = ", try { increment(0) } catch { "error" };
    print "\n"; # 1

    print "zero plus 1 = ", try { increment('zero') } catch { "error" };
    print "\n"; # error

    print "0 plus 1 = ", try { increment(0) } catch { "error" };
    print "\n"; # 1

    sub increment {
        my $int = shift;
        die "That's not an int!" unless defined $int && $int =~ /^[0-9]+\z/;
        return $int+1;
    }

# AUTHOR

Chase Whitener <`capoeirab@cpan.org`>

# LICENSE AND COPYRIGHT

Copyright 1996 by Peter Seibel <`pseibel@cpan.org`>. This original release
was made without an attached license.

Copyright 2016 by Chase Whitener <`capoeirab@cpan.org`>. This re-release contains
none of the original code or structure and is thus re-released under the same
license as Perl itself.

This program is free software; you can redistribute it and/or modify
it under the same terms as Perl itself.
