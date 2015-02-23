# Inline::Lua

This is a Perl 6 module which allows execution of Lua code from Perl 6 code.

## Synopsis

    use Inline::Lua;
    my $L = Inline::Lua.new;

    my $code = q:to/END/;
        local args = {...}
        local n = 0

        for i = 1, args[1] do
            n = n + i
        end

        return n
    END
    my $arg = 1e8;
    my $sum;

    $sum = $L.run: $code, $arg;

    # OR

    my $func = "function sum (...)\n $code\n end";
    $L.run: $func;

    $sum = $L.call: 'sum', $arg;

    # OR

    my &sum = $L.get-global: 'sum';
    $sum = sum $arg;

    say $sum;

## Requirements

Lua 5.1 is currently the only supported version. This allows LuaJIT to be used
as well, though no public option exists yet to load LuaJIT instead of Lua.
Support for other versions of Lua is planned.

Any Rakudo backend with a NativeCall implementation is expected to work, but
testing has only been done under MoarVM on x86-64 Linux.

## Status

Inline::Lua currently allows passing and returning any number of boolean,
number,  string, table, and nil values, including nested tables. Functions,
userdata, and any other types are not implemented.

In Lua, there is no difference between Positional and Associative containers;
both are a table. Lua tables use 1-based indexing. Positional objects passed in
from Perl will have Integer keys incremented by one. Tables returned from Lua
are directly mapped to object hashes; there is no attempt at array detection or
index-adjustment. By contrast, multiple return values from Lua (not packed into
a table) results in an ordinary Perl list instead of an object hash.

Error reporting is crude, and the API is incomplete.

No provisions are made for growing Lua's stack beyond its initial size (which
defaults to 20). Therefore, passing deeply-nested data structures in to Lua may
result in an overflow.

## Usage

### method new ()

Creates, initializes, and returns a new Inline::Lua instance.

### method run (Str:D $code, \*@args)

Compiles $code, runs it with @args, and returns and resulting value(s).

### method call (Str:D $name, \*@args)

Calls the named global function with @args, and returns and resulting value(s).

To compile Lua code for subsequent use, pass it as a global function definition
to the .run method, then use .call to execute it.

### method get-global (Str:D $name)

Returns the Lua value stored in the named global variable.

If the value is a function, it returns a Perl routine which calls the function
as if .call had been used. Note this means the function is looked up by name
for each call, so if the value of the global variable changes, all wrappers in
existence automatically point to the new function.

Otherwise the values are converted in the same way as results from the .run and
.call methods.

A new value is returned every time, making it useless for identity comparison
(e.g. === on the same Lua table or function returned from different .get-global
calls will be False).

### method set-global (Str:D $name, $value)

Sets the value of the named global Lua variable, according to the same
conversions used when passing Perl arguments into Lua code.

## Contact

https://github.com/raydiak/Inline-Lua

raydiak@cyberuniverses.com

raydiak on #perl6 on irc.freenode.net

