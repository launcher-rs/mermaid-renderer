# mermaid-renderer

纯 Rust 实现的 [Mermaid](https://mermaid.js.org/) 图表渲染库，直接输出 SVG：不启动浏览器，不需要 Node.js，解析、布局、文本度量与 SVG 生成全部在进程内完成。

本项目由 [`mermaid-rs-renderer`](https://github.com/1jehuang/mermaid-rs-renderer) 0.3.1（MIT）分叉而来，由 [拾墨（Rumark）](https://github.com/launcher-rs/rumark) 项目维护。尚未发布到 crates.io，也没有打稳定标签，请暂用 git 依赖（由 `Cargo.lock` 锁定具体提交）：

```toml
[dependencies]
mermaid-renderer = { git = "https://github.com/launcher-rs/mermaid-renderer", branch = "main" }
```

## 快速开始

```rust
use mermaid_renderer::{render, render_with_options, RenderOptions};

let diagram = r#"
flowchart LR
    A[开始] --> B{判断}
    B -->|是| C[通过]
    B -->|否| D[拒绝]
"#;

// 一行式渲染
let svg = render(diagram).unwrap();

// 指定主题与布局参数
let svg = render_with_options(diagram, RenderOptions::default()).unwrap();
```

需要复用解析结果或布局结果时，可以单独调用各阶段：

```rust
use mermaid_renderer::{compute_layout, parse_mermaid, render_svg, LayoutConfig, Theme};

let parsed = parse_mermaid("flowchart LR; A-->B-->C").unwrap();
let theme = Theme::modern();
let config = LayoutConfig::default();
let layout = compute_layout(&parsed.graph, &theme, &config);
let svg = render_svg(&layout, &theme, &config);
```

## 支持的图表

流程图、时序图、类图、状态图（`stateDiagram-v2`）、ER 图、饼图、XY 图、象限图、甘特图、时间线、旅程图、思维导图、Git 图等，完整列表见 crate 文档。

## 相对上游 0.3.1 的改动

- 裁掉 `cli` / `png` feature 与 `mmdr` CLI（连同 `clap`、`resvg`、`usvg` 依赖），只保留输出 SVG 的库接口。
- 裁掉 criterion 基准，但保留 `benches/fixtures` 与 `docs/comparison_sources`，它们是各测试套件的回归语料。
- 在 0.3.1 基础上修复的布局问题（源码中以 `fork:` 注释标注）：
  - 状态图：边标签漂移、走线 S 形绕行、直角生硬、正交折线缺少圆角；
  - 时序图：`loop`/`alt` 等分组框误吞框外消息、激活条多覆盖一条消息、`box` 分组缺少边框标签、自消息钩形箭头与 `par`/`alt` 分隔线交叠；
  - 流程图：含管道标签的链式边（如 `A -->|否| B[文本] -.-> C`）第二条边目标与节点标签被吞；
  - 类图：严格解析拒绝官方 `<<interface>>` 等刻板注解，`$`/`*` 成员后缀被误当成返回类型拆开；
  - 性能：收紧精确路由阈值后，24~64 条边的流程图首次渲染耗时从约 1.3 秒降到约 0.3 秒。

## 开发与验证

```bash
cargo fmt --check
cargo clippy --all-targets
cargo test
```

测试套件会用固定语料渲染并对布局与 SVG 形状做回归校验，改动布局代码后必须全绿；任何发布标签都要建立在全绿基线上。

## 许可与出处

MIT。上游原始版权声明保留在 [LICENSE](LICENSE) 中；分叉起点的上游变更历史见 [CHANGELOG.md](CHANGELOG.md)。
