# Rust Systems & High-Performance Architect

[![GitHub Stars](https://img.shields.io/github/stars/Henry-19/rust-systems-architect?style=for-the-badge&logo=github)](https://github.com/Henry-19/rust-systems-architect)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)
[![skills.sh](https://img.shields.io/badge/skills.sh-Compatible-0a0a0a.svg?style=for-the-badge)](https://skills.sh)

A production-grade AI Agent Skill synthesized via **Meta-Skill Synthesizer** for engineering memory-safe, zero-cost Rust 2024/2021 systems, async Tokio/Axum services, CLI accelerators, WebAssembly modules, and PyO3/napi-rs FFI extensions—plus enforcing Rust-powered tooling (`rg`, `fd`, `uv`, `ruff`, `typst`) across all agent workflows.

## ⚡ Quick Install

```bash
npx skills add Henry-19/rust-systems-architect -g
```

---

## Directory Structure

```text
rust-systems-architect/
├── SKILL.md                                     # Core agent instructions & 4-stage pipeline
├── README.md                                    # Overview, synthesis provenance & usage guide
├── references/
│   └── rust-performance-and-tokio-patterns.md   # Zero-copy, Tokio/Axum, PyO3 & Cargo.toml templates
└── checklists/
    └── rust-preflight-audit.md                  # 12-point pre-ship verification checklist
```

---

## Synthesis Provenance Matrix

| Source Skill / Repository | Strengths Extracted | Limitations Identified | Mitigations Engineered |
| :--- | :--- | :--- | :--- |
| `jeffallan/claude-skills@rust-engineer` (7K+ installs) | Modular reference table, ownership-first workflow, strict clippy/fmt validation | Lacks polyglot FFI (PyO3/napi-rs) and release binary optimization profiles | Added Section 3.4 Polyglot Acceleration (`PyO3`/`napi-rs`/`WASM`) and production `Cargo.toml` LTO profiles |
| `manutej/luxor-claude-marketplace@rust-systems-programming` (491 installs) | Comprehensive coverage of async concurrency, unsafe code invariants, and systems design | Monolithic 1,153-line `SKILL.md` causing massive context token bloat | Compressed core `SKILL.md` to <155 lines; extracted code recipes into `references/rust-performance-and-tokio-patterns.md` |
| `omer-metin/skills-for-antigravity@rust-craftsman` | Contrarian anti-clone data-flow philosophy and sharp-edge diagnosis | Missing concrete async backpressure rules, error handling taxonomy, and tooling integration | Added bounded channel/Semaphore rules, `thiserror`/`anyhow` taxonomy, and Rust-Powered Agent Speed Mandate (`rg`, `fd`, `uv`, `typst`) |

---

## Prerequisites & Toolchain

- **Rust Toolchain:** `rustup` (`rustc`, `cargo`, `clippy`, `rustfmt`)
- **Rust-Powered CLI Suite:** `ripgrep` (`rg`), `fd-find` (`fd`), `uv` (`astral-sh.uv`), `typst` (`Typst.Typst`)

---

## Usage Guide

1. Activate when building high-performance backend services (`Axum`/`Tokio`), native CLI tools (`Clap`), WebAssembly modules, or replacing slow Python/Node hotspots via `PyO3`/`napi-rs`.
2. Follow the 4-stage gate pipeline in `SKILL.md` (`Ownership Topology` $\rightarrow$ `Trait/Error Contract` $\rightarrow$ `Zero-Cost Implementation` $\rightarrow$ `Pedantic Clippy & Test Audit`).
3. Verify against `checklists/rust-preflight-audit.md` before committing.
