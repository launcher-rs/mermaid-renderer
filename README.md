# mermaid-renderer

纯 Rust 实现的 [Mermaid](https://mermaid.js.org/) 图表渲染库，直接输出 SVG：不启动浏览器，不需要 Node.js，解析、布局、文本度量与 SVG 生成全部在进程内完成。

## 安装

暂以 git 依赖引入（具体提交由 `Cargo.lock` 锁定）：

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

## 布局与渲染质量

- 状态图：边标签贴线显示，走线避免 S 形绕行与生硬直角，正交折线带圆角。
- 时序图：分组框只包含框内消息，激活条覆盖范围准确，`box` 分组带边框标签，自消息钩形箭头不与 `par`/`alt` 分隔线交叠。
- 流程图：含管道标签的链式边（如 `A -->|否| B[文本] -.-> C`）能正确归属第二条边的目标与标签。
- 类图：支持 `<<interface>>` 等刻板注解，`$`（静态）与 `*`（抽象）成员后缀正常显示。
- 性能：24~64 条边的流程图首次渲染耗时约 0.3 秒。

## 开发与验证

```bash
cargo fmt --check
cargo clippy --all-targets
cargo test
```

测试套件会用固定语料渲染并对布局与 SVG 形状做回归校验，改动布局代码后必须全绿。

## 许可

MIT。本项目分叉自 [`mermaid-rs-renderer`](https://github.com/1jehuang/mermaid-rs-renderer) 0.3.1（MIT），原始版权声明见 [LICENSE](LICENSE)。
