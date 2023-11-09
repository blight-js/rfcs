- Feature Name: Boolean Values
- Start Date: 2023-11-09
- RFC PR: [blight-js/rfcs#0003](https://github.com/blight-js/rfcs/pull/0004)
- Blight Issue: https://github.com/blight-js/blight/issues/4

# Summary

[summary]: #summary

Blight has a boolean type. The compiler will represent these using JavaScript
[booleans](https://tc39.es/ecma262/#sec-terms-and-definitions-boolean-type).
These boolean types are pretty standard so they are incredibly easy to reason
about.

# Motivation

[motivation]: #motivation

Booleans are the most primative type that exists. They are the basis for all
logic. Without booleans, it is impossible to do any sort of logic. This RFC adds
booleans to the language. Fortunately, booleans are a very simple type to
implement.

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

This RFC adds booleans to the language. Booleans are a very simple type to
implement. They are either `true` or `false`. They are used in logic operations.

## Boolean Logic

The following binary operators seen in most other languages are available:

- `&&`: Logical AND
- `||`: Logical OR
- `==`: Logical Equals
- `!=`: Logical Not Equals

The following unary operators are available:

- `!`: Logical Not

The above operators share the
[same precedence and short-circuiting rules as JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_precedence),
to reduce confusion.

There is no undefined behavior when doing logical operations.

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

## Parsing booleans

For reference, the following regular expression is a set of all valid boolean
literals:

```regexp
(true|false)
```

A compiler should implement this in native code than using a regular expression
engine.

Examples of valid boolean literals:

- `true` (a truthy boolean)
- `false` (a falsey boolean)

Examples of invalid boolean literals:

- `true1` (an identifier)

## Operations

Since there are so few operations and types for booleans, we will define every
possible operation here.

### Unary operators

The logical NOT operator is a unary operator that inverts the value of a
boolean.

- `!true` evaluates to `false`
- `!false` evaluates to `true`

### Binary operators

The logical AND operator is a binary operator that returns `true` if both
operands are `true`, and `false` otherwise.

- `true && true` evaluates to `true`
- `true && false` evaluates to `false`
- `false && true` evaluates to `false`
- `false && false` evaluates to `false`

The logical OR operator is a binary operator that returns `true` if either
operand is `true`, and `false` otherwise.

- `true || true` evaluates to `true`
- `true || false` evaluates to `true`
- `false || true` evaluates to `true`
- `false || false` evaluates to `false`

The logical equals operator is a binary operator that returns `true` if both
operands are the same, and `false` otherwise.

- `true == true` evaluates to `true`
- `true == false` evaluates to `false`
- `false == true` evaluates to `false`
- `false == false` evaluates to `true`

The logical not equals operator is a binary operator that returns `true` if both
operands are not the same, and `false` otherwise.

- `true != true` evaluates to `false`
- `true != false` evaluates to `true`
- `false != true` evaluates to `true`
- `false != false` evaluates to `false`

## Constant Expression Evaluation

Expressions that contain only static boolean literals must be evaluated at
compile time.

```blight
let x: bool = (true && false) || (false || true);
```

The implementation logic of the type resolver is as follows:

1. Calculate LHS `(true && false)`
   1. LHS is a boolean literal `true`, which is internally represented as the
      boolean `true`
   2. RHS is a boolean literal `false`, which is internally represented as the
      boolean `false`
   3. Since both sides are compile-time known, this expression evaluates to the
      boolean literal `false`, internally represented by the boolean `false`
1. Calculate RHS `(false || true)`
1. LHS is a boolean literal `false`, which is internally represented as the
   boolean `false`
1. RHS is a boolean literal `true`, which is internally represented as the
   boolean `true`
1. Since both sides are compile-time known, this expression evaluates to the
   boolean literal `true`, internally represented by the boolean `true`
1. Since both sides are compile-time known, this can be evaluated at compile
   time.
1. Assign boolean literal `true` to the variable `x`, which requires type `bool`

## Changes to Identifiers

The following variable names are banned as they refer to built-in types, as they
would shadow the implicit global.

- `/bool|true|false/`

It is [not 100% decided](#variable-naming) how this impacts identifiers in
general. It should probably be the job of that proposal to decide this.

## Compiling `.to(T)`

As there are only number types, this function will only convert to them in most
cases. It may need to compile to `Number(x)` or `BigInt(x)` depending on the
action taken.

For integers (where `Int` represents a type with size greater than 0), the
following rules apply:

- `true.to(Int)` -> `1`
- `false.to(Int)` -> `0`

For floats, the following rules apply:

- `true.to(f64)` -> `1.0`
- `false.to(f64)` -> `0.0`

# Drawbacks

[drawbacks]: #drawbacks

For one, it adds a new type to the language, which increases complexity. This
could technically just be implemented as a type alias for `i1`, but that would
be confusing.

It also prevents the use of the identifiers `true`, `false`, and `bool` as
variable names. This is a minor drawback, as it is a very small set of
identifiers.

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

## Why use `.to(T)`

Converting a boolean into an integer feels like it should be a method on the
boolean. There's many options for how to do this:

- `my_bool.to(i32)`
- `my_bool.toInt(i32)`
- `i32.fromBool(my_bool)`
- `i32(my_bool)`

It also opens up the possibility of `.to(string)` in the future, and so on as a
practice of using `to` as the convention to convert something to another type.

## `.to(same_type)`?

Right now the following is valid:

```blight
true.to(bool).to(bool).to(bool)
```

I think this is okay, it is like how `!!!!!hi` is valid in most languages with
unary `!` for boolean negation.

Later with structure/class/generic/whatever, this may prove handy.

## Why not use `i1`?

Other languages (like C), use numbers for booleans. This has caused a lot of
confusion, and is a common source of bugs. It is also not very intuitive to use
a number for a boolean.

# Prior art

[prior-art]: #prior-art

This introduces a very common type in most languages into Blight. It is not a
new concept.

For instance: Rust, Zig, C, C++, Java, JavaScript, Python, Ruby, and many more
languages have booleans that are syntactically very similar to this proposal.

# Unresolved questions

[unresolved-questions]: #unresolved-questions

## Variable naming

Are variable names like `bool` prohibited as an identifier in general? As
previously suggested in the numerical values RFC, we could allow it. I'm leaning
on not allowing it for now and opening it up as a possibility in the future.

# Future possibilities

[future-possibilities]: #future-possibilities

## More Methods

In the future, we may want to add more logical operations to booleans, such as
`xor`, `nand`, `nor`, and so on. This is not part of this RFC, but is a
possibility.

Also, we may want to add more methods to booleans, such as `to(string)`.

## Other Proposals

Once booleans are implemented, a lot of things become possible. For instance, we
can now implement `if` statements, which is a very important feature. They also
allow for comparison operators such as the following to be implemented for
numbers:

- `==`
- `!=`
- `>`
- `>=`
- `<`
- `<=`
