---
name: ruisearch
description: |
  市场调研、技术调研、竞品分析、行业分析等需要搜索验证并整理成报告的任务。
  文档格式处理：飞书格式输出、标准Markdown排版、Word友好格式、报告整理。
  当用户要求调研某类产品/技术/市场、验证信息、搜集数据、对比分析、整理报告时，必须调用此skill。
version: "2.1.0"
user-invocable: true
argument-hint: "[可选：功能名称，如 research/document]"
---
# Ruisearch
多功能通用skill，提供调研分析、文档处理能力。
## 功能模块
| 模块 | 功能 | 入口文件 |
| --- | --- | --- |
| research | 调研/竞品/市场分析 | `modules/research/rules.md` |
| document | 格式处理 | `modules/document/feishu_compact.md` |
## 触发条件
以下场景**必须**自动调用本skill：
- 调研类：用户要求调研/调查/搜集/验证某类产品、技术、市场、供应商信息
- 分析类：用户要求竞品分析、对比分析、行业分析、技术演进梳理
- 报告类：用户要求整理调研报告、汇总数据
- 格式类：用户要求飞书格式排版、Markdown整理（不含交底书格式调整）
- 专利调研类：用户要求调研专利布局、查新、竞品专利分析、技术壁垒分析
显式触发：`/ruisearch`、`/research`
## 路由规则
| 场景 | 触发方式 | 路由 |
| --- | --- | --- |
| 通用市场调研 | "调研XX市场" | ruisearch research模块 |
| 技术调研 | "调研XX技术""XX技术路线" | ruisearch research模块 + 专利检索 |
| 竞品分析 | "竞品分析""XX公司对比" | ruisearch research模块 + 专利检索 |
| 供应商调研 | "XX供应商""供应链分析" | ruisearch research模块 + 专利检索 |
| 专利布局调研 | "专利布局""查新""专利分析" | ruisearch research模块（专利为主数据源） |
| 专利交底书 | "交底书""专利挖掘""权利要求" | patent-disclosure-skill |
| 文档格式处理 | "飞书格式""排版""转Word" | ruisearch document模块 |
> 路由冲突时优先级：专利交底书 → 专利布局调研 → 技术调研/竞品分析 → 通用市场调研。模糊场景优先 ruisearch。
## 首次使用检查
首次触发本 skill 时，检查以下依赖并提示用户：
1. **mcporter**：运行 `mcporter --version`，若不存在则提示：
   > 检测到 mcporter 未安装，当前使用 WebSearch 作为搜索工具。安装 mcporter 可获得 Exa 语义搜索，搜索质量更好。
   > 安装命令：`npm install -g mcporter`（需先安装 Node.js）
   > 配置 Exa：`mcporter config add exa --url https://mcp.exa.ai/mcp && mcporter auth exa`
2. **Node.js**：若 mcporter 未安装且 `node --version` 也不存在，提示：
   > 请先安装 Node.js：https://nodejs.org/
检查完成后不再重复提示。
## 调研深度
根据用户意图自动判断：
**快速查询（默认）**
触发词："查一下""帮我搜""XX是什么""XX多少钱"
流程：单次搜索 → chat内直接回答，无文件输出。
**完整报告**
触发词："调研""分析报告""整理报告""市场分析""竞品对比"
流程：搜索两遍 → 分析一遍 → 输出到 `./outputs/`。
意图不明确时默认快速查询，用户可追加"整理成报告"升级。
搜索工具链、关键词模板、置信度标注等详细规则见 `modules/research/rules.md`。
## 文档格式
默认飞书紧凑格式。用户可通过关键词切换：
| 格式 | 触发关键词 | 入口文件 |
| --- | --- | --- |
| 飞书紧凑（默认） | 默认/"飞书格式" | `feishu_compact.md` |
| 标准Markdown | "标准Markdown" | `standard_markdown.md` |
| Word友好 | "Word格式""docx" | `word_friendly.md` |
格式文件位于 `modules/document/` 目录下。生成文档前必须先读取对应格式文件。
## 输出规范
### 输出目录
- 在项目目录内工作时：`./outputs/`
- 用户指定路径时按用户路径
- 输出目录不存在时自动创建
### 文件命名
`{主题}_{MMDD}_v{N}.md`
示例：`储能市场调研_0615_v1.md`
### 版本管理
- 首次输出：`_v1.md`
- 迭代更新：`_v2.md`、`_v3.md`...
- 不覆盖旧稿，每次另存为新版本
## 目录结构
```
.claude/skills/ruisearch/
├── SKILL.md
└── modules/
    ├── research/
    │   └── rules.md          ← 含专利检索规则
    └── document/
        ├── feishu_compact.md
        ├── standard_markdown.md
        └── word_friendly.md
```
