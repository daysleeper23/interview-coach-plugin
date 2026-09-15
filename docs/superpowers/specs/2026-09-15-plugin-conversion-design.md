# Interview Coach Plugin Conversion — Design Spec

**Date:** 2026-09-15  
**Status:** Approved for implementation

## Goal

Convert the interview-coach repo from a "rename SKILL.md to CLAUDE.md" distribution model into a proper Claude Code plugin installable via direct GitHub URL, requiring no file copying or project pollution.

The plugin exposes two entry points:

- **`interview-coach:coach`** — full orchestrator for users who want guided coaching. Reads state, detects intent, routes to the right command.
- **`interview-coach:[command]`** — 24 individual command skills for users who know exactly what they want (e.g. `interview-coach:analyze`, `interview-coach:kickoff`).

## Target Install UX

```bash
/plugin install https://github.com/daysleeper23/interview-coach-plugin
```

After install:
```
/skill interview-coach:coach          # guided entry point
/skill interview-coach:analyze        # direct command entry point
/skill interview-coach:kickoff        # etc.
```

> **Risk (acknowledged):** Direct URL install without a marketplace entry is unconfirmed in the Claude Code CLI. The plugin structure built here is identical to what a marketplace would point to, so if direct install doesn't work, adding a one-file marketplace repo is a fallback requiring no changes to this repo.

## File Structure Changes

### Before
```
interview-coach-plugin/
├── SKILL.md
├── references/
│   ├── commands/ (24 files)
│   └── *.md (13 files)
├── releases/
├── README.md
├── LICENSE
└── VERSIONS.md
```

### After
```
interview-coach-plugin/
├── package.json                          ← NEW
├── skills/
│   ├── coach/
│   │   └── SKILL.md                      ← restructured from root SKILL.md
│   ├── analyze/
│   │   └── SKILL.md                      ← NEW (thin, ~20 lines)
│   ├── kickoff/
│   │   └── SKILL.md                      ← NEW (thin, ~20 lines)
│   ├── practice/
│   │   └── SKILL.md                      ← NEW (thin, ~20 lines)
│   └── ... (21 more command skills)      ← NEW (thin, ~20 lines each)
├── references/
│   ├── base.md                           ← NEW (extracted from root SKILL.md)
│   ├── commands/ (24 files)              ← unchanged
│   └── *.md (13 files)                   ← unchanged
├── README.md                             ← updated
├── LICENSE
└── VERSIONS.md
```

**Dropped:** `releases/` directory — redundant with git tags.

**Note:** `references/` stays at the plugin root (not inside `skills/`) so the `pi.skills` scanner doesn't pick it up as a skill.

## package.json

```json
{
  "name": "interview-coach",
  "version": "1.0.0",
  "description": "High-rigor interview coaching for job seekers — PM, Engineering, Design, Data Science, Research, Marketing, and Operations",
  "pi": {
    "skills": ["./skills"]
  }
}
```

Version is managed here and bumped on releases (replaces `releases/` directory convention).

## The Three New Files

### 1. `references/base.md` (new)

Extracted from the current `SKILL.md`. Contains all shared rules that individual command skills need to function correctly in standalone mode:

- Priority hierarchy
- Non-negotiable operating rules (all 12)
- Core rubric (5 dimensions)
- Evidence sourcing standard
- Response blueprints (section headers)
- Coaching voice calibration pointer

The coach skill does **not** read this file — its rules remain embedded in `skills/coach/SKILL.md` (no round trip). Command skills read it once at invocation.

### 2. `skills/coach/SKILL.md` (restructured from root `SKILL.md`)

Functionally identical to the current `SKILL.md`. When routing to a command, reads `references/commands/[command].md` directly — same as today. No structural change to the orchestration logic; only path updates are needed.

### 3. `skills/[command]/SKILL.md` × 24 (new, thin)

Each is ~20 lines. Template:

```markdown
---
name: [command]
description: [one-line description of what this command does]
---

Read `references/base.md` for operating rules, rubric, and coaching voice.
Read `references/commands/[command].md` for the workflow.
[Additional refs if needed, e.g.:]
Also read: references/rubrics-detailed.md, references/calibration-engine.md.

Load `coaching_state.md` if present. Execute [command]. Save state when done.
```

Additional reference files per command (mirrors the File Routing table in `SKILL.md`):

| Command | Additional refs beyond base + commands/[cmd].md |
|---|---|
| `analyze` | transcript-processing.md, transcript-formats.md, rubrics-detailed.md, examples.md, calibration-engine.md, differentiation.md |
| `practice`, `mock` | role-drills.md, calibration-engine.md |
| `prep` | story-mapping-engine.md (when storybank exists) |
| `linkedin`, `resume`, `pitch`, `outreach` | differentiation.md, storybank-guide.md |
| `decode` | cross-cutting.md (Role-Fit Assessment Module) |
| `present` | storybank-guide.md |
| `salary` | commands/negotiate.md |
| `stories` | storybank-guide.md, differentiation.md |
| `progress` | calibration-engine.md |
| All at Directness Level 5 | challenge-protocol.md |
| All others | base.md + commands/[cmd].md only |

## Path Updates

All internal `references/...` paths must be updated to reflect the new location of `references/` at the plugin root.

**Scope:**
- `skills/coach/SKILL.md`: all occurrences of `references/` → `references/` (no change needed — references/ is still at root relative to plugin install)
- Reference files cross-referencing each other: same — `references/` paths remain valid from plugin root
- Command `SKILL.md` files: written with correct paths from the start

**Replacement rule from the old single-skill spec:** The original spec called for `references/ → skills/interview-coach/references/`. This is **superseded**. With `references/` at the plugin root, existing paths in reference files require no rewriting — they already resolve correctly.

## README Update

Replace the current "rename SKILL.md to CLAUDE.md" instructions with:

```bash
# Install
/plugin install https://github.com/daysleeper23/interview-coach-plugin

# Full coaching experience (recommended)
/skill interview-coach:coach

# Jump directly to a command
/skill interview-coach:analyze
/skill interview-coach:kickoff
# ... etc.
```

Keep existing documentation about commands, features, and `coaching_state.md` — only the installation section changes.

## What Does NOT Change

- All skill logic, rubrics, prompts, and workflows — zero behavior changes
- `references/commands/` — 24 files unchanged
- `references/*.md` — 13 shared reference files unchanged
- `coaching_state.md` format — existing user state files remain valid
- `VERSIONS.md` — keep as changelog
- `LICENSE`

## Testing Checklist

### Coach skill
1. Plugin installs without error from the GitHub URL
2. `interview-coach:coach` is listed after install
3. Coach with no `coaching_state.md` → suggests kickoff
4. Coach with existing state → greets with prescriptive recommendation
5. Coach detects "analyze this transcript" intent and executes analyze inline

### Individual command skills
6. `interview-coach:kickoff` works standalone end-to-end (baseline — reads base.md + commands/kickoff.md only)
7. `interview-coach:analyze` works standalone — exercises the most reference file reads
8. No broken path errors (`references/...` not found) during any command skill

### Regression
9. All 24 command skills listed (`/plugin list` or equivalent)
10. State written by one skill (e.g. kickoff) is readable by another (e.g. analyze)
