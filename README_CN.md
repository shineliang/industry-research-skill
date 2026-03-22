# Claude Code 多智能体协同行业调研 Skill

[English](./README.md) | **中文**

一个基于 Claude Code 的行业调研技能（Skill），通过多个 AI Agent 协同工作，自动完成行业调研全流程——从主题分析、并行研究、质量审核到报告合并，最终输出结构化的 Markdown 调研报告。

## 工作原理

```
用户: /research AI Agent 行业现状

     ┌──────────────────────────────────────────────────────────┐
     │                  主 Agent（编排器）                        │
     └──────────────────────┬───────────────────────────────────┘
                            │
     ┌──────────────────────▼───────────────────────────────────┐
     │  阶段 0: 加载配置                                         │
     │  环境变量 → 深度预设 → 用户确认 → config.md                 │
     └──────────────────────┬───────────────────────────────────┘
                            │
     ┌──────────────────────▼───────────────────────────────────┐
     │  阶段 1: 主题分析 → 动态角色分配                            │
     │  拆解主题 → 生成 3-8 个调研角色 → brief.md                  │
     └──────────────────────┬───────────────────────────────────┘
                            │
     ┌──────────────────────▼───────────────────────────────────┐
     │  阶段 2: 并行调研（一次性 spawn N 个 agent）               │
     │                                                           │
     │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │
     │  │ Agent A  │ │ Agent B │ │ Agent C │ │ Agent D │  ...   │
     │  │ WebSearch│ │WebSearch│ │WebSearch│ │WebSearch│       │
     │  │ +Codex   │ │+Gemini │ │ +Codex  │ │+Gemini │       │
     │  └────┬─────┘ └────┬────┘ └────┬────┘ └────┬────┘       │
     │       ▼            ▼           ▼           ▼             │
     │  round-N/agent-*.md（结构化 Markdown 产出）                │
     └──────────────────────┬───────────────────────────────────┘
                            │
     ┌──────────────────────▼───────────────────────────────────┐
     │  阶段 3: 质量审核与迭代                                    │
     │  Reviewer 按 5 个维度评分（满分 50）                        │
     │  通过 → 进入合并 | 不通过 → 带反馈重做（最多 N 轮）         │
     └──────────────────────┬───────────────────────────────────┘
                            │
     ┌──────────────────────▼───────────────────────────────────┐
     │  阶段 4: 合并收敛                                          │
     │  Merger 整合所有调研产出 → 统一报告                         │
     │  Reviewer 审核合并稿                                       │
     └──────────────────────┬───────────────────────────────────┘
                            │
     ┌──────────────────────▼───────────────────────────────────┐
     │  阶段 5: 最终输出 → output/report.md                       │
     └──────────────────────────────────────────────────────────┘
```

### 核心设计理念

- **文件驱动通讯** — 所有中间产物都是结构化 Markdown 文件，agent 间不共享内存，确保可靠性和完整审计记录
- **动态角色分配** — 主 agent 根据调研主题自动生成匹配的调研角色，而非预设固定角色
- **多模型多样性** — 子 agent 可调用 Codex（OpenAI）和 Gemini（Google）CLI 获取不同 AI 模型的补充视角
- **质量门禁与迭代** — 结构化评分标准，既防止放水也防止无限循环
- **全参数可配置** — 支持环境变量、启动时交互选择、深度预设三种配置方式

## 安装

将 Skill 文件复制到 Claude Code 的技能目录：

```bash
# 克隆仓库
git clone https://github.com/shineliang/industry-research-skill.git

# 复制到 Claude Code 技能目录
mkdir -p ~/.claude/skills/industry-research
cp industry-research-skill/SKILL.md ~/.claude/skills/industry-research/SKILL.md
```

或者手动下载 [`SKILL.md`](./SKILL.md) 放到 `~/.claude/skills/industry-research/SKILL.md`。

### 前置要求

- **Claude Code**（必须）— 主 agent 运行环境
- **Codex CLI**（可选）— `npm install -g @openai/codex` — 提供 OpenAI 模型的补充视角
- **Gemini CLI**（可选）— `npm install -g @google/gemini-cli` — 提供 Google 模型的补充视角

CLI 工具是可选的。未安装时 skill 会优雅降级（仅用 WebSearch，不阻塞流程）。

## 使用方法

```bash
/research AI Agent 行业现状             # 标准调研
/research 新能源汽车 --depth=deep       # 深度调研（最多 10 轮迭代）
/research SaaS 市场 --depth=quick       # 快速概览（仅 1 轮）
```

### 可配置参数

所有参数支持三级覆盖：**环境变量 > 启动时用户选择 > 默认值**。

| 参数 | 环境变量 | 默认值 | 说明 |
|------|----------|--------|------|
| Agent 数量 | `RESEARCH_AGENT_COUNT` | `auto` | 设为数字则固定，`auto` 则动态分配 |
| 研究迭代上限 | `RESEARCH_MAX_ROUNDS` | `3` | 研究阶段最多迭代几轮 |
| 合并迭代上限 | `MERGE_MAX_ROUNDS` | `2` | 合并阶段最多迭代几轮 |
| 质量总分阈值 | `QUALITY_THRESHOLD` | `35` | 满分 50 |
| 单项最低分 | `QUALITY_MIN_PER_DIM` | `6` | 满分 10 |
| CLI 工具 | `RESEARCH_CLI_TOOLS` | `codex,gemini` | `codex`、`gemini`、`codex,gemini`、`none` |
| CLI 超时 | `RESEARCH_CLI_TIMEOUT` | `600000` | 毫秒 |
| 输出语言 | `RESEARCH_LANG` | `zh` | `zh` 中文 / `en` 英文 |
| 调研深度 | `RESEARCH_DEPTH` | `standard` | `quick` / `standard` / `deep` |

### 深度预设

| 参数 | quick（快速） | standard（标准） | deep（深度） |
|------|:---:|:---:|:---:|
| Agent 数量 | 2-3 | 3-5 | 5-8 |
| 研究迭代上限 | 1 轮 | 3 轮 | 10 轮 |
| 合并迭代上限 | 1 轮 | 2 轮 | 3 轮 |
| 质量阈值 | 30/50 | 35/50 | 40/50 |
| 最少搜索关键词 | 3 | 5 | 10 |

示例：对复杂主题进行深度调研：
```bash
export RESEARCH_DEPTH=deep
export RESEARCH_MAX_ROUNDS=10
# 启动 Claude Code 后执行 /research <主题>
```

### 质量评分体系

Reviewer 从 5 个维度评分（各 10 分，满分 50）：

| 维度 | 评价内容 |
|------|----------|
| 事实准确性 | 数据是否有源可查、是否存在错误或矛盾 |
| 信源覆盖度 | 是否多元视角、是否有权威信源、是否有明显遗漏 |
| 分析深度 | 是否有因果分析（而非仅罗列）、是否有独到洞察 |
| 内部一致性 | 各 agent 间数据是否矛盾、论述是否自洽 |
| 完整性 | 相对于调研 brief 的覆盖率、必答问题是否解答 |

### 产出文件结构

每次调研在当前目录下生成独立的项目文件夹：

```
{主题}/
├── config.md                    # 本次调研使用的配置
├── brief.md                     # 调研简报（角色分配、必答问题）
├── round-1/
│   ├── agent-{角色}.md          # 各 agent 的调研产出
│   └── review.md               # 质量评审报告（含评分）
├── round-2/                     # 如需迭代
│   └── ...
├── merged/
│   ├── draft-1.md               # 合并稿
│   └── review-1.md              # 合并稿评审
└── output/
    └── report.md                # 最终报告
```

---

## 实际案例：AI Agent 行业调研

[`examples/ai-agent-industry-2025/`](./examples/ai-agent-industry-2025/) 目录包含一次真实调研的完整产出——主题为 **"AI Agent 行业现状"**。

### 运行概要

| 指标 | 数值 |
|------|------|
| 调研主题 | AI Agent 行业现状（2025） |
| 调研深度 | standard |
| Agent 数量 | 4 个（市场分析、技术趋势、竞品格局、应用场景） |
| CLI 工具 | Codex（2 个 agent）+ Gemini（2 个 agent） |
| 研究轮次 | 1 轮（首轮即通过） |
| 合并轮次 | 1 轮（首轮即通过） |
| 第 1 轮评审得分 | **37/50**（阈值 35） |
| 合并稿评审得分 | **39/50** |
| 最终报告大小 | ~33KB（约 8,000 字） |

### 动态角色分配结果

主 agent 分析"AI Agent 行业现状"后，自动生成了 4 个调研角色：

| 角色 | 代号 | 调研方向 | 辅助 CLI |
|------|------|----------|----------|
| 市场规模分析师 | `market-analyst` | 市场规模、增长率、投融资、产业链 | Codex |
| 技术趋势分析师 | `tech-trend` | 技术架构、开发框架、技术突破与瓶颈 | Gemini |
| 竞品格局分析师 | `competitor-analyst` | 头部产品、大厂布局、创业公司 | Codex |
| 应用场景分析师 | `app-scenario` | 落地案例、ROI、商业化挑战 | Gemini |

### 报告核心发现

1. **编码 Agent 率先跑通商业化** — Claude Code、GitHub Copilot、Cursor 合占编码 Agent 市场 70% 以上份额，是目前 PMF 最清晰的子赛道。

2. **企业部署存在巨大的"意愿-行动鸿沟"** — 82-85% 的企业表达采用意愿，但真正规模化部署仅约 2%（Capgemini 口径）。报告系统性地梳理了 9 个矛盾的采用率数据点，按调研方法论逐一拆解。

3. **协议标准化是 2025 年最大基础设施突破** — Anthropic MCP（Agent 到工具）和 Google A2A（Agent 到 Agent）协议，已纳入 Linux Foundation AAIF 统一治理。

4. **"Agent Washing"现象严重** — 数千家供应商中仅约 130 家提供真正的自主 Agent 能力。Gartner 预测超 40% Agent 项目将在 2027 年底前被取消。

5. **资本高度集中于头部** — 2024 年全球 AI 融资超 1000 亿美元，但四笔超大额交易约占 400 亿美元。

### 质量审核亮点

Reviewer 在审核中发现了 4 个跨 agent 的数据矛盾，并要求 Merger 在合并时处理：

| 问题 | 处理方式 |
|------|----------|
| 企业部署率矛盾（2% vs 11% vs 57% vs 82%） | 建立分层拆解表，按调研方法论、样本范围和定义口径逐条分析 |
| Anthropic 年收入 22 亿 vs Claude Code 单品 25 亿的悖论 | 透明标注"待核实"，给出时间节点差异解释 |
| "中国金融智能体 8000 亿元"可疑数据 | 删除，理由：子行业不可能超过全行业总量 |
| 2025 年 AI 融资 894 亿 vs 1500 亿美元差异 | 说明为 VC 口径 vs 全口径融资的差别 |

### 案例文件清单

```
examples/ai-agent-industry-2025/
├── config.md                              # 调研配置
├── brief.md                               # 调研简报（4 个角色定义）
├── round-1/
│   ├── agent-market-analyst.md      (14K) # 市场规模分析
│   ├── agent-tech-trend.md          (18K) # 技术趋势分析
│   ├── agent-competitor-analyst.md  (19K) # 竞品格局分析
│   ├── agent-app-scenario.md        (16K) # 应用场景分析
│   └── review.md                    (11K) # 质量评审：37/50 通过
├── merged/
│   ├── draft-1.md                   (33K) # 合并报告
│   └── review-1.md                   (9K) # 合并评审：39/50 通过
└── output/
    └── report.md                    (33K) # 最终报告
```

---

## 架构设计

### Agent 通讯模型

所有 agent 之间通过 Markdown 文件通讯——没有共享内存，没有直接消息传递。这个设计选择带来以下优势：

- **可靠性**：文件 I/O 是确定性的，不会出现上下文窗口溢出
- **跨 CLI 兼容**：Codex 和 Gemini CLI 都能读写同样的文件
- **审计追踪**：每一步中间过程都被保留
- **无状态迭代**：每轮新建的 agent 读取上轮产出 + 反馈，无需维护状态

```
第 1 轮：
  Agent A 写入 → round-1/agent-a.md
  Agent B 写入 → round-1/agent-b.md
  Reviewer 读取全部 → 写入 round-1/review.md（不通过）

第 2 轮：
  新 Agent A 读取 round-1/agent-a.md + round-1/review.md → 写入 round-2/agent-a.md
  新 Agent B 读取 round-1/agent-b.md + round-1/review.md → 写入 round-2/agent-b.md
  Reviewer 读取全部 → 写入 round-2/review.md（通过）
```

### CLI 调用协议

子 agent 调用 Codex/Gemini CLI 时采用降级重试链：

```
Codex:  xhigh → high → medium → low → Claude 自行分析
Gemini: 完整 prompt → 简化 prompt → 缩减维度 → Claude 自行分析
```

- 超时：600,000ms（10 分钟）
- 临时文件隔离：`mktemp /tmp/research-XXXXXX.txt`
- CLI 不可用：跳过，不阻塞（继续用 WebSearch 完成研究）

### 质量门禁流程

```
                    ┌─────────────────┐
                    │ Reviewer 评分    │
                    │ 5 个维度         │
                    │（各 10 分）       │
                    └────────┬────────┘
                             │
                   ┌─────────▼─────────┐
                   │ 总分 ≥ 阈值       │──否──┐
                   │ 且每项 ≥ 最低分    │      │
                   └─────────┬─────────┘      │
                             │               │
                            是          ┌────▼─────────┐
                             │         │ 轮次 < 上限？  │
                   ┌─────────▼──────┐  └────┬──────────┘
                   │ 进入下一阶段    │      │是     │否
                   └────────────────┘      │       │
                                     ┌────▼────┐ ┌▼──────────┐
                                     │带反馈    │ │上报用户    │
                                     │重新调研  │ │决策        │
                                     └─────────┘ └───────────┘
```

---

## 扩展

### 添加新的深度预设

编辑 `SKILL.md` 中的深度预设表，即可添加自定义档位（如 `exhaustive`，20 轮迭代）。

### 自定义调研输出模板

修改 `SKILL.md` 中"研究 Agent Prompt"部分的输出模板，所有 agent 会自动采用新格式。

### 不使用 CLI 工具

设置 `RESEARCH_CLI_TOOLS=none` 或不安装 Codex/Gemini 即可。Skill 完全基于 Claude Code 的 WebSearch 工作。

---

## 许可证

MIT
