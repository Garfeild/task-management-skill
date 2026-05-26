# claude-task-manager

A lightweight task management skill for [Claude Code](https://claude.ai/code). Tasks live as Markdown files in your project — no external tools, no databases.

## Install

Copy the command file into your project (or globally):

```bash
# Project-scoped (recommended)
mkdir -p .claude/commands
cp path/to/claude-task-manager/commands/task.md .claude/commands/task.md

# Global — available in every project
mkdir -p ~/.claude/commands
cp path/to/claude-task-manager/commands/task.md ~/.claude/commands/task.md
```

Then paste the snippet from `claude-md-snippet.md` into your project's `CLAUDE.md` to enable the automatic session-start behavior.

## Usage

| Command | What it does |
|---------|-------------|
| `/task` | Show active task + next 3 backlog items |
| `/task backlog` | Full backlog overview grouped by priority |
| `/task next` | Start the top backlog item |
| `/task done` | Mark the active task complete |
| `/task wontdo TASK-003` | Dismiss a backlog task without doing it |
| `/task new Fix login bug` | Create a new task and add it to the backlog |
| `/task new PROJ-123 Fix login bug` | Same, with an external ticket reference |
| `/task log Implemented JWT validation` | Append a progress entry to the active task |
| `/task sub new Define data model` | Create a subtask under the active task |
| `/task sub next` | Start the next unstarted subtask |
| `/task sub done` | Complete the active subtask |
| `/task sub log Reviewed with team` | Append a progress entry to the active subtask |

On first use, `/task` automatically creates the `.tasks/` folder structure in your project root — nothing to set up manually.

## File structure

```
.tasks/
├── BACKLOG.md          ← priority-ordered queue
├── IN_PROGRESS.md      ← the single active task
├── DONE.md             ← completed tasks (newest at top)
└── tasks/
    ├── TASK-001.md
    ├── TASK-002.md
    └── ...
```

The index files (`BACKLOG`, `IN_PROGRESS`, `DONE`) are lightweight link tables. All detail lives in the individual task files under `tasks/`.

## Rules

- Only one task in progress at a time
- Task files are never deleted — entries move between index files
- Task IDs are sequential and zero-padded (TASK-001, TASK-002, ...)

## Add to .gitignore (optional)

If you prefer to keep tasks out of version control:

```
.tasks/
```

## Templates

The `templates/` folder contains the raw Markdown templates the skill uses to initialize new projects. You can customize them before copying.
