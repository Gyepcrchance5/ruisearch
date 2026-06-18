# Ruisearch
Claude Code 的调研 skill，用于市场调研、技术调研、竞品分析、行业分析、专利调研等需要搜索验证并整理成报告的任务。
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
    │   └── rules.md          ← 含专利检索规则
    └── document/
        ├── feishu_compact.md
        ├── standard_markdown.md
        └── word_friendly.md
```
安装后重启 Claude Code 会话即可生效。
## 依赖工具
| 工具 | 用途 | 是否必须 | 安装方式 |
| --- | --- | --- | --- |
| mcporter | 调用 Exa 语义搜索（效果最好） | 可选，自动降级 WebSearch | `npm install -g mcporter` |
| Exa MCP 服务 | mcporter 的搜索后端 | 依赖 mcporter | 见下方配置 |
| WebSearch | 内置搜索工具 | Claude Code 自带 | 无需安装 |
| WebFetch | 读取网页全文、专利详情页 | Claude Code 自带 | 无需安装 |
| Jina Reader | 读取网页全文 | 内置 curl 即可 | 无需安装 |
### Exa 搜索配置（可选，强烈推荐）
安装 mcporter 后，配置 Exa 服务以获得最佳搜索效果：
1. 安装 mcporter：
```bash
npm install -g mcporter
```
2. 配置 Exa 服务，在项目目录或全局创建 `mcporter.json`：
```bash
mcporter config add exa --url https://mcp.exa.ai/mcp
```
3. 完成 OAuth 认证：
```bash
mcporter auth exa
```
4. 验证配置：
```bash
mcporter list exa
```
> 不安装 mcporter 也不影响使用，skill 会自动降级到 WebSearch，搜索质量略有差异。
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
