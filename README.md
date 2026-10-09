# NAME

Perl::Critic::Policy::ErrorHandling::RequireCheckedIndirectResults - Check the result of a syscall or an eval that a sub returns to you.

# VERSION

version 0.001

# DESCRIPTION

`InputOutput::RequireCheckedSyscalls`, `InputOutput::RequireCheckedOpen` and
`ErrorHandling::RequireCheckingReturnValueOfEval` report a syscall or an
`eval` block whose result nobody checks.  They look where the builtin is
called.  A sub that returns the result passes it on, so the builtin is
checked as far as they can see, and the caller that drops it is not looked at:

```perl
sub finish { return close $_[0] }
...
finish($fh);               # reported
```

This policy reports such a call.  The value of the sub is the result: the value
of a `return`, or its last statement.  The call is reported when it is a
statement of its own, so that its result goes nowhere.

## Which results

The builtins are those of `InputOutput::RequireCheckedSyscalls`, configured
the same way: `open`, `close`, `print` and `say` by default, and every
builtin that returns a status, `mkdir` among them, with
`functions = :builtins`.  An `eval` block
is always one.  A file that says `use autodie` or `use Fatal` has its
builtins die on failure, so its subs return no syscall result to check.  That
is decided for the whole file.

## Dropped

A call is dropped when it is a statement of its own: alone, or with a postfix
modifier such as `if` or `foreach`.  A call followed by an operator, such as
`or die`, is checked.  The last statement of a block passes its value on,
unless the block belongs to an `if`, a loop or the like, or is the file
itself.  So the last statement of a sub, a `do` or an `eval` is not dropped.

## Where the sub can be

In the same file, by its name.  In another file of the same distribution,
which [Perl::Critic::Distribution](https://metacpan.org/pod/Perl%3A%3ACritic%3A%3ADistribution) reads: a package sub, called by its full
name, as `Some::finish()`, or by its bare name from the same package.  A
policy that reads the distribution through the same library shares its parse.

## What it leaves alone

A call whose result is assigned, tested, returned or passed on.  A sub that
asks `wantarray`.  A result that reaches the return through a variable, and a
`return` inside an inner anonymous sub.  A method call, because the method
that runs can be another sub of the same name.  A bare call of a sub from
another file in another package, because what it imports is not known.  An
`eval` of a string, which `BuiltinFunctions::ProhibitStringyEval` is for.

# CONFIGURATION

```
[ErrorHandling::RequireCheckedIndirectResults]
functions = :builtins
exclude_functions = print
```

`functions` and `exclude_functions` are those of
`InputOutput::RequireCheckedSyscalls`.  Give both policies the same, or this
one reports the results that the other was told to leave alone.

## METHODS

### supported\_parameters

`functions` and `exclude_functions`, as in
`InputOutput::RequireCheckedSyscalls`.

### default\_severity

### default\_themes

### applies\_to

The whole document, because a call can come before the sub that it calls.

### initialize\_if\_enabled

Works out which results count, from `functions` and `exclude_functions`.
Registers what this policy needs from each file of a distribution with
[Perl::Critic::Distribution](https://metacpan.org/pod/Perl%3A%3ACritic%3A%3ADistribution): the package subs whose value is a result, and
which builtin it comes from, whatever the configuration.  A lexical sub is left
out, because no other file can call it.

### violates

## FUNCTIONS

The steps of `violates`, for its tests.

### result\_subs\_in

```perl
my @found = result_subs_in( $ppi, packages_in($ppi) );
```

Each sub of a document whose value is the result of a builtin of
`InputOutput::RequireCheckedSyscalls` or of an `eval` block, as its
statement, its full name and the builtin, `eval` for an `eval` block.  A sub
that asks `wantarray` is not one, and a file under `autodie` or `Fatal` has
none but its `eval` blocks.

### result\_of

The builtin whose result is the value of a block, or undef: that of a
`return` anywhere in it, or of its last statement.  Not a `return` inside an
inner sub, which returns from that sub.

### packages\_in

The `package` statements of a document, in order, for `package_at`.  A
document is searched once, and not once for each element.

### package\_at

```perl
my $package = package_at( $elem, packages_in($ppi) );
```

The package that an element is in: that of the block of a `package NAME { }`
around it, or else that of the last `package NAME;` before it, or `main`.

### is\_dropped

Whether a call is a statement of its own, whose result goes nowhere.
["Dropped"](#dropped) says when.  A method, a hash key and the name in a sub statement
never start a plain statement, so they are never dropped calls.

# AUTHORS

Current Maintainers:

- George S. Baugh <george@troglodyne.net>

# COPYRIGHT AND LICENSE

Copyright (c) 2026 Troglodyne LLC

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
