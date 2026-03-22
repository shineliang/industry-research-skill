# Industry Research Skill for Claude Code

**English** | [中文](./README_CN.md)

Multi-agent collaborative industry research powered by Claude Code. Dynamically assigns research roles, runs parallel research with Codex/Gemini CLI augmentation, iterative quality review, merge & converge — outputs structured Markdown reports.

## How It Works

```
User: /research AI Agent 行业现状

     ┌──────────────────────────────────────────────────────────┐
     │                   Main Agent (Orchestrator)               │
     └──────────────────────┬───────────────────────────────────┘
                            │
     ┌──────────────────────▼───────────────────────────────────┐
     │  Stage 0: Load Config                                     │
     │  ENV vars → depth presets → user confirmation → config.md │
     └──────────────────────┬───────────────────────────────────┘
                            │
     ┌──────────────────────▼───────────────────────────────────┐
     │  Stage 1: Topic Analysis → Dynamic Role Assignment        │
     │  Analyze topic → generate 3-8 researcher roles → brief.md │
     └──────────────────────┬───────────────────────────────────┘
                            │
     ┌──────────────────────▼───────────────────────────────────┐
     │  Stage 2: Parallel Research (N agents spawned at once)    │
     │                                                           │
     │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │
     │  │ Agent A  │ │ Agent B │ │ Agent C │ │ Agent D │  ...   │
     │  │ WebSearch│ │WebSearch│ │WebSearch│ │WebSearch│       │
     │  │ +Codex   │ │+Gemini │ │ +Codex  │ │+Gemini │       │
     │  └────┬─────┘ └────┬────┘ └────┬────┘ └────┬────┘       │
     │       ▼            ▼           ▼           ▼             │
     │  round-N/agent-*.md (structured Markdown output)          │
     └──────────────────────┬───────────────────────────────────┘
                            │
     ┌──────────────────────▼───────────────────────────────────┐
     │  Stage 3: Quality Review & Iteration                      │
     │  Reviewer agent scores across 5 dimensions (50 pts max)   │
     │  Pass: total ≥ threshold AND each dim ≥ min               │
     │  Fail: re-run Stage 2 with feedback (up to N rounds)      │
     └──────────────────────┬───────────────────────────────────┘
                            │
     ┌──────────────────────▼───────────────────────────────────┐
     │  Stage 4: Merge & Converge                                │
     │  Merger agent consolidates all research → unified report   │
     │  Reviewer agent validates merged report                    │
     └──────────────────────┬───────────────────────────────────┘
                            │
     ┌──────────────────────▼───────────────────────────────────┐
     │  Stage 5: Final Output → output/report.md                 │
     └──────────────────────────────────────────────────────────┘
```

**Key design principles:**

- **File-driven communication** — All intermediate outputs are structured Markdown files, enabling reliable inter-agent communication and full audit trail
- **Dynamic role allocation** — The orchestrator analyzes the research topic and generates tailored researcher roles (not hardcoded)
- **Multi-model diversity** — Sub-agents can call Codex and Gemini CLI for supplementary perspectives from different AI models
- **Quality gates with iteration** — Structured scoring rubric prevents both rubber-stamping and infinite loops
- **All parameters configurable** — Environment variables, interactive setup, or depth presets

## Installation

Copy the skill file to your Claude Code skills directory:

```bash
# Clone the repo
git clone https://github.com/shineliang/industry-research-skill.git

# Copy skill to Claude Code
mkdir -p ~/.claude/skills/industry-research
cp industry-research-skill/SKILL.md ~/.claude/skills/industry-research/SKILL.md
```

Or manually: download [`SKILL.md`](./SKILL.md) and place it at `~/.claude/skills/industry-research/SKILL.md`.

### Prerequisites

- **Claude Code** (required) — The orchestrator runtime
- **Codex CLI** (optional) — `npm install -g @openai/codex` — Provides OpenAI model perspectives
- **Gemini CLI** (optional) — `npm install -g @google/gemini-cli` — Provides Google model perspectives

CLI tools are optional. If unavailable, the skill falls back gracefully (WebSearch only, no blocking).

## Usage

```bash
/research AI Agent 行业现状             # Standard research
/research 新能源汽车 --depth=deep       # Deep research (up to 10 iterations)
/research SaaS 市场 --depth=quick       # Quick overview (1 iteration)
```

### Configuration

All parameters support three-level override: **Environment variables > Interactive selection > Defaults**.

| Parameter | Env Var | Default | Description |
|-----------|---------|---------|-------------|
| Agent count | `RESEARCH_AGENT_COUNT` | `auto` | Number or `auto` for dynamic |
| Research iterations | `RESEARCH_MAX_ROUNDS` | `3` | Max research review cycles |
| Merge iterations | `MERGE_MAX_ROUNDS` | `2` | Max merge review cycles |
| Quality threshold | `QUALITY_THRESHOLD` | `35` | Out of 50 total |
| Min per dimension | `QUALITY_MIN_PER_DIM` | `6` | Out of 10 per dimension |
| CLI tools | `RESEARCH_CLI_TOOLS` | `codex,gemini` | `codex`, `gemini`, `both`, `none` |
| CLI timeout | `RESEARCH_CLI_TIMEOUT` | `600000` | Milliseconds |
| Language | `RESEARCH_LANG` | `zh` | `zh` or `en` |
| Depth preset | `RESEARCH_DEPTH` | `standard` | `quick`, `standard`, `deep` |

#### Depth Presets

| Parameter | quick | standard | deep |
|-----------|-------|----------|------|
| Agent count | 2-3 | 3-5 | 5-8 |
| Research iterations | 1 | 3 | 10 |
| Merge iterations | 1 | 2 | 3 |
| Quality threshold | 30/50 | 35/50 | 40/50 |
| Min search keywords | 3 | 5 | 10 |

Example: for deep research on a complex topic:
```bash
export RESEARCH_DEPTH=deep
export RESEARCH_MAX_ROUNDS=10
# Then start Claude Code and run /research <topic>
```

### Quality Scoring

The reviewer evaluates across 5 dimensions (10 points each, 50 total):

| Dimension | What it measures |
|-----------|-----------------|
| Factual Accuracy | Claims backed by sources, data correctness, no contradictions |
| Source Coverage | Multi-perspective, authoritative sources, no major gaps |
| Analysis Depth | Causal analysis beyond listing, original insights, logical rigor |
| Internal Consistency | Cross-agent data alignment, self-consistent reasoning |
| Completeness | Coverage vs. the research brief, key questions answered |

### Output Structure

Each research run creates a self-contained directory:

```
{topic-slug}/
├── config.md                    # Actual config used for this run
├── brief.md                     # Research brief with roles & questions
├── round-1/
│   ├── agent-{role-slug}.md     # Individual agent outputs
│   └── review.md               # Quality review with scores
├── round-2/                     # If iteration needed
│   └── ...
├── merged/
│   ├── draft-1.md               # Merged report
│   └── review-1.md              # Merge review
└── output/
    └── report.md                # Final report
```

---

## Real-World Example: AI Agent Industry Research

The [`examples/ai-agent-industry-2025/`](./examples/ai-agent-industry-2025/) directory contains the complete output from an actual research run on **"AI Agent 行业现状"** (AI Agent Industry Status).

### Run Summary

| Metric | Value |
|--------|-------|
| Topic | AI Agent 行业现状 (2025) |
| Depth | standard |
| Agents | 4 (market analyst, tech trends, competitors, app scenarios) |
| CLI tools used | Codex (2 agents) + Gemini (2 agents) |
| Research rounds | 1 (passed on first round) |
| Merge rounds | 1 (passed on first round) |
| Round 1 review score | **37/50** (threshold: 35) |
| Merged report score | **39/50** |
| Final report size | ~33KB (~8,000 words) |

### Dynamic Role Assignment

The orchestrator analyzed "AI Agent 行业现状" and generated 4 research roles:

| Role | Slug | Focus | CLI Tool |
|------|------|-------|----------|
| 市场规模分析师 | `market-analyst` | Market size, growth, funding, value chain | Codex |
| 技术趋势分析师 | `tech-trend` | Architecture, frameworks, breakthroughs | Gemini |
| 竞品格局分析师 | `competitor-analyst` | Products, big tech strategy, startups | Codex |
| 应用场景分析师 | `app-scenario` | Use cases, ROI, commercialization | Gemini |

### Key Findings from the Report

1. **Coding Agents achieved product-market fit first** — Claude Code, GitHub Copilot, and Cursor lead the coding agent market with 70%+ combined share, the clearest PMF in any agent sub-sector.

2. **Massive "intention-action gap" in enterprise adoption** — 82-85% of enterprises express interest, but only ~2% have scaled deployments (Capgemini). The report systematically reconciles 9 conflicting adoption data points across different survey methodologies.

3. **Protocol standardization is 2025's biggest infrastructure breakthrough** — Anthropic's MCP (agent-to-tool) and Google's A2A (agent-to-agent) protocols, now under Linux Foundation AAIF governance.

4. **"Agent Washing" is rampant** — Only ~130 of thousands of vendors offer genuine autonomous agent capabilities. Gartner predicts 40%+ of agent projects will be cancelled by end of 2027.

5. **Capital concentrated at the top** — 2024 global AI funding exceeded $100B, but four mega-deals accounted for ~$40B.

### Quality Review Highlights

The reviewer identified 4 data contradictions across agents and required the merger to resolve them:

| Issue | Resolution |
|-------|-----------|
| Enterprise deployment rate contradictions (2% vs 11% vs 57% vs 82%) | Created a systematic breakdown table by survey methodology, sample, and definition scope |
| Anthropic revenue $2.2B vs Claude Code single-product $2.5B paradox | Transparently flagged as "pending verification" with timeline explanation |
| "China financial agent 800B CNY" suspect data | Deleted with clear reasoning (sub-sector cannot exceed total industry) |
| 2025 AI funding $89.4B vs $150B discrepancy | Explained as VC-only vs all-source funding scope difference |

### File Listing

```
examples/ai-agent-industry-2025/
├── config.md                              # Research configuration
├── brief.md                               # Research brief with 4 roles
├── round-1/
│   ├── agent-market-analyst.md      (14K) # Market size analysis
│   ├── agent-tech-trend.md          (18K) # Technology trends analysis
│   ├── agent-competitor-analyst.md  (19K) # Competitive landscape analysis
│   ├── agent-app-scenario.md        (16K) # Application scenarios analysis
│   └── review.md                    (11K) # Quality review: 37/50 PASS
├── merged/
│   ├── draft-1.md                   (33K) # Merged report
│   └── review-1.md                   (9K) # Merge review: 39/50 PASS
└── output/
    └── report.md                    (33K) # Final report
```

---

## Architecture

### Agent Communication Model

Agents communicate exclusively through Markdown files — no shared memory or direct messaging between research agents. This design choice enables:

- **Reliability**: File I/O is deterministic; no context window overflow
- **Cross-CLI compatibility**: Codex and Gemini CLI can read/write the same files
- **Audit trail**: Every intermediate step is preserved
- **Iteration without state**: New agents in each round read previous outputs + feedback

```
Round 1:
  Agent A writes → round-1/agent-a.md
  Agent B writes → round-1/agent-b.md
  Reviewer reads all → writes round-1/review.md (FAIL)

Round 2:
  New Agent A reads round-1/agent-a.md + round-1/review.md → writes round-2/agent-a.md
  New Agent B reads round-1/agent-b.md + round-1/review.md → writes round-2/agent-b.md
  Reviewer reads all → writes round-2/review.md (PASS)
```

### CLI Invocation Protocol

Sub-agents invoke Codex/Gemini CLI with a degradation retry chain:

```
Codex:  xhigh → high → medium → low → Claude fallback
Gemini: full prompt → simplified → reduced dimensions → Claude fallback
```

- Timeout: 600,000ms (10 min) per CLI call
- Temp files for isolation: `mktemp /tmp/research-XXXXXX.txt`
- CLI unavailable: skip gracefully (research continues with WebSearch only)

### Quality Gate

```
                    ┌─────────────────┐
                    │ Reviewer scores  │
                    │ 5 dimensions     │
                    │ (10 pts each)    │
                    └────────┬────────┘
                             │
                   ┌─────────▼─────────┐
                   │ Total ≥ threshold  │──No──┐
                   │ AND each dim ≥ min │      │
                   └─────────┬─────────┘      │
                             │               │
                            Yes         ┌────▼─────────┐
                             │         │ Round < max?  │
                   ┌─────────▼──────┐  └────┬──────────┘
                   │ Proceed to     │      │Yes    │No
                   │ next stage     │      │       │
                   └────────────────┘ ┌────▼────┐ ┌▼──────────┐
                                      │Re-run   │ │Ask user   │
                                      │research │ │to decide  │
                                      │with     │ └───────────┘
                                      │feedback │
                                      └─────────┘
```

---

## Extending

### Adding New Depth Presets

Edit the depth preset table in `SKILL.md` to add custom profiles (e.g., `exhaustive` with 20 rounds).

### Customizing Research Output Templates

Modify the output template in the "研究 Agent Prompt" section of `SKILL.md`. All agents will adopt the new format.

### Using Without CLI Tools

Set `RESEARCH_CLI_TOOLS=none` or simply don't install Codex/Gemini. The skill works entirely with Claude Code's WebSearch.

---

## License

MIT
