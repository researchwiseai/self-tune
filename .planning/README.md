# Planning Directory

This folder hosts epics, stories and tasks. The `backlog/config.yml` file sets `.planning` as the Backlog.md project root. To generate a web-based Kanban board, install `backlog.md` and run:

```bash
bun x backlog init .planning
```

Backlog.md will prompt for options like auto commit and remote operations. You can
press **Enter** at each prompt to accept the recommended defaults, which match the
versioned `backlog/config.yml` in the repo.

The command then scans all Markdown files in the `epics/`, `stories/` and
`tasks/` folders and outputs static board data under `./kanban`.
