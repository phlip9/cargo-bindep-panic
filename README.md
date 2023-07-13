# cargo-bindep-panic

This repo reproduces a panic in `cargo` when using the unstable `bindeps`
feature.

### Steps to reproduce

```bash
$ git clone https://github.com/phlip9/cargo-bindep-panic
$ cd cargo-bindep-panic
$ RUST_BACKTRACE=1 cargo check

thread 'main' panicked at 'activated_features for invalid package: features did not find PackageId { name: "some-leaf-dep", version: "0.1.0", source: "/Users/phlip9/dev/cargo-bindep-panic/some-leaf-dep" } ArtifactDep(CompileTarget { name: "x86_64-fortanix-unknown-sgx" })

Stack backtrace:
   0: std::backtrace::Backtrace::create
   1: <anyhow::Error>::msg::<alloc::string::String>
   2: <cargo::core::resolver::features::ResolvedFeatures>::activated_features_int
   3: cargo::core::compiler::unit_dependencies::new_unit_dep_with_profile
   4: cargo::core::compiler::unit_dependencies::compute_deps
   5: cargo::core::compiler::unit_dependencies::deps_of
   6: cargo::core::compiler::unit_dependencies::deps_of
   7: cargo::core::compiler::unit_dependencies::deps_of
   8: cargo::core::compiler::unit_dependencies::deps_of
   9: cargo::core::compiler::unit_dependencies::deps_of
  10: cargo::core::compiler::unit_dependencies::deps_of
  11: cargo::core::compiler::unit_dependencies::deps_of
  12: cargo::core::compiler::unit_dependencies::deps_of_roots
  13: cargo::core::compiler::unit_dependencies::build_unit_dependencies
  14: cargo::ops::cargo_compile::create_bcx
  15: cargo::ops::cargo_compile::compile_ws
  16: cargo::ops::cargo_compile::compile
  17: cargo::commands::check::exec
  18: cargo::cli::main
  19: cargo::main
  20: std::sys_common::backtrace::__rust_begin_short_backtrace::<fn(), ()>
  21: std::rt::lang_start::<()>::{closure#0}
  22: std::rt::lang_start_internal
  23: _main', src/cargo/core/resolver/features.rs:322:14
```

### Crate graph

```
top-level-that-depends-on-bindep
  |
  | [dependencies] artifact = "bin", target = "x86_64-fortanix-unknown-sgx"
  |     - or -
  | [build-dependencies] artifact = "bin", target = "x86_64-fortanix-unknown-sgx"
  V
will-get-bindeped
  |
  | [dependencies]
  V
dep-that-depends-on-build-only-dep
  |
  | [build-dependencies]
  V
build-only-dep
  |
  | [target.'cfg(unix)'.dependencies]
  V
some-leaf-dep
```

### Host Cargo Version

```bash
$ cargo version --verbose

cargo 1.73.0-nightly (45782b6b8 2023-07-05)
release: 1.73.0-nightly
commit-hash: 45782b6b8afd1da042d45c2daeec9c0744f72cc7
commit-date: 2023-07-05
host: aarch64-apple-darwin
libgit2: 1.6.4 (sys:0.17.2 vendored)
libcurl: 7.88.1 (sys:0.4.63+curl-8.1.2 system ssl:(SecureTransport) LibreSSL/3.3.6)
ssl: OpenSSL 1.1.1u  30 May 2023
os: Mac OS 13.4.1 [64-bit]
```
