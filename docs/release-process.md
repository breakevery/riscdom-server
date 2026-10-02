[中文](release-process.zh-CN.md) | English

# Cutting a release

**Status** v1.x (batch ED-5) ｜ **Audience** whoever cuts a `riscdom-server` release.

This repository's CI **builds**, and it **publishes nothing**: no job creates a GitHub Release.
Cutting one is a separate, authorised act, and this document is that act's checklist.

## 1. What CI produces

`server-bundle` — [`.github/workflows/ci.yml`](../.github/workflows/ci.yml) — runs on a `v*` tag
(or a manual dispatch) and leaves **run artifacts**, never Release assets:

| Artifact | Holds | Runner |
|---|---|---|
| `riscdom-server-Linux` | `riscdom-server-<version>-linux-x86_64.tar.gz` and `riscdom-server_<version>_amd64.deb` | ubuntu |
| `riscdom-server-macOS` | `riscdom-server-<version>-macos-arm64.tar.gz` | macOS |

Artifacts are kept for **14 days** (`retention-days`), so move them into the Release while they are
fresh — or re-run the job (it is tag-gated, and a manual dispatch from the tag builds the same
thing).

**Windows is not in that table**, and that is on purpose — see §3.

## 2. What each archive holds

- **`.tar.gz`** — `riscdom-server` (`.exe` on Windows), `README.md`, `settings.example.json`, and
  `web/` (the minimal status page). Run it with `riscdom-server --web-root web`.
- **`.deb`** — the same files laid out on the filesystem: the binary at `/usr/bin/riscdom-server`,
  and `web/` + `settings.example.json` under `/usr/share/riscdom-server/`.

Both are described in [`docs/server-distribution.md`](server-distribution.md).

## 3. Windows: built by hand (M7b-4)

**No repository has a Windows runner**, so the Windows `.zip` is assembled on a Windows machine by
hand. The machine needs the MSVC toolchain (this project has been built with
`x86_64-pc-windows-msvc`), and the working tree must be the **tagged commit**:

```powershell
# 1. the binary
cargo build --release --bin riscdom-server        # -> target\release\riscdom-server.exe

# 2. the tree, exactly the shape the .tar.gz has
$v = (Select-String -Path Cargo.toml -Pattern '^version = "([^"]+)"').Matches[0].Groups[1].Value
$name = "riscdom-server-$v-win-x64"
Remove-Item -Recurse -Force "dist/$name" -ErrorAction SilentlyContinue
New-Item -ItemType Directory -Force "dist/$name" | Out-Null
Copy-Item target\release\riscdom-server.exe "dist/$name/"
Copy-Item README.md "dist/$name/README.md"
Copy-Item -Recurse web "dist/$name/web"
Set-Content -Path "dist/$name/settings.example.json" -Value "{`n  `"version`": 2`n}"

# 3. the archive
Compress-Archive -Path "dist/$name/*" -DestinationPath "dist/$name.zip"
```

Verify it before uploading:

```powershell
Add-Type -AssemblyName System.IO.Compression.FileSystem
[System.IO.Compression.ZipFile]::OpenRead((Resolve-Path "dist/$name.zip")).Entries.FullName
```

That must list `…/riscdom-server.exe`, `…/README.md`, `…/settings.example.json` and
`…/web/index.html`.

> **The Windows asset is the one CI cannot rebuild.** Treat it as the fragile step.

## 4. Naming

```
riscdom-server-<version>-<os>-<arch>.tar.gz     # linux-x86_64, macos-arm64
riscdom-server_<version>_amd64.deb              # the Debian package's own convention
riscdom-server-<version>-<os>-<arch>.zip        # win-x64
```

`<version>` is this repository's `Cargo.toml` version, and the packer reads it rather than being
told it — so an archive cannot disagree with the binary inside it.

## 5. Creating the Release

Nothing does this for you. Once the tag is pushed and the assets are in hand:

```bash
gh release create v1.0.2 --repo breakevery/riscdom-server --title "riscdom-server v1.0.2" \
  --notes-file RELEASE_NOTES.md \
  riscdom-server-1.0.2-linux-x86_64.tar.gz riscdom-server_1.0.2_amd64.deb \
  riscdom-server-1.0.2-macos-arm64.tar.gz riscdom-server-1.0.2-win-x64.zip
```

— or the same thing through the web UI. Attach **both** Linux formats, the macOS archive and the
hand-built Windows `.zip`. **Nothing is signed** (`server-distribution.md` §5); say so in the notes
rather than implying otherwise.

## 6. What the CI does not do

- It does not create a Release, and it does not publish: the two `server-bundle` legs leave run
  artifacts only, and nothing is signed.
- It does not build Windows (§3).
- It does not run on an ordinary push: the job's `if` admits a `v*` tag or a manual dispatch.
