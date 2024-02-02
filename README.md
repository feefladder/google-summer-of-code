# Rust Google Summer of Code project ideas
The Rust project has decided to join [Google Summer of Code 2024](https://summerofcode.withgoogle.com/) (GSoC) for the
first time in 2024! This page contains project ideas that could benefit Rust maintainers and the whole Rust community.

We invite contributors that would like to participate in GSoC to examine the project list and use it as an inspiration
for your project proposals. **You can also propose a project that is not included in this list**, for example
an improvement of some Rust crate. However, please note that ultimately, each project needs at least one mentor. We have
tried to make sure that all the ideas listed here would have a mentor available, but if you propose a different project,
you might have to find someone who would be willing to mentor you.

You can find some tips for applying for a Rust GSoC project [here](proposal-guide.md).

If you would like to discuss projects ideas or anything related to the Rust Project's involvement in GSoC 2024, you can
do so on the [`#gsoc`](https://rust-lang.zulipchat.com/#narrow/stream/421156-gsoc) Zulip stream.

## Index
- **Rust Compiler**
    - [Fast(er) register allocator for Cranelift](#Faster-register-allocator-for-Cranelift)
    - [C codegen backend for rustc](#C-codegen-backend-for-rustc)
    - [Extend annotate-snippets with features required by rustc](#Extend-annotate-snippets-with-features-required-by-rustc)
- **Infrastructure**
    - [Add support for multiple collectors to the Rust benchmark suite](#Add-support-for-multiple-collectors-to-the-Rust-benchmark-suite)
    - [Improve Rust benchmark suite analysis & frontend](#Improve-Rust-benchmark-suite-analysis--frontend)
    - [Improve bootstrap](#Improve-bootstrap)
    - [Improve infrastructure automation tools](#Improve-infrastructure-automation-tools)
- **Cargo**
    - [Move cargo shell completions to Rust](#Move-cargo-shell-completions-to-Rust)
    - [Implement workspace publish in Cargo](#implement-workspace-publish-in-cargo)
- **Crate ecosystem**
    - [Modernize the libc crate](#Modernize-the-libc-crate)
    - [Allow customizing lint levels and reporting in `cargo-semver-checks`](#allow-customizing-lint-levels-and-reporting-in-cargo-semver-checks)
    - [Add more lints to `cargo-semver-checks`](#add-more-lints-to-cargo-semver-checks)
    - [Internationalizing the Rust language]()

# Project ideas
The list of ideas is divided into several categories.

## Rust Compiler

### Fast(er) register allocator for Cranelift

**Description**

The Rust compiler uses various codegen backends to generate executable code (LLVM, GCC, Cranelift).
The Cranelift backend should provide very quick compile times, however its performance is currently
relatively bottlenecked by its register allocator.

The goal of this project is to implement a new register allocator for Cranelift, that would be tuned for very
quick compilation times (rather than optimal runtime performance of the compiled program). A first attempt could
simply create an allocator that spills all registers to stack, and a possible follow-up could be a linear scan allocator.
It would be useful to compare the compilation vs runtime performance trade-offs of various register allocation approaches.

**Expected result**

It will be possible to use a new register allocator in Cranelift that will work at least for simple programs and that
will improve Rust compilation times.

**Desirable skills**

Intermediate knowledge of Rust. Basic knowledge of assembly. Familiarity with compiler technologies is a bonus.

**Project size**

Medium or large.

**Difficulty**

Medium.

**Mentors**
- Amanieu d'Antras ([GitHub](https://github.com/Amanieu), [Zulip](https://rust-lang.zulipchat.com/#narrow/dm/143274-Amanieu))
- Chris Fallin ([GitHub](https://github.com/cfallin), [Zulip](https://rust-lang.zulipchat.com/#narrow/dm/327027-Chris-Fallin))

**Zulip streams**
- [Idea discussion](https://rust-lang.zulipchat.com/#narrow/stream/421156-gsoc/topic/Idea.3A.20fast.20register.20allocator.20for.20Cranelift)
- [Compiler team](https://rust-lang.zulipchat.com/#narrow/stream/131828-t-compiler)

### C codegen backend for `rustc`

**Description**

`rustc` currently has three in-tree codegen backends: LLVM (the default), Cranelift, and GCC.
These live at <https://github.com/rust-lang/rust/tree/master/compiler>, as `rustc_codegen_*` crates.

The goal of this project is to add a new experimental `rustc_codegen_c` backend that could turn Rust's internal
representations into `C` code (i.e. transpile) and optionally invoke a `C` compiler to build it. This will allow Rust
to use benefits of existing `C` compilers (better platform support, optimizations) in situations where the existing backends
cannot be used.

**Expected result**

The minimum viable product is to turn `rustc` data structures that represent a Rust program into `C` code, and write the
output to the location specified by `--out-dir`. This involves figuring out how to produce buildable `C` code from the
inputs provided by `rustc_codegen_ssa::traits::CodegenBackend`.

A second step is to have `rustc` invoke a `C` compiler on these produced files. This should be designed in a pluggable way,
such that any `C` compiler can be dropped in.

**Desirable skills**

Knowledge of Rust and `C`, basic familiarity with compiler functionality.

**Project size**

Large.

**Difficulty**

Hard.

**Mentor**
- Trevor Gross ([GitHub](https://github.com/tgross35), [Zulip](https://rust-lang.zulipchat.com/#narrow/dm/532317-Trevor-Gross))

**Zulip streams**
- [Idea discussion](https://rust-lang.zulipchat.com/#narrow/stream/421156-gsoc/topic/Idea.3A.20C.20codegen.20backend.20for.20.60rustc.60)
- [Compiler team](https://rust-lang.zulipchat.com/#narrow/stream/131828-t-compiler)

### Extend `annotate-snippets` with features required by rustc

**Description**

`rustc` currently has incomplete support for using [`annotate-snippets`](https://github.com/rust-lang/annotate-snippets-rs/)
to emit errors, but it doesn't support all the features that `rustc`'s built-in diagnostic rendering does. The goal
of this project is to execute the `rustc` test suite using `annotate-snippets`, identify missing features or bugs,
fix those, and repeat until at feature-parity.

**Expected result**

More of the `rustc` test suite passes with `annotate-snippets`.

**Desirable skills**

Knowledge of Rust.

**Project size**

Medium.

**Difficulty**

Medium to hard.

**Mentor**
- David Wood ([GitHub](https://github.com/davidtwco), [Zulip](https://rust-lang.zulipchat.com/#narrow/dm/116107-davidtwco))

**Zulip streams**
- [Idea discussion](https://rust-lang.zulipchat.com/#narrow/stream/421156-gsoc/topic/Idea.3A.20extend.20annotate-snippets)
- [Compiler team](https://rust-lang.zulipchat.com/#narrow/stream/131828-t-compiler)

## Infrastructure

### Add support for multiple collectors to the Rust benchmark suite

**Description**

Rust has an extensive [benchmark suite](https://github.com/rust-lang/rustc-perf) that measures the performance of the Rust compiler and Rust programs and
visualizes the results in an interactive web application. Currently, the benchmarks are gathered on a single physical
machine, however we are hitting the limits of how many benchmark runs we can perform per day on a single machine,
which in turn limits the benchmark configurations that we can execute after each commit.

The goal of this project is to add support for splitting benchmark execution across multiple machines. This will
require a refactoring of the existing suite and potentially also database schema modifications and implementation of new
features.

**Expected result**

It will be possible to parallelize the execution of the benchmark suite across multiple machines.

**Desirable skills**

Intermediate knowledge of Rust and database technologies (SQL).

**Project size**

Medium or large.

**Difficulty**

Medium.

**Mentor**
- Jakub Beránek ([GitHub](https://github.com/kobzol), [Zulip](https://rust-lang.zulipchat.com/#narrow/dm/266526-Jakub-Ber%C3%A1nek))

**Zulip stream**
- [Idea discussion](https://rust-lang.zulipchat.com/#narrow/stream/421156-gsoc/topic/Idea.3A.20multiple.20collectors.20for.20Rust.20benchmark.20suite)
- [Compiler performance working group](https://rust-lang.zulipchat.com/#narrow/stream/247081-t-compiler.2Fperformance)

### Improve Rust benchmark suite analysis & frontend

**Description**

Rust has an extensive [benchmark suite](https://github.com/rust-lang/rustc-perf) that measures the performance of the Rust compiler and Rust programs and
visualizes the results in an interactive web application. It received a lot of new features in the past few years, however
some of them are not as polished as they could be.

The goal of this project is to improve both the frontend website and its various dashboards, and also profiling and analysis
tools used to examine benchmarks in the suite. As an example, improvements could be made in the following areas:
- Runtime benchmarks. The suite recently got support for runtime benchmarks that measure the performance of Rust programs
compiled by a specific version of `rustc` (the Rust compiler). There is a lot of features that could be added to get
runtime benchmarks to the same support level as compile-time benchmarks, like adding and visualizing benchmark variance
analysis for them or adding runtime benchmark results to various dashboards in the frontend.
- Analysis of multithreaded performance. The Rust compiler has recently gained support for using multiple threads for its
frontend, but there is no configuration in the suite to parametrize how many threads will be used, nor any analysis of
how well are threads utilized. It would be nice to add analysis and visualisation for this.
- Some pages of the website still use HTML templates. It would be great to port these to the Vue-based frontend.

**Expected result**

New analyses will be available in the Rust benchmark suite, and/or the suite website will contain more useful data and
visualizations.

**Desirable skills**

Basic knowledge of Rust, intermediate knowledge of frontend web technologies (TypeScript, HTML, CSS, Vue).

**Project size**

Medium.

**Difficulty**

Medium.

**Mentor**
- Jakub Beránek ([GitHub](https://github.com/kobzol), [Zulip](https://rust-lang.zulipchat.com/#narrow/dm/266526-Jakub-Ber%C3%A1nek))

**Zulip streams**
- [Idea discussion](https://rust-lang.zulipchat.com/#narrow/stream/421156-gsoc/topic/Idea.3A.20improve.20Rust.20benchmark.20suite)
- [Compiler performance working group](https://rust-lang.zulipchat.com/#narrow/stream/247081-t-compiler.2Fperformance)

### Improve bootstrap

**Description**

The Rust compiler it bootstrapped using a complex set of scripts and programs generally called just `bootstrap`.
This tooling is constantly changing, and it has accrued a lot of technical debt. It could be improved in many areas, for example:

- Design a new testing infrastructure and write more tests.
- Write documentation.
- Remove unnecessary hacks.

**Expected result**

The `bootstrap` tooling will have less technical debt, more tests, and better documentation.

**Desirable skills**

Intermediate knowledge of Rust. Knowledge of the Rust compiler bootstrap process is welcome, but not required.

**Project size**

Medium or large.

**Difficulty**

Medium.

**Mentor**
- AlbertLarsan68 ([GitHub](https://github.com/albertlarsan68), [Zulip](https://rust-lang.zulipchat.com/#narrow/dm/510016-Albert-Larsan))

**Zulip streams**
- [Idea discussion](https://rust-lang.zulipchat.com/#narrow/stream/421156-gsoc/topic/Idea.3A.20improve.20bootstrap)
- [Bootstreap team](https://rust-lang.zulipchat.com/#narrow/stream/326414-t-infra.2Fbootstrap)

### Improve infrastructure automation tools

**Description**

Rust infrastructure uses many custom tools designed for automating pull request merging, handling discussions on Zulip,
managing GitHub permissions etc. It would be a great help to Rust maintainers if these tools were improved. Here are a
few possible tasks that could be implemented:

- Complete the implementation of [bors](https://github.com/rust-lang/bors), our new implementation of a merge queue
bot for GitHub. It currently lacks support for performing merges (it can only perform so-called "try builds").
- Add support for interacting with the Rust team calendar through Zulip, using e.g. some kind of GitHub app bot.
- Add support for creating Zulip streams using the [Rust team data](https://github.com/rust-lang/team) repository.
- Implement a GitHub app for [sync-team](https://github.com/rust-lang/sync-team), our tool for managing permissions of Rust maintainers.

**Expected result**

Rust infrastructure management tools will receive new features, better documentation and tests.

**Desirable skills**

Intermediate knowledge of Rust. Familiarity with GitHub APIs is a bonus.

**Project size**

Medium.

**Difficulty**

Medium.

**Mentors**
- Jakub Beránek ([GitHub](https://github.com/kobzol), [Zulip](https://rust-lang.zulipchat.com/#narrow/dm/266526-Jakub-Ber%C3%A1nek)) (bors, sync-team)
- Jack Huey ([GitHub](https://github.com/jackh726), [Zulip](https://rust-lang.zulipchat.com/#narrow/dm/232957-Jack-Huey)) (triagebot)

**Zulip streams**
- [Idea discussion](https://rust-lang.zulipchat.com/#narrow/stream/421156-gsoc/topic/Idea.3A.20improve.20infrastructure.20automation.20tools)
- [Infra team](https://rust-lang.zulipchat.com/#narrow/stream/242791-t-infra)

## Cargo

### Move cargo shell completions to Rust

**Description**

Cargo maintains Bash and Zsh completions, but they are duplicated and limited in features.
We want to implement completions in Cargo itself, so we can have a single implementation with per-shell skins ([rust-lang/cargo#6645](https://github.com/rust-lang/cargo/issues/6645)).
Most of the implementation will be in clap ([clap-rs/clap#3166](https://github.com/clap-rs/clap/issues/3166)), allowing many tools to benefit from this improvement.

**Expected result**

Cargo shell completion will be extended and implemented in Rust.
This will allow access to easier to add new commands / arguments to commands, richer results, and easier testing.

**Desirable skills**

Intermediate knowledge of Rust. Shell familiarity is a bonus.

**Project size**

Medium.

**Difficulty**

Medium.

**Mentor**
- Ed Page ([GitHub](https://github.com/epage), [Zulip](https://rust-lang.zulipchat.com/#narrow/dm/424212-Ed-Page))

**Zulip streams**
- [Idea discussion](https://rust-lang.zulipchat.com/#narrow/stream/421156-gsoc/topic/Idea.3A.20move.20cargo.20shell.20completions.20to.20Rust)
- [Cargo team](https://rust-lang.zulipchat.com/#narrow/stream/246057-t-cargo)

### Implement workspace publish in Cargo

**Description**

Today, developers can group Rust packages into a workspace to make it easier to operate on all of them at once.
However, `cargo package` and `cargo publish` do not support operating on workspaces ([rust-lang/cargo#1169](https://github.com/rust-lang/cargo/issues/1169)).

The goal of this project is to modify the Cargo build tool to add support for packaging and publishing Cargo workspaces.

**Expected result**

Milestone 1: `cargo package` can be run, with verification, with the standard package selection flags
Milestone 2: `cargo publish` can do the same as above, but also serially post the `.crate` files to the registry when done,
reporting to the user what was posted/failed if interrupted.

**Desirable skills**

Intermediate knowledge of Rust.

**Project size**

Medium.

**Difficulty**

Medium.

**Mentor**
- Ed Page ([GitHub](https://github.com/epage), [Zulip](https://rust-lang.zulipchat.com/#narrow/dm/424212-Ed-Page))

**Zulip streams**
- [Idea discussion](https://rust-lang.zulipchat.com/#narrow/stream/421156-gsoc/topic/Idea.3A.20implement.20workspace.20publish.20in.20Cargo)
- [Cargo team](https://rust-lang.zulipchat.com/#narrow/stream/246057-t-cargo)

## Crate ecosystem

### Modernize the libc crate

**Description**

The [libc](https://github.com/rust-lang/libc) crate is one of the oldest crates of the Rust ecosystem, long predating
Rust 1.0. Additionally, it is one of the most widely used crates in the ecosystem (#4 most downloaded on crates.io).
This combinations means that the current version of the libc crate (`v0.2`) is very conservative with breaking changes and
remains backwards-compatible with all Rust compilers since Rust 1.13 (released in 2016).

The language has evolved a lot since Rust 1.13, and we would like to make use of these features in libc. The main one is
support for `union` types to proper expose C unions.

At the same time there, is a backlog of desired breaking changes tracked in [this issue](https://github.com/rust-lang/libc/issues/3248). Some of these come from
the evolution of the underlying platforms, some come from a desire to use newer language features, while others are
simple mistakes that we cannot correct without breaking existing code.

The goal of this project is to prepare and release the next major version of the libc crate.

**Expected result**

The libc crate is cleaned up and modernized, and released as version 0.3.

**Desirable skills**

Intermediate knowledge of Rust.

**Project size**

Medium.

**Difficulty**

Medium.

**Mentor**
- Amanieu d'Antras ([GitHub](https://github.com/Amanieu), [Zulip](https://rust-lang.zulipchat.com/#narrow/dm/143274-Amanieu))

**Zulip streams**
- [Idea discussion](https://rust-lang.zulipchat.com/#narrow/stream/421156-gsoc/topic/Idea.3A.20modernize.20the.20libc.20crate)
- [Library team](https://rust-lang.zulipchat.com/#narrow/stream/219381-t-libs)

### Allow customizing lint levels and reporting in `cargo-semver-checks`

**Description**

[`cargo-semver-checks`](https://github.com/obi1kenobi/cargo-semver-checks) is a linter for semantic versioning. It ensures
that Rust crates adhere to semantic versioning by looking for breaking changes in APIs.

Currently, semver lints have a hardcoded level (e.g. breaking changes are "major") and are always reported at a "deny"
level: if the release being scanned is a minor version bump, any lints at "major" level are reported as errors.

This can be insufficient for some projects, which may desire to:
- configure some lints to have a different level — e.g. turn a semver "major" lint into a "minor" lint, or vice versa
- turn some lints into warnings instead of hard errorrs — reporting level "warn" instead of the default "deny"
- disable some lints altogether by setting their reporting to "allow"
- (stretch goal) allow customizing lint levels and reporting on a per-module basis

Having such functionality would allow `cargo-semver-checks` to ship additional lints that target changes whose semver
implications are defined by project maintainers on a per-project basis. An example of such a change is bumping the
minimum supported Rust version (MSRV) of a project — some projects consider it a semver-major change, whereas for
others it is minor or patch.

This functionality would also allow us to write lints similar to clippy's ["suspicious" lint group](https://doc.rust-lang.org/nightly/clippy/lints.html#suspicious),
flagging code that is suspect (and deserving of closer scrutiny) but possibly still correct. Such lints should be
opt-in / "warn" tier to avoid annoying users, which is something this project would enable us to do.

**Expected result**

`cargo-semver-checks` lints will be configurable via the [`package.metadata`](https://doc.rust-lang.org/cargo/reference/manifest.html#the-metadata-table) table in `Cargo.toml`
using a clear, simple and expressive way. The design will be suitable for both single-crate projects and workspaces.

**Desirable skills**

Intermediate knowledge of Rust.

**Project size**

Medium to large.

**Difficulty**

Medium.

**Mentor**
- Predrag Gruevski ([GitHub](https://github.com/obi1kenobi/), [Zulip](https://rust-lang.zulipchat.com/#narrow/dm/474284-Predrag-Gruevski-(he-him)))

**Zulip streams**
- [Idea discussion](https://rust-lang.zulipchat.com/#narrow/stream/421156-gsoc/topic/Idea.3A.20allow.20customizing.20.60cargo-semver-checks.60.20lint.20levels)

**Related links**
- [GitHub issue](https://github.com/obi1kenobi/cargo-semver-checks/issues/537)

### Add more lints to `cargo-semver-checks`

**Description**

[`cargo-semver-checks`](https://github.com/obi1kenobi/cargo-semver-checks) is a linter for semantic versioning. It ensures
that Rust crates adhere to semantic versioning by looking for breaking changes in APIs.

It can currently catch ~60 different kinds of breaking changes, so there are hundreds of kinds of breaking changes it
still cannot catch! The goal of this project is to extend its abilities, so that it can catch and prevent more breaking changes, by:
- adding more lints, which are expressed as queries over a database-like schema ([playground](https://play.predr.ag/rustdoc))
- extending the schema, so more Rust functionality is made available for linting

**Expected result**

`cargo-semver-checks` will contain new lints, together with test cases that both ensure the lint triggers when expected
and does not trigger in situations where it shouldn't (AKA false-positives).

**Desirable skills**

Intermediate knowledge of Rust. Familiarity with databases, query engines, or query language design is welcome but
not required.

**Project size**

Small to large (depends on how many lints will be implemented).

**Difficulty**

Small to medium (depends on the choice of implemented lints or schema extensions).

**Mentor**
- Predrag Gruevski ([GitHub](https://github.com/obi1kenobi/), [Zulip](https://rust-lang.zulipchat.com/#narrow/dm/474284-Predrag-Gruevski-(he-him)))

**Zulip streams**
- [Idea discussion]()

**Related Links**
- [Playground where you can try querying Rust data](https://play.predr.ag/rustdoc)
- [GitHub issues describing not-yet-implemented lints](https://github.com/obi1kenobi/cargo-semver-checks/issues?q=is%3Aissue+is%3Aopen+label%3AE-mentor+label%3AA-lint+)
- [Opportunities to add new schema, enabling new lints](https://github.com/obi1kenobi/cargo-semver-checks/issues/241)
- [Query engine adapter](https://github.com/obi1kenobi/trustfall-rustdoc-adapter)

### Internationalizing English Rust

**Description**

Rust -like most programming languages- is an intermediate language between English and Machine Language. If a non-English speaker wishes to learn programming, they now have to learn 2 languages: English and Rust. The goal of this project is not to *translate* Rust, but rather to put the tooling in place to *allow for* translation to happen smoothly (Internationalization in stead of Localization). Even if someone speaks general English well, it could still be beneficial to be able to program and reason about specific field knowledge in their native tongue, and translate it later.

Once internationalization is implemented, it would allow for the following workflow:

> Imagine Aagje, Bas and Carol are working on a mathematical project. Aagje is a mathematician and speaks only Dutch. Bas is intermediate in both computer science and maths and speaks both English and Dutch. Carol is a super smart computer scientist and speaks only English. Aagje thinks she can make a program to solve the Riemann hypothesis. She starts working in Dutch Rust on an implementation -asking Bas for help on computer science-, but the program is too slow. Bas asks Carol for help, but she cannot understand the program too well, since all function names and comments are in Dutch. Luckily, this GSoC project has already been completed. Therefore, the crate has a canonical language set to Dutch. Bas and aagje install the tooling (developed in this project) and translate the relevant names and comments using fluent.rs and . Now, the project is also (partially) available in English! Carol reads the source code and makes some changes (in English). She commits her changes (in English) and the linter gives warnings that her functions and comments are not available in the canonical crate language. This is OK, Bas comes in and translates them, understanding from Carol and explaining the changes to Alice. Then, they jointly earn one million dollars because they have solved the Riemann hypothesis.

Internationalizing Rust is a big, gradual project with many facets:
- A lot of work is currently being done on internationalizing the compiler output ([tracking issue](https://github.com/rust-lang/rust/issues/100717), [dev docs](https://rustc-dev-guide.rust-lang.org/diagnostics/translation.html)). This is important work and both orthogonal to as well as a prerequisite for this project to work well.
- There is [rouille](https://github.com/bnjbvr/rouille), which is both a joke and a starting point: It provides a procedural macro that translates all French keywords and function names to English before compilation. This means that error messages from the compiler will still have English function names.

This project could go as follows:
1. Implement one internationalization from the [tracking issue](https://github.com/rust-lang/rust/issues/100717) (if still available) to get accustomed to the compiler and tooling.
2. Add a compiler plugin/patch so that it understands that tokens have a "canonical" and "local" description.
3. Add a `Cargo.toml` entry for the canonical language and cargo commands for:
   - Setting up an internationalized crate (source code, not output) This:
     - `rustup install`s fluent.rs
     - Creates relevant folders/files.
     - Fills said files with all function names in the canonical language
     - Adds a pre-commit hook for switching back to the canonical language.
     - (stretch) checks all upstream crates for available translations.
     - (further stretches) allows for translating used function names/structs from upstream crates and automatically creates a pull request, adding the translations to the upstream crate. Alternatively, integrate with pontoon.
   - Switching locales: This translates *the entire crate* to another language and emits warnings for all names that are not translated.
4. Implement the developed internationalization tooling in a fork of [rustlings](https://github.com/rust-lang/rustlings) as a testing ground.

**Expected result**

Rustlings can be -and maybe is- partially translated to another language.

**Desirable skills**

Intermediate-good knowledge of Rust. Especially being able to read and understand Rust source code (well-documented). Being a non-native English speaker probably helps with motivation. In any case, there will be a second language to translate to ;)

**Project size**

Medium to large (depending on how far the stretches are taken).

**Difficulty**

Medium to hard.

**Mentor**
- Fee (Fairy) Fladder ([GitHub](https://github.com/feefladder/), [Zulip](HELP))

**Zulip streams**
- [Idea discussion]()

**Related Links**
Inspiration:
- [article that explains the needs much better than I can](https://www.wired.com/story/coding-is-for-everyoneas-long-as-you-speak-english/)
- [Comment](https://internals.rust-lang.org/t/internationalization-of-crate-metadata/13478/21?u=feefladder) on one of many discussions on Rust internals for internationalization.
- [hedy](https://hedy.org/) Multilingual education platform for gradually teaching Python to children.
- [koro](https://github.com/ChimeraCoder/koro) Bengali Go language version
- [babylscript](https://babylscript.plom.dev/) Multilingual Javascript through transpilation. Allows for creating a big multilingual-mess program.
- [plutojl](https://plutojl.org/) Educational environment for Julia. Since it works in-browser, Google translate browser plugins can do their magic.
- [Rust by example](https://doc.rust-lang.org/rust-by-example/) and [Rustlings](https://github.com/rust-lang/rustlings) Of course Rust educational tooling is inspirational!

Pointers:
- [cyn](https://docs.rs/syn/latest/syn/) The Rust parser.
- [Procedural macro blog](https://blog.logrocket.com/procedural-macros-in-rust/) For creating procedural macros.
- [fluent.rs](https://docs.rs/crate/fluent/latest) Translation crate that is used by the compiler.
- [rustc Dev docs](https://rustc-dev-guide.rust-lang.org/part-3-intro.html) parsing section.