# 更新日志（CHANGELOG）

本仓自分叉起单独记录版本变化。分叉前的上游历史见
[mermaid-rs-renderer CHANGELOG](https://github.com/1jehuang/mermaid-rs-renderer/blob/main/CHANGELOG.md)。

## 0.4.0 (2026-11-19)

以 `mermaid-renderer` 命名的首个版本，作为 [`mermaid-rs-renderer`](https://github.com/1jehuang/mermaid-rs-renderer) 0.3.1 的分叉起点。相对上游属于破坏性变更：包名与库名均已修改。

### 变更

- 包名 `mermaid-rs-renderer` → `mermaid-renderer`，库名 `mermaid_rs_renderer` → `mermaid_renderer`。
- 移除 `cli` / `png` feature 与 `mmdr` 可执行文件，连同 `clap`、`resvg`、`usvg` 依赖；本 crate 只输出 SVG。
- 移除 criterion 基准，保留 `benches/fixtures` 与 `docs/comparison_sources` 作为测试套件的回归语料。

### 修复

- 状态图：边标签漂移、走线 S 形绕行、直角生硬，正交折线缺少圆角。
- 时序图：`loop`/`alt` 等分组框误吞框外消息、激活条多覆盖一条消息、`box` 分组缺少边框标签、自消息钩形箭头与 `par`/`alt` 分隔线交叠。
- 流程图：含管道标签的链式边（如 `A -->|否| B[文本] -.-> C`）第二条边目标与节点标签被吞。
- 类图：严格解析拒绝官方 `<<interface>>` 等刻板注解；`$`（静态）与 `*`（抽象）成员后缀被误当成返回类型拆开。

### 性能

- 收紧精确路由阈值后，24~64 条边的流程图首次渲染耗时从约 1.3 秒降到约 0.3 秒。
