---
name: research-flow
description: 交互式深度调研工作流 — 采访→子任务→确认→调deep-research管道→风格重写→拆分存储
model: opus
---

# /research-flow — 交互式深度调研工作流

7 步编排流程，把 deep-research 管道作为研究引擎，Claude 负责采访用户、管理子任务、风格化重写和知识库输出。

## 全局状态

执行过程中维护以下状态变量（不存文件，在当前对话中追踪）：

- `topic`: 研究主题
- `topicSlug`: kebab-case slug，用于文件命名
- `outputStyle`: 用户选择的输出风格（Step 6 确定）
- `confirmedSubtasks`: 确认执行的子任务列表
- `mergedPlanJson`: 合并后的完整研究计划 JSON

## 文件输出约定

| 内容 | 路径 |
|------|------|
| 研究计划 JSON | `research/plan/<topicSlug>.json` |
| 管道原始报告 | `research/material/<topicSlug>-report.md` |
| 风格化重写报告 | `research/material/<topicSlug>-rewrite.md` |
| 小节拆分文档 | `research/material/<topicSlug>-<section>.md` |

### 只读目录

`research/human-output/` 及其子目录为只读区域。这些文件由用户手动创建和维护，AI 只能读取用作参考，**禁止写入、修改或删除**。这一约束同样适用于非 research-flow 流程中的日常交互。

## Step 1: 采访环节

目的：将用户的模糊想法转化为结构化的研究方向。

使用 AskUserQuestion 进行两轮采访。

### 第一轮：核心信息收集

```
questions: [
  {
    question: "你的研究方向或核心问题是什么？",
    header: "研究方向",
    options: [
      { label: "我直接描述", description: "在「其他」中详细描述你的研究主题" }
    ],
    multiSelect: false
  }
]
```

等待用户在「其他」中填写详细描述。如果用户选择了"我直接描述"之外还想选别的，同样以文字补充为准。

### 第二轮：补充信息

基于第一轮的回答，用 AskUserQuestion 收集：

```
questions: [
  {
    question: "你的目标读者是谁？这篇文章的深度和风格应该面向谁？",
    header: "目标读者",
    options: [
      { label: "普通开发者", description: "有编程基础但不一定懂该领域，需要从概念讲起" },
      { label: "领域入门者", description: "对该领域有兴趣但知识不系统，需要结构化的入门引导" },
      { label: "有经验的从业者", description: "已在该领域工作，想看深入分析和不同视角" },
      { label: "完全外行", description: "无技术背景，需要大量类比和解释，目标是科普" }
    ],
    multiSelect: false
  },
  {
    question: "你目前对这个主题的了解程度？",
    header: "了解程度",
    options: [
      { label: "仅知道概念", description: "听说过但没深入了解" },
      { label: "读过一些文章", description: "有碎片化的了解，缺乏系统性" },
      { label: "有一定实践经验", description: "动手做过，想深入理解原理和生态" },
      { label: "比较熟悉", description: "想系统梳理、查漏补缺或验证已有认知" }
    ],
    multiSelect: false
  },
  {
    question: "你有什么特别想了解的角度或疑问？（可多选）",
    header: "特别关注",
    options: [
      { label: "历史与演变", description: "这件事的来龙去脉、关键节点、为什么是现在这样" },
      { label: "技术原理", description: "底层机制、协议/算法细节、架构设计" },
      { label: "生态与工具", description: "周边工具链、社区生态、不同实现的对比" },
      { label: "未来趋势", description: "发展方向、可能的突破、值得关注的项目/论文" }
    ],
    multiSelect: true
  },
  {
    question: "有其他补充信息吗？比如已知的相关项目、文章、或具体想避开的方向。",
    header: "补充信息",
    options: [
      { label: "没有补充", description: "以上信息已经足够" }
    ],
    multiSelect: false
  }
]
```

综合两轮采访结果，提炼出：
- 核心研究问题（一句话）
- 主题摘要（50-100 字）
- topicSlug

向用户展示提炼结果，确认后进入 Step 2。

## Step 2: 子任务生成

基于采访结果，生成 3~6 个子任务。每个子任务是一个独立可研究的方向。

### 子任务 JSON Schema

```json
{
  "id": "t1",
  "title": "子任务标题（一句话，简洁有信息量）",
  "objective": "研究目标（2-3句话，说明要搞清楚什么）",
  "searchQueries": ["英文搜索语句1", "英文搜索语句2", "英文搜索语句3", "英文搜索语句4"],
  "expectedSources": ["official_docs", "community", "tech_blog", "academic", "general"],
  "status": { "kind": "pending" }
}
```

expectedSources 可选值：official_docs, community, tech_blog, academic, general

### 展示格式

仅展示卡片摘要（不写文件）：

```
### 子任务 t1: <标题>
- **目标**: <objective>
- **关键词**: <从searchQueries提取的3-5个核心关键词>
- **搜索源**: <expectedSources映射为中文：官方文档/社区/技术博客/学术/通用>
```

让用户先审阅所有卡片，询问是否需要整体调整（拆分、合并、增删），确认后进入 Step 3。此时不写 JSON。

## Step 3: 逐卡确认

对每个子任务，用 AskUserQuestion 逐卡确认：

```
question: "子任务 t<N>: <title>\n目标: <objective>\n\n下一步操作？"
header: "子任务 t<N>"
options: [
  { label: "执行", description: "确认该子任务，进入执行队列" },
  { label: "删除", description: "移除该子任务" },
  { label: "修改讨论", description: "对目标、搜索方向或范围进行调整" }
]
```

### "修改讨论"分支

1. 用 AskUserQuestion 收集修改意见（选择题 + 「其他」文字补充）
2. 基于反馈生成修改后的子任务卡片
3. 再次让用户确认 [执行] [删除] [继续修改]
4. 最多 2 轮修改

### 全部确认后

1. 汇总所有「执行」的子任务
2. 生成完整的研究计划 JSON（参考下方格式）
3. 写入 `research/plan/<topicSlug>.json`

```json
{
  "id": "plan-<timestamp>-<topicSlug>",
  "version": 1,
  "topic": "<研究主题>",
  "summary": "<主题摘要>",
  "subtasks": [ /* 所有确认执行的子任务，id 重新编号为 t1, t2, ... */ ],
  "createdAt": <unix_timestamp_ms>
}
```

## Step 4: 合并主题 + 选模式

基于 `research/plan/<topicSlug>.json`，将子任务合并为一个完整的研究主题描述。

用 AskUserQuestion 让用户选管道模式：

```
question: "选择 deep-research 管道的执行模式"
header: "管道模式"
options: [
  { label: "Quick 模式", description: "研究简报，500-1500 字，适合快速概览。30 分钟左右。" },
  { label: "Full 模式", description: "完整研究报告，3000-8000 字，涵盖方法、文献、分析、讨论。更全面但更耗时。" }
]
```

向用户展示合并后的研究主题描述，确认后进入 Step 5。

## Step 5: 执行研究

调用 deep-research 管道。

### 调用方式

使用 Skill 工具：
```
Skill("deep-research")
```

### Prompt 构造

传递给 deep-research 的 prompt 必须包含：

1. **研究主题**：合并后的研究主题和子任务作为研究子问题
2. **搜索策略**：所有子任务的 searchQueries 作为 bibliography_agent 的搜索输入
3. **输出约束**：明确告知管道"本报告作为中间研究素材，不需要 APA 7.0 学术格式，输出为结构化的技术研究报告即可，语言简洁直接"
4. **语言**：中文输出（学术术语保留英文）

### 知识库复用

搜索前，先检查 `research/material/README.md` 索引，判断是否有与当前研究主题相关的已有调研文档。如有相关文档，读取并作为补充信息源融入研究。这些文档是内部知识库，**不列入参考资料**。

### 管道输出处理

1. 管道运行完成后，将其完整输出保存到 `research/material/<topicSlug>-report.md`
2. 在文件头部添加元数据：
```markdown
<!--
research-plan: research/plan/<topicSlug>.json
pipeline-mode: quick|full
generated-at: <ISO timestamp>
-->
```

## Step 6: 风格化重写

目的：将管道产出的学术风格报告重写为用户指定的风格。

### 选择风格

用 AskUserQuestion：

```
question: "选择重写风格"
header: "输出风格"
options: [
  { label: "长篇技术博客", description: "深度技术文章风格，有故事线、有观点、可读性强。适合个人博客、技术媒体。" },
  { label: "论文专业解读", description: "保留学术严谨性但语言更通俗，逐一解读关键论文/研究的贡献和局限。" },
  { label: "其他（自定义）", description: "在「其他」中描述你想要的风格" }
]
multiSelect: false
```

### 重写要求

以下要求必须严格执行：

1. **费曼学习法检验**：写完后自检——一个外行看完能理解核心概念和逻辑吗？不能就再简化
2. **去学术化**：不要摘要/引言/方法论/讨论/结论这种学术分段。用自然的文章结构（标题 + 小标题 + 段落）
3. **保留严谨**：事实和数据不能丢失、不能变形。所有来自管道的论据、引用、对比都要保留
4. **mermaid 图表（强制）**：所有能结构化表达的内容，必须用 mermaid 图呈现，不得仅用文字描述。**根据内容语义选择最适合的图表类型**：

   | 内容特征 | 推荐图表类型 | 示例场景 |
   |---------|-------------|---------|
   | 步骤间有明确的先后/交互关系 | `sequenceDiagram` 时序图 | 多人协作流程、API 调用链、用户与 AI 的交互回合 |
   | 跨角色/跨模块的分工流程 | `graph TD` 带 subgraph 的泳道式布局 | 分工流程、管线 Pipeline、多系统交互 |
   | 层级/包含/组织关系 | `graph TD` 架构图 | 技术栈分层、模块依赖、概念层级 |
   | 决策分支/条件路径 | `graph TD/LR` 流程图 | 二选一逻辑、if-else 路径、用户选择分叉 |
   | 时间线/阶段演进 | `timeline` 或 `graph LR` | 产品迭代史、技术演进阶段、项目里程碑 |
   | 多维对比/矩阵关系 | `graph` + subgraph 对比布局 | 竞品差异、模式对比、正反案例对照 |
   | 数量/占比/分布 | `pie` 饼图 | 数据分布、来源占比 |
   | 进度/完成度/状态 | 表格（markdown table） | 状态矩阵、checklist |

   **选择原则**：先想"读者看完这张图应该理解什么关系"，再选能最精准表达那个关系的图表类型。不要所有内容都用 flowchart

   - 此要求同样适用于 Step 5 的管道报告
   - **统一风格**：白底 + 低饱和绿色 `#74aa9c`/`#2d6a4d` + 米棕色 `#d4a574`/`#8b6914`，深色背景用 `#1a1a2e`/`#16213e`，警告色 `#c0392b`/`#7b241c`（仅用于负面/风险标注）。所有历史 mermaid 图均以此配色方案为准，新图必须与历史图配色一致。图内所有节点必须显式声明 style，不得遗漏中间节点
5. **避免 AI 味**：不要用 "In conclusion"、"Furthermore"、"值得注意的是"、 "在当今时代" 这类套话。用自然的、有观点的、有人味的中文
6. **中文排版**：少用破折号（——），用逗号和句号完成语义连接。句号仅在一个段落完全结束时使用，段落中间用逗号、分号或自然换行衔接，避免段落内出现句号
7. **长度**：最终重写文档控制在 3000-8000 字（根据子任务数量弹性调整）
8. **引用标注**：正文中引用数据、观点、事实时，必须以 `[^1]`、`[^2]` 上标形式在句末标注，与文末参考资料编号一一对应。文末添加 `## 参考资料` 小节，格式为：
   ```
   ## 参考资料
   [^1]: 标题: URL
   [^2]: 标题: URL
   ...
   ```
   参考资料仅列来源，不额外说明。此小节不参与 Step 7 拆分。正文引用示例："> Anthropic 内部数据显示，产品团队 65% 的代码由 Claude Tag 生成[^1]。"

### 输出

重写完成后保存到 `research/material/<topicSlug>-rewrite.md`，头部加元数据：

```markdown
<!--
research-plan: research/plan/<topicSlug>.json
pipeline-report: research/material/<topicSlug>-report.md
output-style: <选择的风格>
rewritten-at: <ISO timestamp>
-->
```

## Step 7: 拆分存储

将重写后的文档（rewrite.md）按小节标题拆分为独立 markdown 文件。

### 拆分规则

1. 按 `##` 标题作为小节边界
2. `#` 一级标题作为主文档标题，不单独拆
3. 语义上紧密相关的 `###` 归入父 `##` 小节，不单独拆
4. **`## 参考资料` 小节不拆分**，仅保留在 rewrite.md 完整文档中
5. 每个拆分文件是可独立阅读的完整内容（不只是片段）

### 文件命名

`research/material/<topicSlug>-<section-slug>.md`

其中 section-slug 是 `##` 标题的 kebab-case 转换。

### 拆分文件模板

```markdown
<!--
source: research/material/<topicSlug>-rewrite.md
research-plan: research/plan/<topicSlug>.json
section: <section-title>
keywords: <相关关键词，逗号分隔>
generated-at: <ISO timestamp>
-->

# <section-title>

<独立可读的完整小节内容>
```

### 整理索引

在 `research/material/README.md` 中维护一个索引文件（不存在则创建，存在则追加本次的条目）：

```markdown
## <研究主题> (<生成日期>)

- 完整报告：[管道报告](<topicSlug>-report.md) | [风格重写](<topicSlug>-rewrite.md)
- 小节拆分：
  - [<section-title>](<topicSlug>-<section-slug>.md)
  - ...
```

---

## 完成后

流程走完后，向用户汇报产出文件清单和简要统计（子任务数、总字数、图表数等）。
