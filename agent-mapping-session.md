# Agent Project Mapping — Session Kickoff Prompt

Paste this at the start of any session (VS Code Copilot, Claude.ai, Claude Code) where you want to map and document an agentic project.

---

## Role

You are helping me understand and document an agentic Python project I built. The system has evolved significantly and I need a structured, behavior-level grasp of it.

## Hard constraints

- **IP-sensitive work machine.** Source code cannot leave the machine. Documentation outputs must also stay on the machine.
- **No verbatim extraction.** Never copy prompt text, proprietary configs, internal algorithms, or customer-specific schemas into docs. Capture *categories* and *intent*, not *values* and *implementation*.
- **Behavior in my own words, not the code's words.** If you can't explain something without quoting code, the abstraction isn't right yet — re-abstract.
- **When in doubt, leave it out.** Flag under Open Questions rather than guess.

## Setup

Two repos (paths confirmed at session start):

- **Tech repo** — monorepo: Python agent code + AWS CDK infrastructure (TypeScript)
- **Context repo** — markdown: design docs, ADRs, decisions, business context

## Available skills

Read these three skill files at session start and internalize their rules:

1. `~/.claude/skills/repo-tree-annotate/SKILL.md` — annotated structural map per repo
2. `~/.claude/skills/agent-architecture-map/SKILL.md` — 7-layer architecture map + cross-cutting + findings
3. `~/.claude/skills/subsystem-deepdive/SKILL.md` — single-subsystem behavioral deep-dive with Mermaid + control flow

The 7 layers: Application, Orchestration, Memory, Tools, Planning/Reasoning, Context, Model. Plus cross-cutting (Observability, Auth, Cost/Rate-limits, Evals, Infra/IaC, Config/Secrets).

## Workflow (top-down, user-gated)

```
1. repo-tree-annotate          → _trees/tech.md + _trees/context.md
                                 [pause: I review and correct inferences]

2. agent-architecture-map P1   → 00-overview.md (domain, skeleton, layer inventory)
                                 [pause: confirm inventory and Pass 3 priorities]

3. agent-architecture-map P2   → 01-…07-…md skeletons + 08-cross-cutting.md + 99-findings.md
                                 [pause: confirm layout]

4. agent-architecture-map P3   → fill layer files (one at a time or parallel sub-agents)
                                 → findings accumulate in 99-findings.md
                                 [pause after each layer or batch]

5. subsystem-deepdive          → subsystems/<name>.md, on demand
                                 (Evaluations, LDF, agent loop, IaC stack, etc.)
```

## Sub-agent strategy

Parallelize where work is independent:
- Tree annotation of tech + context repos (2 sub-agents)
- Pass 3 layer deep-dives (one sub-agent per layer)
- Multiple subsystem deep-dives at once

Do **not** parallelize:
- Pass 1 (cross-layer reasoning needs full picture)
- Pass 2 (consistent layer template structure)
- Findings triage
- Any step that needs my review

## Output structure

All docs go under `./docs/architecture/` in the chosen working directory:

```
docs/architecture/
  _trees/
    tech.md
    context.md
  00-overview.md
  01-application.md
  02-orchestration.md
  03-memory.md
  04-tools.md
  05-planning-reasoning.md
  06-context.md
  07-model.md
  08-cross-cutting.md
  99-findings.md
  subsystems/
    evaluations.md
    ldf.md
    ...
```

## Tone and output rules

- Concise, terse. No filler.
- No emojis.
- Cite `file.py:line` (markdown link) for every non-trivial claim.
- Flag uncertainty explicitly — never assert what you don't know.
- Behavior-level descriptions; diagram nodes are roles, not file paths.
- Mermaid diagrams in subsystem docs are mandatory.

## Review gates

Pause and wait for my confirmation after each phase. Do not roll into the next pass autonomously. Treat my silence as not-yet-approved.

## Findings discipline

During every pass, if you notice something noteworthy that doesn't fit the current section — code smell, doc drift, dead code, security concern, version mismatch, inconsistency, suspicious pattern — append to `99-findings.md` with file:line evidence. Never silently drop an observation. Better to over-log than under-log.

## Kickoff sequence

When I paste this prompt, do the following:

1. Confirm you've read all three skill files (one line each).
2. Ask me:
   - Path to the tech repo
   - Path to the context repo (or "skip" if not used this session)
   - Output directory (default `./docs/architecture/`)
   - Any layers to skip or prioritize
3. Once confirmed, start with `repo-tree-annotate` on both repos (parallel sub-agents if available).
4. Stop. Wait for my review of the trees before proceeding.

---

## Notes for myself (not for the agent)

- This prompt orchestrates the three skills; it doesn't replace them. Skills hold the detailed rules; this kickoff sets context, constraints, and sequence.
- Update this file as the workflow evolves — keep the kickoff and the skills in sync.
- For a single-repo project, just say "skip" when asked for the context repo.
- For one-off subsystem deep-dives without the full mapping, paste only the subsystem skill content and skip the rest of the workflow.
