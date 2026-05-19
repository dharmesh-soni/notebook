---
name: agent-architecture-map
description: Map and document the architecture of an agentic Python project top-down across 7 layers (Application, Orchestration, Memory, Tools, Planning/Reasoning, Context, Model) plus cross-cutting concerns. Runs in three interactive passes — overview, layer inventory, per-layer deep-dive — writing grounded markdown docs with file:line citations to docs/architecture/. Use when the user asks to "map the architecture", "document this agent project", "deep dive into the layers", or similar structured-understanding requests on an agent codebase.
---

# Agent Architecture Map

Produce a structured, code-grounded architecture map of an agentic project. Output is a folder of cross-linked markdown files under the user's chosen output directory (default: `./docs/architecture/`).

## Operating principles

1. **Top-down.** Start with domain/purpose, then skeleton, then layer inventory, then per-layer anatomy. Never dive into implementation details before the layer above is documented.
2. **Discover, don't assume.** Scan the actual codebase first. Mark layers as `present`, `partial`, `absent`, or `N/A`. Deep-dive only `present` and `partial`.
3. **Ground every claim in code.** Each non-trivial statement must cite `path/to/file.py:line` (or a line range). No generic textbook descriptions of what a layer "usually" does — describe what *this project* does.
4. **Interactive, gated by user review.** Pause after Pass 1 and Pass 2. Do not begin Pass 3 without user confirmation, and let the user pick which layers to deep-dive in what order.
5. **Idempotent re-runs.** If a target file already exists, update it in place rather than appending or duplicating. Preserve any user-added sections marked with `<!-- user: -->` ... `<!-- /user -->`.
6. **Concise prose, dense with references.** Prefer bullet lists and tables over paragraphs. A reader should be able to jump from any claim to the relevant code in one click.
7. **Never drop a noteworthy observation.** During *any* pass, if you notice something interesting that doesn't fit the current section or the 7-layer structure, append it to `99-findings.md` with file:line evidence. Do not silently discard it and do not jam it into an unrelated section. Examples: cross-cutting code smells, repeated patterns, dead code, version mismatches, missing tests, security concerns, suspicious comments/TODOs, inconsistent naming, dependency oddities. Better to over-log here than under-log.
8. **Code is ground truth; docs are intent.** When code and existing docs disagree, code wins for "what is" — but the doc still tells you "what was meant," which is a finding worth logging.

## The 7 layers (+ cross-cutting)

| # | Layer | Scope |
|---|---|---|
| 1 | Application | User-facing interface (CLI, API, chat UI, webhook entry points) |
| 2 | Orchestration | Agent roles, control flow, handoffs, multi-agent coordination, the loop |
| 3 | Memory | Short-term (conversation, scratchpad) and long-term (vector store, KV, episodic) storage |
| 4 | Tools | Tool registry, function definitions, MCP/external APIs, code interpreters |
| 5 | Planning/Reasoning | Goal decomposition, ReAct/CoT prompts, planners, single-agent thinking strategy |
| 6 | Context | Data ingestion, retrieval, document loaders, ETL, prompt assembly |
| 7 | Model | LLM client(s), model selection, routing, generation params, caching |
| — | Cross-cutting | Observability, auth, cost/rate-limit, evals, infra/deployment, config/secrets |

Note: Planning/Reasoning and Orchestration overlap in single-agent systems. Document the overlap honestly rather than forcing artificial separation.

## Output structure

```
<output_dir>/
  00-overview.md           # domain, skeleton, layer inventory
  01-application.md
  02-orchestration.md
  03-memory.md
  04-tools.md
  05-planning-reasoning.md
  06-context.md
  07-model.md
  08-cross-cutting.md
  99-findings.md           # catch-all for observations that don't fit any layer
```

Files for `absent` or `N/A` layers are still created but contain only a one-line note explaining why. This makes gaps visible.

## Execution: three passes

### Pass 1 — Overview (write `00-overview.md`, then stop)

**Before investigating, check for pre-built tree files** at `<output_dir>/_trees/*.md`. These are produced by the companion `repo-tree-annotate` skill and contain the authoritative structural map: top-level dir roles, entry points, noise flags, and cross-repo references. If present, consume them as the structural prior — do not re-derive structure from scratch and do not overwrite the tree files. If absent, and the project is non-trivial (>~50 files), suggest the user run `repo-tree-annotate` first.

Investigate and document:

- **Domain / Purpose** — one paragraph: what does this agent *do for whom*? Infer from README, top-level docstrings, CLI help text, entry-point names. If unclear, ask the user one targeted question.
- **Skeleton** — entry points (`main.py`, `__main__`, CLI scripts, server endpoints). Trace one representative request from input to output, listing the files/functions touched in order.
- **Stack snapshot** — read `pyproject.toml` / `requirements.txt` / `uv.lock`. List the agent-relevant dependencies grouped by likely layer (e.g., `anthropic` → Model, `langgraph` → Orchestration, `chromadb` → Memory).
- **Layer inventory** — table with one row per layer: `Layer | Status | Evidence (file:line) | Notes`. Status is `present`, `partial`, `absent`, or `N/A`.

After writing, stop and ask the user to review the overview and confirm:
1. Is the domain summary accurate?
2. Is the layer inventory accurate?
3. Which layers should Pass 3 prioritize? (default: all `present` layers in numeric order)

### Pass 2 — Layer stubs (write `01-…` through `08-…` skeletons, then stop)

For each layer file, write a skeleton with these sections (leave most empty for Pass 3):

```markdown
# <N> — <Layer Name>

**Status:** present | partial | absent | N/A
**Primary files:** [path/to/file.py](path/to/file.py), …
**External deps:** anthropic, chromadb, …

## Responsibility in this project
<one-paragraph; filled in Pass 3>

## Key components
<table; filled in Pass 3>

## Data flow
<filled in Pass 3>

## Contracts & interfaces
<filled in Pass 3>

## Notable decisions / trade-offs
<filled in Pass 3>

## Open questions
<filled in Pass 3>
```

Stop and confirm with the user before Pass 3.

### Pass 3 — Per-layer deep dive (one layer at a time, user-gated)

For each layer the user selected, fill in the skeleton sections by reading the relevant files. Per-section guidance:

- **Responsibility** — what *this layer specifically* owns in this codebase (not the textbook definition).
- **Key components** — table: `Component | File:line | Role | Used by`.
- **Data flow** — short numbered list or ASCII diagram showing inputs → transformations → outputs.
- **Contracts & interfaces** — function signatures, data classes, protocols at the layer's edges. Cite each.
- **Notable decisions / trade-offs** — non-obvious choices visible in the code (e.g., "uses sync OpenAI client despite async orchestration — see [foo.py:42](foo.py#L42)"). If something looks like a bug or smell, note it as an open question, not an assertion.
- **Open questions** — things the code alone doesn't answer; flag for the user.

After each layer, stop briefly and let the user redirect (deeper, skip, move on).

## Markdown conventions

- File references use markdown links: `[file.py:42](file.py#L42)` (or ranges `#L42-L51`).
- Use tables for inventories; bullets for flows; fenced code blocks for signatures.
- No emojis unless the user's existing docs use them.
- Keep each layer file under ~200 lines. If a layer needs more, split into `04-tools.md` + `04a-tool-<name>.md`.

## Cross-cutting (`08-cross-cutting.md`)

One section per concern, each with status + evidence:

- **Observability** — logging, tracing (LangSmith/Langfuse/Helicone/etc.), metrics
- **Auth** — user auth, agent-to-tool auth, secrets handling
- **Cost & rate limits** — token accounting, throttling, retries
- **Evals** — test harness, golden sets, regression checks
- **Infra & deployment** — Docker, CI, runtime target
- **Config & secrets** — env vars, config files, secrets manager

Mark missing concerns explicitly — they're often the highest-value gaps to surface.

## Handling existing project documentation

Many repos already ship docs (`README.md`, `/docs`, `ADRs/`, `CHANGELOG.md`, top-level package docstrings, wiki/Notion links). Treat them as a first-class input but never duplicate them.

**Pass 1 — Doc inventory.** Add a "Documentation" subsection to `00-overview.md` listing every doc artifact found, with one line each: path, apparent purpose, last-modified hint (recent vs. stale), and whether it looks authoritative. Include external links referenced in README (Notion, Confluence, design docs) as references-not-followed.

**During all passes — capture the essence, don't restate.** When a layer or concern is already documented:

- Write a **2–4 sentence essence** in the relevant section: the irreducible facts a reader needs to grasp the layer without clicking away. Think "what would I tell a new teammate in 30 seconds?"
- Follow it with a **`See:` line** linking to the source doc(s) for details.
- Never copy paragraphs. If the temptation is to paste, the essence isn't distilled enough — compress further or quote one short line and link.
- The essence should be in *your* words, grounded against the code. If the doc says X but the code does Y, write what the code does and log a `Doc drift` finding.

Format:

```markdown
## Responsibility in this project
The orchestrator runs a single-agent ReAct loop with a hard cap of 8 tool calls per turn; on cap-hit it returns a partial answer rather than failing. Tool errors are surfaced to the model as observations, not raised.

See: [docs/orchestration.md](docs/orchestration.md), [README.md:L88-L120](README.md#L88-L120)
```

**Doc-derived claims must still be verified against code.** Before writing an essence, open the relevant code and confirm the doc is current. If you can't verify in code, mark the claim with `(per docs, unverified)` and add a finding.

**Surface gaps and drift.** Two findings categories handle the doc/code relationship:
- `Doc drift` — docs claim something the code no longer does (or vice versa)
- `Documentation gap` — code clearly does X but no doc mentions it; or a layer is undocumented entirely

## Findings file (`99-findings.md`)

A running log of observations that don't fit a single layer. Append-only during all passes. Each entry is a bullet:

```markdown
- **[Category]** Short observation. Evidence: [file.py:42](file.py#L42). Why it matters: <one line>.
```

Suggested categories: `Smell`, `Duplication`, `Dead code`, `Security`, `Performance`, `Versioning`, `Testing gap`, `Inconsistency`, `Surprise`, `TODO/FIXME`, `Doc drift`, `Documentation gap`.

Group entries under `## <Category>` headings. Initialize the file in Pass 1 with empty category headings so later passes have a place to write. Do not editorialize — state the observation, cite the code, give one line on impact, move on. The user decides what to act on.

## Inputs the skill accepts

- `output_dir` (optional, default `./docs/architecture/`) — where to write
- `project_root` (optional, default current working directory) — where to scan
- `layers` (optional, Pass 3) — subset of layers to deep-dive
- `include` (optional) — additional glob patterns to force-include in scope
- `exclude` (optional) — additional glob patterns to skip
- `tests` (optional, default `"layer-only"`) — `"skip" | "include" | "layer-only"`
- `config_file` (optional, default `./.architecture-map.yml` if present) — path to a YAML scope config

## Scope: include / exclude rules

Control which files the skill reads. Three layers of control, applied in order:

### 1. Built-in defaults (always applied unless overridden)

**Always excluded** (noise, not source-of-truth):

```
__pycache__/   .venv/   venv/   env/   .git/   node_modules/
dist/   build/   *.egg-info/   .pytest_cache/   .mypy_cache/   .ruff_cache/
.tox/   .coverage   htmlcov/   migrations/
*.lock   poetry.lock   uv.lock   package-lock.json   yarn.lock
.DS_Store   *.pyc   *.pyo   *.so
```

**Always included regardless of any exclude** (context, not code):

```
pyproject.toml   setup.py   setup.cfg   requirements*.txt   Pipfile*
README*   CLAUDE.md   AGENTS.md   CHANGELOG*   LICENSE
docs/**   .env.example   Makefile   Dockerfile*   docker-compose*.yml
```

Everything else under `project_root` is included by default.

### 2. User overrides (via inputs or config file)

If a `.architecture-map.yml` exists at `project_root`, read it. Otherwise use inline inputs. Example config:

```yaml
include:
  - "legacy/**"          # force-include something normally noisy
exclude:
  - "experiments/**"
  - "scripts/one_offs/**"
  - "src/generated/**"
tests: "layer-only"      # "skip" | "include" | "layer-only"

layer_scope:             # per-layer subtree scoping (see section 3)
  application: ["src/api/**", "src/cli/**"]
  orchestration: ["src/agents/**", "src/graph/**"]
  memory: ["src/memory/**", "src/store/**"]
  tools: ["src/tools/**"]
  planning_reasoning: ["src/planner/**", "src/prompts/**"]
  context: ["src/ingest/**", "src/retrieval/**"]
  model: ["src/llm/**", "src/clients/**"]
```

User `include` patterns win over defaults' `exclude`. User `exclude` adds to defaults' `exclude`.

### 3. Per-layer conditional scope

Each layer has an implicit subtree of interest. By default, infer the subtree from the layer inventory built in Pass 1 (e.g., the Memory layer is the union of dirs/files where memory-related deps and patterns appear). If `layer_scope` is provided in config, use it verbatim instead.

When deep-diving a layer in Pass 3:

- Read files within that layer's scope first and prioritize them.
- Files outside the layer's scope are read only if they're referenced by in-scope files (e.g., a util imported by an orchestrator).
- This prevents the Model layer's doc from being polluted by frontend code, and vice versa.

### Test handling (`tests` input)

- `"skip"` — never read test files. Cleanest output but loses contract-by-example information.
- `"include"` — treat tests like production code; they appear in components and citations.
- `"layer-only"` (default) — read tests *only* when their behavior clarifies a layer's contracts (e.g., `tests/test_tool_registry.py` is fair game when documenting the Tools layer). Tests never get their own entries in "Key components"; they're cited as supporting evidence under "Contracts & interfaces".

### Behavior rules

- **Excludes are soft.** If an excluded file is referenced (imported, called, or read at runtime) by an included file, do not follow it, but log a one-line entry in `99-findings.md` under `Inconsistency` so the user can decide whether to re-scope. Format: `Excluded file [path](path) is referenced by [other.py:42](other.py#L42) — consider including.`
- **Print the effective scope.** At the top of `00-overview.md`, include a "Scope" section listing the final resolved include patterns, exclude patterns, test policy, and per-layer scopes (if any). This makes the map's coverage auditable.
- **Never silently drop scope.** If a user-provided pattern matches zero files, surface it in Pass 1's output as a warning so typos don't go unnoticed.

## What the skill should NOT do

- Do not invent layers or components that aren't in the code.
- Do not write generic explanations of what an agent layer "is" — only what *this project's* layer does.
- Do not run Pass 2 or Pass 3 without explicit user confirmation after the prior pass.
- Do not create a single mega-document; always use the folder structure.
- Do not modify source code. This skill is read-only on the project, write-only on the output dir.
