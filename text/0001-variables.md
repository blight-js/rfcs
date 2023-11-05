- Feature Name: Variables in Blight
- Start Date: 2023-11-03
- RFC PR: [blight-js/rfcs#0001](https://github.com/blight-js/rfcs/pull/2)
- Blight Issue: https://github.com/blight-js/blight/issues/2

# Summary

[summary]: #summary

We at Blight don't believe that functional programming languages make sense.
Blight is not a functional programming language and thus relies on some form of
state storage. We solve this using variables.

# Motivation

[motivation]: #motivation

To make interesting programs, one needs to be able to store state. Adding state
storage allows for traditional declarative programming. Being able to store
state will allow for future work around variable semantics.

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

This proposal adds variables to the language. There are two types of variables
in Blight: mutable and immutable variables.

Immutable variables store some state which will remain constant forever (in
other words, deeply immutable). Immutable variables may be inligned for
optimization purposes. Let's look at an example of immutable variables in the
context of Blight:

```blight
let <variable_name>: <type> = <value>;
```

i.e.

```blight
let x: <type> = <value>;
x = <value2>; // Error: Cannot reassign immutable variable
```

The type and value stored inside of variables will be defined in a future
proposal.

The variable name can be any valid unicode except for the following conditions:

- Variables may not start with numerical digits
- Variables may not have any whitespace characters
- Variables may not contain with any of the following characters as they are
  reserved for future language use: "'<>(){}[]\|/?,.=+-~`!@#$%^&*

```blight
// Valid
let x: <type> = <value>;
let val10: <type> = <value>;
let こんにちは世界: <type> = <value>;
let 🔥: <type> = <value>;

// Invalid
let 10: <type> = <value>; // Error: Invalid variable name "10" (starts with a number)
let >greentext: <type> = <value>; // Error: Invalid variable name ">greentext" (contains disallowed character ">")
let "variable_name": <type> = <value>; // Error: Invalid variable name ""variable_name"" (contains disallowed character """)
```

The second type of variable has exactly the same semantics except that they are
reassignable. The syntax looks like:

```blight
let mut x: <type> = <value>;
x = <value2>; // no error!
```

In Blight, variables are able to be shadowed. In some languages this is an error
(include Javascript), but here it is not:

```
let x: <type1> = <value1>;
let x: <type2> = <value2>; // no error!
```

Variables are tightly scoped, which is not necessarily the case in plain
Javascript. This is more like "let" than "var".

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

## Variable declaration
Declaring a variable looks like the following regex.

```regex
let\s*(mut\s)?(.+?)(:\s*(.+?))?\s*=\s*(.+)\s*;
```

in perhaps a more readable (but completely made up) IDL:

```idl
let (mut)? <variable_name> (: <type>)? = <value>;
```

Valid types and values will not be discussed in this proposal. They should be discussed in future proposals.

## Variable naming

Variable names are any valid [unicode identifiers](https://unicode.org/reports/tr31/) or [emoji](https://www.unicode.org/Public/15.1.0/ucd/emoji/emoji-data.txt). They may not start with a number, contain whitespace, or contain any of the following characters as they are reserved for future language use: "'<>(){}[]\|/?,.=+-~`!@#$%^&*

## Variable scope

Variables are scoped to the block they are declared in. Blocks have not been specified yet, but they will be in a future proposal. This means that variables declared in a block will not be accessible outside of that block.

## Variable shadowing

Variables can be shadowed. This means that a variable can be declared with the same name as a variable in a parent scope. This is not an error.

## Variable mutability

Variables can be mutable or immutable. Immutable variables cannot be reassigned. Mutable variables can be reassigned. This is the only difference between the two.

Mutable variables are declared with the `mut` keyword. Immutable variables are declared without the `mut` keyword. Variable reassigning follows the following syntax:

```regex
(.+?)\s*=\s*(.+)\s*;
```

in perhaps a more readable (but completely made up) IDL:

```idl
<variable_name> = <value>;
```

## Variable inlining

Variables can be inlined. This means that the variable will be replaced with its value at compile time. This is an optimization that can be done by the compiler. This is not a guarantee, but it is a possibility.

# Drawbacks

[drawbacks]: #drawbacks

Maybe there's a universe where Blight is a functional language with no need for
silly primitives like state, but it's not this universe. Variables are
necessary for any good programming language.

Some people have expressed disinterest for the `let mut` syntax, but we have debated for a long time and nothing else seems to fit. `mut` as a modifier is the best way to go, and plays nicely with other proposals that may come up in the future.

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

I believe this is the best design for variables in a modern programming
language. Explicitness is actually a friend and not a foe in this specific case.
So many bugs come down to modifying a variable that you thought was static.

If we don't use the `mut` keyword now, we would have to invent it down the line anyways. I think we should just get it over with now.

# Prior art

[prior-art]: #prior-art

There's an unbelievable amount of prior art in this field, but in this case we
decided to approach variables syntactically and semantically quite similar to
how Rust does it. All other forms are too bulky, are inconsistent, or are just
not something I'm a fan of.

Let's go through how our variable naming might work if we followed different
languages:

Rust:

```rs
let x: u8 = 10;
let mut x: u8 = 10;
```

Typescript:

```ts
const x: number = 10;
let x: number = 10;
```

C:

```c
const int x = 10;
int x = 10;
```

Swift:

```swift
let x = 10;
var x = 10;
```

Zig:

```zig
const x = 10;
var x = 10;
```

All of these are fine, but I think having immutability be the default and then
enabling mutability if needed is a better way of thinking about programming.

# Unresolved questions

[unresolved-questions]: #unresolved-questions

How will variables interact with future language features? How will they be
transpiled to Javascript? What are valid variable types?

# Future possibilities

[future-possibilities]: #future-possibilities

Lots of future stuff for variables to expand out in future proposals:

- Variable referencing + dereferencing
- Specifying types
- Integration with potential language primitives (iterators?)

The key part of this proposal that extends out in the future is the `mut` keyword. I could easily imagine object and list destructuring:

```blight
let (x, y, mut z)= <value>;
```

I could also easily imagine a future with mutable arguments to functions

```blight
fn foo(mut x: <type>) {
	x = <value>;
}
```

Other keywords do not make sense in my opinion.
