# CLAUDE.md

## 项目概述

多智能体协同行业调研 Skill，支持 Claude Code 和 OpenClaw 两个平台。

- `SKILL.md` — Claude Code 版
- `openclaw/SKILL.md` — OpenClaw 版（部署在 SSH2 上）
- `openclaw/references/` — OpenClaw 版引用文件

## 发布流程

**每次修改 skill 文件后，必须同步三个位置：**

1. **提交并推送 GitHub**
   ```bash
   git add SKILL.md openclaw/SKILL.md
   git commit -m "描述改动"
   git push origin main
   ```

2. **发布到 ClawHub**（OpenClaw 版）
   ```bash
   clawhub login --token clh_uIyIOvdDr9vWKFjJReb6XY7vbDba2fQ-j03hPjAdmJo
   clawhub publish openclaw/ --slug multi-agent-industry-research --version <新版本号> --changelog "改动说明"
   ```
   版本号遵循 semver：bug fix +0.0.1，新功能 +0.1.0，大改 +1.0.0。当前版本：**1.2.0**。

3. **SSH2 上的 OpenClaw 实例**
   - 路径：`/home/shine/.openclaw/workspace/skills/multi-agent-industry-research/`
   - 如果是直接在 SSH2 上改的，需要把改动拉回本地仓库再走步骤 1-2
   - 如果是本地改的，可通过 `clawhub update` 或手动 scp 同步到 SSH2

## SSH2 OpenClaw 运维备忘

- 调研产出目录：`/home/shine/.openclaw/workspace/`
- 子 agent 运行记录：`/home/shine/.openclaw/subagents/runs.json`
- 卡死的 run 特征：有 `startedAt` 但无 `endedAt`，需手动补上 `endedAt` + `outcome.status: "timeout"` + `cleanupHandled: true`
- OpenClaw 并发子 session 上限约 7-8 个（平台限制，非 skill 配置）
- `/model` 命令只影响主 session，子 agent 模型通过 skill 里的 `inheritedModel` 机制继承
