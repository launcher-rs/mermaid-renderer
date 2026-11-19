# AGENTS.md — mermaid-renderer

纯 Rust 的 Mermaid 图表渲染库（只输出 SVG），解析、布局、文本度量与 SVG 生成全部在进程内完成。

## 构建 / 验证

- 构建：`cargo build`；验证顺序：`cargo fmt --check` → `cargo clippy --all-targets` → `cargo test`
- 打包校验：`cargo package --no-verify`（CI 也会跑，注意别把临时文件带进包体）
- 单测聚焦：`cargo test --test layout_suite <测试名>`；库内单元测试用 `cargo test <模块路径>`
- MSRV 1.87（`edition = "2024"`），CI 用 stable 工具链跑 fmt / clippy / test（Linux + Windows）

## 结构与职责

- `src/parser.rs`：Mermaid 文本 → 语法树；`src/ir.rs`：图表中间表示
- `src/layout/`：各图表类型的布局（`flowchart/` 最重，含路由、标签避让、子图等阶段）
- `src/render.rs`：布局 → SVG；`src/text_metrics.rs`：文本度量（依赖 `fontdb`）
- `src/{config,theme,edge_geometry,unicode_width,validator,layout_dump,error}.rs`：配置、主题、几何、宽度、校验、调试输出
- `tests/`：十个 suite（正确性、确定性、布局不变量、形状、鲁棒性、解析错误等）
- `tests/fixtures/`、`benches/fixtures/`、`docs/comparison_sources/`：回归语料；`tests/*baseline*.json`：硬门槛基线

## 代码渊源（内部参考）

- 早期代码移植自 `mermaid-rs-renderer` 0.3.1（MIT）：因此源码里仍有英文注释，且以 `fork:` 前缀标注后续改动点，便于回溯。
- **新增或修改的注释一律用中文**；`fork:` 注释必须是中文，并写清改动原因。
- 不要无谓重排代码、重命名既有标识符或整体重排格式，避免产生与基线无关的巨型 diff。
- 用户可见文档（`README.md`、`AGENTS.md`、`CHANGELOG.md`）与 `Cargo.toml` 的 `description` 用中文；`keywords`、`categories` 等检索用元数据保持英文。

## 硬约束

- 只输出 SVG：不要恢复 `cli` / `png` feature，不要引入 `resvg` / 浏览器 / Node.js / 子进程依赖。
- 语料与基线是回归防线：改动布局代码后 `cargo test` 必须全绿；golden 文件需要更新时优先用套件自带的 `UPDATE_*` 环境变量重新生成，并在提交信息里说明原因。
- 文本度量依赖本机字体：可移植 golden 一律用 `fast_text_metrics = true`，不要把依赖字体的数值写进 golden。
- 换行统一 LF（`.gitattributes` 已声明）：golden 比对是逐字节的，Windows 上出现 CRLF 会让测试假失败；本仓约定 `core.autocrlf=false`。
- 依赖保持精简，新增依赖前先确认必要性；tree-sitter 之类的可选重型依赖不属于本仓。
- 版本遵循 semver；0.x 期间 minor 可含破坏性变更，但必须在 `CHANGELOG.md` 明确写出。

## 发布

- 已发布 `0.1.0` 到 crates.io（发布命令需显式指定 `--registry crates-io`，本机 cargo 默认走 rsproxy 镜像）。
- 发布流程：改 `Cargo.toml` 版本 → 在 `CHANGELOG.md` 补该版本段与日期 → 测试全绿 → 提交 → 打 `v<版本>` 标签 → 推送 → `cargo publish --registry crates-io`。
- 双远端：GitHub（`launcher-rs/mermaid-renderer`）为主仓，Gitee 为镜像；`main` 与标签两边同步推送。

## 提交

- 提交信息用 `type: 中文描述`（`feat` / `fix` / `perf` / `docs` / `chore` / `test` / `ci`）。
- 一个提交只做一件事；重排代码、改名、修缺陷不要混在同一个提交里。
