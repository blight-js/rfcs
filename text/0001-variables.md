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

This is the technical portion of the RFC. Explain the design in sufficient
detail that:

- Its interaction with other features is clear.
- It is reasonably clear how the feature would be implemented.
- Corner cases are dissected by example.

The section should return to the examples given in the previous section, and
explain more fully how the detailed proposal makes those examples work.

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

The actual syntax is just what I prefer at the time of writing, there are a lot
of alternatives discussed in the prior art section.

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
