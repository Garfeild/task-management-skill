# CLAUDE.md snippet — Task Manager

Paste the block below into your project's `CLAUDE.md`. It handles the automatic
session-start behavior. Explicit task operations go through the `/task` skill.

---

```markdown
## Task Management

Tasks are tracked in `.tasks/` using the [claude-task-manager](https://github.com/YOUR_USERNAME/claude-task-manager) skill.

Install: copy `commands/task.md` → `.claude/commands/task.md` in this repo.

### Starting a session

When the user says "let's continue", "what's next", or similar:

1. Read `.tasks/IN_PROGRESS.md`
2. If a task is listed — summarize its status and resume work
3. If empty — read `.tasks/BACKLOG.md` and ask whether to start the top task

Use `/task` for all task operations (start, complete, new, log).
```
