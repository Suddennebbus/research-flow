# Changelog

All notable changes to research-flow will be documented in this file.

## [1.0.0] — 2026-07-28

### Added

- 7 步交互式深度调研工作流（采访 → 子任务生成 → 逐卡确认 → 合并选模式 → 执行研究 → 风格重写 → 拆分存储）
- 两轮结构化采访（AskUserQuestion），覆盖研究方向、目标读者、了解程度、特别关注
- 子任务卡片机制：JSON Schema 约束，逐卡确认（执行/删除/修改），修改讨论最多 2 轮
- deep-research 管道编排（Quick / Full 双模式）
- 6 种风格重写（长篇技术博客、知乎专业回答、GitHub 项目推荐、论文专业解读、自定义）
- 强制 mermaid 图表 + 统一配色规范（白底 + 低饱和绿 + 米棕）
- 费曼学习法自检 + 去 AI 味规则
- 小节拆分 + 知识库索引（README.md）
- 只读区域保护（research/human-output/）
- 参考资料标注系统（上标 + 文末列表）
- 依赖检测：deep-research 前置校验
