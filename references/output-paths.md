# 文件输出路径约定

research-flow 在项目根目录下的 `research/` 目录中组织所有产物。

## 目录结构

```
research/
├── plan/            ← 研究计划 JSON
├── material/        ← 管道报告、风格重写、小节拆分
└── human-output/    ← 只读区域（用户手动维护）
```

## 文件命名规则

| 内容 | 路径 | 格式 |
|------|------|------|
| 研究计划 | `research/plan/<topicSlug>.json` | JSON |
| 管道原始报告 | `research/material/<topicSlug>-report.md` | Markdown + HTML 元数据注释 |
| 风格化重写 | `research/material/<topicSlug>-rewrite.md` | Markdown + HTML 元数据注释 |
| 小节拆分 | `research/material/<topicSlug>-<section-slug>.md` | Markdown + HTML 元数据注释 |
| 知识库索引 | `research/material/README.md` | Markdown |

`topicSlug` 为研究主题的 kebab-case 转换，例如 `attention-mechanism-survey`。
`sectionSlug` 为 `##` 小节标题的 kebab-case 转换。

## 文件元数据格式

每个产出文件头部包含 HTML 注释元数据：

### 管道报告（-report.md）

```markdown
<!--
research-plan: research/plan/<topicSlug>.json
pipeline-mode: quick|full
generated-at: <ISO timestamp>
-->
```

### 风格重写（-rewrite.md）

```markdown
<!--
research-plan: research/plan/<topicSlug>.json
pipeline-report: research/material/<topicSlug>-report.md
output-style: <选择的风格>
rewritten-at: <ISO timestamp>
-->
```

### 小节拆分（-<section>.md）

```markdown
<!--
source: research/material/<topicSlug>-report.md
research-plan: research/plan/<topicSlug>.json
section: <section-title>
keywords: <相关关键词，逗号分隔>
generated-at: <ISO timestamp>
-->
```

## 知识库索引格式

`research/material/README.md` 中按研究主题维护索引：

```markdown
## <研究主题> (<生成日期>)

- 完整报告：[管道报告](<topicSlug>-report.md) | [风格重写](<topicSlug>-rewrite.md)
- 小节拆分：
  - [<section-title>](<topicSlug>-<section-slug>.md)
  - ...
```

## 只读区域

`research/human-output/` 及其子目录为只读区域。这些文件由用户手动创建和维护，Skill 只读取用作参考，**禁止写入、修改或删除**。这一约束同样适用于非 research-flow 流程中的日常交互。
