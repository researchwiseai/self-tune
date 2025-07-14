Below is the **updated, Bun‑first development plan**.  Major changes from the previous draft:

- Switched the monorepo to **Bun** as the package‑manager/runtime (Nx 19.1+ supports `--packageManager bun`).
- Added **Vitest 3** for TypeScript tests.
- Clarified **GitHub Actions** CI matrix (Node 22 LTS & Bun 1.2) and automatic docs deployment to **GitHub Pages**.
- Introduced an optional **WebAssembly kernel pathway** (Rust/C & AssemblyScript) alongside Python workers.
- Embedded strong UX & validation principles (progress bars, hashed resume, plain‑English errors).
- Recorded the *latest stable versions* of Node.js, Bun, Python, Vitest and the GitHub Actions runner.

---

## 🔑 Quick‑Glance Version Matrix  *(July 14 2025)*

| Tool                      | Current Stable / LTS    | Notes                                         |
| ------------------------- | ----------------------- | --------------------------------------------- |
| **Node.js**               | **22.17.0 (LTS “Jod”)** | Active LTS until Oct 2025, maint. to Apr 2027 |
| **Bun**                   | **1.2.18**              | Released 3 July 2025                          |
| **Nx**                    | **19.6**                | Bun support landed in 19.1                    |
| **Python**                | **3.12.11**             | Latest maintenance release, June 2025         |
| **Vitest**                | **3.0.0**               | Stable since Jan 2025 (4.x in beta)           |
| **GitHub Actions runner** | **2.326.0**             | Latest as of 10 July 2025                     |

---

## 0  Project Foundation

### 0.1 Repo & Tooling

- **Nx monorepo** – initialise with `bunx create-nx-workspace@latest myspace --packageManager bun`.
- **Bun** is our default JS runtime, task‑runner & package‑manager; Node 22 LTS remains in the tool‑chain for polyglot tasks (via `engines` field).
- Directory layout unchanged (`ingestors/`, `preprocessors/`, `workers/py/`, etc.).
- **Vitest** replaces Jest – configure via `vitest.config.ts`, plus `@nx/vite` executor.
- Source formatting: `eslint`, `prettier`, strict `tsconfig` shared across apps.
- Conventional Commits + semantic‑release.

### 0.2 Hash‑based Incrementalism

- **Nx computation hashing** guarantees that tasks only re‑run when their *input graph* changes; local & remote cache (optional Nx Cloud/S3).
- Each pipeline stage writes artefacts to deterministic paths keyed by SHA‑256 of the inputs (mirrors Temporal’s idempotency guarantees).

---

## 1  Orchestration Layer (TypeScript)

- **Temporal Server** (Apache‑2.0) via Docker compose; for local demos and CI we ship a single‑binary **Temporalite**.
- Workflows in TypeScript using the Temporal TS SDK.
- Activity workers can be Node 22 + Bun runtime, or Python via gRPC bridge.

---

## 2  Connector Framework

- Based on the **Airbyte Connector Spec** but implemented in TypeScript.
- Command helpers (`bun run connector:dev`) hot‑reload via Bun’s ultrafast watcher.
- Starter connectors: RSS (WordPress/Substack), YouTube (captions -> text), LinkedIn (OAuth 2 `/ugcPosts`), Git (shallow clone).
- **Singer‑tap adapter** for re‑using existing community taps.

---

## 3  Pre‑processing & Dataset Builder

- **Media → Text**: FFmpeg strip‑audio → Whisper transcription.
- Chunk & enrich via LangChain splitters.
- Optional PII scrub (Presidio) and vector‑store write (`pgvector`/`Qdrant`).
- Artefact folder structure remains deterministic (`datasets/uuid.jsonl`).

---

## 4  Training Sub‑system

- Uniform `training.yaml` manifest (model family, LoRA/QLoRA, epochs, dataset URI).
- Back‑ends: Hugging Face PEFT (local GPU), OpenAI jobs (hosted).
- Outputs: `models/<provider>/<run‑id>` + evaluation report.

---

## 5  WebAssembly & Native Kernels *(New)*

To support CPU‑bound feature extraction or model kernels that aren’t feasible in Python:

1. **AssemblyScript** – compile strict‑typed TypeScript to `.wasm` for seamless JS interop.
2. **Rust** – contributors can ship `wasm-pack build --target bun` modules; Bun’s WASI‑like runtime can load them natively.
3. **C/C++** – via `clang --target=wasm32-wasi`.  Provide simple generator `nx generate wasm-lib <name>` scaffolding boilerplate.

The CLI auto‑detects `.wasm` plugins in `plugins/wasm/` and registers them as Tasks.

---

## 6  User‑Experience & Validation *(New)*

- **Plain‑English progress** – each Temporal Activity streams updates; CLI prints spinners + human‑friendly advice.
- **Front‑loaded validation** – connectors validate credentials, file paths & quotas before enqueueing a Workflow.
- **Checkpoint resume** – hashes of source inputs & intermediate artefacts allow skipping already‑completed steps.
- **Automatic issue suggestions** – common errors mapped to FAQ links.

---

## 6A  Agentic AI Execution Guidance *(New)*

> These rules are written for autonomous or semi‑autonomous agents (LLMs, tool‑calling chains, or LangGraph flows) that will actually **run the workflows** defined above. Embed them verbatim in every agent prompt so that generated code stays current and maintainable.

### 🔄 Always Verify the Latest Versions

1. **Before starting any Activity or writing code**, the agent **MUST** call one of the following to fetch the latest stable version and compare with the version matrix:
   - `curl https://api.github.com/repos/oven-sh/bun/releases/latest`  → `tag_name` ✅
   - `node -p "require('node:process').versions.node"` ✅
   - `python - <<'PY' import sys, json, urllib.request, re, ssl; print(json.load(urllib.request.urlopen('https://pypi.org/pypi/pip/json'))['info']['version']) PY` ✅
   - `gh release view --json tagName --repo nx/devkit` ✅
   - For any NPM package: `bunx npm:latest <pkg>` or `npm view <pkg> version` ✅
2. If the fetched version **differs** from the Quick‑Glance table *or* the code sample about to be generated, the agent **MUST** adjust the code snippet accordingly **and** emit a note in the progress log: “🔄 Auto‑updated to Bun 1.2.19 (released 2025‑07‑12)”.
3. Prefer **LTS or Stable tags**; avoid experimental/nightly unless explicitly requested.

### 📚 Consult Official Documentation First

- Query docs at their canonical URLs (`docs.bun.sh`, `docs.nx.dev`, `docs.temporal.io`, `docs.python.org/3.12`, etc.) **before** relying on memory.  Use the first‑party example closest to the task.
- If an API has been renamed or deprecated, the agent **MUST** substitute the new call and leave an inline comment citing the change.
- Never copy code that contains deprecation warnings or references to EOL versions (e.g., Node 14).

### 🛠️ Self‑Test & Lint Before Commit

1. Every generated TypeScript file **MUST** compile via `bunx tsc --noEmit`.  If not, fix before proceeding.
2. Run `bun vitest run` for the affected package.  Use failing tests as a signal to refine the snippet.
3. Lint with `bunx eslint . --max-warnings 0` and auto‑fix where possible.

### 📈 Emit Plain‑English Feedback

- Stream progress in the format `step‑name  ✔/✖  (<duration>) – <hint>`.
- On error, supply **one‑paragraph practical advice** (e.g., “Looks like Bun can’t find `@types/node`.  Run `bun add -D @types/node@latest` and retry.”).

### ♻️ Idempotent & Incremental

- Leverage **Nx computation hashing** and Temporal `|ContinueAsNew|` patterns.  If the SHA‑256 of inputs hasn’t changed, skip the Activity and emit “(cached)”.
- Use deterministic artefact paths so that reruns pick up where they left off.

### 🔐 Security & Supply‑Chain Hygiene

- Verify downloaded binaries via SHA‑256 or Sigstore when available.
- Pin versions in `bun.lockb` and `requirements.txt`.
- Generate SBOM via `nx audit` each run.

---

## 7  CLI & Minimal Web UI

- **CLI** (`packages/cli`) built with Oclif; uses Bun runtime.
- UI dashboard (`apps/web`) shows dataset graphs & workflow timelines via Temporal gRPC‑Web.

---

## 8  CI/CD & Docs

- **GitHub Actions**
  - Matrix: `node:22-bun`, `os: ubuntu-latest, macos-latest`.
  - Use **Vitest** for unit & integration tests (`bun run test`).
  - Temporalite spun up in a service container.
  - Cache restoration via Nx’s remote cache.
- **Docs** generated with Docusaurus and deployed via `actions/deploy-pages` to **GitHub Pages**.

---

## 9  Security & Compliance

- Tokens encrypted with AWS KMS or `age` locally.
- Provenance: SHA‑256 signature chain from raw input → chunk → dataset → model.
- SBOM (`nx audit`) attached to every release artefact.

---

## 10  Governance & Community

- Apache‑2.0 everything.
- Separate repo per connector; CLI registry JSON.
- Monthly minor releases; weekly canary.

---

## 11  Phase Timeline

| Weeks | Milestone                                  |
| ----- | ------------------------------------------ |
| 1‑2   | Repo scaffolding; Nx+Bun; Temporal scripts |
| 3‑4   | Vitest baseline; GitHub Actions CI + Pages |
| 5‑6   | Core Connector interface + RSS/YouTube     |
| 7‑8   | Pre‑processing worker (FFmpeg + Whisper)   |
| 9‑10  | Dataset builder & vector store             |
| 11‑12 | Training back‑ends (PEFT & OpenAI)         |
| 13‑14 | WebAssembly plug‑in scaffold               |
| 15‑16 | End‑to‑end demo & polishing                |

---

## 12  Epics, Stories & Continuous Improvement *(New)*

This section formalises **agile planning, progress tracking and continuous feedback** for both humans and agentic AIs working on the project.

### 12.1 Directory Structure

```
.planning/
  epics/              # high‑level goals (3‑6 weeks)
  stories/            # user‑facing slices (3‑5 days)
  tasks/              # atomic work units (<1 day)
  retrospectives/     # sprint review docs
  progress/           # auto‑generated KPI & burn‑down reports
```

All files live in Markdown for easy diffing and PR review; GitHub renders them nicely.

### 12.2 Epic Guidance

- **Definition** – a significant theme delivering user value (e.g., *“Self‑Hosted Web UI”*).
- **Duration** – normally spans **2‑3 sprints** (≈3‑6 weeks).
- **Breakdown** – sliced into **stories** that adhere to the **INVEST** criteria (Independent, Negotiable, Valuable, Estimable, Small, Testable).
- **Template** (`epics/<id>-<slug>.md`):
  ```md
  ---
  id: EP-2025-07
  name: Self-Hosted Web UI
  owner: @will
  target-date: 2025-09-01
  kpi: "Daily active self-hosted users"
  ---
  ## Objective
  Build a local-first dashboard so users can monitor workflows without cloud services.

  ## Narrative
  <persona> as an <actor>, I want <capability> so that <benefit>.

  ## Acceptance Criteria
  - [ ] User can see list of workflows
  - [ ] Error logs stream live
  - [ ] Docs deployed via GitHub Pages

  ## Linked Stories
  - STY-042
  - STY-047
  ```

### 12.3 Story Guidance

- **Scope** – delivers a slice of user value in **≤5 dev‑days**.
- **Template** (`stories/STY-042-<slug>.md`):
  ```md
  ---
  id: STY-042
  epic: EP-2025-07
  points: 5
  ---
  ## Story
  As a hobbyist, I can start the Web UI with a single `bun cli serve` so that I don’t need Docker.

  ## Tasks
  - [ ] TSK-221 Scaffold Vite SPA
  - [ ] TSK-222 Add Temporal gRPC-Web client
  ```

### 12.4 Task Guidance

- **Scope** – should fit in **one working day** or be further decomposed.
- **Template** (`tasks/TSK-221-<slug>.md`):
  ```md
  ---
  id: TSK-221
  story: STY-042
  est-hours: 6
  ---
  ## Task
  Generate a Vite + Bun template under `apps/web`.

  ### Definition of Done
  - `bun vitest` passes
  - CI workflow succeeds
  ```

### 12.5 Sprint Cadence & Retrospectives

- **Sprint length** – **2 weeks** (Mon → Fri next week).  Sprint 0 is project setup.
- End of sprint:
  1. Demo shippable increments.
  2. Automated **progress report** is generated by a GitHub Action that collates:
     - Closed Stories / Tasks (`gh api` search)
     - Nx affected graph stats (LOC, bundle size)
     - Temporal workflow counts & success rate
  3. **Retrospective** doc saved to `retrospectives/2025-08-01-sprint-01.md` with *What went well*, *What didn’t*, *Action items*.

### 12.6 Agentic AI Checklist for Planning

1. **Validate Story size**: refuse to create a story >5 days; split automatically.
2. **Linkage enforcement**: every Task must reference a Story; every Story an Epic.
3. **Auto‑generate IDs** using a monotonic counter stored in `.planning/ids.json`.
4. **Review latest docs** before creating new files to avoid duplicates.

### 12.7 Automated Progress Reports

- Nightly GitHub Action runs `scripts/report.ts` which:
  - Queries GitHub Issues & PRs for `state:closed label:story`
  - Pulls Temporal metrics via API
  - Generates markdown under `progress/2025-08-report.md` and commits.
- Dashboards visualised in the Web UI using Recharts.

---

## 13  Markdown Task Management Tooling *(New)*

> This toolkit turns the `.planning/` directory into a living, AI‑friendly backlog while remaining pleasant for humans to read in GitHub.

| Need | Selected Tool | Why |
|------|---------------|-----|
| **CLI + Web Board** | **Backlog.md** | Single command (`bunx backlog init`) transforms Markdown issues into a Kanban board with drag‑and‑drop UI and JSON export, ideal for AIs to both read and write fileciteturn1file0L9-L11 |
| **Lightweight Static Board** | **KanbanBoardInMarkdown** | Zero‑config generator renders README‑style task lists into an HTML Kanban board, perfect for GitHub Pages mirrors fileciteturn1file0L12-L13 |
| **IDE Board View** | **markdown‑kanban** (VS Code) | Lets devs view/update tasks in‑editor; two‑way sync with Markdown source fileciteturn1file0L15-L16 |
| **Task Parsing & Metrics** | **taskparser** | Parses todos from Markdown into JSON/CSV for nightly progress reports fileciteturn1file0L25-L26 |
| **Deep Analysis** | **mrkdwn_analysis** | AI can run this Python lib to extract headings/lists and compute completion % fileciteturn1file3L1-L2 |
| **Story Mapping** | **Featmap** | Optional story‑map UI for stakeholders who prefer a big‑picture view fileciteturn1file0L37-L38 |
| **Visual Charts** | **Mermaid & mkdocs‑charts‑plugin** | Embed burndown charts directly in Markdown and auto‑render in docs site fileciteturn1file0L24-L25 |

### 13.1 Folder Conventions

```
.planning/
  kanban/              # Backlog.md data cache
  charts/              # Mermaid or Vega‑Lite JSON specs
  metrics/             # JSON exports (taskparser)
```

- `backlog.md sync` keeps `kanban/` JSON in step with Epics/Stories/Tasks.
- GitHub Action `planning-report.yml` runs `taskparser` → outputs `metrics/burnup.json`, then commits a Mermaid burndown chart under `charts/`.

### 13.2 Agent Guidance for Tooling

1. **When creating/updating tasks**, prefer Backlog.md CLI (`backlog add --story STY‑042 "task title"`).
2. **Before modifying a Markdown file**, parse existing tasks via `taskparser` to avoid ID clashes.
3. **Generate Mermaid diagrams** only after `metrics/*.json` exists.
4. **If VS Code is detected**, suggest installing `markdown‑kanban` extension.

### 13.3 CI Integration

- `bunx backlog validate` runs in the lint stage; fails the build if any Story‑→‑Task linkage breaks.
- `taskparser --summary` outputs CSV consumed by the weekly progress Action.
- `mkdocs build` uses `mkdocs‑charts‑plugin` so burndown lines appear in the public docs site.

### 13.4 Human Viewing Experience

- Default README badge: `![Sprint Burn‑down](planning/charts/burndown.svg)`.
- GitHub Pages hosts the **KanbanBoardInMarkdown** board at `/board/` for quick stakeholder access.

---

### ✅ Outcome

A **Bun‑powered, Nx‑structured** stack that anyone can run locally with zero licences, boasting lightning‑fast tests (Vitest), deterministic caching, and first‑class UX.  The architecture leaves room for high‑performance WebAssembly kernels and scales upward via Temporal’s battle‑tested workflow engine.

