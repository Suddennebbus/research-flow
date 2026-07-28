# Mermaid 图表配色规范

research-flow 的 Step 6（风格化重写）强制要求所有结构化内容使用 mermaid 图表呈现。本文件定义统一配色方案，确保所有产出图表风格一致。

## 配色方案

白底 + 低饱和鼠尾草绿 + 米棕 + 深灰文字 + 圆角细边框。

## 色板

| 色值 | 角色 | 用途 |
|------|------|------|
| `#A4BCA4` | 主色填充 | 核心节点、源节点、终点节点 |
| `#8CA88C` | 主色描边 | 核心节点边框、信号线 |
| `#BFCFBF` | 次要绿填充 | 活跃节点、角色背景 |
| `#E8EDE4` | 极浅绿灰填充 | 链接节点、中间节点背景 |
| `#C4D4C0` | 浅绿描边 | 链接节点边框 |
| `#D8CFC4` | 米棕填充 | 项目级节点、辅助节点 |
| `#C0B4A0` | 米棕描边 | 项目节点边框 |
| `#E0D8CC` | 浅米棕填充 | 归档节点、次要区域 |
| `#C4B8A8` | 浅米棕描边 | 归档节点边框 |
| `#F0EBE0` | 暖白米填充 | 时序图 Note 背景 |
| `#4A4A4A` | 深灰文字 | 所有浅色背景上的文字 |
| `#fff` | 白色文字 | 主色节点上的文字 |

## 风格要求

- 白底为基础
- 以低饱和绿色为主色调，米棕色为辅助色调
- 不使用高饱和色（红、橙、亮绿、亮蓝等）
- 所有节点圆角、细边框
- 文字统一用 `#4A4A4A`（深灰），主色填充节点用 `#fff`

## 时序图 (sequenceDiagram)

通过 `%%{init}%%` 注入 themeVariables：

```
%%{init: {'theme': 'base', 'themeVariables': {
  'primaryColor': '#BFCFBF',
  'primaryTextColor': '#4A4A4A',
  'primaryBorderColor': '#A4BCA4',
  'lineColor': '#A4BCA4',
  'secondaryColor': '#E8EDE4',
  'tertiaryColor': '#E0D8CC',
  'noteBkgColor': '#F0EBE0',
  'noteTextColor': '#4A4A4A',
  'actorBkg': '#BFCFBF',
  'actorBorder': '#A4BCA4',
  'actorTextColor': '#4A4A4A',
  'actorLineColor': '#A4BCA4',
  'signalColor': '#8CA88C',
  'signalTextColor': '#4A4A4A'
}}}%%
```

## 图表类型选择

根据内容语义选择最适合的图表类型：

| 内容特征 | 图表类型 | 示例场景 |
|---------|---------|---------|
| 步骤间有明确的先后/交互关系 | `sequenceDiagram` | 多人协作流程、API 调用链、用户与 AI 的交互回合 |
| 跨角色/跨模块的分工流程 | `graph TD` 带 subgraph | 分工流程、管线 Pipeline、多系统交互 |
| 层级/包含/组织关系 | `graph TD` 架构图 | 技术栈分层、模块依赖、概念层级 |
| 决策分支/条件路径 | `graph TD/LR` 流程图 | 二选一逻辑、if-else 路径 |
| 时间线/阶段演进 | `timeline` 或 `graph LR` | 产品迭代史、技术演进阶段 |
| 多维对比/矩阵关系 | `graph` + subgraph | 竞品差异、模式对比、正反案例对照 |
| 数量/占比/分布 | `pie` | 数据分布、来源占比 |

**选择原则**：先想"读者看完这张图应该理解什么关系"，再选最精准表达那个关系的图表类型。
