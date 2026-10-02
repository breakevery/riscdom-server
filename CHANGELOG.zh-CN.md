[English](CHANGELOG.md) | 中文

# 变更日志

本文件记录项目的所有重要变更。

格式基于 [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)，
版本号遵循 [Semantic Versioning](https://semver.org/spec/v2.0.0.html)。

## [未发布]

## [1.0.2] - 2026-10-02

### 新增

- **一个面向 Linux 的 Debian 包**（v1.x 批 ED-5）：`server-bundle` 作业的 ubuntu leg 现在还会用 `dpkg-deb` 构建 `riscdom-server_<version>_amd64.deb` —— 二进制在 `/usr/bin/riscdom-server`，而 `web/` 与 `settings.example.json` 在 `/usr/share/riscdom-server/` 下。macOS leg 未变。
- **把发布流程写下来**（v1.x 批 ED-5）：`docs/release-process.md`（+ zh）—— CI 产出什么、保留多久，每种归档含什么，**人工构建的 Windows `.zip`** 步骤（M7b-4：没有任何仓有 Windows runner），命名规范，以及收尾用的 `gh release create`。
- **一个最小 web 状态页**（v1.x 批 ED-4）：`web/index.html` 是一个小的、自包含页面，每五秒读一次 `/v0/status` —— 节点的版本、运行时长、连接数、SSE 订阅者数与 agent id。它在一个输入框里接收 bearer token，并把它存在该标签页的 `sessionStorage` 里；页面自身的资产从不携带秘密，而 API 仍然对每一次调用鉴权。tag 触发的 `server-bundle` 作业现在会把 `web/` 复制进归档，所以 `riscdom-server --web-root web` 无需额外构建即可提供它。

### 变更

- **README 不再自称 web UI 尚未发布**（v1.x 批 ED-4）：「not bundled yet —— 拆仓到 M8-4b 时它才来」这句已过期（拆仓已收尾），而 `--web-root` 那段描述的是 `ui/` —— 本仓自 v1.0 M8-4b 起就没有这个目录了。两者现在都改讲 `web/`，并说明完整管理 UI 在哪里（`riscdom-adminapp`）。
- **README 里过期的版本示例已修**（v1.x 批 ED-5）：`/v0/health` 的应答曾写成 `"version":"0.8.0"`；它是程序自己的版本（修这一处时是 `1.0.0`）。
- **`docs/server-distribution.md` 覆盖 `.deb`**（v1.x 批 ED-5）：§1 说明其文件落在何处，§4 的平台表多了 Linux `.deb` 一行。

### 移除

- **`.gitignore` 的 `ui/` 残留规则**（v1.x 批 ED-5）：四行（`/ui/node_modules`、`/ui/dist/*`、`!/ui/dist/.gitkeep`、`/ui/src-tauri/gen/schemas`）指向一个本仓自 v1.0 M8-4b 起就没有的目录。

### 修复

- **`src/lib.rs` 的模块注释**（v1.x 批 ED-5）：它写着 crate「sits beside `ui/src-tauri`」；那个程序已在 v1.0 M8-4b 离开、去了 `riscdom-adminapp`。

## [1.0.1] - 2026-10-02

### 修复

- **路由表漂移守卫又能编译了**（v1.0 批 EC-2）：`src/routes.rs` 用 `include_str!("../../docs/…")` 读它那两份文档 —— crate 落在仓根后这条路径就跨出了仓界 —— 于是 `clippy --all-targets` 与 `cargo test` 编不出 test target，**本仓的 CI 自 M8-4a 起就一直红**。那两份文档（`control-plane-api.md`、`tool-schema-control-plane.md`，及其 `.zh-CN.md` 双胞胎）现在快照在 `docs/` 下 —— 内核保留单一真源 —— 而 `routes.rs` 改读 `../docs/…`。

### 变更

- **一个标记 tag，不是一次发布**：`v1.0.1` 标记的是内核拆仓的收尾（内核批 ED，决策 §170）。它不 bump 版本 —— 本仓 `Cargo.toml` 仍写 `1.0.0` —— 也**没有为它发布任何 Release**。这个 tag 命名的是收尾，不是一次新构建。

## [1.0.0] - 2026-10-01

### 新增

- **作为独立程序的控制平面**（v1.0 M8-4a）：从内核（`breakevery/riscdom`）拆出 —— 架在内核门面的可移植半边（`host-core`，以钉在内核 `v1.0.0` tag 上的 git 依赖引入）之上的 Layer 3，API 文档冻结的 HTTP + SSE 表面、bearer token，以及拆分前由内核测试断言过的同一张路由表。
