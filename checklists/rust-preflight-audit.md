# Rust Systems Architect — Pre-Flight Audit Checklist

Run this 12-point checklist before marking any Rust implementation or refactoring complete:

## 1. Memory Safety & Ownership
- [ ] **1.1 Zero Borrow-Checker Clones:** No `.clone()` calls added solely to silence borrow checker errors where `&T`, `&str`, `&[T]`, or `Cow<'_, T>` suffices.
- [ ] **1.2 Unsafe Justification:** Zero `unsafe` blocks, OR every `unsafe` block is preceded by an explicit `// SAFETY:` proof comment.
- [ ] **1.3 Slice Parameters:** Public functions accept `&str`, `&[T]`, and `&Path` instead of `&String`, `&Vec<T>`, or `&PathBuf`.

## 2. Error Handling & Resilience
- [ ] **2.1 Zero Production Panics:** Zero `.unwrap()`, `.expect()`, `panic!()`, or `todo!()` calls in non-test code (`#[deny(clippy::unwrap_used)]`).
- [ ] **2.2 Typed Errors:** Libraries define structured `thiserror::Error` enums; binaries wrap I/O with `anyhow::Context`.
- [ ] **2.3 5-State Resilience:** Empty inputs, timeouts, backpressure (`TrySendError::Full`), and graceful shutdown signals are handled explicitly.

## 3. Concurrency & Performance
- [ ] **3.1 Non-Blocking Async:** Zero synchronous blocking I/O or CPU-intensive loops inside `async fn` without `tokio::task::spawn_blocking`.
- [ ] **3.2 Bounded Channels:** All `mpsc` channels and worker pools have explicit capacity limits and `tokio::time::timeout` guards.
- [ ] **3.3 Release Profile:** `Cargo.toml` defines `[profile.release]` with `lto = "thin"`, `codegen-units = 1`, and `strip = true`.

## 4. Governance & Hygiene
- [ ] **4.1 License & Pure-Rust Deps:** All dependencies use permissive licenses (`MIT`/`Apache-2.0`/`BSD`) and prefer pure-Rust (`rustls`) over C-linked bindings (`native-tls`).
- [ ] **4.2 Zero-Warning Verification:** `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` pass with exit code `0`.
- [ ] **4.3 Sole Author Git Commit:** Committed with strictly one author (`git config user.name` / `user.email`) and zero `Co-authored-by:` trailers.
