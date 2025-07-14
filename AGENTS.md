# Agentic AI Execution Guidance

These rules are written for autonomous or semi‑autonomous agents (LLMs, tool‑calling chains,
or LangGraph flows) that will actually **run the workflows** defined in this repo. Embed them
verbatim in every agent prompt so that generated code stays current and maintainable.

### 🔄 Always Verify the Latest Versions

1. **Before starting any Activity or writing code**, the agent **MUST** call one of the
   following to fetch the latest stable version and compare with the version matrix:
   - `curl https://api.github.com/repos/oven-sh/bun/releases/latest` → `tag_name` ✅
   - `node -p "require('node:process').versions.node"` ✅
   - 
     ```bash
     python - <<'PY'
     import sys, json, urllib.request, ssl
     print(
       json.load(
         urllib.request.urlopen(
           'https://pypi.org/pypi/pip/json',
           context=ssl.create_default_context()
         )
       )['info']['version']
     )
     PY
     ```
   - `gh release view --json tagName --repo nx/devkit` ✅
   - For any NPM package: `bunx npm:latest <pkg>` or `npm view <pkg> version` ✅
2. If the fetched version **differs** from the Quick‑Glance table or the code sample about
   to be generated, the agent **MUST** adjust the snippet accordingly **and** emit a note:
   “🔄 Auto‑updated to Bun 1.2.19 (released 2025‑07‑12)”.
3. Prefer **LTS or Stable tags**; avoid experimental/nightly unless explicitly requested.

### 📚 Consult Official Documentation First

- Query the canonical docs (`docs.bun.sh`, `docs.nx.dev`, `docs.temporal.io`,
  `docs.python.org/3.12`, etc.) **before** relying on memory.
- If an API is renamed or deprecated, the agent **MUST** substitute the new call and leave
  an inline comment citing the change.
- Never copy code containing deprecation warnings or referencing EOL versions.

### 🛠️ Self‑Test & Lint Before Commit

1. Ensure every generated TS file **MUST** compile via `bunx tsc --noEmit`.
2. Run `bun vitest run` on the affected package.
3. Lint with `bunx eslint . --max-warnings 0` and auto‑fix where possible.

### 📈 Emit Plain‑English Feedback

- Stream progress as `step-name  ✔/✖  (<duration>) – <hint>`.
- On error, supply **one‑paragraph practical advice**.

### ♻️ Idempotent & Incremental

- Use **Nx computation hashing** and Temporal `ContinueAsNew` patterns. If inputs SHA‑256
  hasn’t changed, skip the Activity and emit “(cached)”.
- Use deterministic artifact paths for seamless reruns.

### 🔐 Security & Supply‑Chain Hygiene

- Verify binaries via SHA‑256 or Sigstore when available.
- Pin versions in `bun.lockb` and `requirements.txt`.
- Generate SBOM via `nx audit` each run.
