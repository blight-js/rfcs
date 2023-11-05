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

This proposal adds variables to the language. Variables are a way to store state between operations.

This RFC only defines immutable variables. All immutable values are deeply immutable. Syntax for mutable variables may be defined in a future RFC.


Let's look at an example of variables in the context of Blight:

```blight
let <variable_name>: <type> = <value>;
```

i.e. with an example type of "u8" and an example value of "10"

```blight
let x: u8 = 10;
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
let\s+(.+?)\s*=\s*(.+);
```

in perhaps a more readable (but completely made up) IDL:

```idl
let <variable_name> (: <type>)? = <value>;
```

Valid types and values will not be discussed in this proposal. They should be discussed in future proposals.

## Variable naming

Variable names are any valid [unicode identifiers](https://unicode.org/reports/tr31/) or [emoji](https://www.unicode.org/Public/15.1.0/ucd/emoji/emoji-data.txt). They may not start with a number, contain whitespace, or contain any of the following characters as they are reserved for future language use: "'<>(){}[]\|/?,.=+-~`!@#$%^&*

## Variable scope

Variables are scoped to the block they are declared in. Blocks have not been specified yet, but they will be in a future proposal. This means that variables declared in a block will not be accessible outside of that block.

## Variable shadowing

Variables can be shadowed. This means that a variable can be declared with the same name as a variable in a parent scope. This is not an error. See the guide-level explanation for more information.

## Variable mutability

Variables currently are only immutable. This means that once a variable is declared, it cannot be changed. This is a design decision that may be changed in a future RFC.

## Variable inlining

Variables can be inlined. This means that the variable will be replaced with its value at compile time. This is an optimization that can be done by the compiler. This is not a guarantee, but it is a possibility.

# Drawbacks

[drawbacks]: #drawbacks

Maybe there's a universe where Blight is a functional language with no need for
silly primitives like state, but it's not this universe. Variables are
necessary for any good programming language.

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

I believe this is the best design for variables in a modern programming
language. Explicitness is actually a friend and not a foe in this specific case.
So many bugs come down to modifying a variable that you thought was static.

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

We choose to follow Rust and Swift in this case for immutable variables.

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
- Mutable variables
- Variable destructuring
