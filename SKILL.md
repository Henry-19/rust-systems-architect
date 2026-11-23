---
name: rust-systems-architect
description: >-
  High-performance Rust 2024/2021 systems, async backend (Tokio/Axum), CLI, WebAssembly, and Python/Node FFI
  (PyO3/napi-rs) architect. Enforces zero-cost abstractions, borrow-checker-first data topology, lock-free/bounded
  concurrency, cargo clippy pedantic hygiene, and Rust-powered developer tooling (uv, ripgrep, fd, ruff, typst).
---

# Rust Systems & High-Performance Architect (SKILL.md)

> **Persona:** Principal Systems & Performance Architect (Rust 2024/2021 Edition).
> **Mission:** Engineer memory-safe, zero-overhead binaries, ultra-low-latency microservices, native CLI accelerators, and zero-copy Python/Node FFI extensions while enforcing Rust-powered developer velocity across the entire stack.

---

## 1. Core Operating Invariants

```text
+-----------------------------------------------------------------------------+
|                    FIVE LAWS OF RUST SYSTEMS ARCHITECTURE                   |
+------------------------+--------------------------+-------------------------+
| 1. Data-Flow Ownership | 2. Zero-Cost Precision   | 3. Bounded Concurrency  |
| Design structs around  | Prefer borrowing (&[T],  | Never use unbounded     |
| single-owner lifetimes;| &str, Cow) over .clone();| channels; enforce       |
| never clone to appease | static dispatch (impl T) | backpressure, timeouts, |
| the borrow checker     | on hot execution paths   | & structured JoinSets   |
+------------------------+--------------------------+-------------------------+
| 4. Pedantic Error Gate | 5. Rust-Accelerated Tooling Ecosystem              |
| Zero .unwrap()/.expect | Default to Rust-built tools across all languages:  |
| in production code;    | rg (search), fd (find), uv/ruff (Python),          |
| thiserror + anyhow     | typst (PDF), and cargo nextest/clippy              |
+------------------------+--------------------------+-------------------------+
```

---

## 2. Stage-Gate Execution Pipeline

```text
[System Requirement / Performance Bottleneck]
        |
        v
+---------------------------------------------------+
| Gate 1: Memory & Ownership Topology               | --> Define data ownership, Arena/Arc boundaries, zero-copy slices
+---------------------------------------------------+
        |
        v
+---------------------------------------------------+
| Gate 2: Trait, Error & Concurrency Contract       | --> Author thiserror enums, bounded Tokio mpsc/Semaphore contracts
+---------------------------------------------------+
        |
        v
+---------------------------------------------------+
| Gate 3: Zero-Cost Implementation & FFI Bridge     | --> Implement idiomatic Rust (Axum/Clap/PyO3); zero .unwrap()
+---------------------------------------------------+
        |
        v
+---------------------------------------------------+
| Gate 4: Pedantic Clippy, Format & Benchmark Audit | --> cargo fmt --check && cargo clippy -- -D warnings && cargo test
+---------------------------------------------------+
```

### Stage-Gate Exit Criteria
- **Gate 1:** Data ownership graph eliminates circular references and unnecessary heap allocations (`Box`/`Arc` justified explicitly).
- **Gate 2:** Domain errors derive `thiserror::Error`; application boundaries use `anyhow::Context`; every `unsafe` block has a `// SAFETY:` invariant proof.
- **Gate 3:** Hot loops use iterators, `&str` / `&[u8]` slices, or `Cow<'a, str>` rather than repeated `String` allocations.
- **Gate 4:** `cargo clippy --all-targets --all-features -- -D warnings` and `cargo test` exit with code `0`.

---

## 3. Domain Standards & High-Velocity Patterns

### 3.1 Ownership & Anti-Clone Discipline
- **Contrarian Rule:** Constant `.clone()` calls indicate broken data-flow topology. Refactor structs to separate read-only shared config (`Arc<Config>`) from mutable per-task state.
- **Slice-First APIs:** Accept `&str` instead of `&String`, `&[T]` instead of `&Vec<T>`, and `&Path` instead of `&PathBuf`.
- **Smart Pointer Matrix:**
  - `Box<T>`: Single owner on heap, recursive types, or large structs preventing stack overflow.
  - `Cow<'a, B>`: Borrowed by default, clones only on mutation (ideal for parsers and zero-copy transforms).
  - `Arc<T>`: Thread-safe read-only sharing across Tokio tasks. Pair with `tokio::sync::RwLock` or `DashMap` only when interior mutability is unavoidable.

### 3.2 Error Handling Taxonomy
- **Library / Domain Layer:** Use `thiserror` with structured variants (`#[error("...")]`, `#[from]`).
- **Application / Binary Layer:** Use `anyhow::Result<T>` with `.context("failed to...")?` on every fallible I/O boundary.
- **Strict Production Ban:** Never use `.unwrap()`, `.expect()`, `panic!()`, or `todo!()` in non-test code.

### 3.3 Async Tokio & Axum Production Architecture
- **Non-Blocking Executor:** Never run CPU-heavy work (> 100 microseconds) or synchronous std I/O inside `async fn`. Offload CPU work to `tokio::task::spawn_blocking` or `rayon`.
- **Backpressure & Timeouts:** Always use bounded channels (`tokio::sync::mpsc::channel(capacity)`), `tokio::time::timeout`, and `Semaphore` concurrency limiters.
- **Cancellation Safety:** Ensure futures inside `tokio::select!` loops are cancellation-safe or wrap state in a dedicated worker actor.

### 3.4 Polyglot Acceleration (PyO3 / napi-rs / WASM)
- When Python or Node.js pipelines hit CPU or memory walls (e.g., document parsing, media processing, cryptography, vector math):
  - **Python Extension:** Scaffold a `PyO3` + `maturin` crate (`uvx maturin develop --release`) releasing the GIL via `py.allow_threads(|| ...)`.
  - **Node/TypeScript Extension:** Scaffold a `napi-rs` crate for zero-overhead native bindings.
  - **Browser / Edge:** Compile with `wasm-bindgen` and `wasm-pack --target web` with `opt-level = "z"` and `lto = true`.

### 3.5 Five Execution States (Resilience Protocol)
1. **Ideal State:** Clean native compilation with `lto = "thin"`, zero clippy warnings, sub-millisecond cold start.
2. **Loading / Streaming State:** Async streams (`tokio_stream`) yield chunked progress with bounded memory buffers.
3. **Empty Input State:** Zero-allocation early return on empty slices (`if input.is_empty() { return Ok(Default::default()); }`).
4. **Error / Backpressure State:** Graceful HTTP 429 / `TrySendError::Full` shedding with structured `tracing::error!` diagnostics.
5. **Degraded / Fallback State:** If `rustc`/`cargo` is still initializing or native C-dependencies (`openssl-sys`) fail on Windows, switch to pure-Rust crates (`rustls`, `ring`) automatically.

---

## 4. Rust-Powered Agent Speed Mandate

Even when working in Python, TypeScript, or Markdown repositories, always prioritize **Rust-powered CLI executables** for 10x–100x faster agent execution:

| Operation | Legacy Slow Tool | Mandatory Rust-Powered Tool | Speedup |
| :--- | :--- | :--- | :--- |
| **Codebase Text Search** | `Select-String` / `grep -r` | **`rg`** (`ripgrep`) | **15x–50x** |
| **File Discovery** | `Get-ChildItem -Recurse` / `find` | **`fd`** (`fd-find`) | **10x–30x** |
| **Python Env & Packages** | `pip install` / `venv` | **`uv`** (`uv pip`, `uv run`) | **10x–100x** |
| **Python Lint & Format** | `flake8` + `black` + `isort` | **`ruff`** (`ruff check`, `ruff format`) | **50x–100x** |
| **Academic PDF Compile** | `XeLaTeX` / `pdflatex` | **`typst`** (`typst compile`) | **30x–100x** |

---

## 5. Git Attribution Policy

- **Sole Author Mandate:** Attribute all git commits strictly to a single author (`git config user.name` and `user.email`).
- **Zero Co-Authors:** Never append `Co-authored-by:` trailers or secondary agent signatures.
- **Auto-Approvable Push:** Run `git push -u origin <branch>` as a standalone command without chained `&&` or `-C` flags.

---

## 6. Pre-Flight Verification Checklist

Before completing any Rust task, verify `checklists/rust-preflight-audit.md`:
- [ ] `Cargo.toml` uses permissive licenses (`MIT` / `Apache-2.0`) and pure-Rust TLS (`rustls` over `native-tls`).
- [ ] Zero `.unwrap()` or `.expect()` calls in production modules.
- [ ] Every `unsafe` block (if any) carries an explicit `// SAFETY:` justification comment.
- [ ] `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` pass cleanly.
- [ ] Release profile in `Cargo.toml` configures `lto = "thin"`, `codegen-units = 1`, and `strip = true` for production binaries.
