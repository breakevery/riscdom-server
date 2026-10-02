[中文](server-distribution.zh-CN.md) | English

# Distributing the control plane

**Status** v1.0 specification (M7b-1; control-plane only since M8-4d) ｜ **Date** 2026-09-29 ｜
**Audience** whoever builds, ships or unpacks a `riscdom-server` package.

> **The relay is not here.** The **connection layer's server** (`riscdom-relay`) stays in the kernel
> repository — its binary is `net`'s — and its package is documented in
> <https://github.com/breakevery/riscdom/blob/v1.0.0/docs/server-distribution.md>. This document
> covers the one program *this* repository ships: the **single-node control plane**.

**What this document is.** This repository ships one program that a **deployer** runs: the
**single-node control plane** (`riscdom-server`). This document says what its **package** holds, how
one is built, and what it does not carry.

## 1. The single-node control plane

`riscdom-server-<version>-<platform>` holds:

| Entry | What it is |
|---|---|
| `riscdom-server` (`.exe` on Windows) | The control plane: HTTP + SSE over the kernel's host facade ([README.md](../README.md)). |
| `web/` | A **minimal status page** (`index.html`), served at `/` by `--web-root web` (unauthenticated; the API is not). The full management UI is [`riscdom-adminapp`](https://github.com/breakevery/riscdom-adminapp)'s build; point `--web-root` at it instead if you want that. |
| `README.md` | [README.md](../README.md) — how to build, run, the endpoints, authentication. |
| `settings.example.json` | A minimal, valid settings document (`{"version": 2}`), for a first run. |

**On Linux the same files also ship as a `.deb`** (`riscdom-server_<version>_amd64.deb`): the
binary at `/usr/bin/riscdom-server`, and `web/` and `settings.example.json` under
`/usr/share/riscdom-server/`.

**Run it** with `riscdom-server --bind <addr> --workspace <dir> --data-dir <dir> --web-root web`. The
full flag set is the program's own `--help` (`--bind`, `--workspace`, `--data-dir`, `--web-root`,
`--heartbeat-ms`, `--auth` / `--no-auth`, `--log-level`; exit codes `0`/`1`/`2`).

## 2. What the package does not carry

- **No credential, ever.** The bearer token is generated into `<data-dir>/token` on the first start;
  a package never carries it, and a packer must never copy a data directory in.
- **No data directory at all** — no `settings.json`, `peers.json`, `sessions.db` or `audit.db`
  travels with the software. `settings.example.json` is an example, not state.
- **No QEMU, and no RISC-V GCC.** The control plane is a plain Rust binary. (It still *needs* QEMU
  and a RISC-V GCC to boot guests — but that is the operator's installed toolchain, the same one the
  desktop app uses, not something a package bundles: see
  [qemu-setup.md](https://github.com/breakevery/riscdom/blob/v1.0.0/docs/qemu-setup.md).)
- **No second copy of the management UI.** `web/` is this repository's own minimal status page
  (hand-written, one self-contained `index.html`, no build step). The full management UI's source
  is **not** here — it is `riscdom-adminapp`'s, and a package does not carry it.

## 3. Building a package

**This repository has no separate packer.** Its tag-gated `server-bundle` job —
[`.github/workflows/ci.yml`](../.github/workflows/ci.yml) — builds the release binary, copies
`README.md` and `web/`, writes a `settings.example.json`, and leaves the `.tar.gz` as a run
artifact. The kernel's packer
(`riscdom/scripts/pack.{sh,ps1}`) builds the **relay**, which is that repository's program.

The **version** comes from this repository's `Cargo.toml` (its own workspace), so a package cannot
disagree with the binary inside it. The **platform** is the runner's (`linux-x86_64`,
`macos-arm64`, …). The artifacts land under `dist/` in the job, which is not committed.

**Nothing is signed**, and nothing is published by this job: it is tag-gated, and it only leaves the
archive as a run artifact.

## 4. Platforms

| Platform | Format | Notes |
|---|---|---|
| Linux | `.tar.gz` | Built by the `server-bundle` job on a `v*` tag. |
| Linux | `.deb` | The same job's ubuntu leg: `riscdom-server_<version>_amd64.deb`. |
| macOS | `.tar.gz` | Same. |
| Windows | `.zip` | **Built by hand** — there is no Windows runner yet (M7b-4). |

## 5. What is not signed

**Nothing here is signed.** There is no Authenticode signature on the Windows binary, no notarization
on macOS and no package signing anywhere — the same state the desktop installers are in.
An operator who needs a signature gets one by building from source and signing it themselves. A
signing story would be its own decision, with its own credentials, and is not assumed here.

## 6. What is not covered

- **The relay's package** (`riscdom-relay-<version>-<platform>`) — it belongs to the kernel
  repository (<https://github.com/breakevery/riscdom>), which still ships `net` and therefore builds
  the relay; its document is that repository's `docs/server-distribution.md` (at its `v1.0.0` tag).
- **Windows CI packaging** — M7b-4: no Windows runner is wired up yet.
- **The release act** — cutting a `v*` tag and attaching archives to a release needs its own
  authorisation, and it is what makes a package public.
- **A `riscdom-backup` package inside this one** — the portability tool has its own spec in the
  kernel repository
  ([backup.md](https://github.com/breakevery/riscdom/blob/v1.0.0/docs/backup.md)); it is not bundled
  with the control plane.
- **The desktop application** — it is distributed as the Tauri bundles
  [`riscdom-adminapp`](https://github.com/breakevery/riscdom-adminapp) produces (`.dmg`, `.deb`,
  `.rpm`, `.AppImage`, and the Windows installers built by hand).
