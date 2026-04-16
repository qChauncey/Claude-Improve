# Global Claude Instructions

## Large Task Guardian (Anti-Timeout Protocol)

Before starting ANY task, assess its complexity. If MEDIUM or HIGH risk, invoke the `large-task-guardian` skill immediately — do not wait for the user to ask.

**MEDIUM or HIGH risk means ANY of:**
- Touching >10 files
- Bash commands likely to run >30 seconds
- Spawning sub-agents for multi-step work
- Full codebase refactors, installs, builds, or migrations
- Any task where you think "this might take a while"

This is a BLOCKING requirement: invoke `large-task-guardian` BEFORE doing any work on such tasks.
