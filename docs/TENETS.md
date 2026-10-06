# Tenets of the Tick Programming Language

This document establishes the foundational design tenets of **Tick**. Unlike traditional programming languages that optimize primarily for human readability and ergonomic typing on mechanical keyboards, Tick is designed primarily for **AI coding agents** as authors, and for **deterministic cross-language transpilation**.

---

## 1. High Token Efficiency (Context Optimization)
* **Rationale:** Context windows in Large Language Models (LLMs) are finite and directly impact cost, latency, and reasoning depth. Verbose boilerplate consumes unnecessary attention budget.
* **Principles:**
  - Maximize semantic density per token.
  - Minimize ceremony, repetitive boilerplate, and redundant declarations without sacrificing semantic precision.
  - Choose compact, expressive syntactic constructs that LLM tokenizers compress effectively.

---

## 2. Unambiguous, Orthogonal Grammar (Zero Hallucination Surface)
* **Rationale:** Ambiguity, multiple valid ways to write the exact same construct, and complex operator precedence increase the probability of syntax errors and hallucinated patterns.
* **Principles:**
  - One clear, canonical way to express any given operation.
  - Strictly deterministic context-free grammar constructs that are easy for both LLM probability distributions and compiler parsers to predict and validate.
  - Strict syntax rules that disallow ambiguous edge cases (e.g., mandatory statement-terminating semicolons with zero automatic insertion heuristics, no implicit type coercions, no hidden scope-escaping variable shadowing).

---

## 3. Strict, Explicit Static Semantics & Rust-Style Ownership
* **Rationale:** Latent bugs and undefined behavior across target platforms are costly to debug. Strong invariants allow agents to reason about code locally without whole-program global inference.
* **Principles:**
  - **Compile-Time Ownership & Borrow Verification:** Rust-inspired ownership, borrowing, and lifetime semantics operate strictly as compile-time static proof invariants within the frontend. Because the compiler statically proves non-aliasing and consumption invariants before code emission, the generated code safely lowers to idiomatic, native references in garbage-collected targets (Python, TypeScript, Go) and moves/borrows in systems targets (Rust, Swift) with zero runtime shims.
  - **Explicit Static Typing:** Strong, static typing where all types, effects, and mutability are explicitly knowable.
  - **Sound Multi-Target Lowering:** Code that compiles is guaranteed to map soundly to both garbage-collected and manually managed target environments.

---

## 4. Agentic Diagnostic & Repair Ergonomics
* **Rationale:** When an AI agent makes a syntax or semantic error, traditional human compiler diagnostics often lack actionable precision, resulting in endless debugging loops.
* **Principles:**
  - Compiler errors and warnings must be machine-actionable, providing structured output (e.g., JSON schemas) alongside human-readable text.
  - Diagnostics must provide exact source spans, clear explanations of failed invariants, and suggested canonical patches/fixes that the agent can apply directly.
  - Fast feedback loops: instantaneous compiler checks allow agents to validate drafts in sub-second cycles.

---

## 5. First-Class Compile-Time Evaluation (CTFE) & Metaprogramming
* **Rationale:** Agents frequently need to generate repetitive glue or compute configuration/table values. Rather than writing external code generators or template scripts, this logic should live natively in the language.
* **Principles:**
  - A unified interpreter that allows executing Tick code at compile time (`comptime`).
  - Pure functions that can resolve directly to constants at compile time.
  - Ability to synthesize types, tables, and functions programmatically before transpilation passes.

---

## 6. Locality of Reasoning (Self-Contained Units)
* **Rationale:** AI agents have finite attention and struggle when understanding a function requires chasing deeply nested inheritance, ambient global variables, or implicit dependencies across dozens of files.
* **Principles:**
  - Code must be understandable and verifiable using only its immediate local context and explicit signatures.
  - No global mutable state; all data flow is explicit through function inputs, outputs, and ownership transfers.
  - Local type inference only: type inference never leaks past function boundaries; all public APIs and boundary signatures are fully explicit.
  - No implicit contextual "magic" (such as hidden thread-locals, ambient dependency injection, or invisible lifecycle hooks).

---

## 7. Determinism Over Cleverness (Predictable Semantics & Explicit Failure)
* **Rationale:** Complex language "magic" (operator overloading surprises, implicit coercions, hidden exception flows) triggers subtle hallucinations and regressions in AI agent reasoning.
* **Principles:**
  - **No Implicit Conversions:** Type casting and conversions must always be explicit.
  - **Deterministic Evaluation Order:** Strict, unambiguous left-to-right evaluation across all expressions.
  - **Explicit Error Flow (Result Unions):** Recoverable errors are represented as typed return value unions/results (inspired by rich error aggregation proposals like Kotlin KEEP-0441 and languages like Rust and Zig). The caller is forced by the compiler to explicitly handle or intentionally propagate errors.
  - **Fatal-Only Exceptions:** Exceptions/panics are reserved exclusively for unrecoverable, fatal program failure (process death). They are never used for ordinary control flow.

## 8. Single Canonical Representation (One Way To Do It)
* **Rationale:** Languages with multiple syntactic forms for the same semantic construct (e.g., three different function declaration syntaxes, four looping constructs, truthy vs explicit checks) dilute LLM probability distributions, leading to inconsistent generation and edge-case bugs.
* **Principles:**
  - Exactly **one** canonical way to express any given operation or construct.
  - **Single Function Syntax:** Unified declaration syntax for free functions, methods, and closures/lambdas.
  - **Single Iteration Model:** One unified loop construct rather than multiple overlapping primitives (`for`, `while`, `do-while`, `forEach`).
  - **Strict Boolean Conditions:** Conditionals (`if`, loop guards) require an explicit `bool` expression. No implicit "truthiness" (e.g., non-zero numbers, non-empty strings, or implicit null pointer conversions).
  - **Zero Syntactic Synonyms:** Eliminates stylistic fragmentation across AI-authored codebases.

## 9. Token-Dense Grammar without Punitive Punctuation
* **Rationale:** Traditional human languages are full of multi-token ceremony (`function`, `implements`, `public static void`) and punctuation that wastes finite LLM context windows or triggers tokenizer fragmentation and bracket-drift errors.
* **Principles:**
  - **Deterministic Statement Termination:** Mandatory semicolons terminate statements, eliminating complex Automatic Semicolon Insertion (ASI) heuristics and ensuring multi-line expressions and refactoring patches never terminate prematurely.
  - **No Redundant Punctuation:** Eliminate superfluous parentheses, braces, or decorative ceremony where grammar constructs are already unambiguous.
  - **Robust Block Structure:** Avoid purely whitespace-sensitive indentation pitfalls (which cause off-by-one whitespace bugs in LLMs) while keeping block delimiters lightweight and easy for models to track and close.
  - **Zero Boilerplate Ceremony:** No mandatory enclosing classes or boilerplate namespaces required to author standalone functions, types, or tests.

## 10. First-Class Invariants & Contract Verification
* **Rationale:** Specifying explicit constraints as formal contracts allows AI agents to reason about code soundness locally, author robust logic without hallucinations, and verify behavior without needing to read distant implementation details.
* **Principles:**
  - **Explicit Function Contracts:** Functions can declare formal preconditions (`pre`) and postconditions (`post`) directly on their signatures.
  - **Type Invariants:** Data structures can declare invariants that must hold true after instantiation and mutation.
  - **Multi-Tier Contract Verification:** Contracts are evaluated at compile time during CTFE when expressions operate on compile-time constants; a small, tractable subset of simple structural invariants can be verified statically without a heavy generic solver, while general contracts are enforced via runtime assertions in debug/test builds and leveraged for property-based fuzz testing.
  - **Contract-as-Interface:** Calling agents rely entirely on the function's contract guarantees rather than needing to parse the function's internal body.

## 11. Decoupled, Single-Pass Syntactic Parsing (Zero Semantic Feedback)
* **Rationale:** While LLMs generate code strictly autoregressively (left-to-right, one token at a time) without future lookahead, compiler parsing is deterministic with bounded lookahead. Pass 1 is strictly a single-pass syntactic check that constructs a valid AST from the token stream using bounded lookahead (Pratt parsing for expressions, LL(k) for declarations), completely independent of symbol tables or type information. Languages that require unbounded lookahead, backtracking, or semantic type-feedback during parsing (such as C/C++ where parsing depends on symbol tables to distinguish types from expressions) induce high-frequency syntax generation errors.
* **Principles:**
  - **Pure Single-Pass Syntactic AST Construction (Pass 1):** Pass 1 is strictly a single-pass syntactic check that constructs a valid AST from the token stream using bounded lookahead (combining LL(k) declarations and Pratt expression parsing), completely independent of symbol tables or type information.
  - **Zero Lexer/Parser Semantic Feedback:** Parsing grammar never requires knowing whether an identifier is a type, variable, or function (unlike C/C++). Lexing and parsing operate with zero semantic feedback loops.
  - **Unambiguous Prefix Grammar:** Constructs declare their identity up front via explicit prefix keywords (`fn`, `let`, `type`, `loop`), allowing both token prediction and compiler parsing to proceed deterministically with bounded lookahead.
  - **Unambiguous Generic Delimiters:** Generic parameters use unambiguous delimiters (such as bracketed generics `[T]`) to eliminate lookahead ambiguities and parser backtracking.
  - **Sub-Millisecond Parse Performance:** Pure syntactic parsing enables instant structural feedback for AI agent drafting and validation loops.

## 12. Composable, Flat Typing over Deep Inheritance (Traits Only)
* **Rationale:** Class implementation inheritance hierarchies introduce fragile base class problems, complex method resolution order (MRO) bugs, and hidden parent state mutations that confuse AI reasoning. Crucially, implementation inheritance maps poorly and inconsistently across targets (Rust and Go lack it entirely).
* **Principles:**
  - **Zero Implementation Inheritance:** No `class ... extends` hierarchies.
  - **Pure Data & Pure Interfaces:** Clean separation between pure data records/structs and behavioral interface contracts (`trait`).
  - **Composition Over Inheritance:** Code reuse is achieved via explicit data composition and trait implementation.
  - **Universal Transpilation Alignment:** Maps seamlessly 1:1 onto Rust `trait`, Swift `protocol`, Kotlin/Java `interface`, TypeScript `interface`, and Go `interface`.

## 13. First-Class Native Testing & Verification
* **Rationale:** Writing tests in traditional ecosystems forces AI agents to juggle disparate testing frameworks, annotation magic, and external harnesses (e.g., JUnit, pytest, Vitest, XCTest), resulting in mismatched assertions and environment confusion.
* **Principles:**
  - **Native Language Construct:** Tests are first-class syntax within the language (e.g., `test "name" { ... }`), placed directly adjacent to the units they verify without requiring external framework imports.
  - **Zero-Boilerplate Assertions:** Built-in assertion primitives with rich, machine-readable diff output on failure.
  - **Dual-Verification Model:**
    - **Fast Local Feedback:** Tests run instantly in Tick's native interpreter (`tick test`) for sub-second agent drafting and CTFE iteration loops.
    - **Authoritative Target Verification:** Tests automatically transpile into target test suites (e.g., `#[test]` in Rust, `pytest` in Python, `XCTest` in Swift), serving as the authoritative ground truth for cross-compiled behavior across all platforms.

## 14. Lossless Comments & Intent Anchors
* **Rationale:** AI agents collaborate by reading and writing rationale alongside code. When comments are treated as disposable compiler trivia, critical context and downstream documentation are lost during compilation and transpilation.
* **Principles:**
  - **Lossless Comment-Preserving AST:** Doc comments and intent annotations are first-class nodes bound directly to the AST items they precede, rather than discarded during lexing.
  - **Target-Idiomatic Doc Emission:** Comments authored in Tick are automatically transformed into native documentation standards in generated target code (e.g., KDoc for Kotlin, DocC `///` for Swift, docstrings `"""` for Python, Rustdoc `///` for Rust, JSDoc for TypeScript).
  - **Structured Intent Anchors:** Standard, token-efficient annotations (e.g., `/// @why`, `/// @invariant`) allow agents to communicate critical design rationale with minimal token overhead.

## 15. Semantic Diff & Patch Stability (Minimal Syntax Churn)
* **Rationale:** AI agents modify codebases primarily through line diffs and targeted block replacements. Syntactic choices that trigger cascading edits (like re-indenting entire files or juggling missing commas on adjacent lines) inflate patch token costs and create merge/patch collisions.
* **Principles:**
  - **Trailing Commas Supported Everywhere:** Trailing commas in argument lists, struct declarations, match arms, and arrays mean adding or removing an element touches exactly one isolated line.
  - **Zero Indentation-Cascade Traps:** Block scoping prevents massive whitespace reflow diffs when nesting or un-nesting logic.
  - **Localized Statement Boundaries:** Statements and declarations are self-terminating or clearly bounded, ensuring patch tools and agents can apply atomic, surgical edits.

## 16. Immutable Defaults, Move-By-Default & Reusable Identifiers
* **Rationale:** Uncontrolled mutable aliasing and hidden in-place object mutations across function calls are leading causes of silent regressions in AI-generated code.
* **Principles:**
  - **Immutable by Default:** All variable bindings and function parameters are strictly immutable unless explicitly declared with `mut`.
  - **Move Semantics by Default:** Passing a value to another function or binding moves ownership by default, eliminating accidental shared mutable aliasing across boundaries.
  - **Explicit Lexical Shadowing:** Re-declaring a binding with `let` in the same scope legally shadows prior bindings of the same identifier. This eliminates synthetic variable churn (`user1`, `user2`, `updated_user`) while preserving deterministic lexical scoping and enabling unambiguous transpiler renaming (`user_1`, `user_2`) for targets that disallow shadowing.
  - **Zero Aliasing Ambiguity:** Multiple concurrent mutable references to the same data are strictly forbidden at compile time.

## 17. Exhaustive Null Safety (No Implicit Missing Values)
* **Rationale:** Implicit null pointer and undefined reference exceptions represent the single most frequent category of runtime bugs in AI-authored code.
* **Principles:**
  - **Non-Nullable by Default:** Every type is strictly non-nullable unless explicitly marked as optional.
  - **No Implicit Null Literals:** Missing or optional values are represented exclusively through explicit, typed algebraic optionality rather than untyped universal null pointers.
  - **Compiler-Enforced Unwrapping:** Accessing fields or methods on an optional value without explicit checking or pattern-matching is a compile-time error.
  - **Universal Target Mapping:** Maps natively to the target languages' modern optionality systems (e.g., Swift optionals, Kotlin nullable types, Rust optional types, TypeScript union types).

## 18. Order-Independent Declarations (Safe Append-Only Generation)
* **Rationale:** In languages where declaration order matters (or where forward declarations and temporal dead zones exist), AI agents frequently fail when appending new helper functions or types to existing files. Because syntax parsing (Pass 1) is cleanly decoupled from semantic analysis, subsequent passes traverse the complete AST to resolve symbol tables, check types, and verify methods, making top-level declarations fully order-independent and safe for append-only generation.
* **Principles:**
  - **Multi-Pass Semantic Analysis:** Pure syntactic parsing produces a complete AST without requiring type feedback, allowing subsequent passes to resolve symbol tables, check types, and verify methods across the entire AST.
  - **Order-Independent Symbol Resolution:** Functions, types, traits, and constants can be declared in any order within a module or package without forward declarations or prototype headers.
  - **Safe Append-Only Modification:** AI agents can append new symbols to the end of a file with guaranteed reference resolution by earlier functions in that file.
  - **Zero Temporal Dead Zones:** Resolving top-level symbols is declarative rather than dependent on linear script execution ordering.

## 19. Single Unified Toolchain (Zero Configuration Sprawl)
* **Rationale:** In fragmented ecosystems (e.g., JS/TS or Python), AI agents waste significant token budget and tool invocations reconciling disparate, conflicting external tools (formatters, linters, test harnesses, bundlers) and drifting config files.
* **Principles:**
  - **All-in-One Canonical Binary:** The core `tick` toolchain natively integrates formatting, linting, type-checking, testing, transpilation, and local test interpretation into a single binary.
  - **Zero Config Churn:** Sane, strict, opinionated defaults work out of the box without requiring sprawling configuration files across repositories.
  - **Uniform Machine-Readable Interface:** Every toolchain command provides standardized structured output (e.g., `--format=json`) so agents can parse results, errors, and diffs programmatically without brittle text scraping.

## 20. Stable Error Codes & Actionable Repair Recipes
* **Rationale:** Generic prose error messages force AI agents to guess at solutions, often leading to counterproductive trial-and-error edits or incorrect type casts.
* **Principles:**
  - **Stable Error Code Catalog:** Every diagnostic is assigned a unique, immutable error code (e.g., `E0142`).
  - **Machine-Queryable Repair Recipes:** The toolchain provides canonical repair templates via CLI (e.g., `tick explain E0142`) and structured JSON output, explicitly demonstrating the violated invariant and the exact pattern to resolve it.
  - **Deterministic Self-Healing Loops:** Provides AI coding agents with direct, deterministic paths to fix syntax and type discrepancies in a single edit step.

## 21. Explicit Capabilities & Zero Ambient Environment Access
* **Rationale:** Implicit access to ambient host state (system clocks, environment variables, hardware randomness, unconstrained filesystem paths) introduces non-determinism, breaks compile-time execution (CTFE), and creates subtle platform incompatibilities when transpiling between mobile SDKs and backends.
* **Principles:**
  - **Zero Implicit Ambient Access:** Code cannot implicitly reach into host environment variables, read system timestamps, or generate entropy without an explicit capability or context passed in.
  - **Deterministic Compile-Time Evaluation:** All logic executed at compile time (CTFE) is guaranteed to produce byte-for-byte identical results regardless of host OS, architecture, or environment.
  - **Testability & Portability:** Functions that interact with the outside world require explicit handles, ensuring unit tests can mock or control environmental factors effortlessly across all target platforms.

## 22. Native Semantic Slicing & Codebase Introspection
* **Rationale:** Navigating large multi-file codebases forces AI agents to consume vast token budgets reading irrelevant function bodies and implementation details just to understand API surfaces or change impact.
* **Principles:**
  - **First-Class Symbol Skeletons:** The toolchain can extract high-density, token-minimized interface skeletons (public types, function signatures, contracts) on demand, omitting private implementation bodies.
  - **Compiler-Powered Blast Radius Queries:** Built-in semantic commands allow agents to query call graphs, type references, and symbol usages directly, identifying the exact impact of a refactor before altering code.
  - **Toolchain as Agent Coprocessor:** The compiler doubles as a semantic query engine specifically designed to feed compact, structured context directly into agent prompts and tools.

---

## 23. Transpilation-First Invariants (Pivot-Language Purity)
* **Rationale:** Tick code must transpile faithfully and idiomatically to targets like Rust, Python, TypeScript, Swift, and Kotlin.
* **Principles:**
  - Language primitives must represent the clean intersection of modern target semantics, avoiding reliance on quirks unique to any single host runtime.
  - Deterministic execution semantics and value equivalence ensure that business logic behaves identically across all targets (Python, Swift, Rust), without depending on host-specific physical memory layouts or garbage-collection implementations.
  - Standard library abstractions are designed to map to target language idioms rather than forcing heavy runtime shims.
