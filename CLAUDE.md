# Argus — reviewer guidance

When reviewing changes, in addition to bugs and security issues, flag violations of these project guidelines on changed lines and cite the rule id.

# Argus review guidelines

These are the project's review rules. Reviewers (human or automated) should flag violations
of any rule below as a review comment on the offending lines, in addition to correctness and
security bugs. Keep comments specific: name the rule (e.g. **[G3 Dependency injection]**),
say what is wrong, and suggest the fix.

## G1 — Composition over inheritance
- Do not introduce class inheritance to share behaviour. Prefer composition: hold a
  collaborator and delegate. Inheritance is acceptable only for genuine *is-a* relationships
  with a stable abstract interface (e.g. implementing a `Protocol`/interface, a Go interface,
  an abstract base class with no state).
- Flag: subclasses that override concrete parent methods to change behaviour; base classes
  that exist only to share helpers; inheritance depth > 1; mixins carrying state.

## G2 — Pragmatic SOLID
- **Single responsibility**: a type or function does one thing. Flag functions > ~60 lines that
  mix I/O, parsing and business logic; types that both manage lifecycle and implement policy.
- **Open/closed**: new variants should be added by adding a new implementation of an existing
  interface, not by adding another `if/else`/`switch` arm on a type tag. Flag growing type
  switches on an enum/string kind where an interface already exists.
- **Liskov**: implementations must honour the documented contract of the interface (no
  "not supported" panics in a method the interface requires, no stricter preconditions).
- **Interface segregation**: pass the narrowest interface a function needs (`io.Reader`, not
  `*os.File`; a `Protocol` with one method, not the whole service).
- **Dependency inversion**: core logic depends on abstractions (the provider `Protocol`s, Go
  interfaces, C ABI), never on a concrete provider/SDK/global.
- Pragmatic: do not demand abstractions for one-off code; flag only where a second
  implementation exists or is clearly imminent (providers, stores, transports).

## G3 — Dependency injection
- Collaborators (clients, stores, clocks, random sources, providers, loggers) are passed in
  through constructors/parameters, not created inside the function and not read from globals,
  module-level singletons, or environment variables deep inside logic.
- Time and randomness are injected (`clock`, `now()`, `rng`) so code is testable.
- Flag: `time.Now()`/`datetime.now()`/`Date.now()` inside business logic that has a clock
  available; `new`/`make_provider`/`os.environ`/`process.env` inside non-constructor code;
  hidden network calls in constructors.

## G4 — Error handling & boundaries
- Errors are returned/raised with context, never swallowed (`except Exception: pass`,
  `_ = err`, empty `catch`). Flag silent error drops and `log-and-continue` on errors that
  affect correctness.
- Validate at the boundary (input parsing), trust inside.

## G5 — Comment and documentation conventions
- Public functions/types have a doc comment stating *what* and *why*, preconditions and
  units (ms vs ns, normalized vs pixel coordinates).
- Comments describe intent, not mechanics. Flag comments that restate the code
  (`i += 1  # increment i`), **stale or misleading comments** (comment contradicts the code),
  commented-out code, `TODO`/`FIXME` without an owner or issue reference, and docstrings
  whose parameter lists do not match the signature.
- Language conventions: Go doc comments start with the identifier name; Python uses
  Google-style docstrings; TS uses TSDoc `/** */`; C++ public headers use `///` Doxygen style.

## G6 — Naming and units
- Names carry units and clock domain where relevant (`ts_ms`, `wall_ns`, `retention_s`).
  Flag unit-less time/size names and mixed clock domains in one expression.

## What not to flag
- Formatting handled by the formatter (ruff/gofmt/prettier/clang-format).
- Test code structure, unless a test is misleading.
