# Spec: /project-audit skill

> Automated per-project audit pipeline — from raw repo to standardized product docs.

## Problem

28 projects, no unified documentation. Manual audit per project takes 30+ min of human attention. Need an automated pipeline that scans a repo, generates positioning, evaluates readiness, and produces standardized docs.

## Usage

```bash
# Audit current repo
/project-audit

# Audit specific repo
/project-audit h2t-snap

# Audit all high-potential projects
/project-audit --tier product

# Dry run (show what would be generated, don't write)
/project-audit --dry-run
```

## Pipeline Stages

### Stage 1: SCAN (automated, no LLM)

**Input:** repo path or name
**Actions:**
- Read CLAUDE.md, README.md, package config (pyproject.toml, setup.py, etc.)
- `git log --oneline -20` — recent activity
- `gh issue list --state open --limit 10` — open issues
- Check for: tests/, docs/, examples/, LICENSE, .claude-plugin/
- Count commits, contributors, file types
- Load card from `h2t-landings/projects.yaml` if exists

**Output:** `scan_result.json` — structured facts about the repo

```json
{
  "id": "h2t-snap",
  "has_claude_md": false,
  "has_readme": true,
  "has_tests": true,
  "has_docs_dir": true,
  "has_landing": true,
  "has_license": false,
  "has_examples": false,
  "commit_count": 14,
  "last_commit": "2026-03-28",
  "open_issues": 0,
  "primary_language": "python",
  "existing_card": { ... }
}
```

### Stage 2: COUNCIL (LLM, ceo-council skill)

**Input:** scan_result.json + existing README content
**Actions:**
- Invoke `h2t:ceo-council` with mode `strategic`
- Prompt: "Position this project: what is it, for whom, why now, competitive landscape, MVP scope"
- Extract structured output

**Output:** `docs/product/positioning.md` (from template)

**Cost control:** Single LLM call, structured output, max 1000 tokens response.

### Stage 3: JUDGE (LLM, h2t-evals if available)

**Input:** scan_result.json + positioning.md
**Actions:**
- If h2t-evals is wired: submit eval job
- If not: run inline LLM judge with rubric:
  - Code quality (has tests, has types, has CI): 0-3
  - Documentation (CLAUDE.md, README, examples): 0-3
  - Product readiness (positioning, landing, marketing): 0-3
  - Architecture (modularity, dependencies, API surface): 0-3
- Total score out of 12

**Output:** `docs/product/eval-report.md`

```markdown
## Readiness Score: 7/12

| Area | Score | Notes |
|------|-------|-------|
| Code quality | 2/3 | Has tests, no CI |
| Documentation | 1/3 | README only |
| Product readiness | 1/3 | No positioning, no landing |
| Architecture | 3/3 | Clean API, modular |

### Recommendations
1. Add CLAUDE.md (use template)
2. Create landing page
3. Write positioning doc
```

### Stage 4: DOCS (LLM, template-based)

**Input:** scan_result.json + positioning.md + eval-report.md + templates
**Actions:**
- Generate CLAUDE.md from template + scan facts + positioning
- Generate/update README.md from template + scan facts + positioning
- Generate MVP scope if missing (3-5 bullet DoD)

**Output:**
- `CLAUDE.md` (new or updated)
- `README.md` (new or updated)
- `docs/product/mvp-scope.md`

**Human gate:** Show generated docs, ask for confirmation before writing.

### Stage 5: REPORT (automated, no LLM)

**Input:** all outputs from stages 1-4
**Actions:**
- Update `h2t-landings/projects.yaml` with new doc statuses
- Update acceptance matrix scores
- Print summary: what was created, what changed, next steps

**Output:** Updated projects.yaml + console summary

## Architecture

```
/project-audit [repo] [--tier product] [--dry-run]
        |
        v
  ┌─────────┐    ┌──────────┐    ┌─────────┐    ┌──────┐    ┌────────┐
  │  SCAN   │───>│ COUNCIL  │───>│  JUDGE  │───>│ DOCS │───>│ REPORT │
  │ (local) │    │ (LLM)    │    │ (LLM)   │    │(LLM) │    │(local) │
  └─────────┘    └──────────┘    └─────────┘    └──────┘    └────────┘
       |               |              |             |            |
  scan_result    positioning.md  eval-report   CLAUDE.md    projects.yaml
                                     .md       README.md     (updated)
                                               mvp-scope.md
```

## Implementation

**Where:** `h2t-skills` repo (or `claude-agent-skills`), as a new skill.

**Skill file:** `commands/project-audit.md`

**Dependencies:**
- `h2t:ceo-council` skill (for COUNCIL stage)
- `h2t-evals` (optional, for JUDGE stage — fallback to inline judge)
- Templates from `h2t-landings/templates/`
- `projects.yaml` from `h2t-landings/`

**Execution model:** Sequential stages, human gate before writing files. Can be run in batch mode for `--tier product` (iterates over matching projects).

## Cost Estimate

Per project:
- SCAN: 0 tokens (local reads)
- COUNCIL: ~2K input + 1K output = ~3K tokens
- JUDGE: ~2K input + 500 output = ~2.5K tokens
- DOCS: ~3K input + 2K output = ~5K tokens
- **Total: ~10K tokens per project**

For 9 high-potential projects: ~90K tokens total (~$0.30 with Haiku, ~$3 with Sonnet).

## Batch Mode

```bash
/project-audit --tier product --batch
```

Iterates over all projects in projects.yaml where `product_potential: high`. Runs pipeline for each, with human confirmation between projects (unless `--auto` flag).

## Storage Standard

Each repo gets:
```
docs/
  product/
    positioning.md    # from COUNCIL
    eval-report.md    # from JUDGE
    mvp-scope.md      # from DOCS
CLAUDE.md             # root, from DOCS
README.md             # root, from DOCS (updated)
```

Cross-repo tracking in `h2t-landings/projects.yaml`.

## Open Questions

1. Should COUNCIL use ceo-council skill or a simpler focused prompt?
2. Should JUDGE scores feed into h2t-evals DB or stay local?
3. Should batch mode create PRs per repo or commit directly?
4. Integration with h2t-factory: audit as prerequisite for "package" stage?

## Success Criteria

- [ ] Running `/project-audit h2t-snap` produces all 5 outputs
- [ ] Running `/project-audit --tier product` audits all 9 projects
- [ ] Acceptance matrix in dashboard updates automatically
- [ ] Templates are followed consistently
- [ ] Human gate works (confirmation before writes)
