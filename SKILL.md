---
name: industry-research
description: |
  Multi-agent collaborative industry research. Dynamically assigns research roles,
  runs parallel research with Codex/Gemini CLI augmentation, iterative quality review,
  merge & converge, outputs structured Markdown report.

  All parameters are configurable via environment variables or interactive setup.

  Trigger: /research, 行业调研, industry research, 调研报告
metadata:
  version: 1.0.0
  openclaw:
    emoji: "\U0001F50D"
    homepage: https://github.com/shineliang/industry-research-skill
    requires:
      anyBins:
        - codex
        - gemini
        - claude
---

# 多智能体协同行业调研

协调多个 AI agent 并行完成行业调研，通过迭代审核和合并收敛产出高质量报告。

## Commands

```bash
/research <topic>                    # 启动行业调研（如：/research AI Agent 行业现状）
/research <topic> --depth=deep       # 深度调研模式
/research <topic> --depth=quick      # 快速概览模式
```

## Prerequisites

- **Claude Code** — 主 agent 运行时
- **Codex CLI** (`codex`) — 可选，提供 OpenAI 模型补充视角
- **Gemini CLI** (`gemini`) — 可选，提供 Google 模型补充视角
- CLI 工具未安装不影响核心流程，仅跳过补充视角

## 可配置参数

所有决策参数均可配置，优先级：**环境变量 > 启动时用户选择 > 默认值**。

| 参数 | 环境变量 | 默认值 | 说明 |
|------|----------|--------|------|
| 研究 agent 数量 | `RESEARCH_AGENT_COUNT` | `auto` | 设为数字则固定，`auto` 则动态分配 |
| 研究迭代上限 | `RESEARCH_MAX_ROUNDS` | `3` | 研究阶段最多迭代几轮 |
| 合并迭代上限 | `MERGE_MAX_ROUNDS` | `2` | 合并阶段最多迭代几轮 |
| 质量总分阈值 | `QUALITY_THRESHOLD` | `35` | 满分 50，总分需达到此值 |
| 单项最低分 | `QUALITY_MIN_PER_DIM` | `6` | 满分 10，每个维度不低于此值 |
| 启用 CLI 工具 | `RESEARCH_CLI_TOOLS` | `codex,gemini` | 可选值：`codex`、`gemini`、`codex,gemini`、`none` |
| CLI 超时 | `RESEARCH_CLI_TIMEOUT` | `600000` | 毫秒 |
| 输出语言 | `RESEARCH_LANG` | `zh` | `zh` 中文 / `en` 英文 |
| 报告深度 | `RESEARCH_DEPTH` | `standard` | `quick` / `standard` / `deep` |

### 深度预设

| 参数 | quick | standard | deep |
|------|-------|----------|------|
| Agent 数量 | 2-3 | 3-5 | 5-8 |
| 研究迭代上限 | 1 | 3 | 10 |
| 合并迭代上限 | 1 | 2 | 3 |
| 质量总分阈值 | 30 | 35 | 40 |
| WebSearch 最少关键词数 | 3 | 5 | 10 |

设置 `RESEARCH_DEPTH=deep` 一键启用深度模式，也可再单独覆盖个别参数。

## 工作流

### 阶段 0：配置加载与确认

收到调研主题后，**先完成配置再开始调研**：

1. 读取环境变量覆盖默认值：
```bash
# 读取所有 RESEARCH_* 环境变量
RESEARCH_DEPTH="${RESEARCH_DEPTH:-standard}"
RESEARCH_AGENT_COUNT="${RESEARCH_AGENT_COUNT:-auto}"
RESEARCH_MAX_ROUNDS="${RESEARCH_MAX_ROUNDS:-3}"
MERGE_MAX_ROUNDS="${MERGE_MAX_ROUNDS:-2}"
QUALITY_THRESHOLD="${QUALITY_THRESHOLD:-35}"
QUALITY_MIN_PER_DIM="${QUALITY_MIN_PER_DIM:-6}"
RESEARCH_CLI_TOOLS="${RESEARCH_CLI_TOOLS:-codex,gemini}"
RESEARCH_CLI_TIMEOUT="${RESEARCH_CLI_TIMEOUT:-600000}"
RESEARCH_LANG="${RESEARCH_LANG:-zh}"
```

2. 如果设置了 `RESEARCH_DEPTH`，先应用对应预设，再用环境变量中的具体参数覆盖

3. 检查 CLI 工具可用性：
```bash
command -v codex && codex --version || echo "CODEX_UNAVAILABLE"
command -v gemini && gemini --version || echo "GEMINI_UNAVAILABLE"
```
如果配置了某 CLI 但不可用，自动从 `RESEARCH_CLI_TOOLS` 中移除并提示用户。

4. 通过 AskUserQuestion 展示配置，让用户确认或调整：
   - 选项 A：使用当前配置开始
   - 选项 B：调整参数（逐项询问需修改的值）
   - 选项 C：切换为深度调研模式
   - 选项 D：切换为快速调研模式

5. 创建项目目录，将最终配置写入 `{topic-slug}/config.md`

### 阶段 1：主题分析与角色动态分配

配置确认后：

1. **分析主题**：拆解调研领域，识别 3-8 个关键维度（具体数量由 `RESEARCH_AGENT_COUNT` 决定）
2. **动态生成角色**，每个角色包含：
   - 角色名称和代号（如 `market-analyst`、`tech-trend`）
   - 调研范围和具体任务描述
   - 关键问题清单（3-5 个必答问题）
   - 推荐 CLI 工具（Codex/Gemini/仅 WebSearch）
3. **生成 brief.md**：写入调研简报
4. **用户确认**：通过 AskUserQuestion 展示角色方案，允许增减或调整

### 阶段 2：并行调研

1. 在一条消息中 **并行 spawn N 个研究 agent**（每个 Agent 调用使用不同 name）
2. 每个 agent 使用 **研究 Agent Prompt**（见下方模板），包含：
   - 角色定义、任务范围、必答问题
   - 输出模板
   - CLI 调用协议
   - 目标文件路径：`{topic-slug}/round-{N}/agent-{role-slug}.md`
3. 如果是迭代轮次 > 1，agent prompt 中额外包含上轮产出路径和 reviewer 反馈路径
4. 等待所有 agent 完成后进入审核

### 阶段 3：审核与迭代

1. **Spawn reviewer agent**，使用 **Reviewer Prompt**（见下方模板）
2. Reviewer 读取当前轮所有 agent 产出，按质量标准评分
3. Reviewer 写入 `{topic-slug}/round-{N}/review.md`
4. 主 agent 读取 review.md，解析评分和结论：
   - **通过**（总分 ≥ `QUALITY_THRESHOLD` 且单项 ≥ `QUALITY_MIN_PER_DIM`）→ 进入阶段 4
   - **不通过且轮次 < `RESEARCH_MAX_ROUNDS`** → 回到阶段 2（spawn 新 agent，prompt 中包含上轮产出 + 反馈）
   - **不通过且已达上限** → AskUserQuestion 让用户决策：
     - 接受当前质量，继续合并
     - 追加 N 轮迭代
     - 调整质量标准后重新评审
     - 手动介入

### 阶段 4：合并收敛

1. **Spawn merger agent**，使用 **Merger Prompt**（见下方模板）
2. Merger 读取最终通过轮次的所有产出，整合为 `{topic-slug}/merged/draft-{M}.md`
3. **Spawn reviewer agent** 审核合并稿，写入 `{topic-slug}/merged/review-{M}.md`
4. 判断逻辑同阶段 3，上限为 `MERGE_MAX_ROUNDS`

### 阶段 5：最终输出

1. 读取通过审核的合并稿
2. 最终格式化和润色
3. 写入 `{topic-slug}/output/report.md`
4. 向用户展示报告摘要和文件路径

---

## Agent Prompt 模板

### 研究 Agent Prompt

spawn 每个研究 agent 时，使用以下 prompt（替换 `{...}` 变量）：

```
你是行业调研团队的 {角色名}（{角色代号}）。

## 你的任务
{来自 brief.md 的角色定义和任务描述}

## 必答问题
{来自 brief.md 的关键问题清单，每个问题单独一行}

## 工作流程
1. 使用 WebSearch 搜索相关信息（至少搜索 {config.minSearchKeywords} 个不同关键词，覆盖中英文）
2. 整理初步研究结果，识别核心发现
3. 调用 {推荐的 CLI 工具} 获取补充视角（遵循下方 CLI 调用协议）
4. 按照输出模板格式，将研究报告写入文件：{目标文件路径}
5. 完成后返回结果

{如果是迭代轮次 > 1，包含以下段落：}
## 上轮产出和反馈
请先阅读以下文件：
- 你的上轮产出：{上轮文件路径}
- Reviewer 反馈：{review 文件路径}

重点关注 reviewer 对你（{角色名}）的具体修改指令，逐条落实。保留上轮中已获好评的内容，只改进被指出的问题。
{/如果}

## 输出模板（必须严格遵循此格式）

# {调研维度名称}

> 调研员：{角色名} | 轮次：{N} | 日期：{今日日期}

## 核心发现
1. {发现1 — 一句话概括}
2. {发现2}
3. {发现3}
（3-5 条）

## 详细分析

### {子主题 1}
{分析内容，包含数据、趋势、案例}

### {子主题 2}
{分析内容}

## 数据与信源
| 数据点 | 数值 | 来源 | 可信度 |
|--------|------|------|--------|
| {数据名} | {具体数值} | {来源名称和链接} | {高/中/低} |

## 补充视角
> 以下内容通过 {Codex/Gemini} CLI 获取（如未使用则删除此节）

{CLI 返回的补充分析}

## 风险与不确定性
- {不确定因素1}
- {不确定因素2}

## 待深入研究
- {可进一步挖掘的方向}

## CLI 调用协议

如果需要调用 Codex 或 Gemini CLI 获取补充视角：

1. 将你已收集的研究要点写入临时文件：
   RESEARCH_FILE=$(mktemp /tmp/research-XXXXXX.txt)

2. 调用 CLI（Bash 工具必须设置 timeout: {config.cliTimeout}）：
   - Codex: cat $RESEARCH_FILE | codex exec "基于以下调研内容，请补充遗漏的关键数据和观点，并指出可能存在的偏见或不准确之处。以{输出语言}输出。" 2>&1
   - Gemini: cat $RESEARCH_FILE | gemini -p "基于以下调研内容，请补充遗漏的关键数据和观点，并指出可能存在的偏见或不准确之处。以{输出语言}输出。" 2>&1

3. 超时降级：
   - Codex: 追加 "Use reasoning effort: high" → medium → low → Claude 自行分析
   - Gemini: 简化 prompt → 缩减维度 → Claude 自行分析
   - 最多重试 4 次

4. 清理：rm -f $RESEARCH_FILE

5. CLI 未安装或全部失败 → 跳过，不阻塞。如用 Claude 自行补充，标注 "[Claude Fallback]"

## 规则
- 数据必须标注来源和可信度评估
- 不要编造数据，找不到就明确标注"未找到可靠数据"
- 对不确定的判断使用"可能""推测""初步判断"等措辞
- 保持客观中立，呈现多元视角
- 优先引用一手数据和权威信源
```

### Reviewer Agent Prompt

```
你是行业调研团队的质量审核员。你的职责是严格、公正地评估调研产出的质量。

## 你的任务
审核第 {N} 轮所有调研 agent 的产出，按评分标准评审，给出结构化反馈。

## 评审标准（5 维度，各 10 分，满分 50）

| 维度 | 评分标准 |
|------|----------|
| 事实准确性 | 声明是否有据可查、数据是否标注来源、是否存在明显错误或自相矛盾 |
| 信源覆盖度 | 是否涵盖多元视角（中英文、不同机构）、是否有一手/权威信源、是否存在明显遗漏领域 |
| 分析深度 | 是否有原因分析而非仅罗列事实、是否有独到洞察、逻辑推理是否严密 |
| 内部一致性 | 各 agent 之间数据是否矛盾、同一 agent 内部论述是否自洽、结论是否与论据匹配 |
| 完整性 | 相对于 brief.md 中定义的调研范围和必答问题，覆盖率如何、有无关键遗漏 |

## 通过标准
总分 ≥ {config.qualityThreshold}/50，且每个单项 ≥ {config.qualityMinPerDim}/10

## 工作流程
1. 阅读调研简报：{brief.md 路径}
2. 逐个阅读所有 agent 产出：{各文件路径列表}
3. 按 5 个维度逐一打分
4. 交叉验证各 agent 之间的数据一致性
5. 对每个 agent 写出具体、可执行的修改指令
6. 做出通过/不通过决定
7. 按以下模板写入文件：{review.md 路径}
8. 完成后返回结论（通过/不通过 + 总分）

## 输出模板（必须严格遵循）

# 第 {N} 轮评审报告

## 评审结论：{通过 / 不通过}
## 总分：{X}/50

## 各维度评分
| 维度 | 得分 | 说明 |
|------|------|------|
| 事实准确性 | {X}/10 | {一句话说明扣分/加分原因} |
| 信源覆盖度 | {X}/10 | {一句话说明} |
| 分析深度 | {X}/10 | {一句话说明} |
| 内部一致性 | {X}/10 | {一句话说明} |
| 完整性 | {X}/10 | {一句话说明} |

## 亮点
- {值得保留的优质内容}

## 逐 Agent 修改指令

### Agent: {角色名}
1. {具体修改指令 — 明确说明需要补充/修改什么、期望的产出是什么}
2. {具体修改指令}

### Agent: {角色名}
1. {具体修改指令}

## 整体建议
{对整体调研方向或方法的建议，如有}

## 规则
- 评分必须基于实际内容质量，不可随意给高分放水
- 修改指令必须具体可执行（不可只说"需要改进"，要说明改进什么、怎么改）
- 交叉验证各 agent 之间的数据，发现矛盾必须指出
- 关注信源的权威性、时效性和多元性
- 如果某 agent 产出已经很好，明确说明"保持现状"而非无病呻吟
```

### Merger Agent Prompt

```
你是行业调研团队的报告整合专家。你的职责是将多个调研 agent 的产出整合为一份完整、连贯、有洞察的行业调研报告。

## 你的任务
将以下 agent 的调研产出合并为一份统一报告。

## 工作流程
1. 阅读调研简报：{brief.md 路径}
2. 阅读调研配置：{config.md 路径}
3. 逐个阅读所有 agent 最终产出：{各文件路径列表}
4. 提取各维度的核心发现
5. 识别并处理各 agent 之间的矛盾或重叠
6. 构建完整叙事逻辑，形成有洞察的报告
7. 按以下结构输出到：{merged/draft-N.md 路径}
8. 完成后返回结果

{如果是修订轮，包含以下段落：}
## Reviewer 反馈
请先阅读 reviewer 的审核意见：{review 文件路径}
按反馈意见逐条修订报告。保留已获好评的内容。
{/如果}

## 报告结构

# {调研主题} — 行业调研报告

> 日期：{今日日期} | 调研深度：{config.depth} | 迭代轮次：{实际轮次}

## 执行摘要
{500 字以内，概括核心发现、关键数据和主要结论}

## 1. 行业概述
{行业定义、边界、发展历程简述}

## 2. {维度 1 标题}
{整合该维度所有 agent 的研究成果}

## 3. {维度 2 标题}
{整合内容}

...（按实际维度数量扩展）

## N. 关键发现与洞察
{跨维度的核心洞察，不是简单罗列而是有分析的判断}

## N+1. 风险与挑战
{整合所有 agent 提到的风险，去重并分类}

## N+2. 展望与建议
{基于分析得出的行动建议}

## 附录
### 数据来源汇总
{所有信源的汇总表}

### 调研方法说明
{本次调研使用的方法、工具、agent 角色说明}

## 规则
- 去重合并时保留最准确、最有信源支持的版本
- 对矛盾数据必须标注说明（"A 数据显示 X，而 B 数据显示 Y"），不可静默丢弃
- 保持报告的叙事连贯性，避免简单拼接各 agent 的内容
- 所有数据保留原始信源标注
- 执行摘要必须能独立阅读，包含最重要的 3-5 个发现
```

---

## 主 Agent 编排逻辑

收到 `/research <topic>` 后，按以下逻辑编排：

### 阶段 0 实现

1. 运行 Bash 读取环境变量：
```bash
echo "RESEARCH_DEPTH=${RESEARCH_DEPTH:-standard}"
echo "RESEARCH_AGENT_COUNT=${RESEARCH_AGENT_COUNT:-auto}"
echo "RESEARCH_MAX_ROUNDS=${RESEARCH_MAX_ROUNDS:-}"
echo "MERGE_MAX_ROUNDS=${MERGE_MAX_ROUNDS:-}"
echo "QUALITY_THRESHOLD=${QUALITY_THRESHOLD:-}"
echo "QUALITY_MIN_PER_DIM=${QUALITY_MIN_PER_DIM:-}"
echo "RESEARCH_CLI_TOOLS=${RESEARCH_CLI_TOOLS:-codex,gemini}"
echo "RESEARCH_CLI_TIMEOUT=${RESEARCH_CLI_TIMEOUT:-600000}"
echo "RESEARCH_LANG=${RESEARCH_LANG:-zh}"
```

2. 根据 RESEARCH_DEPTH 应用预设（如未被环境变量显式覆盖的参数取预设值）：
   - quick: maxRounds=1, mergeRounds=1, threshold=30, minSearch=3, agentCount=2-3
   - standard: maxRounds=3, mergeRounds=2, threshold=35, minSearch=5, agentCount=3-5
   - deep: maxRounds=10, mergeRounds=3, threshold=40, minSearch=10, agentCount=5-8

3. 检查 CLI 可用性（Bash: `command -v codex && command -v gemini`）

4. AskUserQuestion 展示配置并确认

5. 创建目录并写入 config.md：
```bash
mkdir -p "{topic-slug}/round-1" "{topic-slug}/merged" "{topic-slug}/output"
```

### 阶段 1 实现

分析主题 → 生成角色 → 写入 brief.md → AskUserQuestion 确认角色方案

### 阶段 2 实现

在一条消息中发送多个 Agent tool call，并行 spawn 所有研究 agent：
- 每个 agent 使用 `name: "researcher-{role-slug}"`
- 使用 `mode: "bypassPermissions"`（agent 需要执行 Bash 调用 CLI）
- prompt 填入研究 Agent Prompt 模板

### 阶段 3 实现

Spawn reviewer agent（name: "reviewer-round-{N}"），读取所有产出并评审。
主 agent 读取 review.md 解析分数，根据通过/不通过决定下一步。

### 阶段 4 实现

Spawn merger agent（name: "merger-{M}"），读取通过轮次的所有产出。
再 spawn reviewer（name: "merge-reviewer-{M}"）审核合并稿。

### 阶段 5 实现

主 agent 直接读取通过的合并稿，做最终润色，写入 output/report.md。
向用户展示摘要 + 文件路径。

---

## 文件结构

每次调研在当前工作目录下创建：

```
{topic-slug}/
├── config.md                    # 本次调研的实际配置
├── brief.md                     # 调研简报（主题、范围、角色分配）
├── round-1/
│   ├── agent-{role-slug}.md     # 各 agent 调研产出
│   └── review.md               # Reviewer 评审结果
├── round-2/                     # 如需迭代
│   ├── agent-{role-slug}.md
│   └── review.md
├── merged/
│   ├── draft-1.md               # 合并稿
│   ├── review-1.md              # 合并稿审核
│   └── ...
└── output/
    └── report.md               # 最终报告
```
