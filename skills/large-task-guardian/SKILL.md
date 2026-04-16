---
name: large-task-guardian
description: "ALWAYS invoke this skill BEFORE starting any task when your own assessment determines the task is MEDIUM or HIGH complexity — do not wait for the user to ask. Trigger criteria (any one is enough): touching >10 files, bash commands likely >30s, spawning sub-agents for multi-step work, full codebase refactors, installs/builds/migrations, or any task where you think 'this might take a while'. Purpose: prevent 'API Error: Stream idle timeout - partial response received' by proactively decomposing work and maintaining stream heartbeats."
---

# Large Task Guardian Skill

Prevent `Stream idle timeout - partial response received` errors by proactively analyzing task size, decomposing work into safe chunks, and maintaining stream heartbeats during execution.

## When This Skill Applies

Activate this skill when the task has ANY of these risk factors:
- Touching more than 10 files
- Running bash commands that may take >30s (builds, installs, tests, migrations)
- Spawning sub-agents for complex multi-step work
- Processing large datasets or files (>1MB)
- Refactoring across many modules
- Tasks the user describes as "large", "complex", "everything", or "all of X"

## Workflow

Make a todo list for all tasks in this workflow, and work through them one by one.

### 1. Risk Assessment

Before starting any work, evaluate the task:

**Estimate idle risk:**
- LOW: Single file edit, simple query, <5 tool calls expected
- MEDIUM: 5–20 tool calls, one long bash command, single sub-agent
- HIGH: 20+ tool calls, multiple sub-agents, long-running processes, full codebase ops

If risk is LOW, proceed normally without this skill's overhead.

If risk is MEDIUM or HIGH, continue with steps below.

**Announce the plan:**
```
Task complexity: [MEDIUM/HIGH]
Estimated tool calls: ~N
Decomposition strategy: [brief description]
```

### 2. Decompose Into Chunks

Break the work into phases where each phase:
- Completes a meaningful, verifiable unit of work
- Takes at most ~60 seconds of silent processing
- Ends with visible output (a status line, a count, a result)

**Example decomposition patterns:**

| Large Task | Chunked Version |
|---|---|
| "Edit all 50 files" | Batch by directory, 5–10 files per batch |
| "Run full test suite" | Run one test file at a time, report per file |
| "Install + build + test" | Separate tool calls with status between each |
| "Refactor module X" | Read → plan → edit file-by-file → verify |
| "Migrate database" | Schema changes → data migration → validation |

### 3. Heartbeat Protocol During Execution

For any operation that could run silently for >20 seconds, apply the heartbeat protocol:

**For Bash commands:**
- Prefer commands that stream output (`--progress`, `-v`, line-buffered)
- Add explicit progress markers:
```bash
echo "=== Phase 1/3: Installing dependencies ===" && npm install && \
echo "=== Phase 2/3: Building ===" && npm run build && \
echo "=== Phase 3/3: Running tests ===" && npm test
```
- For long silent operations, break into smaller sequential calls with status output between them
- Use `run_in_background: true` only for genuinely parallel work; poll with status messages

**For sub-agent calls (Agent tool):**
- Add to the prompt: "After every major step, output a one-line status update like '✓ Step N done: [what was done]'"
- Structure the prompt with explicit numbered phases so the agent outputs progress
- Prefer `run_in_background: false` (foreground) for long agents so you receive incremental updates

**For file edits:**
- Process files in batches of 5–10
- After each batch, output: `✓ Edited N/Total files`

### 4. Between-Chunk Status Updates

After completing each chunk, output a status line before starting the next:

```
✓ [Phase name] complete — [brief result]. Starting [next phase]...
```

This keeps the stream active AND gives the user visibility.

### 5. Sub-agent Monitoring

When spawning agents with the Agent tool for complex work:

**Structure the agent prompt to include:**
```
IMPORTANT: This task may take a while. To prevent stream timeouts:
1. Output a one-line progress update after each major step
2. Break your work into phases and announce each phase start
3. Never go more than 20 seconds without producing output
```

**After the agent returns:**
- Verify the output is complete (check for truncation signals)
- If the result seems partial, re-run with a more targeted prompt on the remaining work

### 6. Recovery When Timeout Occurs

If a timeout does occur mid-task:

1. **Assess state**: What completed? What was in-progress? What is pending?
2. **Output a recovery plan**:
```
Recovery status:
- Completed: [list]
- Interrupted: [what was running]
- Remaining: [list]
Resuming from: [specific point]
```
3. Resume from the exact interruption point — do not restart from scratch
4. Use smaller chunks than the previous attempt

### 7. Wrap Up

After all chunks complete:
- Summarize what was done across all phases
- Note any phases that were split due to complexity
- Confirm the work is complete and consistent

## Quick Reference: Safe Command Patterns

```bash
# BAD: silent, potentially long
npm install

# GOOD: with progress visibility  
npm install --progress 2>&1 | tail -5

# BAD: one giant find+replace
find . -name "*.py" -exec sed -i 's/old/new/g' {} \;

# GOOD: batched with progress
find . -name "*.py" | head -10 | xargs sed -i 's/old/new/g' && echo "Batch 1/5 done"

# BAD: full test suite in one call
pytest

# GOOD: one module at a time
pytest tests/module1/ -v && echo "module1 done" && pytest tests/module2/ -v && echo "module2 done"
```

## Signals That a Task Needs This Skill

The user says things like:
- "重构整个..."  / "Refactor the entire..."
- "更新所有..."  / "Update all..."
- "迁移..."       / "Migrate..."
- "安装并配置..." / "Install and configure..."
- "运行完整..."  / "Run the full..."
- Any task where you think "this might take a while"
