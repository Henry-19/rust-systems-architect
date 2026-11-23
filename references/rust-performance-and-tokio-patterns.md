# Rust Performance, Tokio/Axum & PyO3 Reference Patterns

## 1. Production `Cargo.toml` Profile & Lint Hardening

```toml
[package]
name = "app-core"
version = "0.1.0"
edition = "2021"
license = "MIT OR Apache-2.0"

[lints.rust]
unsafe_code = "warn"

[lints.clippy]
all = { level = "deny", priority = -1 }
pedantic = { level = "warn", priority = -1 }
unwrap_used = "deny"
expect_used = "deny"
panic = "deny"
todo = "deny"

[profile.release]
opt-level = 3
lto = "thin"
codegen-units = 1
panic = "abort"
strip = true
```

---

## 2. Structured Domain Errors (`thiserror`) + Boundary Context (`anyhow`)

```rust
use thiserror::Error;

#[derive(Debug, Error)]
pub enum StorageError {
    #[error("record `{id}` not found")]
    NotFound { id: String },

    #[error("serialization failure")]
    Serialization(#[from] serde_json::Error),

    #[error("database I/O failure")]
    Io(#[from] std::io::Error),
}
```

---

## 3. Bounded Async Concurrency & Backpressure (`Tokio`)

```rust
use std::sync::Arc;
use tokio::sync::{mpsc, Semaphore};
use tokio::task::JoinSet;
use tokio::time::{timeout, Duration};

pub async fn process_bounded_batch<T, F, Fut>(
    items: Vec<T>,
    max_concurrency: usize,
    task_timeout: Duration,
    worker: F,
) -> anyhow::Result<()>
where
    T: Send + 'static,
    F: Fn(T) -> Fut + Send + Sync + 'static,
    Fut: std::future::Future<Output = anyhow::Result<()>> + Send + 'static,
{
    let sem = Arc::new(Semaphore::new(max_concurrency));
    let worker = Arc::new(worker);
    let mut set = JoinSet::new();

    for item in items {
        let permit = sem.clone().acquire_owned().await?;
        let w = worker.clone();
        set.spawn(async move {
            let _permit = permit;
            timeout(task_timeout, w(item)).await??;
            Ok::<_, anyhow::Error>(())
        });
    }

    while let Some(res) = set.join_next().await {
        res??;
    }
    Ok(())
}
```

---

## 4. Zero-Copy String / Slice Transformation (`Cow<'a, str>`)

```rust
use std::borrow::Cow;

/// Returns a borrowed slice if no sanitization is needed; allocates only when modified.
pub fn sanitize_token(input: &str) -> Cow<'_, str> {
    if input.bytes().any(|b| b == b'\r' || b == b'\n') {
        Cow::Owned(input.replace(['\r', '\n'], " "))
    } else {
        Cow::Borrowed(input)
    }
}
```

---

## 5. High-Speed Python FFI Bridge (`PyO3` + `Rayon`)

```rust
use pyo3::prelude::*;
use rayon::prelude::*;

#[pyfunction]
fn parallel_token_count(py: Python<'_>, texts: Vec<String>) -> PyResult<Vec<usize>> {
    // Release Python GIL during multi-core Rust execution
    let counts = py.allow_threads(|| {
        texts
            .par_iter()
            .map(|t| t.split_whitespace().count())
            .collect()
    });
    Ok(counts)
}
```
