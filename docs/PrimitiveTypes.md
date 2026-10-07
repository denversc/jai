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
