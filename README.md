# Ruisearch

Ruisearch 是面向 Claude Code 的智能调研技能模块（Skill），提供**市场调研、技术分析、竞品追踪、专利检索、文档格式化**五位一体的自动化调研能力。通过多级搜索工具链与结构化输出引擎，将自然语言查询意图转化为可复用的分析报告。

## 核心能力

Ruisearch 围绕四个设计目标构建：

**搜索即服务** — 以 Exa 语义搜索引擎为主力，内置多级降级策略。优先走高质量语义结果，失败时无感切换到 WebSearch，搜索链路对用户透明。

**调研流程化** — 根据意图深度自动选择路径：事实类查询走快速通道（单次搜索，即时回答），分析类需求走完整流水线（两轮搜索 → 一轮分析 → 结构化输出），无需用户指定模式。

**报告可交付** — 输出遵循固定的报告架��：市场概况、竞争格局、技术趋势、关键结论四章必选骨架，产业链、专利布局、应用场景等模块按需插入，保证每份报告的下限质量。

**格式自适应** — 默认飞书兼容的紧凑 Markdown，可根据用户意图切换为标准 Markdown 或 Word 友好格式。

## 调研能力矩阵

| 能力 | 触发场景示例 | 数据源 | 输出 |
|------|-------------|--------|------|
| 市场调研 | "中国储能市场规模 2025" | Exa / WebSearch | 报告（市场概况 + 竞争格局 + 趋势 + 结论） |
| 技术调研 | "固态电池量产进展" | Exa + Google Patents | 报告 + 专利布局章节 |
| 竞品分析 | "特斯拉 vs 小鹏 智驾方案对比" | Exa + Google Patents | 多维对比报告 + 专利矩阵 |
| 供应商分析 | "宁德时代 储能电池 技术路线" | Exa + 官网 + 专利 | 供应商画像 + 技术壁垒分析 |
| 专利检索 | "DMS 驾驶员监测 麦格纳 专利" | Google Patents | 专利摘要卡 + 法律状态 + 同族 |
| 事实查询 | "小米港股现在多少钱" | 腾讯行情 API / Exa | 即时表格 + 来源标注 |
| 文档格式化 | "把这个报告转成 Word 格式" | 本地文件 | 格式转换后输出 |

## 搜索架构

```
用户意图
    │
    ├─ Exa 语义搜索（首选）
    │   └─ 失败 → WebSearch（自动降级，无感切换）
    │
    ├─ 技术/竞品类 → 自动触发专利检索
    │   ├─ 专利号 → Google Patents 详情页
    │   └─ 关键词 → WebSearch site:patents.google.com
    │
    └─ 深度分析 → Jina Reader 全文抓取
```

### 工具链

| 层级 | 工具 | 角色 | 可用性要求 |
|------|------|------|-----------|
| L1 | Exa (via mcporter) | 语义搜索，质量最优 | 需安装 mcporter + 配置 Exa API |
| L2 | WebSearch | 降级搜索，Claude Code 内置 | 零配置 |
| L3 | Google Patents | 专利详情提取 | 零配置 |
| L4 | Jina Reader | 网页全文读取 | 零配置 |
| L5 | 腾讯行情 API | A 股 / 港股实时数据 | 零配置 |

## 快速开始

### 安装

```bash
git clone https://github.com/Gyepcrchance5/ruisearch.git ~/.claude/skills/ruisearch
```

安装后重启 Claude Code 会话即可生效。

### 环境搭建

Ruisearch 以 Exa 语义搜索为首选引擎。不配置 Exa 也能使用，但搜索质量会下降。

**前置条件**：Node.js ≥ 18（[下载](https://nodejs.org/)）

```bash
# 1. 安装 mcporter
npm install -g mcporter

# 2. 注册 Exa MCP 后端
mcporter config add exa --url https://mcp.exa.ai/mcp
mcporter auth exa

# 3. 验证链路（关键步骤）
mcporter call 'exa.web_search_exa(query: "test", numResults: 1)'
```

返回正常 JSON 即配置成功。若失败，检查网络和 OAuth 认证状态。

> Ruisearch 首次触发时会自动执行三级环境自检（Node.js → mcporter → Exa 可用性），并在会话中标注当前搜索链路状态。

### 可选增强

需要搜索社交媒体（小红书、抖音、B 站、微博、LinkedIn、YouTube）时，额外安装 [agent-reach](https://github.com/Panniantong/Agent-Reach)。Ruisearch 的核心搜索链路不依赖它。

## 触发方式

| 方式 | 示例 |
|------|------|
| 显式调用 | `/ruisearch` 或 `/research` |
| 自然语言 | "调研 XX 市场""查一下 XX 技术""XX 竞品分析""整理成报告" |
| 专利场景 | "查新 XX""XX 专利布局""XX 公司专利分析" |
| 格式处理 | "飞书格式排版""转 Word 格式" |

## 路由规则

Ruisearch 与 [patent-disclosure-forge](https://github.com/your-org/patent-disclosure-forge)（专利交底书 Skill）存在边界交叉场景，按以下优先级路由：

| 优先级 | 场景 | 路由目标 |
|--------|------|---------|
| 1 | 专利交底书撰写 / 权利要求起草 | patent-disclosure-forge |
| 2 | 专利布局调研 / 查新 / 竞品专利分析 | Ruisearch（专利为主数据源） |
| 3 | 技术调研 / 竞品分析 / 供应商分析 | Ruisearch + 自动专利检索 |
| 4 | 通用市场调研 / 事实查询 | Ruisearch |

模糊场景默认走 Ruisearch。

## 输出规范

| 维度 | 规则 |
|------|------|
| 文件命名 | `{主题}_{MMDD}_v{N}.md`，如 `储能市场调研_0716_v1.md` |
| 输出目录 | `./outputs/`（工作目录内）；用户指定路径时按用户路径 |
| 版本管理 | 不覆盖旧稿，每次迭代另存为新版本 |
| 报告骨架 | 必选：市场概况 / 竞争格局 / 技术趋势 / 关键结论 |
| 可选模块 | 产业链分析 / 应用场景 / 专利布局 / 财务对比（按需插入） |
| 表格约束 | ≤ 4 列，每格 ≤ 30 字，长文本用表格 + 展开两层结构 |
| 置信度 | 高 / 中 / 低三级标注，多源交叉验证 |
| 来源标注 | 每条关键数据标注出处（官方文档 / 行业报告 / 媒体报道） |

## 目录结构

```
ruisearch/
├── SKILL.md                     # Skill 入口（路由表、触发条件、环境自检）
├── README.md                    # 本文档
└── modules/
    ├── research/
    │   └── rules.md             # 调研规则引擎（搜索工具链、关键词模板、报告架构）
    └── document/
        ├── feishu_compact.md    # 飞书兼容紧凑 Markdown 格式规范
        ├── standard_markdown.md # 标准 Markdown 格式规范
        └── word_friendly.md     # Word 友好格式规范
```

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| v2.2.0 | 2026-07 | 自包含化：合并 Exa 完整说明，移除 agent-reach 硬依赖；新增三级环境自检；股票行情查询模块 |
| v2.1.0 | 2026-06 | 新增专利检索能力（Google Patents + WebSearch） |
| v2.0.0 | 2026-06 | 初始发布：Exa 语义搜索 + 多格式输出 + 调研流水线 |

## License

MIT
