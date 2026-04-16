# Claude-Improve

A collection of Claude Code skills and configurations to improve AI-assisted development workflows.

---

## Skills

### `large-task-guardian`

**Purpose:** Prevent `API Error: Stream idle timeout - partial response received` errors that occur when Claude processes large or complex tasks silently for too long.

**How it works:**

1. **Risk Assessment** — Before starting any task, Claude evaluates complexity as LOW / MEDIUM / HIGH based on the number of files, expected command duration, and sub-agent usage.
2. **Task Decomposition** — MEDIUM/HIGH tasks are broken into chunks of ≤60 seconds each, ensuring visible output between phases.
3. **Heartbeat Protocol** — Long bash commands are structured with progress markers (`echo "=== Phase N/M ==="`). Sub-agents are instructed to output status lines every step.
4. **Sub-agent Monitoring** — Agent prompts include explicit instructions to report progress and never go >20 seconds without output.
5. **Timeout Recovery** — If a timeout does occur, Claude assesses state, outputs a recovery plan, and resumes from the exact interruption point.

**Trigger criteria (any one is enough):**
- Touching more than 10 files
- Bash commands likely to run >30 seconds
- Spawning sub-agents for multi-step work
- Full codebase refactors, installs, builds, or migrations

**Auto-trigger:** Enabled via `CLAUDE.md`. Claude invokes this skill automatically when it assesses a task as MEDIUM or HIGH risk — no manual `/large-task-guardian` needed.

**Installation:**

```bash
# Copy skill to your Claude skills directory
mkdir -p ~/.claude/skills/large-task-guardian
cp skills/large-task-guardian/SKILL.md ~/.claude/skills/large-task-guardian/
```

> **Auto-trigger setup:** On first use, the skill will detect whether the auto-trigger rule exists in your `~/.claude/CLAUDE.md` and ask if you want it added. It appends to your existing file — nothing is overwritten.

---

## Files

```
Claude-Improve/
├── CLAUDE.md                              # Global Claude instructions (auto-trigger rules)
└── skills/
    └── large-task-guardian/
        └── SKILL.md                       # Skill definition
```

---

---

# Claude-Improve（中文说明）

一套用于改善 AI 辅助开发工作流的 Claude Code 技能与配置集合。

---

## 技能

### `large-task-guardian`（大任务守护者）

**用途：** 防止在 Claude 处理大型或复杂任务时因长时间无输出而触发 `API Error: Stream idle timeout - partial response received` 错误。

**工作原理：**

1. **风险评估** — 在开始任何任务前，Claude 根据涉及文件数量、命令预计耗时、是否使用 sub-agent 等因素，将任务复杂度评为 LOW / MEDIUM / HIGH。
2. **任务拆分** — MEDIUM/HIGH 任务会被拆分为每块 ≤60 秒的安全单元，每阶段结束后输出可见状态。
3. **心跳协议** — 长时间运行的 bash 命令会加入进度标记（`echo "=== 阶段 N/M ==="`），确保 stream 保持活跃。
4. **Sub-agent 监督** — 向 agent 的 prompt 中注入指令，要求每步输出状态行，且不超过 20 秒无输出。
5. **超时恢复** — 若发生超时，Claude 会评估当前状态、输出恢复计划，并从中断点继续，而非从头重来。

**触发条件（满足任意一条即触发）：**
- 涉及超过 10 个文件
- bash 命令预计运行时间 >30 秒
- 为多步骤工作派生 sub-agent
- 全量代码库重构、依赖安装、构建或数据迁移

**自动触发：** 通过 `CLAUDE.md` 全局指令启用。Claude 在自行判断任务为 MEDIUM/HIGH 风险时会自动调用此技能，无需用户手动输入 `/large-task-guardian`。

**安装方式：**

```bash
# 将技能复制到 Claude 技能目录
mkdir -p ~/.claude/skills/large-task-guardian
cp skills/large-task-guardian/SKILL.md ~/.claude/skills/large-task-guardian/
```

> **自动触发配置：** 首次使用时，skill 会检测 `~/.claude/CLAUDE.md` 中是否已有自动触发规则，并询问是否添加。采用**追加**方式写入，不会覆盖你现有的任何配置。

---

## 文件结构

```
Claude-Improve/
├── CLAUDE.md                              # 全局 Claude 指令（自动触发规则）
└── skills/
    └── large-task-guardian/
        └── SKILL.md                       # 技能定义文件
```
