[中文](CHANGELOG.zh-CN.md) | English

# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- **A minimal web status page** (v1.x batch ED-4): `web/index.html` is a small, self-contained
  page that reads `/v0/status` — the node's version, uptime, connections, SSE subscribers and
  agent id — every five seconds. It takes the bearer token in a field and keeps it in
  `sessionStorage` for the tab; the page's own assets never carry a secret, while the API still
  authenticates every call. The tag-gated `server-bundle` job now copies `web/` into the
  archive, so `riscdom-server --web-root web` serves it with no extra build.

### Changed

- **The README no longer claims the web UI is unshipped** (v1.x batch ED-4): the "not bundled
  yet — it arrives when the split reaches M8-4b" note was stale (the split is closed out), and
  the `--web-root` text described `ui/`, a directory this repository has not had since v1.0
  M8-4b. Both now describe `web/`, and say where the full management UI lives
  (`riscdom-adminapp`).

## [1.0.1] - 2026-10-02

### Fixed

- **The route-table drift guards compile again** (v1.0 batch EC-2): `src/routes.rs` read its two
  documents with `include_str!("../../docs/…")`, which leaves the repository once the crate sits
  at the root — so `clippy --all-targets` and `cargo test` could not compile the test target and
  **this repository's CI was red from M8-4a**. The two documents (`control-plane-api.md`,
  `tool-schema-control-plane.md`, and their `.zh-CN.md` twins) are now snapshotted under `docs/`
  — the kernel keeps the single source of truth — and `routes.rs` reads `../docs/…`.

### Changed

- **A marker tag, not a release**: `v1.0.1` marks the kernel split's close-out (kernel batch ED,
  decision §170). It bumps no version — this repository's `Cargo.toml` still reads `1.0.0` — and
  **no release was published** for it. The tag names the close-out, not a new build.

## [1.0.0] - 2026-10-01

### Added

- **The control plane as its own program** (v1.0 M8-4a): split out of the kernel
  (`breakevery/riscdom`) — Layer 3 over the kernel facade's portable half (`host-core`, a git
  dependency pinned to the kernel's `v1.0.0` tag), the HTTP + SSE surface the API document
  freezes, the bearer token, and the same route table the kernel's tests asserted before the
  split.
