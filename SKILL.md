---
name: ruisearch
description: |
  多功能通用 skill：17 平台搜索、市场调研、竞品分析、股票行情、文档格式处理。
  覆盖平台：网页搜索(Exa)、小红书、抖音、微博、推特、B站、V2EX、Reddit、LinkedIn、GitHub、YouTube、播客、微信公众号、RSS。
  产出能力：调研报告(含专利检索)、飞书/标准MD/Word三种文档格式。
version: "3.0.0"
user-invocable: true
argument-hint: "[可选：功能名称]"
triggers:
  - search: 搜/查/找/search/搜索/查一下/帮我搜
  - social:
    - 小红书: xiaohongshu/xhs/小红书/红书
    - 抖音: douyin/抖音
    - Twitter: twitter/推特/x.com/推文
    - 微博: weibo/微博
    - B站: bilibili/b站/哔哩哔哩
    - V2EX: v2ex
    - Reddit: reddit
  - career: 招聘/职位/求职/linkedin/领英/找工作
  - dev: github/代码/仓库/gh/issue/pr/分支/commit
  - web: 网页/链接/文章/公众号/微信文章/rss/读一下/打开这个
  - video: youtube/视频/播客/字幕/小宇宙/转录/yt
  - finance: 雪球/股票/stock/xueqiu/行情/股价/基金/财报
  - research: 调研/报告/分析报告/竞品分析/市场分析/行业分析/供应商/技术路线
  - patent: 专利/查新/专利布局/技术壁垒/知识产权/IP
  - format: 飞书格式/排版/Word格式/标准Markdown/整理报告
---

# Ruisearch

多功能通用 skill，提供 17 平台搜索、调研分析、股票行情、文档处理能力。

## 路由表

| 用户意图 | 路由 | 详细文档 |
|---------|------|---------|
| 网页搜索/代码搜索 | search | [references/search.md](references/search.md) |
| 小红书/抖音/微博/推特/B站/V2EX/Reddit | social | [references/social.md](references/social.md) |
| 招聘/职位/LinkedIn | career | [references/career.md](references/career.md) |
| GitHub/代码 | dev | [references/dev.md](references/dev.md) |
| 网页/文章/公众号/RSS | web | [references/web.md](references/web.md) |
| YouTube/B站/播客字幕 | video | [references/video.md](references/video.md) |
| 股票/行情/股价/基金 | finance | `modules/research/rules.md`（股票行情查询章节） |
| 调研/报告/竞品/专利 | research | `modules/research/rules.md` |
| 文档格式处理 | format | `modules/document/` |

> 路由冲突优先级：调研报告 → 股票行情 → 社交媒体 → 通用搜索。模糊场景默认快速查询。

## 零配置快速命令

```bash
# Exa 网页搜索
mcporter call 'exa.web_search_exa(query: "query", numResults: 5)'

# 通用网页阅读
curl -s "https://r.jina.ai/URL"

# 股票行情（腾讯接口，无鉴权）
curl -s "https://qt.gtimg.cn/q=r_hk01810" | iconv -f GBK -t UTF-8

# GitHub 搜索
gh search repos "query" --sort stars --limit 10

# Twitter 搜索
twitter search "query" --limit 10

# YouTube/B站字幕
yt-dlp --write-sub --skip-download -o "/tmp/%(id)s" "URL"

# Reddit 搜索
rdt search "query" --limit 10

# V2EX 热门
curl -s "https://www.v2ex.com/api/topics/hot.json" -H "User-Agent: agent-reach/1.0"
```

## 环境自检（首次触发时自动执行）

首次触发本 skill 时，必须执行以下检查，确保搜索链路可用。

### 检查 1：Node.js
```bash
node --version
```
若不存在，提示安装 Node.js（≥18）。

### 检查 2：mcporter
```bash
mcporter --version
```
若不存在，提示 `npm install -g mcporter`。

### 检查 3：Exa MCP 可用性
```bash
mcporter call 'exa.web_search_exa(query: "test", numResults: 1)'
```
- 返回正常 JSON → 通过 ✅，后续调研走 Exa 优先链路
- 返回错误/超时 → 提示配置 Exa，当前降级 WebSearch

### 状态标注
每次调研开始前标注当前搜索链路：`[Exa]` 或 `[WebSearch]`。

## 调研深度

### 快速查询（默认）
触发词："查一下""帮我搜""XX是什么""XX多少钱""股价"
流程：单次搜索 → chat 内直接回答，标注来源和置信度。不生成文件。

### 完整报告
触发词："调研""分析报告""整理报告""市场分析""竞品对比"
流程：搜索两遍 → 分析一遍 → 输出到 `./outputs/`。
详细规则见 [modules/research/rules.md](modules/research/rules.md)。

意图不明确时默认快速查询，用户可追加"整理成报告"升级。

## 文档格式

默认飞书紧凑格式。用户可通过关键词切换：

| 格式 | 触发关键词 | 入口文件 |
| --- | --- | --- |
| 飞书紧凑（默认） | 默认/"飞书格式" | `modules/document/feishu_compact.md` |
| 标准Markdown | "标准Markdown" | `modules/document/standard_markdown.md` |
| Word友好 | "Word格式""docx" | `modules/document/word_friendly.md` |

## 输出规范

### 输出目录
- 项目内：`./outputs/`
- 用户指定路径时按用户路径
- 目录不存在时自动创建

### 文件命名
`{主题}_{MMDD}_v{N}.md`，如 `储能市场调研_0615_v1.md`

### 版本管理
首次 `_v1.md`，迭代 `_v2.md` → `_v3.md`，不覆盖旧稿。

## 工作区规则

**不要在 skill 目录创建文件。** 使用 `/tmp/` 存放临时输出，项目 `./outputs/` 存放正式产出。

## 目录结构

```
.claude/skills/ruisearch/
├── SKILL.md
├── references/                      ← 平台工具文档
│   ├── search.md
│   ├── social.md
│   ├── web.md
│   ├── video.md
│   ├── career.md
│   └── dev.md
└── modules/
    ├── research/
    │   └── rules.md                 ← 调研规则、专利检索、股票API、关键词模板
    └── document/
        ├── feishu_compact.md
        ├── standard_markdown.md
        └── word_friendly.md
```
