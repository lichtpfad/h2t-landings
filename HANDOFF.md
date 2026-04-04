# Handoff — h2t-landings

> Session: 2026-04-04 | Duration: ~90 min | Branch: main

## What was done

### Issue #11 — Documentation Standard (CLOSED)
- Created 2 new templates: `mvp-scope.md.template`, `eval-report.md.template` (5 total)
- Built acceptance matrix: filesystem scan of all 28 repos, verified docs presence
- Expanded `projects.yaml` docs block: added `positioning`, `mvp_scope`, `eval_report` fields
- Fixed h2t-snap `marketing_docs: true → false` (only discrepancy found)
- Updated registry dashboard HTML: 10-column matrix + Top 5 Gaps section
- Committed, pushed, issue auto-closed via `closes #11`

### Issue #12 — Per-project deep audit (IN PROGRESS: 4/9 done)

Ran `/project-audit` on 4 product-tier projects:

| Project | Score change | Readiness | Key blocker | Repo |
|---------|-------------|-----------|-------------|------|
| h2t-dcc | 0/8 → 5/8 | 5/12 | hou-expert wiring, issue #1 (bash quotes) | h2t-dcc |
| depthkit | 0/8 → 5/8 | 6/12 | .tox packaging (#19) | depthkit (C:/work/depthkit) |
| creative-thinking | 1/8 → 5/8 | 7/12 | data licensing (Chernyshov content) | creative-thinking |
| h2t-snap | 5/8 (pre-session) | — | marketing docs | h2t-snap |

Each audit produced: positioning.md, eval-report.md, mvp-scope.md, CLAUDE.md, README.md (where missing).

Competitive research completed for all 3:
- **h2t-dcc:** 3 TD + 3 Houdini competitors. Moat = operator registry + knowledge graph.
- **depthkit:** 6 TD competitors (depth + 3DGS). Moat = only depth→splat pipeline.
- **creative-thinking:** 0 direct competitors. Intersection of structured methodology + graphs + LLM is empty.

### Key correction applied
h2t-dcc and h2t-ai are the SAME project (dcc=prod, ai=dev). Fixed all docs that said "requires h2t-ai backend". Saved to memory.

## What's next

### Immediate (issue #12 continuation)
5 product projects remaining for audit:
1. **SpecDesigner** (0/8, remote-only) — needs `gh` clone or remote scan
2. **newsengine** (1/8, remote-only, MacBook) — same
3. **h2t-vision** (2/8, local C:/dev) — good candidate
4. **h2t-transcription** (2/8, local C:/dev) — good candidate
5. **h2t-graphs** (3/8, local C:/dev) — good candidate

### After #12
- **#5 Landing sprint** — now unblocked, all audited projects have positioning docs
- **#9 Ontology v2** — restructure domains.yaml

## Commits this session

```
h2t-landings:
  2c7f268  feat: acceptance matrix verified by filesystem scan, closes #11
  c50f253  chore: update registry after h2t-dcc project-audit (0/8 → 5/8)
  3618bc6  chore: update registry after depthkit project-audit (0/8 → 5/8)
  7b4c5ae  chore: update registry after creative-thinking project-audit (1/8 → 5/8)

h2t-dcc:
  b2063d7  feat: project-audit — positioning, eval-report, mvp-scope, CLAUDE.md, README
  b63a04b  fix: h2t-ai is dev repo not dependency, snap bundled, houdini in MVP

depthkit:
  aa1a6ab  feat: project-audit — README, positioning, eval-report, mvp-scope
  6cb2ecd  chore: add competitive landscape to positioning

creative-thinking:
  b399385  feat: project-audit — CLAUDE.md, positioning, eval-report, mvp-scope
  336166f  chore: add competitive landscape to positioning
```

## State

- **Branch:** main (all repos)
- **Unpushed:** nothing
- **Untracked in h2t-landings:** .claude/, .playwright-mcp/, 3 registry screenshots (cosmetic)
- **depthkit location:** C:/work/depthkit (not C:/dev/)
