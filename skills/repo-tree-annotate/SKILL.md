---
name: repo-tree-annotate
description: Produce an annotated structural map of one or more repos — directory tree with one-liner roles per top-level dir, entry points, noise flags, LOC for non-trivial files, markdown titles, and cross-repo references. Structure-only; no semantic or behavior analysis. Output is a reviewable per-repo markdown file under docs/architecture/_trees/. Use when the user asks to "map the repo structure", "produce a repo tree", "annotate the tree", "scan the repo before deep-dive", or as a preprocessing step for agent-architecture-map. Supports single-repo and multi-repo (e.g., tech + context) setups.
---

# Repo Tree Annotate

Produce a compact, reviewable structural map of one or more repos. The output is meant to be eyeballed and corrected by the user, then consumed downstream (by `agent-architecture-map` or directly by a human reader).

## Operating principles

1. **Structure only, no semantics.** Describe what's there, not what it does. Roles are one-liners inferred from filenames, READMEs, package docstrings, and `pyproject.toml` — never from reading function bodies. Behavior documentation is `agent-architecture-map`'s job.
2. **Fast and cheap.** This skill should run in seconds to minutes, not hours. Avoid reading file contents beyond what's needed for annotation (first lines for headings, line counts, top-level imports for entry detection).
3. **Mark inferences explicitly.** When a one-liner is inferred (not from a doc), append `(inferred)` so the user knows what to review. The Review Checklist at the end of each tree file lists all inferences.
4. **Idempotent re-runs.** If a target tree file already exists, update it in place. Preserve any user edits inside `<!-- user: --> ... <!-- /user -->` blocks.
5. **One file per repo.** Even in multi-repo setups, each repo gets its own file. Cross-repo links are surfaced but trees stay separate.

## Inputs

- `repos` — one or more repos to scan. Accepts:
  - A single path (single-repo mode)
  - A list of `{name, path}` objects (multi-repo mode), e.g., `[{name: tech, path: ~/code/tech}, {name: context, path: ~/code/context}]`
  - If the user provides two unnamed paths, prompt for short names (e.g., `tech`, `context`)
- `output_dir` (optional, default `./docs/architecture/_trees/`)
- `depth` (optional, default `3`) — max tree depth to display
- `exclude` (optional) — additional glob patterns to skip
- `config_file` (optional, default `./.architecture-map.yml` if present) — reuses the same config as `agent-architecture-map`

## Default exclusions (always applied)

```
__pycache__/   .venv/   venv/   env/   .git/   node_modules/
dist/   build/   *.egg-info/   .pytest_cache/   .mypy_cache/   .ruff_cache/
.tox/   .coverage   htmlcov/   .DS_Store
*.lock   poetry.lock   uv.lock   package-lock.json   yarn.lock
*.pyc   *.pyo   *.so
```

## Output structure

```
<output_dir>/
  <repo-name>.md      # one file per repo
```

If only one repo, name it after the repo's directory. If multi-repo, use the user-provided names (`tech.md`, `context.md`).

## Per-repo file format

```markdown
# Tree: <repo-name>

**Path:** `<absolute or relative path>`
**Generated:** <YYYY-MM-DD>
**Depth:** <N>
**Detected languages/frameworks:** Python (FastAPI, LangGraph), TypeScript (AWS CDK)
**Recent activity:** last commit <relative date>, <N> commits in last 30 days

## Tree

```text
<repo-name>/
├── README.md                          # "Project Foo: agentic data pipeline"
├── pyproject.toml                     # Python 3.11, deps: anthropic, langgraph, chromadb
├── src/
│   ├── agents/                        # agent definitions (inferred)
│   │   ├── planner.py                 → entry  (180 LOC)
│   │   └── executor.py                (240 LOC)
│   ├── tools/                         # tool registry and implementations (inferred)
│   │   └── ... (12 files)
│   └── main.py                        → entry  (60 LOC)
├── cdk/                               # AWS CDK infrastructure (TypeScript)
│   ├── cdk.json                       → entry  (CDK app)
│   └── lib/
│       └── stack.ts                   (320 LOC)
├── tests/                             (noise: layer-only relevance)
├── legacy/                            (noise: archived 2025)
└── docs/
    └── architecture.md                # "Architecture overview"
```

## Top-level annotations

| Path | Role | Source of annotation |
|------|------|----------------------|
| `src/agents/` | Agent definitions | inferred from filenames |
| `src/tools/` | Tool registry and implementations | inferred |
| `cdk/` | AWS CDK infrastructure | `cdk.json` present |
| `docs/` | Project documentation | filenames + first headings |

## Entry points

- [src/main.py](src/main.py) — CLI entry (`if __name__ == "__main__"`)
- [src/agents/planner.py](src/agents/planner.py) — invoked by main
- [cdk/cdk.json](cdk/cdk.json) — CDK app entry

## Cross-repo references

(Only present in multi-repo mode.)

- `src/agents/planner.py` references string `"context/decisions/0007"` — see [context/decisions/0007-retry-policy.md](../../../../context/decisions/0007-retry-policy.md)
- `context/README.md` references `tech/cdk/` — confirmed exists

## Notable files

Files >300 LOC or with high import counts (likely architectural hotspots):

- [src/agents/executor.py](src/agents/executor.py) — 240 LOC, imports from 8 internal modules
- [cdk/lib/stack.ts](cdk/lib/stack.ts) — 320 LOC

## Review checklist

Items the skill is least confident about. Please confirm or correct:

- [ ] Role of `src/agents/`: "Agent definitions" — correct?
- [ ] `legacy/` flagged as noise — should it be excluded from downstream analysis?
- [ ] Detected entry `src/agents/planner.py` — is this actually run, or only imported?
- [ ] Cross-repo reference `"context/decisions/0007"` — is this an active link or a stale string?

<!-- user:
Add any corrections or notes here. They will be preserved on re-run.
-->
<!-- /user -->
```

## Detection rules

### Entry points (mark with `→ entry`)

- Python: files containing `if __name__ == "__main__":` or referenced in `pyproject.toml` `[project.scripts]` or `[tool.poetry.scripts]`
- AWS Lambda: files matching `handler` patterns, referenced in `cdk` stack or `serverless.yml`
- AWS CDK: `cdk.json` (the app entry); the `app` field points to the entry file
- Node/TS: files referenced in `package.json` `"main"`, `"bin"`, or `"scripts"`
- FastAPI/Flask/etc: files with `app = FastAPI()` or `app = Flask(__name__)` at module level
- Make a best-effort detection; mark `→ entry?` (with question mark) when uncertain

### Noise flags (mark with `(noise: <reason>)`)

- Generated dirs: `generated/`, `gen/`, anything mentioned as `output_dir` in build configs
- Archived/legacy: `legacy/`, `_archive/`, `old/`, `deprecated/`, dirs containing only files last modified >12 months ago
- Vendor: `vendor/`, `third_party/`, `external/`
- Tests: `tests/`, `test/`, `__tests__/` — flag as `(noise: layer-only relevance)` rather than excluding, to remind downstream that they're contract evidence
- Always grep README/docs for a sentence like "this folder is deprecated" or "do not edit" — if found, surface as the reason

### LOC display

- Show LOC for files >50 lines
- Show LOC for files >300 lines in **bold** (architectural hotspots — listed again under "Notable files")
- For dirs with many small files: collapse as `... (N files)`

### Top-level dir one-liners

Source of inference, in priority order:
1. Explicit description in a top-level README or `docs/structure.md`
2. Module docstring of `__init__.py`
3. README inside that directory
4. Inferred from filenames / contained file types (mark as `(inferred)`)

Never read function bodies to infer dir purpose — that's downstream work.

### Markdown titles (context repos especially)

For every `.md` file in scope, extract the first `# Heading` (or first non-empty line if no heading). Display inline next to the filename. This handles ADR/Notion-export filename opacity.

### Cross-repo references (multi-repo mode)

For each repo, grep for:
- The other repo's name (case-insensitive)
- The other repo's top-level dir names (e.g., `context/`, `decisions/`)
- Common ADR/decision-doc filename patterns (e.g., `0001-`, `ADR-`)

Surface matches with file:line and verify the referenced path actually exists in the other repo. Mismatches (referenced but missing) go in the Review Checklist.

### Language/framework detection

Quick signals from manifests, not deep imports:
- `pyproject.toml`/`requirements.txt` deps → Python frameworks
- `package.json` deps → JS/TS frameworks
- `cdk.json` present → AWS CDK
- `serverless.yml` → Serverless Framework
- `Dockerfile`, `docker-compose*.yml` → containerization
- `.tf` files → Terraform

List 3-5 top signals; don't enumerate every dep.

### Recent activity

Run `git log -1 --format=%cr` for last-commit recency. Run `git log --since="30 days ago" --oneline | wc -l` for activity volume. If not a git repo, skip this section.

## Handoff to `agent-architecture-map`

The architecture skill's Pass 1 should look for files in `<output_dir>/_trees/*.md`. If present, it consumes them as the structural prior instead of re-deriving from scratch. The tree files are the authoritative source for: top-level dir roles, entry points, noise flags, and cross-repo references. The architecture skill should not overwrite tree files; it only reads them.

## What this skill does NOT do

- Does not read function bodies or analyze behavior
- Does not document architecture or design decisions
- Does not run the code or its tests
- Does not modify source code or existing docs in the repos being scanned (output goes only to `output_dir`)
- Does not infer roles beyond top-level directories (leaf files get LOC and entry detection only, not roles)