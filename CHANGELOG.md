# Changelog — `armature-lambda`

All notable changes to this crate will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this crate adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Earlier changes are recorded in the workspace [`CHANGELOG.md`](../CHANGELOG.md).

## [Unreleased]

### Fixed

- **Breaking:** `impl_request_handler!` is renamed `impl_lambda_handler!`, and `RequestHandler`/`async_trait` are exported so its `$crate` paths resolve. The documented integration path did not compile: the macro called a private free function that was never a method on `Application`.
- **Breaking:** request and response headers preserve duplicates, so a handler can emit two `Set-Cookie` lines — previously impossible.

### Changed — `0.2.0` → `0.2.1`

- Migrated onto `armature-core` `0.8`'s `Bytes`-backed request and response types. No behavior change beyond what that migration implies; see [`armature-core/CHANGELOG.md`](../armature-core/CHANGELOG.md).
- The captured method and body are read through the request's new accessors.
- **Breaking:** `impl_request_handler!` is renamed to `impl_lambda_handler!`. The old name expanded to `$crate::runtime::RequestHandler`, a path in a private module that never resolved outside this crate, and to a bare `async_trait::async_trait` that only resolved in crates depending on `async-trait` under that exact name. The macro now names both through `$crate`, and `RequestHandler` and `async_trait` are re-exported at the crate root.
- **Breaking:** `LambdaRequest::headers` and `LambdaResponse::headers` are `Vec<(String, String)>` instead of `HashMap<String, String>`, so repeated field names survive in both directions — most importantly a handler can now emit more than one `Set-Cookie`. `LambdaResponse::header` appends; the new `set_header` replaces. New readers: `header_values` on both types and `header_value` on the response. A request header whose value is not UTF-8 is still dropped, but now logs a warning instead of vanishing silently.

### Documentation

- The crate docs, `RequestHandler`/`LambdaRuntime` rustdoc and the README no longer claim that an Armature `Application` becomes a handler on its own. No `HttpRequest`/`HttpResponse` conversion exists here; `impl_lambda_handler!` targets a user-supplied inherent `handle_request` method, and its required shape is now documented.

## [0.4.0] - 2026-08-05

### Changed

- **Requires `armature-core` 0.9 (breaking).** The requirement moved `0.8` →
  `0.9`. `armature-core 0.9.0` itself moves `armature-h1` across a breaking
  0.x boundary; because `armature-core` types appear in this crate's own
  public API, the requirement change is breaking here too and the minor moves
  with it. Under Cargo's 0.x caret rules the 0.8 and 0.9 types are distinct
  and do not unify, so a consumer holding an `armature-core 0.8` type cannot
  pass it to this crate. Part of the `armature-core 0.9.0` release train; see
  `armature-core`'s CHANGELOG for the publish order.

## [0.3.1] - 2026-08-04

### Fixed

- Requirements on sibling armature crates name a minor instead of `0`. Under
  Cargo's 0.x rules `version = "0"` matches any release ever made, and edition
  2024 selects the MSRV-aware resolver, so a consumer declaring an older
  `rust-version` was handed the oldest version satisfying it — resolving
  `armature-core = "0"` on Rust 1.89 produced `armature-core 0.2.3` while an
  explicit `armature-core = "0.8"` elsewhere in the same graph pulled 0.8.2.
  Two copies of core, and a build failing on symbols the older one lacks. Each
  0.x minor in this family is a breaking change, so the requirement now names
  one. No API change.
