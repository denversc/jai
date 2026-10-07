# Primitive Types of the Tick Programming Language

This document records the formal definitions of primitive types in **Tick**.

---

## General Rules

### Case Sensitivity
Tick is strictly **case sensitive**. Identifiers, keywords, and literals must match their exact casing. For example:
* `true` is a boolean literal keyword.
* `True` is an identifier (or syntax error where an identifier is not permitted), distinct from `true`.

---

## 1. Boolean (`bool`)

### Definition
The `bool` type represents a logical truth value.

* **Type Identifier:** `bool`
* **Values:** `true`, `false`

### Semantics
* `bool` has exactly two valid literal values: `true` and `false`. There are no alternative names or aliases (such as `boolean`).
* Control-flow guards (such as `if` conditions and loop guards) strictly require an expression evaluating to `bool`.
* There are zero implicit conversions to or from `bool`. Integers (`0`, `1`), empty collections, nulls/options, or references cannot be coerced implicitly to `bool`.

---

## 2. Integer Types

Tick provides fixed-width signed and unsigned integers, as well as pointer-sized integers.

### Type Identifiers and Ranges

#### Signed Fixed-Width Integers (Two's Complement)
* `i8`: 8-bit signed integer, range $[-128, 127]$ ($-2^7$ to $2^7 - 1$)
* `i16`: 16-bit signed integer, range $[-32,768, 32,767]$ ($-2^{15}$ to $2^{15} - 1$)
* `i32`: 32-bit signed integer, range $[-2,147,483,648, 2,147,483,647]$ ($-2^{31}$ to $2^{31} - 1$)
* `i64`: 64-bit signed integer, range $[-9,223,372,036,854,775,808, 9,223,372,036,854,775,807]$ ($-2^{63}$ to $2^{63} - 1$)

#### Unsigned Fixed-Width Integers
* `u8`: 8-bit unsigned integer, range $[0, 255]$ ($0$ to $2^8 - 1$)
* `u16`: 16-bit unsigned integer, range $[0, 65,535]$ ($0$ to $2^{16} - 1$)
* `u32`: 32-bit unsigned integer, range $[0, 4,294,967,295]$ ($0$ to $2^{32} - 1$)
* `u64`: 64-bit unsigned integer, range $[0, 18,446,744,073,709,551,615]$ ($0$ to $2^{64} - 1$)

#### Pointer-Sized Integers
* `usize`: Unsigned integer with the same bit-width as a pointer/memory address on the target architecture. Primarily used for collection indexing, container lengths, and memory offsets.
* `isize`: Signed integer with the same bit-width as a pointer/memory address on the target architecture. Primarily used for pointer or offset differences.

### Integer Literals

* **Decimal Only:** Integer literals are represented strictly in decimal (base 10). Alternate base prefixes (such as hexadecimal `0x`, binary `0b`, or octal `0o`) are not supported.
* **No Literal Suffixes:** Integer literals do not permit type suffixes (for example, `42_i32` and `42i32` are invalid syntax).
* **Digit Separators:** Underscores (`_`) are permitted as digit separators to enhance readability (e.g., `1_000_000`, `42_000`).
  * An underscore cannot appear as the first character of a number literal (which would conflict with identifier syntax).
  * An underscore cannot appear as the trailing character of a literal (e.g., `100_` is invalid).
  * Consecutive underscores (e.g., `1__000`) are invalid.
* **Default Literal Type:** When an integer literal appears without an explicit type annotation and cannot be inferred from immediate local assignment context, it defaults canonically to `i32`. For example:
  ```tick
  let x = 42; // x is inferred as i32
  let y = -24; // y is inferred as i32
  ```

### Conversions & Type Invariants

* **Strictly Explicit Conversions:** There are zero implicit conversions between numeric types (no implicit widening and no implicit narrowing). Converting between any integer types (such as `u8` to `i32`, `i32` to `i64`, or `usize` to `u32`) requires an explicit type conversion.
* **Zero Mixed-Type Arithmetic:** Binary arithmetic and bitwise operations require both operands to have the exact same integer type. Mixing types without an explicit conversion is a compile-time error.

