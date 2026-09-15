# Interview Coach Plugin Conversion — Design Spec

**Date:** 2026-09-15  
**Status:** Approved for implementation

## Goal

Convert the interview-coach repo from a "rename SKILL.md to CLAUDE.md" distribution model into a proper Claude Code plugin installable via direct GitHub URL, requiring no file copying or project pollution.

## Target Install UX

```bash
/plugin install https://github.com/daysleeper23/interview-coach-plugin
```

After install, the skill is available globally:
```
/skill interview-coach:interview-coach
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
│   └── interview-coach/
│       ├── SKILL.md                      ← moved from root
│       └── references/                   ← moved from root
│           ├── commands/ (24 files)
│           └── *.md (13 files)
├── README.md                             ← updated
├── LICENSE
└── VERSIONS.md
```

**Dropped:** `releases/` directory — redundant with git tags.

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

## Path Updates

All internal `references/...` paths must be updated to `skills/interview-coach/references/...`.

**Scope:**
- `SKILL.md`: 27 occurrences (confirmed via grep)
- Reference files cross-referencing each other: ~70 occurrences across files including `cross-cutting.md`, `story-mapping-engine.md`, `calibration-engine.md`, `rubrics-detailed.md`, `storybank-guide.md`, `schema-migration.md`, `coaching-voice.md`, `role-drills.md`, `coaching-state-schema.md`

**Replacement rule:**
```
references/ → skills/interview-coach/references/
```

Applied as a global find-and-replace across all `.md` files in the repo. No exceptions — every `references/` path must resolve from the plugin install root.

## README Update

Replace the current "rename SKILL.md to CLAUDE.md" instructions with:

```bash
# Install
/plugin install https://github.com/daysleeper23/interview-coach-plugin

# Use
/skill interview-coach:interview-coach
```

Keep existing documentation about commands, features, and `coaching_state.md` — only the installation section changes.

## What Does NOT Change

- All skill logic, rubrics, prompts, and workflows — zero behavior changes
- `coaching_state.md` format — existing user state files remain valid
- `VERSIONS.md` — keep as changelog
- `LICENSE`

## Testing Checklist

1. Plugin installs without error from the GitHub URL
2. Skill is listed after install (`/plugin list` or equivalent)
3. `kickoff` command works end-to-end (reads no reference files at start, so baseline test)
4. `analyze` command works — exercises the most reference file reads (transcript-processing, rubrics-detailed, calibration-engine, differentiation, examples)
5. No broken path errors (`references/...` not found) during any command
