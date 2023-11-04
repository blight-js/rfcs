- Feature Name: `numeric-values`
- Start Date: 2023-11-04
- RFC PR: WIP / [blight-js/rfcs#0003](https://github.com/blight-js/rfcs/pull/0003)
- Blight Issue: https://github.com/blight-js/blight/issues/0003

# Summary

[summary]: #summary

Blight has fixed-size signed integers, unsigned intgers and floating point types. The compiler will represent these using JavaScript [numbers](https://tc39.es/ecma262/#sec-terms-and-definitions-number-value) or [BigInt](https://tc39.es/proposal-bigint). These number types are designed to be easy to reason about as a developer, as well as easy for a compiler to optimize usage. Conversions between types are explicit, and breaking the bounds is safety-checked undefined behavior.

# Motivation

[motivation]: #motivation

Numbers are universal for computation, obviously, but it is important to build a primative in a way that can be efficiently compiled to JavaScript. The proposal shown, for the most part, compiles most operations directly to the JS operators.

# Guide-level explanation

[guide-level-explanation]: #guide-level-explanation

Instead of relying on a single type for numbers, there are explicit types. These are split into integer and floating point numbers.

## Integers

Integers have two signednesses:

- **Unsigned Integer**: Unsigned numbers only represent `0` positive numbers. It is declared with `u` followed by the number of bits, such as `u8`, `u32`, `u64`, `u29`, `u512`, etc. The `u` means "unsigned", `u32` can be read as "U-int 32".
- **Signed Integers**: These can represent negative numbers and positive numbers. It is declared with an `i` followed by the number of bits, such as `i8`, `i32`, etc. The `i` means "integer", `i32` can be read as "Int 32".

When using integers, it is recommended to not go above `u53`/`i54`, as it allows the compiler to prevent using JavaScript's `BigInt` to represent the number. (see JS [Number.MAX_SAFE_INTEGER](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER))

## Built-in Properties on Integer Types

All number types have built-in constants. These are accessed by using property access notation. The variable name `Int` will serve as a placeholder for the integer type used.

> [!NOTE]
> There is not yet a property access notation RFC. This will assume JS-like dot notation.

- `Int.min`: The minimum value of this integer type. ex: `i32.min == -2147483648`
- `Int.max`: The maximum value of this integer type. ex: `u8.max == 255`

## Built-in Properties on Integer Values

All integer values can use property access notation to access built-in properties and methods. The variable name `val` will serve as a placeholder for the integer value used.

- `val.to(T)`: Converts the integer's value to a the given type. T can be
  - `f64`, which it will convert the value to a [floating point number](#floats)
  - Another Integer type like `u8`, which it will convert the value to that type. It is undefined behavior if this integer is out of range of `T.min`..`T.max`.
- `val.pow(y)`: Raise this integer to the power of `y`.

If the integer is a signed integer, the following methods are available:

- `val.abs()`: Get the absolute value of this integer. Returns an unsigned integer type.

## Integer Arithmetic

The following binary operators seen in most other languages are available:

- `+`: Addition
- `-`: Subtraction
- `*`: Multiplication
- `/`: Division
- `%`: Remainder
- `<<`: Bitwise left shift
- `>>`: Bitwise right shift
- `&`: Bitwise AND
- `|`: Bitwise OR
- `^`: Bitwise XOR

The following unary operators are available:

- `-`: Negation
- `~`: Bitwise NOT

The above operators share the [same precedence as JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_precedence), to reduce confusion.

These all function as JavaScript does, except bitwise operators do not [convert the value to a 32-bit integer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Left_shift#description). Instead, it will keep the number at whatever it's bit width is.

```js
0 ^ 4294967296;
// JavaScript -> 0
// Blight     -> 4294967296
```

Since fixed-width integer types have a specific minimum and maximum, it is undefined behavior to error to overflow or underflow when doing arithmatic. In a safe build, this will safely crash with a stack trace and information on what values were provided, but at runtime this will cause undefined behavior.

The following operations will cause undefined behavior:

- `+`: If the result of the addition is above `Int.max`
- `-`: If the result of the subtraction is above `Int.min`.
- `*`: If the result of the multiplication is above `Int.max` or below `Int.min`.
- `/`: If there is a remainder when dividing.
- `/`: If the divisor is `0`.
- `%`: If the divisor is `0`.
- Unary `-` can underflow if when negating `Int.max`.

## Floats

There is one floating point type: `f64`, which is a 64-bit floating point that follows the [IEEE 754 standard](https://en.wikipedia.org/wiki/IEEE_754).

## Built-in Properties on the Float Type

`f64` has built in properties. These are accessed by using property access notation.

- `f64.min`: Constant `1.7976931348623157e+308`
- `f64.max`: Constant `5e-324`
- `f64.inf`: Constant `Infinity`

## Built-in Properties on Float Values

Similarly to integers, floats have built-in properties and methods. The variable name `val` is a placeholder for the float value used.

- `val.to(T)`: Converts the float's value to a the given type. T can be
  - An integer type like `u8`, which it will convert the value to that type, rounding the float if needed.
    - It is safety-checked undefined behavior if the float is out of range of `T.min`..`T.max`.
    - It is safety-checked undefined behavior if the float is not an integer.
  - `f64`, which it will do nothing.
- `val.round()`: Round this float to the nearest integer, while keeping it a float.
- `val.floor()`: Round this float down to the nearest integer, while keeping it a float.
- `val.ceil()`: Round this float up to the nearest integer, while keeping it a float.
- `val.pow(y)`: Raise this float to the power of `y`.
  - It is safety-checked undefined behavior to call `1.pow(f64.inf)`.
- `val.sqrt()`: Get the square root of this float.
  - It is safety-checked undefined behavior if the value is negative.
- `val.abs()`: Get the absolute value of this float.

## Float Arithmetic

The following binary operators seen in most other languages are available:

- `+`: Addition
- `-`: Subtraction
- `*`: Multiplication
- `/`: Division
- `%`: Remainder

The following unary operators are available:

- `-`: Negation

The following operations will cause undefined behavior:

- `/`: If the divisor is `0`.
- `%`: If the divisor is `0`.

Note that floating point operations cannot overflow, but instead will return `f64.inf` as IEEE 754 specifies.

## NaN

There is no `NaN` constant. Instead, any operation that could produce `NaN` is either safety-checked undefined behavior (like dividing by zero), or will return an optional `f64`. When representing `NaN` is required, it should be represented as an optional `f64`, to indicate the possibility of `NaN`.

If it was missed when writing this RFC. Any math operation that would produce `NaN` should be safety-checked undefined behavior.

> [!NOTE]
> There is not yet an RFC for optionals.

# Reference-level explanation

[reference-level-explanation]: #reference-level-explanation

## Parsing numbers

For reference, the following regular expression is a set of all valid numeric literals:

```regexp
^(?:\d(?:\d|_\d)*)?\.?(?:\d(?:\d|_\d)*)+(?:e[+-]?(?:\d(?:\d|_\d)*))?\b
```

A compiler should implement this in native code than using a regular expression engine.

Examples of valid numeric literals:

- `10` (an integer)
- `512_000.000` (an integer)
- `63_2.42` (a float)
- `.425_000` (a float)
- `50.2e+3` (an integer)
- `4_2.1_2e8_2` (an integer)

Examples of invalid numeric literals:

- `10.` (this dot may be seen as a property access)
- `1.5.1` (potentially a property access)
- `4e-4.0` (potentially a property access)
- `_52` (an identifier)
- `10_` (parse error: numeric separator must be between two numbers)
- `1__3` (parse error: numeric separator must be between two numbers)

## Type of numeric literals

Numeric literals do not have explicit types when in expressions, but rather adapt to the runtime-known types. This works exactly how [Zig's `comptime_int` type](https://ziglang.org/documentation/master/#toc-Runtime-Integer-Values) works (the type of integer literals), but this RFC does not give this type an observable name.

For example, if a variable of type `u32` named `x` existed, and the following code was run:

```blight
x + 512
```

`512` is widened to a u32. If the rhs was larger than `u32.max`, then this would be a compile error. Explicit widening of `x` is needed.

```
x.to(u64) + u32.max + 123
```

You cannot use [unary `~`](#integer-arithmetic) on an integer literal, it must be given a type, such as:

```blight
~512 // compile error: cannot infer type of constant numeric literal
     // hint: unary operator '~' needs a result type

~512.to(u32) // ok
```

Similarly, runtime-mutable variables cannot be assigned without annotating their type.

> [!NOTE]
> There is not yet an RFC for variable declarations. We will assume `var name: type` is the syntax.

```blight
var x = 512 // compile error: cannot infer type of constant numeric literal

var x: u32 = 512 // ok
var x = 512.to(u32) // ok, less preferred approach
```

## Constant Expression Evaluation

Expressions that contain only numeric literals must be evaluated at compile time.

```blight
var x: u32 = (2.5 + 2.5) / 5
// valid as `1` can coerce to `u32`

var x: u32 = 5 / 6
// compile error: cannot coerce `0.8333333333333334` to `u32`
```

If undefined behavior is caused by a constant expression, it is a compile error.

```blight
var x: u32 = 5 / 0
// compile error: division by zero
```

## Widening during arithmetic

When integers of different sizes are used in arithmetic, the compiler will widen the smaller integer to the larger integer.

## Changes to Identifiers

The following variable names are banned as they refer to built-in types, as they would shadow the implicit global.

- `/[uif][0-9]+/`

It is [not 100% decided](#variable-naming) how this impacts identifiers in general. It should probably be the job of that proposal to decide this.

## Compiling `.to(T)`

As there are only number types, this function will do nothing in most cases. It may need to compile to `Number(x)` or `BigInt(x)` depending on the action taken.

## Compiling bitwise operations

Spec compliant bit-shifting is not trivial to implement, it involves preserving the size for larger integers. The simplest way to do this is to convert JS numbers into [BigInt](https://tc39.es/proposal-bigint) and use the BigInt operators. This is not ideal, as it is slower than using the JS operators directly. It is also not ideal because it is not possible to use the BigInt operators on `f64` values.

## Compiling arithmetic safety checks

Saftey checks can be implemented somewhat trivially using a single temporary variable. This can be left as a global. It is unfortunate that so much code, as safety checks off allow this to compile to 3 characters.

> [!NOTE]
> Blight does not yet have an decided assignment operator or variable system, `=` will be used as such in this example.

Input:

```blight
a = 255.to(u8)
b = 1
c = a + b
```

```js
const _u8_max = 255;
let _tmp;

let a = 255;
let b = 1;
let c = (_tmp = a + b) > _u8_max ? _integerOverflow(a, b) : _tmp;
```

> [!NOTE]
> A future RFC will figure out what `_integerOverflow` does.

# Drawbacks

[drawbacks]: #drawbacks

Giving the developer explicit control and the quantity of options over the numberic types will increase cognitive load. Most scripting languages simply make all numbers floating point because of this.

More reasonable systems for bitwise logic puts a performance hit due to the language's design.

The removal of NaN might be problematic for some uses. I think it's worth it.

# Rationale and alternatives

[rationale-and-alternatives]: #rationale-and-alternatives

## Why undefined behavior / crash on overflow?

This is a design decision to make faster builds, the `+` operation with all safety checks disabled will compile to `+`, allowing for fast execution. When a specific behavior is desired for overflow, this should be explicitly opted into. However, it will be left to [another proposal to add these APIs](#saturation-and-wrapping-arithmetic).

## Why methods instead of built-in functions / globals?

I personally think the number of global symbols should be kept to a minimum. And I think it's better organization.

## Why use `.to(T)`

Converting a float into an integer feels like it should be a method on the float. There's many options for how to do this:

- `my_float.to(i32)`
- `my_float.toInt(i32)`
- `i32.fromFloat(my_float)`
- `i32(my_float)`
- a global symbol like Zig `@intFromFloat`

I went with the first one because it is concise, and gives the same API to both floating point and integer types. It also opens up the possibility of `.to(string)` and `.to(bool)`, and so on as a practice of using `to` as the convention to convert something to another type.

## `.to(same_type)`?

Right now the following is valid:

```blight
(123).to(u32).to(u32).to(u32)
```

I think this is okay, it is like how `!!!!!hi` is valid in most languages with unary `!` for boolean negation.

Later with structure/class/generic/whatever, this may prove handy.

### Alternative: like Zig RLS

Another alternative to consider for conversion is doing something similar to Zig's Result Location Semantics<sup>[citation needed]</sup>. Prior art:

```zig
const x: i32 = 1;
// the result loation `y` is u8, so `@intCast` must produce a u8,
// so it converts `x` to a u8.
const y: u8 = @intCast(x);
// the result loation (the argument) is u16, so `@intCast` must produce a u16,
// so it converts `x` to a u16.
callExpectsU16(@intCast(x))
```

A mechanism like this would introduce a lot of compiler complexity, and kinda covers a different set of uses than exactly what the `.to(T)` function does. It is another RFC's job to decide this.

## Why not use `number`?

If we assigned a type to all JS numbers, like [TS `number`](https://www.typescriptlang.org/docs/handbook/basic-types.html#number), this leads to ambiguity, primarily in if the intent is to use an integer or a floating point number.

If a developer really wants to have `number` and Blight accepts a 'types as values'-like proposal, they could assign `f64` to a constant named `number`.

## Why only single float type `f64`?

In the primary compilation target, JavaScript, uses 64-bit floats for all numbers. It does not give access to other percisions (32-bit, 128-bit). It would be impossible to compile any other float type than `f64` without embedding a library for correct calculations. For that reason, Blight does not have any other floating point types.

`f32` could be implemented as [Float32Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Float32Array), but it would likely be slower than `f64`, which would be misleading to the developer.

An alternative to this, which was used in an earlier version of this document, was `float`. It is more keys to type, but it looks cleaner, especially if other float sizes will not be added.

## Why use a function `.pow` for exponentiation instead of `**`?

It is ambiguous with a potential `*` unary operator. do you parse `a**b` as:

- `a` `**` `b`
- `a` `*` `(` `*b` `)`

## Floating point overflow

Should `f64.inf + 1` be `f64.inf` or a crash. This RFC defines that it is `f64.inf`.

# Prior art

[prior-art]: #prior-art

A large part of this RFC is adapted from how Zig's [Integer](https://ziglang.org/documentation/0.11.0/#toc-Integers) and [Float](https://ziglang.org/documentation/0.11.0/#Floats) types work.

The pronounciation of the types is adapted from the C99 [stdint.h](https://en.wikipedia.org/wiki/C_data_types#stdint.h) (`uint32_t`, `int64_t`, etc).

Properties on the type definitions themselves are adapted from Rust's [Integer's Associated Constants](https://doc.rust-lang.org/std/primitive.i32.html)

# Unresolved questions

[unresolved-questions]: #unresolved-questions

## Handling Overflow Errors

A way to handle overflow errors should be given. Making these methods on the values seems like the path of least resistance, but it will lead to long code like `something.tryAdd(63)` or whatever.

Maybe later when we figure out error handling in general we can prescribe a way to handle operator errors easily.

Similarly, this should be the case for `.sqrt`/`.pow` and so on

## Saturating and Wrapping Arithmetic

This proposal does not include saturating or wrapping arithmetic.

Two approaches to consider:

- Rust-like `left.addSaturating(right)`
- Zig-like `left +| right`

This is a job for a different RFC, as getting the base numberic semantics working is more important.

## Variable naming

Are variable names like `f64` prohibited as an identifier in general? Some languages, including JavaScript, have interesting rules for this.

Valid JavaScript:

```ts
const x = { class: 2 };
console.log(x.class);
```

Valid Zig:

```zig
pub fn main() void {
  const x = struct {
    const @"null" = 1;
    const @"u8" = 2;
  };

  const y = x.null;
  const z = x.u8;

  @import("std").debug.print("{d} {d}", .{y, z});
}
```

I'm inclined to allow it, but that isn't neccecarily about this RFC, but about a property access RFC.

## Naming compile-time known integers

This RFC does not name the type of [numeric literals](#type-of-numeric-literals). A pre-requisite to doing this realistically would be adding a full equivilent to `comptime`, which may not be desirable, and is a job for a different RFC.

## Hex/Octal/Binary literals

To keep the scope down, this RFC does not address these literals. Currently, all numeric literals are base 10.

# Future possibilities

[future-possibilities]: #future-possibilities

## More Methods

Later on, it will be cool to add more properties / methods to the integer types, such as `i32.parse(str)`. Exactly how this would compile is not clear at the moment, as `parseInt` is a weird API that we may not want to expose as-is. The integer value can contain methods like `.string()` and others.

We should also, at some point, expose ways to use the raw JS operators on `float` types.

## Comparison operators

Once booleans come into play, we can add the following operators:

- `==`
- `!=`
- `>`
- `>=`
- `<`
- `<=`

Since we elimiated `NaN` for an optional, the identity `x == x` will **always** hold, and a compiler and a developer can optimize it away without looking at the value.

For comparisons, no thought has to be put into how to handle `NaN`, as it is never occurs.
