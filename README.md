# Self‑Tune

> **Personal‑data ingestion & fine‑tuning platform - run locally, extend infinitely**

<!-- ![CI](https://github.com/researchwiseai/self-tune/actions/workflows/ci.yml/badge.svg) -->
[![License: Apache‑2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)

---

## ✨ Key Features

| Area | Highlights |
|------|------------|
| **Local‑first** | Works entirely on your laptop with **Bun 1.2+**, **Temporalite**, and optional GPUs. |
| **Pluggable Connectors** | RSS, YouTube, LinkedIn, Git and more—drop‑in modules based on the Airbyte spec. |
| **Deterministic Pipelines** | Temporal workflows + Nx hashing for resumable, cache‑friendly runs. |
| **Multi‑runtime Kernels** | Swap heavy lifting between Python, Rust/Wasmtime or AssemblyScript WebAssembly. |
| **Fine‑tuning Hub** | Train via OpenAI, local Hugging Face PEFT, or your own Ollama server—all share one manifest. |
| **Human‑friendly UX** | Plain‑English progress, actionable errors, visual dashboards & markdown Kanban board. |

---

## 🚀 Quick Start

```bash
# 1. Clone repo
$ git clone https://github.com/your‑org/self-tune && cd self-tune

# 2. Install deps (Bun will infer)
$ bun install

# 3. Bootstrap Temporalite & Postgres (Docker Compose)
$ bun run dev:stack

# 4. Run first ingestion demo (example blog RSS)
$ bun cli ingest rss https://yourblog.com/rss.xml

# 5. Fine‑tune with OpenAI GPT‑3.5
$ bun cli train openai --model gpt-3.5-turbo --dataset datasets/demo.jsonl
```

> Need help? `bun cli --help` prints every command with examples.

---

## 🏗️ Architecture

```
           +-----------+       +---------------+
           | Connectors | ---> |  Preprocess   | --+-->  Vector Store
           +-----------+       +---------------+   |
                                                   |
 Git / RSS / Video / Audio               +---------v--------+
                                         |  Fine‑tune Jobs  |
                                         +---------+--------+
                                                   |
                                          Model Registry (HF, OpenAI, Ollama)
```

Workflows are orchestrated by **Temporal** (or Temporalite locally) while Nx handles computation‑caching and project graph.

---

## 📄 Documentation

Full docs—including API reference, tutorials and design decisions—live on **GitHub Pages**:

> <https://your‑org.github.io/self-tune>

The detailed engineering roadmap is in [`development-plan.md`](./development-plan.md).

---

## 🛠️ Project Structure

```
apps/
  web/            # Dashboard (Vite + Bun)
packages/
  cli/            # Oclif‑powered CLI
  ingestors/      # Source connectors
  preprocessors/  # Text, audio, video transforms
  workers/py/     # Python ML tasks (Whisper, PEFT)
  wasm/           # Optional WebAssembly kernels
.planning/        # Epics, stories, tasks, Kanban board
```

---

## 🤝 Contributing

We welcome PRs big & small!  Start by reading the [contrib guide](CONTRIBUTING.md) and picking an open Story in `.planning/stories/`.

* Run `bun test` before committing.
* Follow Conventional Commits (`feat:`, `fix:`...).
* All code is linted & formatted via ESLint + Prettier.
* Generate the Kanban board with `bun x backlog init .planning --out .planning/kanban`.

---

## 🔐 License

Self‑Tune is **Apache‑2.0** licensed.  See [LICENSE](./LICENSE) for details.

---

© 2025 Self‑Tune Contributors
