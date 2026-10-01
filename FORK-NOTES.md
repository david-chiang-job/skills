# Fork notes

Local deltas against `mattpocock/skills`. **The rule is: cite upstream, do not
rewrite it.** Anything that can live outside these files — extra guardrails,
Traditional Chinese adaptations, repo-specific conventions — belongs in a thin
shell skill in `ai-dotfiles/claude/skills/` that points back at the upstream
body, not in an edit here. This file exists so every remaining edit has to be
justified in writing.

Branch layout: `mine` is upstream plus the deltas below, and is the branch
`~/.claude/skills` links into. The upstream baseline to diff against is the
remote ref `upstream/main`; this fork's own `main` is not maintained.
`mine` is also the fork's GitHub default branch (since 2026-09-27): the global
default-branch guard reads `origin/HEAD`, so while that pointed at `main` a
direct commit to `mine` went through unchecked.

Why `upstream/main` and not a release branch or a tag (reviewed 2026-09-27):
upstream's `release/vX.Y` branches are staging. Each one is merged into main
and tagged there before it ships (v1.1, v1.2 both went that way), so main
never misses a release; it receives it on release day. A release branch is
still being edited, and a new minor means a new branch name to re-point to.
Tags lag too far to track: the last one, `v1.2.3`, is from 2026-08-06, and main
has moved 54 commits since. What main does not promise is that its tip loads,
which is what the check step in Syncing is for.

## Syncing

Every update lands through a pull request into `mine`, like every other repo
of this owner. Merge, never rebase: rebasing rewrites the local commits, and
the only way to publish rewritten commits is a force push.

```
git fetch upstream origin
git merge-tree --write-tree --name-only origin/mine upstream/main   # preview: exit 0 = clean, else lists conflicting files
git switch -c sync/<date> origin/mine
git merge upstream/main
py ../ai-dotfiles/bootstrap/strip_invocation_flag.py skills/engineering skills/productivity
py ../ai-dotfiles/bootstrap/check_skill_frontmatter.py --baseline origin/mine skills/engineering skills/productivity
# exit 1 = a skill would be dropped: stop and do not merge. Otherwise commit
# any flag changes, add a dated entry below (with the ADDED/REMOVED list), then:
gh pr create --repo david-chiang-job/skills --base mine
gh pr merge sync/<date> --repo david-chiang-job/skills --merge --delete-branch   # a merge commit; --rebase would rewrite history again
git switch mine && git pull --ff-only
py ../ai-dotfiles/bootstrap/bootstrap.py --apply
```

The check line exists because a SKILL.md whose frontmatter does not parse is
dropped from the skill list with no error. Replayed on 2026-09-27 against the
commit between #905 and #911 (see the 2026-08-29 entry below), it fails four
installed skills; one commit later it passes. It also lists skills that came
or went since the old `mine`, which is for reading, not a failure.

`--repo` is on every `gh` line because this clone has two remotes and no
`gh repo set-default`, so without it `gh` may pick `mattpocock/skills` and open
the PR in public.

Conflicts land on exactly the lines listed here, every time. The resolution is
always: **keep upstream's prose, re-apply only the switch.**

Until 2026-09-26 this section said `git rebase upstream/main`, which made every
sync end in a force push; it was switched to merge so the fork follows the same
PR-then-merge rule as the rest. Rebase's one selling point, seeing every
local delta at a glance, survives the switch: after a full merge,
`git diff upstream/main mine` is exactly that list. Measured the same day: the
first merge-based sync needed zero hand-resolved hunks, and a dry-run merge of
`release/v1.3` was clean too.

## Why each delta exists

### 1. Invocation flags (`disable-model-invocation`)

Not a preference — a mechanical necessity. The flag is a frontmatter key read by
the harness; no symlink, setting, or manifest entry can override it from
outside, so switching it means editing the upstream file. Which skills are
switched, and why, is declared in one place:
`ai-dotfiles/bootstrap/strip_invocation_flag.py` (`KEEP_FLAGGED` /
`FORCE_FLAGGED`). Do not hand-edit a SKILL.md to change this — add the name to a
list there and re-run the script, so the next sync reproduces it.

### 2. Two description rewrites

`grilling` and `grill-with-docs` carry locally written `description:` lines that
disambiguate the pair. Upstream's own descriptions do not cross-reference each
other, and the model picks between them from the description alone — so this is
the one place where citing upstream verbatim measurably degrades routing.

Reviewed 2026-08-13 and kept. Re-check on each sync: upstream has been actively
aligning this family (`grill-me-align`, PR #788), and the moment its own text
disambiguates, delete these two overrides and go back to citing it.

Re-checked 2026-08-29 and kept again: upstream's `grilling` description still
never names `grill-with-docs`, and vice versa, so the routing problem these two
lines solve is still there. Note both overrides still carry em-dashes, which
upstream swept out of the repo in #905 — deliberate, since they are our prose,
not a stale copy of upstream's.

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

- **2026-08-29 sync (main tip, 37 commits past `v1.2.3`):** upstream added
  `disable-model-invocation: true` to `grill-with-docs` (#880, "stop skills from
  calling other user-invoked skills"). We keep stripping it: the local rule is
  that the agent reaches for skills from natural language, and an
  `interview-prep` hook actively suggests this one by name, so muting it would
  leave that suggestion pointing at something the model cannot call. The cost is
  a new permanent conflict site — this skill now carries two deltas at once, the
  description override and the stripped flag, on adjacent lines.

  Also absorbed: the repo-wide em-dash sweep (#905) and the fix for the invalid
  YAML front-matter it created (#911, six descriptions whose new `: ` sequences
  made the skills unparseable). Both are in this sync, so the net effect is
  none — but a sync landing between those two commits would have silently
  dropped `code-review`, `to-spec`, `setup-matt-pocock-skills`, and
  `wait-what` from discovery (the fourth found by the 2026-09-27 replay). Worth remembering the next time "main tip is fine" comes up.

  Two new skills arrived in `skills/in-progress/` (`retro`, `implement-spec`).
  Nothing to do: `bootstrap/manifest.toml` links only `engineering` and
  `productivity`, so `in-progress` skills are never installed — which is also
  why upstream's `retro` cannot collide with the local `/retro`.

- **2026-09-26 sync (main tip, 15 commits):** adds `pr` to `in-progress/` and
  sharpens `retro` (mechanical findings become deterministic checks). Neither
  is installed yet, so nothing changes at runtime. Merge was clean. This is
  the first sync done the merge-and-PR way (see Syncing).

  The previous entry's "cannot collide" only held while `retro` sat in
  `in-progress/`. `release/v1.3` graduates `retro`, `pr` and `implement-spec`
  into `engineering/`, and a personal skill hides a project skill of the same
  name with no error. Handled ahead of v1.3, outside this repo:
  interview-prep renamed its skill to `interaction-review`; `bootstrap.py`
  now refuses any same-name skill (`test_skill_collisions.py`); and
  `KEEP_FLAGGED` holds `retro` and `implement-spec`. A rehearsal rebase onto
  `release/v1.3` replayed all five local commits without conflict.

  Still due when v1.3 reaches main: v1.3 renames `CONTEXT.md` to
  `GLOSSARY.md` (so `ai-dotfiles/delegation/CONTEXT.md` and the quote in
  `wait-what-zh` follow), and retires `resolving-merge-conflicts` (its link is
  pruned by bootstrap; three interview-prep docs name it). Done in the
  2026-10-01 entry below.

- **2026-10-01 sync (v1.3, main tip `d81f3a1`, 21 commits):** v1.3 reached
  main through #1120. Merge was clean (merge-tree preview exit 0, zero
  hand-resolved hunks); the strip script changed nothing.
  Frontmatter check against the old `mine`:
  ADDED `implement-spec`, `pr`, `retro`; REMOVED `resolving-merge-conflicts`.
  `retro` and `implement-spec` stay user-only through `KEEP_FLAGGED`, as
  planned in the previous entry; `pr` ships model-invocable. The
  `CONTEXT.md` to `GLOSSARY.md` rename and the retired skill were followed
  up in `ai-dotfiles` and `interview-prep` the same day.

  The check step could not run at first: `PyYAML` was missing on the machine
  even though `python_sync.py` reported the Python spec as matching, because
  the spec did not list it. `ai-dotfiles` #104 added `pyyaml` to
  `manifest.toml` `[python]`, so the next sync on any machine has it.
