# AI Agent 竞品格局分析

> 调研员：竞品格局分析师 | 轮次：1 | 日期：2026-03-22

## 核心发现

1. **AI Agent 市场已进入"群雄逐鹿"阶段**：全球 AI Agent 市场 2025 年约 78 亿美元，预计 2030 年达 527 亿美元（CAGR 46.3%）。竞争已从"谁有最强模型"转向"谁能占据分发入口和企业工作流"。
2. **协议标准化成为新战场**：Anthropic 的 MCP 协议已成为 Agent-工具连接的事实标准（10,000+ 公共服务器），Google 的 A2A 协议补位 Agent 间互操作。MCP 和 A2A 均已捐赠给 Linux Foundation 下的 Agentic AI Foundation（AAIF），行业标准化进程加速。
3. **编码 Agent 是第一个跑通 PMF（产品-市场匹配）的子赛道**：Claude Code、GitHub Copilot、Cursor 三家年收入均突破 10 亿美元大关，合占编码 Agent 市场 70%+ 份额，远超其他 Agent 类别的商业化成熟度。
4. **企业软件巨头才是 Agent 落地的真正守门人**：Microsoft（Copilot + 365 + Azure）、Salesforce（Agentforce）、ServiceNow 掌握企业预算和业务入口，模型厂商更多扮演"能力供给方"角色。
5. **中国市场呈"头部集中、生态分化"格局**：字节 Coze（扣子）开源后生态优势显著，百度文心、腾讯元器、阿里百炼各据一方，但整体商业化仍落后于海外 1-2 个身位。

## 详细分析

### 全球头部产品/平台

AI Agent 市场不是一个单一市场，需要按四个层次理解竞争格局：

#### 第一层：模型/能力层

| 产品/模型 | 公司 | 定位 | 核心能力 | 市场地位 |
|-----------|------|------|----------|----------|
| GPT-5.x 系列 | OpenAI | 通用大模型 + Agent 能力供给 | 最广泛的开发者生态、Agents SDK、ChatGPT Agent | 消费端市占率 ~68%（从 87% 下滑） |
| Gemini 3 | Google | 多模态模型 + Agent 执行 | 搜索/Android 生态整合、coding/agent/多模态综合最强 | 消费端市占率 ~18%（从 5.4% 飙升） |
| Claude (Opus/Sonnet) | Anthropic | 安全导向模型 + Agent 基础设施 | MCP 协议生态、Computer Use、Claude Code | 消费端 ~2%，但企业端增速迅猛（年收入 $22 亿+） |
| DeepSeek | 深度求索 | 高性价比推理模型 | 极致的推理性价比，开源生态影响力大 | 模型层强势，但不等于完整 Agent 平台 |
| Qwen (通义千问) | 阿里 | 开源大模型 + 云端整合 | 阿里云生态、开源社区活跃 | 中国开源模型第一梯队 |

#### 第二层：Agent 开发平台层

| 平台 | 公司 | 定位 | 核心能力 |
|------|------|------|----------|
| OpenAI Agents SDK | OpenAI | 轻量 Python 框架 | 多 Agent 工作流、tracing、guardrails、GitHub 11K+ Stars |
| Vertex AI Agent Builder | Google | 企业级 Agent 开发 | 与 Google Cloud 深度整合、Agent Engine、Agentspace |
| Azure AI Foundry Agent Service | Microsoft | 企业 Agent 运行时 | Azure 生态、最广泛的企业连接器 |
| AWS Bedrock Agents | Amazon | 云端 Agent 服务 | 多 Agent 协作、企业接入、与 AWS 服务联动 |
| Coze / 扣子 | 字节跳动 | 一站式 Agent 开发 | 2025.7 开源（Apache 2.0）、48h 获 9K+ Stars、低门槛 |
| 阿里云百炼 | 阿里 | 企业 Agent 开发 | 深度整合阿里云生态、代码质量和安全规范 |
| LangChain / LangGraph | LangChain Inc. | 开源 Agent 框架 | v1.0 durable execution、human-in-the-loop、provider-agnostic |

#### 第三层：企业流程 Agent 层（嵌入式）

| 产品 | 公司 | 定位 | 核心能力 |
|------|------|------|----------|
| Microsoft 365 Copilot Agents | Microsoft | 办公 Agent | M365 + Teams + Graph 原生整合，最广企业连接器 |
| Salesforce Agentforce 360 | Salesforce | CRM/销售 Agent | CRM 数据 + 业务流程 + AgentExchange 市场，单季 6000 企业客户 |
| ServiceNow AI Agents | ServiceNow | ITSM/HR/客服 Agent | 2026.1 与 OpenAI 三年合作、2025 年收购 Moveworks |
| SAP Joule Agents | SAP | ERP Agent | 嵌入 ERP 流程、企业数据治理 |
| Oracle AI Agent Studio | Oracle | 企业应用 Agent | 全面覆盖 Oracle 应用生态 |
| IBM watsonx Orchestrate | IBM | 企业工作流 Agent | 多技能编排、企业级治理 |
| UiPath Agentic Automation | UiPath | RPA + Agent 融合 | LLM + Agent + RPA + Workflow，运营流程场景 |

#### 第四层：终端执行型/编程 Agent

| 产品 | 公司 | 定位 | 核心能力 | 市场数据 |
|------|------|------|----------|----------|
| Claude Code | Anthropic | AI 编码 Agent | 基于 MCP、CLI 原生、90-100% 代码生成 | 年化收入 $25 亿+，2025.5 发布后 8 个月超越 Copilot |
| GitHub Copilot | Microsoft/GitHub | AI 编码助手 | 最早的 AI 编码产品、GitHub 生态、coding agent 模式 | 年收入突破 $10 亿 |
| Cursor | Anysphere | AI 编码 IDE | Background Agents、全编辑器体验 | 2026.1 融资 $4 亿 |
| Devin | Cognition | 全自主编码 Agent | 最高自主性、端到端任务执行 | 降价至 $20/月 + 按用量计费 |
| Manus | Meta（2025.12 收购） | 通用执行型 Agent | Browser Operator、Wide Research、多 Agent 并行 | 被 Meta 以 $20-30 亿收购 |
| ChatGPT Agent | OpenAI | 消费级通用 Agent | 联网执行、自主操作、背景运行 | 依托 ChatGPT 用户基础 |

### 大厂 Agent 战略布局

#### OpenAI：从模型霸主到 Agent 平台

- **产品线**：Agents SDK（2025.3 发布）、ChatGPT Agent、AGENTS.md 规范（6 万+开源项目采用）
- **战略重心**：CEO 应用负责人 Fidji Simo 明确表示"一年后回答问题将是 AI 最不有用的事"，全力转向主动式 Agent
- **生态策略**：参与 AAIF 联合创始、拥抱 MCP 标准、贡献 AGENTS.md 规范
- **竞争优势**：消费端最大用户基数（68% 市占率）、最成熟的开发者生态
- **挑战**：消费端份额持续下滑、企业端面临 Microsoft/Salesforce 挤压

#### Google/Alphabet：全栈布局、增速最快

- **产品线**：Gemini 3（综合性能最强）、Vertex AI Agent Builder / Agent Engine、Agentspace（企业知识搜索）、Project Mariner（浏览器 Agent，仍在 research 阶段）
- **协议贡献**：A2A（Agent-to-Agent）协议，定位 Agent 间互操作标准，已捐赠 Linux Foundation
- **战略重心**：消费端利用搜索 + Android 生态快速扩张（市占率 5.4% -> 18.2%）、企业端通过 Google Workspace 切入
- **竞争优势**：搜索流量入口 + Android 分发 + Cloud 企业客户 + 模型综合能力
- **挑战**：企业软件生态深度不及 Microsoft

#### Anthropic：基础设施标准制定者

- **产品线**：Claude 模型系列、Claude Code（编码 Agent 年收入 $25 亿+）、Claude Cowork（企业协作 Agent）、Claude Code Channels（跨平台通信）
- **协议贡献**：MCP（Model Context Protocol）——"AI 的 USB-C"，10,000+ 公共服务器、被 ChatGPT/Cursor/Gemini/VS Code 等采用；Agent Skills 开放标准
- **战略重心**：enterprise-first，30 万+企业客户；通过 MCP 建立生态锁定
- **竞争优势**：MCP 成为事实标准后的生态控制力 + 编码 Agent 收入第一 + 安全/可靠性品牌
- **挑战**：消费端份额仅 ~2%，过度依赖 API/开发者渠道

#### Microsoft：最强企业分发方

- **产品线**：Microsoft 365 Copilot Agents、Copilot Studio（自定义 Agent）、Azure AI Foundry Agent Service、GitHub Copilot coding agent
- **战略重心**：利用 M365 + Teams + Azure + GitHub 的分发优势，把 Agent 嵌入企业日常工作流
- **竞争优势**：全球最广泛的企业软件入口、最多的预置企业连接器、同时是 OpenAI 最大投资者
- **挑战**：Copilot 定价争议、用户活跃度数据未公开

#### 字节跳动：中国 Agent 平台领跑者

- **产品线**：Coze / 扣子（Agent 开发平台）、Coze Space / 扣子空间（AI 协同办公）、Coze Studio + Coze Loop（开源）
- **战略重心**：2025.7 全面开源（Apache 2.0），从封闭平台转向开源生态；向"职场 AI / 可交付工作产物 / 技能生态"升级
- **竞争优势**：抖音/飞书生态流量 + 开源社区快速增长 + 低代码门槛 + 最活跃的中文 Agent 社区
- **挑战**：企业级场景深度不足、海外扩张受地缘政治影响

#### 阿里巴巴：云 + 模型 + 企业服务三位一体

- **产品线**：通义千问（Qwen 模型，开源）、阿里云百炼（Agent 开发平台）、Qoder（AI 编码）
- **战略重心**：深度整合阿里云生态，从开发到部署全链路 AI 支持；开源 Qwen 模型影响力大
- **竞争优势**：中国最大云厂商之一 + 开源模型生态 + 电商/供应链场景
- **挑战**：Agent 平台知名度不及字节 Coze

#### 百度：最早布局但增速放缓

- **产品线**：文心智能体平台、千帆 AppBuilder
- **战略重心**：利用搜索 + 自动驾驶 + 云服务的生态优势
- **竞争优势**：中文搜索入口、最早的大模型布局者之一
- **挑战**：在 DeepSeek/Qwen 冲击下模型层竞争力下滑，Agent 平台生态活跃度不及 Coze

#### 腾讯：社交生态 + 企业服务

- **产品线**：腾讯元器（Agent 平台）、腾讯云 ADP（企业知识/工作流/多智能体编排）
- **竞争优势**：微信/企业微信生态（月活 13 亿+）、社交场景天然入口
- **挑战**：大模型能力认知度低于竞品

### 创业公司赛道分析

#### 赛道热度与融资规模

- 2025 年 H1 Agentic AI 创业公司融资总额达 $28 亿
- 2025 年全球 AI 创业公司融资近 $1500 亿，占全球 VC 的 40%+
- 中国新兴 AI Agent 创业公司占国内平台市场的 27.8%

#### 主要创业赛道与代表公司

**1. 编码/开发者 Agent（PMF 最清晰）**
- Cursor/Anysphere（$4 亿融资，编码 IDE）
- Cognition/Devin（全自主编码 Agent）
- Codegen、Augment、Poolside 等
- 切入角度：开发者愿为效率买单，ROI 可量化

**2. 通用执行型 Agent**
- Manus（被 Meta $20-30 亿收购前获 Benchmark 领投 $7500 万）
- Genspark、Kortix
- 切入角度：浏览器操控、多 Agent 并行研究、端到端任务执行

**3. 垂直行业 Agent**
- Harvey（法律 AI Agent）
- Trace（企业 Agent 采纳，$300 万种子轮）
- 九科信息（中国，RPA + Agent 服务商）
- 切入角度：深耕特定行业 know-how，解决通用 Agent 无法覆盖的专业场景

**4. Agent 基础设施/中间件**
- LangChain（Agent 框架领导者，LangGraph v1.0）
- CrewAI、AutoGen（多 Agent 编排框架）
- 切入角度：做"卖铲子的人"，服务 Agent 开发者

**5. 企业员工助手/搜索型 Agent**
- Moveworks（被 ServiceNow 收购，代表行业整合信号）
- Glean、Aisera
- 切入角度：企业内部知识搜索 + 自动化 + Agent 市场

#### 创业公司的竞争优势

1. **场景聚焦**：大厂做平台、创业公司做场景；垂直深耕带来更高的用户粘性和转化率
2. **速度优势**：决策链短、迭代快，能在大厂反应前抢占心智（如 Manus 的现象级传播）
3. **开源策略**：通过开源获取开发者社区，再向企业端延伸（如 LangChain 路径）
4. **被收购退出**：Manus 被 Meta 收购、Moveworks 被 ServiceNow 收购，印证了大厂"买不如买"的整合逻辑

#### 创业公司的核心挑战

- 88% 的 AI POC 未能进入大规模部署（商业化鸿沟）
- 大厂平台能力快速跟进，窗口期短
- 资金消耗大（模型调用成本 + 算力成本）
- 企业客户对安全、合规、治理要求高

### 市场格局与趋势

#### 竞争态势关键变化

1. **护城河转移**：从"谁有最强模型"转向"谁掌握分发入口、集成能力和治理体系"。企业买单看的是身份权限、业务系统对接、审计监控、审批流——不只是模型聪明度。

2. **协议/标准之战**：
   - MCP（Anthropic）：Agent-工具/数据连接标准，已成事实标准
   - A2A（Google）：Agent-Agent 互操作标准
   - AGENTS.md（OpenAI）：Agent 项目指令标准
   - Agent Skills（Anthropic）：Agent 技能共享标准
   - 四者均在 Linux Foundation 下的 AAIF 中推进

3. **RPA 与 Agent 融合**：企业真实落地路径是 LLM + Agent + Workflow + RPA + Human Approval，而非纯自治 Agent。UiPath 等 RPA 厂商已明确转向 agentic automation。

4. **中国与海外竞争逻辑不同**：
   - 中国依赖企业微信/飞书/钉钉/微信生态 + 低代码 + 交付型场景
   - 海外依赖 Microsoft 365/Salesforce/ServiceNow/Google Workspace/GitHub 等成熟软件入口

5. **多智能体不是默认最优解**：2025-2026 年多数生产场景中，单 Agent + Workflow + 人审更可控、更便宜、更容易上线。多 Agent 协作仍在早期阶段。

6. **"有限自治"是主流**（Bounded Autonomy）：2025 年可用的 Agent 产品核心卖点是在边界内自治、关键动作需人工确认，而非彻底替代人。

#### 市场集中度

- **消费端**：ChatGPT（68%）+ Gemini（18%）占据 86%，高度集中但 ChatGPT 份额加速下滑
- **编码 Agent**：GitHub Copilot + Claude Code + Cursor 占 70%+，三巨头格局初定
- **企业流程 Agent**：Salesforce + ServiceNow + Microsoft 三足鼎立，但市场极度碎片化
- **中国市场**：字节 Coze + 百度文心 + 腾讯元器占第一梯队，但创业公司占比 27.8%

#### 2026 年趋势预判

- Gartner 预测 40% 企业应用将集成 AI Agent（2025 年不足 5%）
- IDC 预计 AI copilot 将嵌入近 80% 企业工作场所应用
- 82% 组织计划整合 AI Agent，商业化进程 2026-2027 年进入爆发期
- 周鸿祎预测进入"百亿智能体时代"
- 技术范式从"预训练+微调"让位于"通用基座+行业专精+推理时进化"

## 数据与信源

| 数据点 | 数值 | 来源 | 可信度 |
|--------|------|------|--------|
| 全球 AI Agent 市场规模 2025 | $78.4 亿 | MarketsandMarkets | 高 |
| 全球 AI Agent 市场预测 2030 | $526.2 亿（CAGR 46.3%） | MarketsandMarkets | 中（预测值） |
| 中国 AI Agent 市场预测 2028 | 8520 亿元（CAGR 72.7%） | 前瞻产业研究院 | 中 |
| ChatGPT 消费端市占率 | ~68%（2026.1） | Similarweb | 高 |
| Gemini 消费端市占率 | ~18.2%（2026.1） | Similarweb | 高 |
| Claude 消费端市占率 | ~2% | Similarweb | 高 |
| Anthropic 年化收入 2025 | $22 亿 | 多家媒体报道 | 中 |
| Claude Code 年化收入 | $25 亿+ | Codegen/CNBC 报道 | 中 |
| 编码 Agent 市场总规模 | $40 亿 | Codegen Blog | 中 |
| Anthropic 企业客户数 | 30 万+ | Anthropic 官方 | 高 |
| MCP 公共服务器数量 | 10,000+ | Pento/Anthropic | 高 |
| AGENTS.md 采用项目数 | 60,000+ | OpenAI/PulseMCP | 中 |
| Salesforce Agentforce 单季新增客户 | 6,000 | WebProNews | 中 |
| Agentforce 单季收入 | $5.4 亿 | WebProNews | 中 |
| Cursor 2026.1 融资额 | $4 亿 | CNBC | 高 |
| Manus 被 Meta 收购价 | $20-30 亿（推测） | 多家媒体 | 中 |
| 2025 H1 Agentic AI 创业融资总额 | $28 亿 | AI Agents Directory | 中 |
| 2025 年全球 AI 创业公司融资 | ~$1500 亿 | Yahoo Finance | 高 |
| 企业 AI Agent 生产部署率 | 57%（G2 调查） | G2 2025.8 | 中 |
| 40% 企业应用将集成 Agent by 2026 | Gartner 预测 | Gartner | 高 |
| AI POC 未能大规模部署比率 | 88% | 36Kr | 中 |

## 补充视角

> 以下内容通过 Codex CLI（OpenAI Codex, model: gpt-5.4）获取

### 建议补充的关键产品

Codex CLI 指出原始调研遗漏了以下重要玩家，已在上文详细分析中整合：

- **企业软件厂商**：SAP Joule Agents、Oracle AI Agent Studio、Workday Illuminate Agents、IBM watsonx Orchestrate——这些厂商在"已有业务系统内嵌 Agent"这条线上对大企业采购和落地极其重要
- **RPA 转型厂商**：UiPath 已明确转向 agentic automation，代表 RPA + Agent 融合趋势
- **被收购标的**：Moveworks 被 ServiceNow 收购是行业整合的关键信号
- **AWS Bedrock Agents**：不能把云厂商只写成模型托管方，AWS 在 Agent runtime 和多 Agent 协作上竞争力强
- **腾讯云 ADP**：中国部分不能只写字节/阿里/百度，腾讯在企业知识、工作流、多智能体编排上是明确玩家

### Codex 指出的潜在偏见与不准确点

1. **不要把所有厂商都并列为"Agent 厂商"**：模型供应商、Agent 平台方、消费级产品方、企业软件方的定位截然不同，不能混写
2. **不要把 GPTs/智能体商店等同于企业级 Agent 平台**：前者偏轻量分发，后者需要集成、治理、权限、监控
3. **用"市场份额"做强结论需谨慎**：多数厂商不单独披露 Agent 收入/活跃用户的可比数据
4. **区分 research/preview/beta/GA**：如 Project Mariner、部分 Computer Use 能力仍在实验阶段
5. **不要高估"完全自治"**：2025 年可用的 Agent 产品核心是 bounded autonomy
6. **DeepSeek 不等于 Agent 平台**：更准确的定位是高性价比模型/推理能力供给方

### 核实链接（Codex 提供）

- OpenAI: [New tools for building agents](https://openai.com/index/new-tools-for-building-agents/), [Introducing ChatGPT agent](https://openai.com/index/introducing-chatgpt-agent/)
- Anthropic: [MCP](https://docs.anthropic.com/en/docs/mcp), [Computer use](https://docs.anthropic.com/en/docs/agents-and-tools/computer-use), [Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview)
- Google: [Vertex AI Agent Builder](https://cloud.google.com/products/agent-builder), [A2A](https://developers.googleblog.com/en/google-cloud-donates-a2a-to-linux-foundation/)
- Microsoft: [Azure AI Foundry Agent Service](https://learn.microsoft.com/en-us/azure/ai-services/agents/concepts/agent-catalog)
- Salesforce: [Agentforce 2dx](https://investor.salesforce.com/news/news-details/2025/Salesforce-Launches-Agentforce-2dx)
- ServiceNow: [AI Agents](https://www.servicenow.com/products/ai-agents.html)

## 风险与不确定性

- **市场规模数据差异大**：不同研究机构对 AI Agent 市场的定义和口径差异显著（$52B vs $103B vs $183B by 2030/2032/2033），建议关注趋势方向而非绝对数字
- **企业部署与商业化鸿沟**：57% 企业称有 Agent 在生产环境，但 88% 的 POC 未能大规模部署，两个数据之间存在张力
- **收入数据多为推测**：除上市公司外，多数 Agent 产品收入数据来自媒体估算而非官方披露
- **地缘政治风险**：中国 Agent 厂商出海受限、芯片限制影响算力成本
- **"Agent"定义模糊**：行业对什么算"Agent"缺乏统一定义，部分厂商将传统自动化/Chatbot 包装为 Agent
- **安全与合规**：Agent 自主执行操作带来的安全、隐私、责任归属问题尚无成熟解决方案

## 待深入研究

- Agent 在具体垂直行业（如 HR/劳动力管理、制造、零售）的落地案例和 ROI 数据
- 盖雅工场所在的劳动力管理（WFM）行业中 AI Agent 的应用现状和竞品动态
- MCP vs A2A 协议的采用率对比和演进路径
- 中国 Agent 市场的监管政策走向（如生成式 AI 管理办法对 Agent 的适用性）
- 多 Agent 系统在企业真实部署中的可靠性和成本效益数据
- Agent 安全性/可控性的技术方案和行业标准进展
