[English](release-process.md) | 中文

# 剪一个发行版

**状态** v1.x（批 ED-5）｜ **受众** 剪 `riscdom-server` 发行版的人。

本仓的 CI **只构建**、**什么都不发布**：没有任何作业会创建 GitHub Release。剪一个 Release 是
独立的、需授权的动作，本文件就是该动作的清单。

## 1. CI 产出什么

`server-bundle` —— [`.github/workflows/ci.yml`](../.github/workflows/ci.yml) —— 在 `v*` tag（或一次
手动派发）上运行，只留**运行制品**，绝不直接成为 Release 资产：

| 制品 | 内含 | runner |
|---|---|---|
| `riscdom-server-Linux` | `riscdom-server-<version>-linux-x86_64.tar.gz` 与 `riscdom-server_<version>_amd64.deb` | ubuntu |
| `riscdom-server-macOS` | `riscdom-server-<version>-macos-arm64.tar.gz` | macOS |

制品保留 **14 天**（`retention-days`），所以趁新鲜把它们搬进 Release —— 或重跑该作业（它是 tag
触发的，从该 tag 手动派发会构建出同样的东西）。

**Windows 不在上表里**，这是有意的 —— 见 §3。

## 2. 每种归档含什么

- **`.tar.gz`** —— `riscdom-server`（Windows 上为 `.exe`）、`README.md`、`settings.example.json`，以及
  `web/`（最小状态页）。用 `riscdom-server --web-root web` 运行它。
- **`.deb`** —— 同样的文件铺在文件系统上：二进制在 `/usr/bin/riscdom-server`，而 `web/` 与
  `settings.example.json` 在 `/usr/share/riscdom-server/` 下。

两者都写在 [`docs/server-distribution.md`](server-distribution.zh-CN.md) 里。

## 3. Windows：人工构建（M7b-4）

**没有任何仓有 Windows runner**，所以 Windows `.zip` 在一台 Windows 机器上人工拼。该机器需要 MSVC
工具链（本项目用 `x86_64-pc-windows-msvc` 构建过），且工作树必须是**该 tag 的提交**：

```powershell
# 1. 二进制
cargo build --release --bin riscdom-server        # -> target\release\riscdom-server.exe

# 2. 目录树，形状与 .tar.gz 完全一致
$v = (Select-String -Path Cargo.toml -Pattern '^version = "([^"]+)"').Matches[0].Groups[1].Value
$name = "riscdom-server-$v-win-x64"
Remove-Item -Recurse -Force "dist/$name" -ErrorAction SilentlyContinue
New-Item -ItemType Directory -Force "dist/$name" | Out-Null
Copy-Item target\release\riscdom-server.exe "dist/$name/"
Copy-Item README.md "dist/$name/README.md"
Copy-Item -Recurse web "dist/$name/web"
Set-Content -Path "dist/$name/settings.example.json" -Value "{`n  `"version`": 2`n}"

# 3. 归档
Compress-Archive -Path "dist/$name/*" -DestinationPath "dist/$name.zip"
```

上传前核对：

```powershell
Add-Type -AssemblyName System.IO.Compression.FileSystem
[System.IO.Compression.ZipFile]::OpenRead((Resolve-Path "dist/$name.zip")).Entries.FullName
```

它必须列出 `…/riscdom-server.exe`、`…/README.md`、`…/settings.example.json` 与
`…/web/index.html`。

> **Windows 资产是 CI 唯一无法重建的那个。** 把它当作脆弱的一步。

## 4. 命名

```
riscdom-server-<version>-<os>-<arch>.tar.gz     # linux-x86_64、macos-arm64
riscdom-server_<version>_amd64.deb              # Debian 包自己的惯例
riscdom-server-<version>-<os>-<arch>.zip        # win-x64
```

`<version>` 是本仓 `Cargo.toml` 的版本，由打包器**读取**而非人工输入 —— 所以归档不可能与里面的
二进制不一致。

## 5. 创建 Release

没有东西替你做这件事。tag 推上去、资产到手之后：

```bash
gh release create v1.0.2 --repo breakevery/riscdom-server --title "riscdom-server v1.0.2" \
  --notes-file RELEASE_NOTES.md \
  riscdom-server-1.0.2-linux-x86_64.tar.gz riscdom-server_1.0.2_amd64.deb \
  riscdom-server-1.0.2-macos-arm64.tar.gz riscdom-server-1.0.2-win-x64.zip
```

—— 或经网页 UI 做同样的事。**两种 Linux 格式**、macOS 归档与人工构建的 Windows `.zip` 都要附上。
**没有任何签名**（`server-distribution.zh-CN.md` §5）；在说明里如实写出来，不要暗示相反的东西。

## 6. CI 不做什么

- 不创建 Release、不发布：`server-bundle` 两条腿只留运行制品，且什么都不签名。
- 不构建 Windows（§3）。
- 不在普通 push 上运行：该作业的 `if` 只接受 `v*` tag 或手动派发。
