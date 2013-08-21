<img src="https://rawgithub.com/recursedoka/album/master/logo.svg" />

Album (meaning list in Latin) is a programming language that attempts to gather
the best ideas and features from Lisps and other functional languages, and bring
them into a single, robust environment that encourages continual experimentation
and improvement and without sacrificing stability or performance. It is meant
for both developing new, innovative technology and bringing that technology to
products under the ideology that companies that go directly from concept to
reality the fastest maintain an advantage over their competitors.

To keep the core language small, special forms and the standard library are
limited. Many features are modularly obtained with packages, including IO. Some
things that are standardized are error handling and concurrency, which both
require underlying access to the VM and are important to keep all code
fundamentally compatible. For example, C does not standardize their error
handling mechanism, which ends up causing a lot of confusion in the language.

All things in the core language are up to scrutiny. Improvements that challenge
the core language are encouraged through the GitHub pull request system. Those
that prove to be a better solution can be implemented in the core language. With
the power of homoiconicity, old code can easily be updated to use new features
when necessary.

Using the power of git branches, several versions of the language can be
maintained at once. This allows stability to be maintained within a single
branch for production software, with important bug fixes from the latest version
propogating to older versions that are still maintained. As such, new
maintainers are always beneficial to the community, and those interested in
helping should get involved in pull requests and issues.

## Features

- Simple macro system
- All functions are generic based on type and arity
- Generic set
- Destructuring
- Restarts
- Dynamic bindings
- Simplified CLOS-style type system
- Erlang-style strings
- Package system with management software
- JIT compiler built on custom static compiler
- Scheme pairs/lists
- Tail recursion
- Arc brevity including function composition
- Clojure concurrency
- Clojure hash tables and meta data
- Compiles to optimized bytecode
- Support for ARM with NEON, x86/64 with AVX

## Documentation

An interactive tutorial is in the works with video explanations and an
interactive REPL. This may be part of a development environment built with the
language. Most users should start here.

See the [specification](spec/index.md) for more information about the exact
implementation requirements. If you already know a lot about Lisp, this might be
a good place to start.

## Get It

You can't yet.
