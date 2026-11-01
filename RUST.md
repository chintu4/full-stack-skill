# 🦀 Golden Rules for Rust Backends

Compiling and passing tests is the easy part. These rules are about surviving real traffic, real
failures and a 3 a.m. page.

Each rule has a **Check** line — something you can grep, run, or look at.

---

## Part 1 — Project & Toolchain

1. **Pin the toolchain.** A `rust-toolchain.toml` means every machine and CI builds the same compiler.
   *Check:* the file exists and CI uses it.

2. **Commit the lockfile.** For a binary, `Cargo.lock` belongs in git or builds aren't reproducible.
   *Check:* `Cargo.lock` is tracked and not in `.gitignore`.

3. **Split library from binary.** Domain logic in a lib crate (testable, no `main`), HTTP wiring in the bin.
   *Check:* the lib crate has no `#[tokio::main]` and its tests don't boot a server.

4. **Audit dependencies in CI.** `cargo audit` for advisories, `cargo deny` for licences, bans and
   duplicate versions.
   *Check:* both run on every PR and fail the build.

5. **Know your MSRV and hold it.** Bumping the minimum Rust version is a breaking change for downstream.
   *Check:* `rust-version` in `Cargo.toml`, tested in CI.

---

## Part 2 — Async Discipline

6. **Never block a runtime worker.** No `std::thread::sleep`, `std::fs`, `reqwest::blocking`,
   `std::sync::mpsc::recv` or long CPU loops inside `async`.
   *Check:* grep for those in files containing `async fn`. `reqwest::blocking` inside a Tokio runtime
   can panic outright, not just stall.

7. **Move blocking work to `spawn_blocking`.** Hashing, image processing, PDF generation, sync file IO.
   *Check:* every heavy call sits behind `spawn_blocking` or a dedicated thread.

8. **`std::sync::Mutex` by default; `tokio::sync::Mutex` only if the guard must cross an `.await`.**
   A std guard alive at an `.await` makes the future `!Send` and holds the lock while suspended.
   *Check:* no `.lock()` whose guard is still in scope at an `await`.

9. **Every task has an owner.** `JoinSet` or `TaskTracker`; never drop a `JoinHandle` on the floor.
   *Check:* no bare `tokio::spawn` whose handle is discarded for work you care about.

10. **Every queue is bounded.** `mpsc::channel(n)`, not `unbounded_channel`, unless the producer is
    provably bounded.
    *Check:* grep `unbounded_channel`.

11. **`select!` branches must be cancel-safe** when the loop repeats. Losing a future mid-write loses data.
    *Check:* no `read_exact` / `write_all` / `Sender::send` directly inside a looped `select!`.

---

## Part 3 — Errors

12. **One typed error per surface.** `thiserror` for libraries and domain errors; `anyhow` at the
    application edge. Both together is normal.
    *Check:* no `Result<T, String>` in public APIs.

13. **`AppError: IntoResponse` is the only place that maps errors to status codes.**
    *Check:* handlers return `Result<T, AppError>`; no handler builds a `500` by hand.

14. **Never leak the cause of a 5xx to the client.** Log the detail; return a generic body. A database
    error string can expose hostnames and topology.
    *Check:* the `IntoResponse` impl branches on `status.is_server_error()`.

15. **No `unwrap()`/`expect()` on fallible I/O in a request path.** Reserve them for startup invariants
    where crashing loudly is correct.
    *Check:* enable `#![warn(clippy::unwrap_used)]` in production crates; clippy won't flag it otherwise.

16. **Don't swallow results.** `let _ = ...` on a `Result` is a silent bug.
    *Check:* grep `let _ =` and justify each.

17. **Add context at `spawn` boundaries.** `handle.await??` loses which task failed; attach context first.
    *Check:* spawned tasks wrap errors with context before crossing the boundary.

---

## Part 4 — Timeouts, Limits, Load

18. **Bound every outbound call.** `reqwest` has no default timeout. An unbounded `await` is a latent outage.
    *Check:* client built with `.timeout()` and `.connect_timeout()`; `tokio::time::timeout` elsewhere.

19. **Bound inbound requests.** `TimeoutLayer` on the router, plus a body size limit.
    *Check:* a slow request returns 408; an oversized body returns 413.

20. **Build clients and pools once.** One `reqwest::Client`, one `PgPool` in `AppState`, cloned freely.
    *Check:* no `Client::new()` inside a handler.

21. **Rate-limit public endpoints.** Per-IP and global caps so one client can't exhaust the box.
    *Check:* a limiter layer exists, or the endpoint is behind auth.

22. **Retries need backoff *and* jitter.** Synchronised retries are a thundering herd.
    *Check:* retry helper has a random component and a max attempt count.

---

## Part 5 — Shutdown & Health

23. **Graceful shutdown: SIGTERM → readiness 503 → stop accepting → drain in-flight → cancel tasks → close pools.**
    *Check:* `axum::serve(...).with_graceful_shutdown(...)`; a deploy causes no 502s.

24. **Separate liveness from readiness.** Liveness means "restart me"; readiness means "route to me".
    Liveness must not depend on the database.
    *Check:* two endpoints; readiness goes 503 when a dependency is down, liveness stays 200.

25. **Health checks must be cheap and side-effect free.**
    *Check:* `/health` does no DB query, no outbound call.

---

## Part 6 — Observability

26. **Structured logging with `tracing`, JSON in production, level from the environment.**
    *Check:* logs parse as JSON in prod; no `println!` in library code.

27. **A correlation/request ID on every request**, propagated to outbound calls.
    *Check:* the same ID appears in every log line for one request.

28. **Measure RED (rate, errors, duration) and USE (utilisation, saturation, errors).**
    *Check:* latency histograms exist per route, not just a global average.

29. **Never log secrets or tokens.** Redact at the subscriber, not at each call site.
    *Check:* grep logs for `token`, `key`, `password`, `authorization`.

---

## Part 7 — Security

30. **Secrets from the environment, never in code or git.** A committed `.env` is in history forever.
    *Check:* no `.env` in the tree; secrets rotated if one ever was.

31. **Parameterised queries only.** Never build SQL with `format!`.
    *Check:* grep `format!("SELECT`, `format!("INSERT` — should be zero.

32. **Validate and type config at startup.** A typed struct, validated once; fail loudly rather than
    defaulting silently mid-request.
    *Check:* the process refuses to start with a missing or malformed variable.

33. **CORS, CSRF and security headers are part of the feature**, not an afterthought.
    *Check:* explicit allow-list origins, not `*`, on anything authenticated.

34. **Container hygiene:** non-root user, minimal/distroless image, read-only filesystem where possible.
    *Check:* `USER` is not root in the Dockerfile; the image has no shell.

35. **`unsafe` needs a written justification and a test.** Every block.
    *Check:* grep `unsafe`; each has a `// SAFETY:` comment.

---

## Part 8 — Build & Release

36. **A deliberate release profile.** `lto`, `codegen-units = 1`, and a conscious choice on `panic`.
    *Check:* `[profile.release]` exists. If `panic = "abort"`, a panic kills the whole process —
    pair it with an orchestrator restart. If you keep unwinding, add `CatchPanicLayer` so one bad
    request returns 500 instead of taking down the worker.

37. **Ship small and stripped.** `strip = "symbols"` for the binary; source maps/artifacts archived separately.
    *Check:* the release binary size is tracked.

---

## Part 9 — Testing

38. **Test the HTTP surface, not just units.** `tower::ServiceExt::oneshot` against the real router.
    *Check:* every route has a test asserting status codes for success, auth failure and bad input.

39. **Test the error mapping.** A 404, a 401 and a 500 each have a test.
    *Check:* `#[tokio::test]` cases cover the `IntoResponse` branches.

40. **No test-only code on the production path.** A hand-written `FakeDb` implementing the whole DB trait
    drifts the moment the trait changes — which is exactly what it did in MMD.
    *Check:* mocks are generated or minimal, and CI compiles tests (`--all-targets`).

---

## Part 10 — Data

41. **Migrations are versioned and applied by tooling**, never by hand.
    *Check:* a migrations directory with ordered files.

42. **Watch for N+1 queries.** A query per row in a loop is the most common performance bug.
    *Check:* no DB call inside a `for` over query results.

43. **Every cache is bounded and has a TTL.** An unbounded map is a memory leak with extra steps.
    *Check:* cache struct has a capacity limit and expiry.

44. **Store timestamps as UTC, with a timezone-aware type.** Naive datetimes cause silent bugs across DST
    and regions.
    *Check:* no `NaiveDateTime` at API boundaries.

45. **Idempotency for anything that mutates money, health records or sends messages.**
    *Check:* a client-supplied idempotency key is honoured on retry.
