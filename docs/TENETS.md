# Tenets of the Tick Programming Language

This document establishes the foundational design tenets of **Tick**. Unlike traditional programming languages that optimize primarily for human readability and ergonomic typing on mechanical keyboards, Tick is designed primarily for **AI coding agents** as authors, and for **deterministic cross-language transpilation**.

---

## 1. Transpilation-First Invariants (Pivot-Language Purity & Ephemeral Emission)
* **Rationale:** Tick is a pivot language designed to transpile deterministically to heterogeneous targets (Rust, Python, TypeScript, Swift, Kotlin, Go). It avoids host-specific runtime quirks and treats generated target code as an ephemeral build artifact.
* **Principles:**
  - **Intersection Semantics:** Language computational primitives represent the clean semantic intersection of modern target languages, avoiding reliance on runtime quirks unique to any single execution environment.
  - **Deterministic Target Equivalence:** Deterministic execution semantics and value equivalence ensure that business logic behaves identically across all targets (Python, Swift, Rust), without depending on host-specific physical memory layouts or garbage-collection implementations.
  - **Deterministic Memory Model Lowering:** While runtime computational primitives represent the clean semantic intersection of targets, the ownership and aliasing model operates as an ahead-of-time static proof system. By proving consumption and non-aliasing invariants at compile time, Tick eliminates heavy runtime borrow checkers or custom GC engines, lowering cleanly to target idioms and minimal shims where target value semantics diverge: data structures are acyclic by construction under Tick's affine ownership model to prevent retain leaks under ARC (Swift), lifetimes are cleanly erased for GC targets (Python, Go, Kotlin, TypeScript), and emitted code is guaranteed to pass `rustc` borrow checking without requiring `unsafe` blocks.
  - **Flexible, Minimal Runtime Footprint (Dual Distribution & Aggregated Shims):** Standard library abstractions support two consumption models: (1) linking an official pre-packaged Tick standard library dependency in target projects for convenience, or (2) tree-shaking and synthesizing strictly the minimal shims and wrappers required by the generated code into a custom namespace. Multiple independent compilations can aggregate their standard library requirements into a single deduplicated namespace, guaranteeing zero unused runtime bloat.
  - **Ephemeral Build Artifacts (Debuggable, Not Hand-Crafted):** Generated target code is treated as an ephemeral compilation artifact rather than checked-in source code. While the transpiler makes a minimal effort to maintain general legibility and faithfully preserve Tick identifiers and symbols for stack trace inspection and debugging, it makes zero guarantees of human stylistic elegance or target linter compliance.

---

## 2. High Token Efficiency & Empirical BPE Co-Design
* **Rationale:** Context windows in Large Language Models (LLMs) are finite and directly impact cost, latency, and reasoning depth. Verbose boilerplate and unoptimized syntax consume valuable attention budget.
* **Principles:**
  - **Maximize Semantic Density:** Maximize semantic density per token; eliminate ceremonial boilerplate, redundant declarations, and mandatory enclosing class/namespace envelopes.
  - **Single-Token Keywords:** Favor concise, established keywords that tokenize into single BPE tokens (e.g., `fn`, `let`, `mut`, `ret`, `type`, `use`).
  - **Empirical Tokenizer Alignment (BPE Vocabulary Co-Design):** Keywords, operators, delimiter sequences, and canonical formatting conventions (such as indentation width) are empirical choices co-designed and benchmarked against frontier LLM tokenizers to guarantee single-token representations and eliminate multi-token punctuation splintering.
  - **Semantic Token ROI (Investment Over Waste):** High token efficiency does not mean omitting specification; tokens invested in formal contracts (`pre`/`post`), type invariants, and intent anchors (`@why`) yield high net token savings by eliminating multi-file context expansion, defensive runtime boilerplate, and iterative debugging turns.
  - **No Redundant Decorative Punctuation:** Eliminate superfluous parentheses around control conditions, redundant type annotations where locally unambiguous, and decorative ceremony where grammar constructs are already unambiguous.

---

## 3. Single Canonical Representation & Decoupled Prefix Grammar
* **Rationale:** Languages with multiple syntactic forms for the same semantic construct dilute LLM probability distributions, triggering hallucinated variants and edge-case bugs. Syntactic parsing must be decoupled from semantic typing to enable instant AST construction with bounded lookahead.
* **Principles:**
  - **Single Canonical Construct:** Exactly **one** canonical way to express any given operation or construct; zero syntactic synonyms.
  - **Single Function Syntax:** Unified declaration syntax for free functions, methods, and closures/lambdas.
  - **Single Iteration Model:** One unified loop construct rather than multiple overlapping primitives (`for`, `while`, `do-while`, `forEach`).
  - **Strict Boolean Conditions:** Conditionals (`if`, loop guards) require an explicit `bool` expression with no implicit "truthiness" conversions.
  - **Unambiguous Prefix Grammar:** Constructs declare their identity up front via explicit prefix keywords (`fn`, `let`, `type`, `loop`), allowing both token prediction and compiler parsing to proceed deterministically with bounded lookahead.
  - **Unambiguous Generic Delimiters:** Generic parameters use unambiguous delimiters (such as bracketed generics `[T]`) to eliminate lookahead ambiguities and parser backtracking.
  - **Pure Single-Pass Syntactic AST Construction (Pass 1):** Pass 1 is strictly a single-pass syntactic check that constructs a valid AST from the token stream using bounded lookahead (combining LL(k) declarations and Pratt expression parsing), completely independent of symbol tables or type information.
  - **Zero Lexer/Parser Semantic Feedback:** Parsing grammar never requires knowing whether an identifier is a type, variable, or function (unlike C/C++). Lexing and parsing operate with zero semantic feedback loops.
  - **Order-Independent Declarations (Multi-Pass Semantics):** Pure syntactic parsing (Pass 1) enables subsequent semantic passes to resolve symbol tables and check types across the entire AST, guaranteeing order-independent symbol declarations and enabling safe append-only declarations for AI agents.
  - **Layout-Deterministic Delimiters:** Formatting rules are compiler-enforced to prevent stylistic variance.

---

## 4. Lossless Comments & Intent Anchors
* **Rationale:** AI agents collaborate by reading and writing rationale alongside code. When comments are treated as disposable compiler trivia, critical context and downstream documentation are lost during compilation and transpilation.
* **Principles:**
  - **Lossless Comment-Preserving AST:** Doc comments and intent annotations are first-class nodes bound directly to the AST items they precede, rather than discarded during lexing.
  - **Target-Idiomatic Doc Emission:** Comments authored in Tick are automatically transformed into native documentation standards in generated target code (e.g., KDoc for Kotlin, DocC `///` for Swift, docstrings `"""` for Python, Rustdoc `///` for Rust, JSDoc for TypeScript).
  - **Structured Intent Anchors:** Standard, token-efficient annotations (e.g., `/// @why`, `/// @invariant`) allow agents to communicate critical design rationale with minimal token overhead.

---

## 5. Semantic Diff & Patch Stability (Minimal Syntax Churn)
* **Rationale:** AI agents modify codebases primarily through line diffs and targeted block replacements. Syntactic choices that trigger cascading edits inflate patch token costs and create merge collisions.
* **Principles:**
  - **Canonical Trailing Commas:** Trailing commas are mandatory for all multi-line lists (argument lists, struct fields, match arms, arrays) to ensure modifying an element touches exactly one isolated line, and strictly forbidden on single-line lists, enforcing exactly one canonical representation per layout.
  - **Deterministic Statement Termination (Mandatory Semicolons):** Mandatory semicolons explicitly terminate statements. This eliminates ambiguous Automatic Semicolon Insertion (ASI) heuristics, guarantees that multi-line expressions never break across line diffs, and prevents patches from terminating statements prematurely.
  - **Robust Explicit Block Structure (Zero Indentation Cascades):** Explicit block scoping prevents massive whitespace reflow diffs when nesting or un-nesting logic, eliminating off-by-one whitespace slips and indentation-cascade traps common in whitespace-sensitive languages.
  - **Localized Statement Boundaries:** Statements and declarations are self-terminating or clearly bounded, ensuring patch tools and agents can apply atomic, surgical edits.

---

## 6. Compile-Time Affine Ownership, Immutability & Lexical Shadowing
* **Rationale:** Latent bugs, uncontrolled mutable aliasing, and hidden in-place object mutations across function calls are leading causes of silent regressions in AI-generated code.
* **Principles:**
  - **Compile-Time Ownership & Borrow Verification:** Rust-inspired ownership, borrowing, and lifetime semantics operate strictly as compile-time static proof invariants within the frontend. The compiler statically enforces consumption and non-aliasing rules before code emission, ensuring that memory safety and value semantics are guaranteed at authoring time (see Tenet 1 for target runtime lowering).
  - **Immutable by Default:** All variable bindings and function parameters are strictly immutable unless explicitly declared with `mut`.
  - **Move Semantics by Default:** Passing a value to another function or binding moves ownership by default, eliminating accidental shared mutable aliasing across boundaries.
  - **Explicit Lexical Shadowing:** Re-declaring a binding with `let` in the same scope legally shadows prior bindings of the same identifier. This eliminates synthetic variable churn (`user1`, `user2`, `updated_user`) while preserving deterministic lexical scoping and enabling unambiguous transpiler renaming (`user_1`, `user_2`) for targets that disallow shadowing.
  - **Zero Aliasing Ambiguity:** Multiple concurrent mutable references to the same data are strictly forbidden at compile time.

---

## 7. Determinism Over Cleverness (Predictable Semantics & Explicit Failure)
* **Rationale:** Complex language "magic" (operator overloading surprises, implicit coercions, hidden exception flows) triggers subtle hallucinations and regressions in AI agent reasoning.
* **Principles:**
  - **No Implicit Conversions:** Type casting and conversions must always be explicit.
  - **Deterministic Evaluation Order:** Strict, unambiguous left-to-right evaluation across all expressions.
  - **Explicit Error Flow (Result Unions):** Recoverable errors are represented as typed return value unions/results (inspired by rich error aggregation proposals like Kotlin KEEP-0441 and languages like Rust and Zig). The caller is forced by the compiler to explicitly handle or intentionally propagate errors.
  - **Fatal-Only Exceptions:** Exceptions/panics are reserved exclusively for unrecoverable, fatal program failure (process death). They are never used for ordinary control flow.

---

## 8. Locality of Reasoning (Self-Contained Units)
* **Rationale:** AI agents have finite attention and struggle when understanding a function requires chasing deeply nested inheritance, ambient global variables, or implicit dependencies across dozens of files.
* **Principles:**
  - **Local Context Verifiability:** Code must be understandable and verifiable using only its immediate local context and explicit signatures.
  - **Zero Global Mutable State:** No global mutable state; all data flow is explicit through function inputs, outputs, and ownership transfers.
  - **Strict Local Type Inference:** Local type inference only: type inference never leaks past function boundaries; all public APIs and boundary signatures are fully explicit.
  - **Zero Ambient Magic:** No implicit contextual "magic" (such as hidden thread-locals, ambient dependency injection, or invisible lifecycle hooks).

---

## 9. First-Class Invariants & Multi-Tier Contract Verification
* **Rationale:** Specifying explicit constraints as formal contracts allows AI agents to reason about code soundness locally, author robust logic without hallucinations, and verify behavior without needing to read distant implementation details.
* **Principles:**
  - **Explicit Function Contracts:** Functions can declare formal preconditions (`pre`) and postconditions (`post`) directly on their signatures.
  - **Type Invariants:** Data structures can declare invariants that must hold true after instantiation and mutation.
  - **Multi-Tier Contract Verification:** Contracts are evaluated at compile time during CTFE when expressions operate on compile-time constants; a small, tractable subset of simple structural invariants can be verified statically without a heavy generic solver, while general contracts are enforced via runtime assertions in debug/test builds and leveraged for property-based fuzz testing.
  - **Contract-as-Interface:** Calling agents rely entirely on the function's contract guarantees rather than needing to parse the function's internal body.

---

## 10. Composable, Flat Typing over Deep Inheritance (Traits Only)
* **Rationale:** Class implementation inheritance hierarchies introduce fragile base class problems, complex method resolution order (MRO) bugs, and hidden parent state mutations that confuse AI reasoning. Crucially, implementation inheritance maps poorly and inconsistently across targets (Rust and Go lack it entirely).
* **Principles:**
  - **Zero Implementation Inheritance:** No `class ... extends` hierarchies.
  - **Pure Data & Pure Interfaces:** Clean separation between pure data records/structs and behavioral interface contracts (`trait`).
  - **Composition Over Inheritance:** Code reuse is achieved via explicit data composition and trait implementation.
  - **Universal Transpilation Alignment:** Maps cleanly onto Rust `trait`, Swift `protocol`, Kotlin/Java `interface`, TypeScript `interface`, and Go `interface`.

---

## 11. Exhaustive Null Safety (No Implicit Missing Values)
* **Rationale:** Implicit null pointer and undefined reference exceptions represent the single most frequent category of runtime bugs in AI-authored code.
* **Principles:**
  - **Non-Nullable by Default:** Every type is strictly non-nullable unless explicitly marked as optional.
  - **No Implicit Null Literals:** Missing or optional values are represented exclusively through explicit, typed algebraic optionality rather than untyped universal null pointers.
  - **Compiler-Enforced Unwrapping:** Accessing fields or methods on an optional value without explicit checking or pattern-matching is a compile-time error.
  - **Universal Target Mapping:** Maps natively to the target languages' modern optionality systems (e.g., Swift optionals, Kotlin nullable types, Rust optional types, TypeScript union types).

---

## 12. Deterministic Compile-Time Evaluation (CTFE) & Explicit Capabilities
* **Rationale:** Agents frequently need to compute lookup tables, constants, and precomputed static data arrays natively without runtime initialization overhead. Furthermore, implicit access to ambient host state breaks compile-time execution and introduces platform non-determinism.
* **Principles:**
  - **Unified Compile-Time Interpreter:** A unified interpreter that allows executing pure Tick code at compile time (`comptime`).
  - **Pure Constant Evaluation & Static Data Generation:** CTFE is strictly scoped to pure constant expression evaluation and static data generation (e.g., precomputing lookup tables, numeric constants, and static data arrays).
  - **Explicit Source Declarations (Zero Dynamic Code Synthesis):** CTFE cannot dynamically synthesize arbitrary types, signatures, or functions. All types, interfaces, and functions must be declared explicitly in author-written Tick source code, strictly preserving Pass 1 single-pass syntactic AST construction and ensuring every emitted target construct maps 1:1 to an explicit author-written Tick AST token span in source maps.
  - **Zero Implicit Ambient Access:** Code cannot implicitly reach into host environment variables, read system timestamps, or generate entropy without an explicit capability handle passed in.
  - **Deterministic CTFE:** All logic executed at compile time (CTFE) is guaranteed to produce byte-for-byte identical results regardless of host OS, architecture, or environment.
  - **Testability & Portability:** Functions that interact with the outside world require explicit handles, ensuring unit tests can mock environmental factors effortlessly across all target platforms.

---

## 13. Unified Toolchain, Agentic Diagnostics & Semantic Introspection
* **Rationale:** AI agents require instantaneous feedback, actionable repair instructions, and deep codebase introspection without configuration sprawl or brittle text scraping.
* **Principles:**
  - **All-in-One Canonical Binary:** The core `tick` toolchain natively integrates formatting, linting, type-checking, testing, transpilation, and local test interpretation into a single binary.
  - **Minimal Configuration & Pure Code Generation (`tick.toml`):** A single minimal configuration file (`tick.toml`) configures Tick compiler and transpilation settings. Tick focuses strictly on generating source code and source maps, leaving downstream project packaging and target package manifests (`package.json`, `Cargo.toml`, `pyproject.toml`) to the developer or authoring agent.
  - **Dual-Verification Model:** Tick's language specification and native execution semantics are the single, authoritative ground truth.
    - *Fast Local Feedback:* Tests run instantly in Tick's native interpreter (`tick test`) to verify business logic against Tick's canonical semantics during drafting and CTFE iteration loops.
    - *Target Conformance Verification:* Target test suites (e.g., `#[test]` in Rust, `pytest` in Python, `XCTest` in Swift) verify transpilation conformance and backend regression resistance (ensuring the transpiler faithfully preserved Tick's canonical semantics on target platforms), rather than target platforms acting as competing/divergent ground truths.
  - **Machine-Actionable Structured Diagnostics:** Compiler errors and warnings provide structured JSON output alongside human-readable text, with exact source spans and clear explanations of violated invariants.
  - **Stable Error Catalog & Repair Recipes:** Every diagnostic has an immutable error code (e.g., `E0142`) backed by machine-queryable repair recipes (`tick explain E0142`) that provide canonical patches for deterministic self-healing loops.
  - **High-Fidelity Token Source Maps:** Transpilation generates precise, token-level source maps linking all emitted target constructs back to their originating Tick AST spans, enabling external build harnesses and agent toolchains to trace downstream compiler errors or runtime stack traces directly back to Tick source.
  - **Native Semantic Slicing & Blast-Radius Queries:** The toolchain can extract high-density symbol skeletons (public signatures and contracts) on demand, omitting private implementation bodies, and query call graphs and symbol blast radii before refactoring.
