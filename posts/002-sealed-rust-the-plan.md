# Sealed Rust - The Plan

**Sealed Rust** is the effort led by [Ferrous Systems GmbH] to qualify the Rust Language and Compiler for use in the Safety Critical domain. This is the second post in a series detailing our plans around this effort. This post expands on concepts discussed in our first article, [Sealed Rust - The Pitch].

Our goal is to **improve the status-quo of software quality and correctness in safety critical domains** by enabling the use of the Rust Programming Language for safety critical software development. We believe that Rust is a significant improvement over existing tools both from a safety and quality perspective, as well as an improvement for developer and software team productivity.

You can follow the progress on these efforts on the [Ferrous Systems' Blog], by [subscribing to our newsletter], or by [contacting us directly].

[Ferrous Systems GmbH]: https://ferrous-systems.com
[Ferrous Systems' Blog]: https://ferrous-systems.com/blog/
[Sealed Rust - The Pitch]: https://ferrous-systems.com/blog/sealed-rust-the-pitch/
[subscribing to our newsletter]: http://eepurl.com/guoC6P
[contacting us directly]: mailto:sealed-rust@ferrous-systems.com

Since our last post in June 2019, we've been busy speaking to all of the wonderful folks that have reached out to us. In general, we have had the following primary groups of people get in contact:

* Companies interested in using a qualified version of the Rust Language for safety critical purposes
* Companies interested in parts of Sealed Rust, outside of safety critical industries, but instead working on **Mission Critical** infrastructure or research
* Companies that provide relevant tooling that would like to add Rust support, such as Compiler, RTOS, and Static Analysis tooling providers
* Interested individuals, including folks interested in using or contributing to Sealed Rust

Over these past months, we've made **two primary advances**:

1. We have a rough idea of our initial approach at qualification of Sealed Rust
2. We have refined our potential funding approaches for realizing these efforts

In this post, we'll discuss our initial approach to the problem, and in our next post, we will discuss our potential funding approaches.

## The Initial Approach

Our plan involves splitting the problem domain of qualifying a Rust compiler into the following parts:

1. Specifying the Rust Language, and any critical libraries or tools required for a minimal environment
2. Specifying an Intermediate Representation of the Rust Language that will be produced by Rust Compiler Frontend(s), and consumed by Rust Compiler Backend(s) or Static/Dynamic Analysis Tools
3. Validating a Rust Compiler Frontend in its ability to produce correct Intermediate Representation from a given Source Code input that is coherent with the Rust Language Specification
4. Validating a Rust Compiler Backend in its ability to produce correct binaries or machine code from a given Intermediate Representation
5. Domain Specific Qualification efforts, e.g. for the Automotive, Medical, or Avionics-relevant Tool Qualification standards

### The Rust Language Specification

Although standards vary in their specific requirements for using tools like languages or compilers as part of delivering a safety critical system, the overarching goals from these standards can be simplified down to "Does the tool do what it is supposed to do, as directed or automated by the developers, at an appropriate level of confidence for the application it will be used for?".

Without a specification of what the Rust Language is, it can be difficult to prove that a compiler is doing something correctly. A language specification helps to turn the evaluation of compiler behavior from Subjective (e.g. it "seems right"), to Objective (e.g. it is "definitely right").

In the realm of safety critical, we usually refer to specifications as "normative", or mandatory/required, or "non-normative", which can be helpful for understanding, but not necessarily binding. Think of "Normative" specifications like written laws, while "non-normative" specifications are like informational pamphlets about those laws.

Today, Rust has a wealth of non-normative specifications (some more comprehensive than normative specifications of other tools/languages!), including:

* [The Rust Reference]
* [The Rust Book]
* [Unsafe Code Guidelines]
* [The Rustonomicon]
* The [Core Library] and [Standard Library] Docs
* [Rust RFCs] and their associated pull requests

However, today there is only one *de facto* normative specification of the Rust Language: The [Rust Compiler] itself. In short, "the implementation is the specification" today. As there are dozens of commits to the Rust Compiler every day, and the codebase itself is composed of hundreds of thousands of lines of code, it would be exceedingly difficult to validate the compiler against itself.

[The Rust Reference]: https://doc.rust-lang.org/stable/reference/
[The Rust Book]: https://doc.rust-lang.org/book/
[The Rustonomicon]: https://doc.rust-lang.org/nomicon/
[Unsafe Code Guidelines]: https://rust-lang.github.io/unsafe-code-guidelines/
[Core Library]: https://doc.rust-lang.org/core/index.html
[Standard Library]: https://doc.rust-lang.org/std/index.html
[Rust Compiler]: https://github.com/rust-lang/rust/
[Rust RFCs]: https://github.com/rust-lang/rfcs

We propose preparing a normative specification for a subset of the Rust Language, that includes:

* A subset of the Language and its features
* A subset of critical language libraries necessary for a minimal operating environment, such as parts of `libcore`
* A subset of critical language tooling necessary for a minimal operating environment, such as `rustc` and perhaps `cargo`

We aim to contribute to the existing non-normative references, and extend them in a way that can be used for normative and verification purposes. We also plan to utilize academic research and formal proofs/modeling of components of the Rust Language, such as the Type System and Borrow Checker.

We intend for this specification to only extend to components of the language that have reached the state of "Sealed", as described in our [first post][Sealed Rust - The Pitch].

### Specifying an Intermediate Representation

In Rust, there are several layers to the code generation process. Source starts as Rust Code, is translated in roughly the following steps (many omitted for the sake of brevity):

* Rust Source Code
* AST - Abstract Syntax Tree
* HIR - High Intermediate Representation
* MIR - Middle Intermediate Representation

From here, MIR is traditionally translated to LLVM IR, which is fed to LLVM to make the final process to target-specific object code. However, recent work has been started to add a new code generation backend to Rust - Cranelift. Cranelift is targeted as a simpler code generation backend that is useful for WASM applications, as well as faster unoptimized builds at some point in the future.

This new code generation point is interesting and relevant to us, as it has made the Rust compiler more modular - adding an explicit segmentation point for multiple compiler code generation backends.

When validating a compiler from source code to object code, there is a lot of ground to cover, with wildly different domain models. On one end, you have the theoretical Rust model of how program flow, ownership/borrowing, and other language concerns work. On the other end, you have the operational model of each architecture or even each target you could be generating code for. Although not all qualified usage of Rust needs to be verified down to object code, standards like DO-178C's Level A, used in flight control systems, do require this end to end qualification.

We think the best way to make this challenge reasonable is to **split the problem in the middle**: specifying an Intermediate Representation somewhere in the neighborhood of Rust's existing MIR layer - somewhere where there is contextual information about the original source code still available, but the code has been transformed into a much less Rust-specific form that can be verified independently of the actual binary/object code produced.

Although this IR needs to be specified for the purposes of validation, it does not need to have the same stability guarantees as the Rust code it represents. This means that our IR can change from release to release of Sealed Rust, which allows for compiler internals to change over time, without changing Rust's backwards compatible guarantees.

This specified IR also has the useful side effect of being an *excellent* output for static analysis tools, such as those that analyze for things like max stack usage, code complexity, or even as a part of simulation and formal modeling.

### Validating a Compiler Frontend

Once we have a specification of what the Rust Language represents, and a specified IR, we have now established our "inputs" and "outputs" for validation! Or at least, the first set of inputs and outputs.

We'll leave a more comprehensive discussion of this step for a later post, however we imagine that there will be a number of automated tests developed that exercise the specification of the language, perhaps using tools like [MIRI, the MIR Interpreter], as a target environment for validation efforts. The use of a tool like MIRI would allow us to validate the operational semantic **behavior** of the emitted IR, rather than just a string comparison with expected IR output.

[MIRI, the MIR Interpreter]: https://github.com/rust-lang/miri

We initially plan to validate the existing `rustc` codebase against the written specification, providing automated tests that ensure correctness for all portions covered by the Sealed Rust initiative.

In the future, it may be necessary to increase the level of confidence in this validation, which could prompt the need for a "simpler" and more verifiable reimplementation of `rustc`, written in a way that is willing to trade simplicity/verifiability for features or performance, or perhaps even using only the Sealed Rust portions of the language. However, we still see the most reasonable path to an initial deliverable being the improvement and qualification of the existing Rust compiler.

### Validating a Compiler Backend

Similar to validating the Compiler Frontend, we will need to validate the translation from the specified IR down to a suitable level, such as Object Code.

One important thing to note here is that many qualified C or C++ compilers already have an internal IR used for code generation, and additionally, there are vendors that provide test suites for the purposes of qualification of open source C/C++ compilers such as LLVM or GCC for use in safety critical domains.

By partnering with companies offering one of these products, the effort to validate the "last mile" of code generation could be greatly shortened, by only being required to specify an "IR to IR" translation layer, which should be vastly less complex than validation down to the final machine code.

### Domain Specific Validation

Different safety critical domains require different deliverables for tool usage. This is typically referred to as Tool Qualification, and involves documentation and analysis of the tool and its fitness for purpose.

We'll also expand on this step more in future steps, where we discuss the requirements for tools in different safety critical domains.

### Cooperation with the upstream Rust Project

As mentioned in our previous post, we intend for all of the work enumerated above to be done collaboratively with the upstream Rust project, and where possible and relevant, upstream any changes or improvements to the compiler, documentation, testing, or analysis made as part of these efforts. What this means in practice is:

* We want any changes we propose to fit the overall design and intent of the Rust project
* We aim to get buy-in from the upstream language, library, and compiler contributors and relevant teams
* We aim to not add any additional burden for contributors to the Rust project, instead funding additional contributions, to the benefit of the Rust project as a whole

Said shortly, we don't ever wish to see Sealed Rust as a fork of the upstream project. We see Rust's strength in central contribution of improvements that benefit the whole ecosystem, and we believe that the efforts to enable qualified usage of Rust can benefit the Rust community as a whole.

## What's next for Sealed Rust?

In the next post, we'll discuss more about the financial details and opportunities for partnerships in making Sealed Rust a possibility.

For now, more than ever so far, **we need to hear from companies that are interested in being a part of the Sealed Rust initiative!** In particular, we are interested in hearing from parties such as:

* Companies interested in using Rust in safety critical domain and projects
* Companies interested in using Rust for mission critical infrastructure, particular around modeling and verification of code written in Rust
* Compiler vendors interested in partnering with us to add Rust support to their qualified offerings
* Academic researchers interested in Rust for verification and validation purposes
* Investors interested in funding these efforts

If you or your company are interested in these efforts, please [send us an email][contacting us directly]!

Thank you to [Mazdak Farrokhzad] and [Niko Matsakis] from the Rust project for their review and feedback on this post.

[Mazdak Farrokhzad]: https://github.com/centril
[Niko Matsakis]: https://github.com/nikomatsakis
