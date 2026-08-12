# Fork notes

Local deltas against `mattpocock/skills`. **The rule is: cite upstream, do not
rewrite it.** Anything that can live outside these files — extra guardrails,
Traditional Chinese adaptations, repo-specific conventions — belongs in a thin
shell skill in `ai-dotfiles/claude/skills/` that points back at the upstream
body, not in an edit here. This file exists so every remaining edit has to be
justified in writing.

Branch layout: `main` mirrors `upstream/main` untouched. `mine` is `main` plus
the commits below, and is the branch `~/.claude/skills` links into.

## Syncing

```
git fetch upstream
git rebase upstream/main        # replays the config commits below
py ../ai-dotfiles/bootstrap/strip_invocation_flag.py skills/engineering skills/productivity
py ../ai-dotfiles/bootstrap/bootstrap.py --apply
```

Conflicts land on exactly the lines listed here, every time. The resolution is
always: **keep upstream's prose, re-apply only the switch.**

## Why each delta exists

### 1. Invocation flags (`disable-model-invocation`)

Not a preference — a mechanical necessity. The flag is a frontmatter key read by
the harness; no symlink, setting, or manifest entry can override it from
outside, so switching it means editing the upstream file. Which skills are
switched, and why, is declared in one place:
`ai-dotfiles/bootstrap/strip_invocation_flag.py` (`KEEP_FLAGGED` /
`FORCE_FLAGGED`). Do not hand-edit a SKILL.md to change this — add the name to a
list there and re-run the script, so the next rebase reproduces it.

### 2. Two description rewrites

`grilling` and `grill-with-docs` carry locally written `description:` lines that
disambiguate the pair. Upstream's own descriptions do not cross-reference each
other, and the model picks between them from the description alone — so this is
the one place where citing upstream verbatim measurably degrades routing.

Reviewed 2026-08-13 and kept. Re-check on each sync: upstream has been actively
aligning this family (`grill-me-align`, PR #788), and the moment its own text
disambiguates, delete these two overrides and go back to citing it.

### 3. `FORK-NOTES.md`

This file. Never upstream's concern.

## Known upstream breaking changes absorbed

- **2026-08-13 sync (v1.2):** `writing-great-skills` was renamed to
  `writing-for-agents` (`1fc6573`) and restructured — its `GLOSSARY.md` split
  into `SKILL.md` (universal) plus `SKILL-MECHANICS.md` (skill-specific).
  Anything referring to the old name or the old file is stale.

  A rename is not a new skill. This one was briefly muted the same day on the
  belief that it was newly arrived and newly overlapping a local rule; it had in
  fact been running model-invoked under its old name since 2026-07-25. Check
  `git log --follow` before treating a name you have not seen as new.
