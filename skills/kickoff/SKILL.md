---
name: kickoff
description: Initialize a coaching profile. Collects your background, resume, targets, and builds a personalized coaching plan. Start here if you're new.
---

Read `references/base.md` for operating rules, rubric, and coaching voice.
Read `references/commands/kickoff.md` for the kickoff workflow.

If `coaching_state.md` already exists: inform the user they already have a coaching profile and suggest `/skill interview-coach:coach` for a returning session. Do not re-run kickoff unless they explicitly confirm they want to reset.

Execute kickoff. Save state to `coaching_state.md` when done.
