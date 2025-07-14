# Planning Directory

This folder hosts epics, stories and tasks. To generate a web-based Kanban board, install `backlog.md` and run:

```bash
bun x backlog init .planning --out .planning/kanban
```

The command scans all Markdown files in the `epics/`, `stories/` and `tasks/` folders and outputs static board data under `./kanban`.
