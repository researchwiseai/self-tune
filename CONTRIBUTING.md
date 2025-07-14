# Contributing

Thank you for helping improve Self-Tune! This project uses **Bun** and **Nx**, so getting up and running is quick.

## Install dependencies

1. Install [Bun](https://bun.sh) `1.2+` and ensure `bun --version` prints a recent release.
2. Clone the repo and run:

   ```bash
   bun install
   ```

This will download all workspace packages defined in `package.json`.

## Run the test suite

Before opening a pull request, run the tests:

```bash
bun run test
```

`bun run test` invokes the Nx test target (powered by Vitest). If no tests exist, the command exits with a non‑zero code.

## Commit conventions

All commits should follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/). Typical prefixes are:

- `feat:` for new features
- `fix:` for bug fixes
- `docs:` for documentation only

This format keeps the changelog tidy and allows automated versioning.

## Automated agent workflows

Automated contributions run by agents follow additional rules. See [`AGENTS.md`](./AGENTS.md) for details on version checks, linting and CI steps.
