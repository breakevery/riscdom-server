[English](server-distribution.md) | 中文

# 分发控制平面

**状态** v1.0 规格（M7b-1；自 M8-4d 起只讲控制平面）｜ **日期** 2026-09-29 ｜ **受众** 构建、发布或
解包 `riscdom-server` 包的人。

> **relay 不在这里。** **连接层的服务器**（`riscdom-relay`）留在内核仓 —— 它的二进制属于 `net` ——
> 其包文档在
> <https://github.com/breakevery/riscdom/blob/v1.0.0/docs/server-distribution.md>。本文件只讲*本仓*仍
> 发布的那个程序：**单节点控制平面**。

**本文件是什么。** 本仓发布一个**部署者**要运行的程序：**单节点控制平面**（`riscdom-server`）。本文件
说明它的**包**包含什么、如何构建、以及它不带什么。

## 1. 单节点控制平面

`riscdom-server-<version>-<platform>` 包含：

| 条目 | 是什么 |
|---|---|
| `riscdom-server`（Windows 上为 `.exe`） | 控制平面：内核 host 门面之上的 HTTP + SSE（[README.md](../README.md)）。 |
| `web/` | 一个**最小状态页**（`index.html`），由 `--web-root web` 挂在 `/`（页面无需鉴权；API 需要）。完整管理 UI 是 [`riscdom-adminapp`](https://github.com/breakevery/riscdom-adminapp) 的构建产物；想要那个就把 `--web-root` 指向它。 |
| `README.md` | [README.md](../README.md) —— 怎么构建、怎么运行、端点、鉴权。 |
| `settings.example.json` | 一份最小、合法的 settings 文档（`{"version": 2}`），供首次运行。 |

**运行它**：`riscdom-server --bind <addr> --workspace <dir> --data-dir <dir> --web-root web`。完整
flag 集以程序自己的 `--help` 为准（`--bind`、`--workspace`、`--data-dir`、`--web-root`、
`--heartbeat-ms`、`--auth` / `--no-auth`、`--log-level`；退出码 `0`/`1`/`2`）。

## 2. 这个包不带什么

- **绝无凭据。** bearer token 在首次启动时生成到 `<data-dir>/token`；包从不携带它，打包器也绝不能把
  data 目录复制进去。
- **完全不带 data 目录** —— 没有 `settings.json`、`peers.json`、`sessions.db` 或 `audit.db` 随软件
  走。`settings.example.json` 是示例，不是状态。
- **不带 QEMU，也不带 RISC-V GCC。** 控制平面是一个普通 Rust 二进制。（它仍**需要** QEMU 与 RISC-V
  GCC 来启动访客 —— 但那是运维自己装好的工具链、与桌面应用同一套，不是包要捆绑的东西：见
  [qemu-setup.md](https://github.com/breakevery/riscdom/blob/v1.0.0/docs/qemu-setup.md)。）
- **不带管理 UI 的第二份拷贝。** `web/` 是本仓自己的最小状态页（手写、一个自包含 `index.html`、无构建步骤）。完整管理 UI 的源码**不**在这里 —— 它是 `riscdom-adminapp` 的，包也不携带它。

## 3. 构建一个包

**本仓没有独立的打包器。** 它的 tag 触发作业 `server-bundle` ——
[`.github/workflows/ci.yml`](../.github/workflows/ci.yml) —— 构建 release 二进制、复制 `README.md`
与 `web/`、写一份 `settings.example.json`，把 `.tar.gz` 作为运行制品
留下。内核的打包器（`riscdom/scripts/pack.{sh,ps1}`）打的是 **relay**，那是那个仓的程序。

**版本**来自本仓 `Cargo.toml`（它自己的 workspace），所以包不可能与里面的二进制不一致。**平台**是
runner 的（`linux-x86_64`、`macos-arm64`、……）。制品落在该作业的 `dist/` 下，不提交。

**没有任何签名**，该作业也不发布任何东西：它是 tag 触发的，只把归档留作运行制品。

## 4. 平台

| 平台 | 格式 | 备注 |
|---|---|---|
| Linux | `.tar.gz` | 由 `server-bundle` 作业在 `v*` tag 时构建。 |
| macOS | `.tar.gz` | 同上。 |
| Windows | `.zip` | **人工构建** —— 还没有 Windows runner（M7b-4）。 |

## 5. 什么没有签名

**这里没有任何签名。** Windows 二进制没有 Authenticode 签名，macOS 没有公证，任何地方都没有包签名
—— 与桌面安装包处于同一状态。需要签名的运维者可以自行从源码构建并签名。签名方案会是它自己的裁决、
带它自己的凭据，这里不做假定。

## 6. 什么不在覆盖范围

- **relay 的包**（`riscdom-relay-<version>-<platform>`）—— 它属于内核仓
  （<https://github.com/breakevery/riscdom>），内核仍发布 `net`、因而构建 relay；其文档是该仓的
  `docs/server-distribution.md`（在其 `v1.0.0` tag 处）。
- **Windows CI 打包** —— M7b-4：还没有接上 Windows runner。
- **发布动作** —— 打 `v*` tag 并把归档挂到 release 上，需要它自己的授权，也正是它让一个包公开。
- **把 `riscdom-backup` 打进这个包** —— 可移植工具在内核仓自有规格
  （[backup.md](https://github.com/breakevery/riscdom/blob/v1.0.0/docs/backup.md)）；不与控制平面一同
  打包。
- **桌面应用** —— 它以 [`riscdom-adminapp`](https://github.com/breakevery/riscdom-adminapp) 产出的
  Tauri 包（`.dmg`、`.deb`、`.rpm`、`.AppImage`，以及人工构建的 Windows 安装包）分发。
