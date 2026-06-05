# RustDesk Guide

## Project Layout

### Directory Structure
* `src/` Rust app
* `src/server/` audio / clipboard / input / video / network
* `src/platform/` platform-specific code
* `src/ui/` legacy Sciter UI (deprecated)
* `flutter/` current UI
* `libs/hbb_common/` config / proto / shared utils
* `libs/scrap/` screen capture
* `libs/enigo/` input control
* `libs/clipboard/` clipboard
* `libs/hbb_common/src/config.rs` all options

### Key Components
- **Remote Desktop Protocol**: Custom protocol implemented in `src/rendezvous_mediator.rs` for communicating with rustdesk-server
- **Screen Capture**: Platform-specific screen capture in `libs/scrap/`
- **Input Handling**: Cross-platform input simulation in `libs/enigo/`
- **Audio/Video Services**: Real-time audio/video streaming in `src/server/`
- **File Transfer**: Secure file transfer implementation in `libs/hbb_common/`

### UI Architecture
- **Legacy UI**: Sciter-based (deprecated) - files in `src/ui/`
- **Modern UI**: Flutter-based - files in `flutter/`
  - Desktop: `flutter/lib/desktop/`
  - Mobile: `flutter/lib/mobile/`
  - Shared: `flutter/lib/common/` and `flutter/lib/models/`

## Rust Rules

* Avoid `unwrap()` / `expect()` in production code.
* Exceptions:

  * tests;
  * lock acquisition where failure means poisoning, not normal control flow.
* Otherwise prefer `Result` + `?` or explicit handling.
* Do not ignore errors silently.
* Avoid unnecessary `.clone()`.
* Prefer borrowing when practical.
* Do not add dependencies unless needed.
* Keep code simple and idiomatic.

## Tokio Rules

* Assume a Tokio runtime already exists.
* Never create nested runtimes.
* Never call `Runtime::block_on()` inside Tokio / async code.
* Do not hide runtime creation inside helpers or libraries.
* Do not hold locks across `.await`.
* Prefer `.await`, `tokio::spawn`, channels.
* Use `spawn_blocking` or dedicated threads for blocking work.
* Do not use `std::thread::sleep()` in async code.

## Editing Hygiene

* Change only what is required.
* Prefer the smallest valid diff.
* Do not refactor unrelated code.
* Do not make formatting-only changes.
* Keep naming/style consistent with nearby code.

## Project Delivery Rules

* GitHub Actions is the authoritative build and compilation environment.
* Do not rely on local compilation as proof that a Windows installer works.
* Local checks should be limited to inspection, formatting, metadata, and other
  lightweight validation unless the user explicitly requests a local build.
* The target artifact is a production-ready Windows `setup.exe` for the custom
  InforiaConnect RustDesk client.
* The installed client must retain the configured InforiaConnect relay server,
  ID server, and public key settings.
* Diagnose build and installer failures independently from previous AI notes.
  Treat handoff prompts as context, verify their claims against the repository,
  dependency graph, workflow logs, and generated artifacts.
* Review the complete relevant execution path when diagnosing failures,
  including GitHub Actions, `build.py`, Cargo, Flutter, vcpkg, packaging,
  installer generation, and runtime server configuration.
* Prefer reproducible, pinned dependencies and deterministic CI behavior.

## Revision Logging And Git

* Every repository change must be documented in `logging/revisions.txt`.
* Add the revision entry after the implementation is complete and before any
  commit or push.
* Each entry must include the date, a concise title, changed files, what
  changed, why it changed, and validation performed.
* Keep the log readable, chronological, and suitable for technical auditing.
* Never commit or push implementation changes without the matching revision
  entry.
* Push completed changes to the configured GitHub remote after validation and
  revision logging.
* Do not claim a push succeeded until the remote operation has completed.
* Do not overwrite or rewrite unrelated user changes while preparing a commit.

## GitHub Actions Runs

* Manually triggered workflows must accept a descriptive build name.
* Set the workflow-level `run-name` so the GitHub Actions tab clearly shows
  what change or hypothesis is being compiled.
* Use a concise, specific name such as
  `Fix FFmpeg linkage - Windows portable installer`.
* Record the intended Actions run name in `logging/revisions.txt` when a change
  is expected to trigger a build.
* Inspect the resulting GitHub Actions logs and artifacts before deciding the
  next fix. Do not make speculative chains of unrelated changes.
