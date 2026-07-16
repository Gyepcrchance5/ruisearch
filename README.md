# Ruisearch
Claude Code 的调研 skill，用于市场调研、技术调研、竞品分析、行业分析、专利调研等需要搜索验证并整理成报告的任务。
## 环境搭建（必须先做）
Ruisearch 依赖 Exa 语义搜索引擎获取高质量结果。**不配置搜索后端的话，只能降级到内置 WebSearch，搜索质量会明显下降。**
### 前置条件
需要 Node.js（≥18），没有的话先装：https://nodejs.org/
验证：
```bash
node --version   # 应 ≥ v18.0.0
```
### 安装 mcporter
```bash
npm install -g mcporter
```
验证：
```bash
mcporter --version
```
### 配置 Exa 搜索后端
```bash
mcporter config add exa --url https://mcp.exa.ai/mcp
mcporter auth exa
```
验证 Exa 是否真正可用（关键一步，很多人漏掉）：
```bash
mcporter call 'exa.web_search_exa(query: "test", numResults: 1)'
```
如果返回正常的搜索结果（JSON），说明配置成功。如果报错，检查：
- 网络是否能访问 `https://mcp.exa.ai/mcp`
- OAuth 认证是否完成（重新跑 `mcporter auth exa`）
> **为什么必须验证？** 裸装 `mcporter` 不加 Exa 后端等于没装。首次使用时会自动检测，但不提前配好的话，每次调研都先试 Exa → 失败 → 降级 WebSearch，白白浪费时间。
### 可选增强：社交平台搜索
如果需要搜索小红书/抖音/B站/微博/领英/YouTube 等社交平台，额外安装 [agent-reach](https://github.com/Panniantong/Agent-Reach) skill。Ruisearch 的核心搜索链路不依赖它。
## 安装
将 `ruisearch` 文件夹复制到 Claude Code 的 skills 目录：
```
~/.claude/skills/ruisearch/
```
目录结构：
```
ruisearch/
├── SKILL.md
├── README.md
└── modules/
    ├── research/
    │   └── rules.md          ← 含专利检索规则、Exa 搜索
    └── document/
        ├── feishu_compact.md
        ├── standard_markdown.md
        └── word_friendly.md
```
安装后重启 Claude Code 会话即可生效。
## 依赖工具
| 工具 | 用途 | 是否必须 | 安装方式 |
| --- | --- | --- | --- |
| mcporter + Exa | 语义搜索（首选，搜索质量显著优于内置搜索） | **强烈推荐** | 见上方「环境搭建」 |
| WebSearch | 内置搜索工具（Exa 不可用时的降级方案） | Claude Code 自带 | 无需安装 |
| WebFetch | 读取网页全文、专利详情页 | Claude Code 自带 | 无需安装 |
| Jina Reader | 读取网页全文 | 内置 curl 即可 | 无需安装 |
## 触发方式
- 显式：`/ruisearch` 或 `/research`
- 自然语言：说"调研""查一下""竞品分析""整理报告""专利布局"等
## 功能
### 快速查询（默认）
触发词："查一下""帮我搜""XX是什么"
流程：单次搜索 → chat 内直接回答，无文件输出。
### 完整报告
触发词："调研""分析报告""整理报告"
流程：搜索两遍 → 分析一遍 → 输出到 `./outputs/`
报告架构：
- **必选**：市场概况、竞争格局、技术趋势、关键结论
- **可选**：产业链/成本、应用场景深挖、专利布局、财务分析（按需插入）
### 专利调研（新增）
触发词："专利布局""查新""专利分析""技术壁垒"
数据源：
- 专利号精准查询：WebFetch Google Patents（标题+摘要+权利要求+法律状态+同族）
- 关键词查专利：WebSearch + CNIPA爬虫
- 竞品专利分析：WebSearch
技术调研和竞品分析时自动启用专利检索，报告默认包含专利布局章节。
## 路由规则
| 场景 | 路由 |
| --- | --- |
| 通用市场调研 | ruisearch research模块 |
| 技术调研/竞品分析/供应商调研 | ruisearch research模块 + 专利检索 |
| 专利布局调研 | ruisearch research模块（专利为主数据源） |
| 专利交底书 | patent-disclosure-skill（独立skill） |
## 文档格式
默认飞书紧凑格式，可通过关键词切换：
- "标准Markdown" → 标准 Markdown
- "Word格式"/"docx" → Word 友好格式
## 输出规范
- 文件命名：`{主题}_{MMDD}_v{N}.md`
- 输出目录：`./outputs/`（项目内工作时）
- 版本管理：不覆盖旧稿，每次迭代另存为新版本
- 表格不超过 4 列，每格 ≤ 30 字
