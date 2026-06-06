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
* Work only on tasks explicitly requested by the user and the minimum
  supporting work required to complete and validate those tasks.
* Do not make opportunistic changes, unrelated hardening, cleanup, upgrades,
  or configuration changes.
* Stability is a release requirement. Review every implementation after
  editing and test the affected behavior before considering it complete.
* Use the strongest practical validation available for the affected layer:
  focused tests, formatting/static checks, GitHub Actions, artifact inspection,
  runtime checks, or end-to-end acceptance tests.
* Never claim that a change works without recording what was actually tested
  and any remaining limitation.

## Infrastructure Safety Boundary

* Access to the Linux VM is limited to the user's requested InforiaConnect
  tasks.
* Never change network configuration on the VM, Windows workstation, router,
  firewall, DNS, VPN, virtual switch, port forwarding, interfaces, routes, or
  security groups.
* Never enable, disable, or modify UFW or other firewall rules without a new,
  explicit user instruction approving that exact network change.
* Security reviews of network and firewall state are read-only unless the user
  separately authorizes a specific remediation.
* Do not modify SSH server configuration or authentication policy. The
  dedicated `codex-inforia` key account is only an access mechanism.
* Do not install packages, upgrade the OS, alter unrelated services, or change
  system-wide policy unless directly required by an approved task.
* Before every VM change, identify the exact requested outcome, inspect the
  current state, back up affected persistent data when appropriate, make the
  smallest change, and validate both the intended behavior and rollback path.

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

* Client changes must be documented in `logging/revisions.txt`.
* Server, Linux VM, Docker, firewall, SSH, deployment, and hbbs/hbbr changes
  must be documented in `logging/revisions_server.txt`.
* Record confirmed client build or packaging failures in
  `logging/BuildError_log.txt`.
* Record confirmed server build, image, deployment, or runtime failures in
  `logging/BuildErrorServer_log.txt`.
* Do not duplicate full server entries in the client logs. A short
  cross-reference is sufficient when one coordinated change affects both.
* Keep each build-error entry concise and human-readable. Include only an ISO
  8601 timestamp with timezone, a short error summary, what was attempted to
  fix it, and the outcome.
* Do not copy full GitHub Actions logs or lengthy diagnostic output into the
  build-error log.
* Update an open build-error entry when its GitHub Actions validation finishes.
  If a fix fails, preserve the attempted fix and add the repeated failure
  instead of rewriting history.
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

## Server Documentation

* Keep the Windows client and Linux server technical documentation separate.
* Create `logging/InforiaConnectServer_Tehnicni_pregled.docx` after the custom
  server is deployed and end-to-end acceptance testing succeeds.
* The final server document must be in Slovenian and cover architecture,
  Docker deployment, ports, DNS, keys, ID blocking, security controls, UFW,
  monitoring, backups, rollback, incident response, and verified test results.
* Never include the private server key, passwords, access tokens, or private
  SSH key material in documentation or Git.

## GitHub Actions Runs

* Manually triggered workflows must accept a descriptive build name.
* Set the workflow-level `run-name` so the GitHub Actions tab clearly shows
  what change or hypothesis is being compiled.
* Keep the Windows build split into two profiles:
  * `baseline` is the default push and release-validation path;
  * `accelerated` is manually selected for FFmpeg hardware codecs and VRAM.
* A failing accelerated build must not block baseline installer delivery.
* Use a concise, specific name such as
  `Fix FFmpeg linkage - Windows portable installer`.
* Record the intended Actions run name in `logging/revisions.txt` when a change
  is expected to trigger a build.
* Inspect the resulting GitHub Actions logs and artifacts before deciding the
  next fix. Do not make speculative chains of unrelated changes.
