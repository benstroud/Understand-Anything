---
name: sync-fork
description: >
  Syncs this fork (benstroud/Understand-Anything) with the upstream repo
  (Lum1104/Understand-Anything), then enforces the fork's one invariant:
  no openclaw files or references exist anywhere in the tree.
  Also ensures README.md contains the correct fork notice and all
  installation URLs point to benstroud/Understand-Anything.

  Use this skill whenever the user says anything like "sync the fork",
  "pull in upstream changes", "bring in the latest from upstream",
  "update from Lum1104", "keep the fork current", or asks to maintain
  the fork. The skill handles the full workflow end-to-end.
---

# sync-fork skill

This fork of Lum1104/Understand-Anything exists for a single reason:
organizations whose security policies block software containing files that
reference "openclaw". The fork is otherwise identical to upstream. Your job
when this skill triggers is to keep it that way — current with upstream,
and free of every trace of openclaw.

## Context

- Upstream: `https://github.com/Lum1104/Understand-Anything.git`
- Fork: `https://github.com/benstroud/Understand-Anything`
- Fork invariants:
  1. No openclaw files, directories, or text references anywhere
  2. README.md contains the fork notice callout (see below)
  3. All install commands and INSTALL.md fetch URLs reference `benstroud/Understand-Anything`

> [!IMPORTANT]
> OpenCode is **not** openclaw. OpenCode references are fine to keep.
> Only remove things mentioning "openclaw" (case-insensitive).

---

## Step 1 — Sync upstream

### 1a. Ensure the upstream remote exists
```bash
git remote get-url upstream 2>/dev/null || git remote add upstream https://github.com/Lum1104/Understand-Anything.git
```

### 1b. Fetch and merge
```bash
git fetch upstream
git merge upstream/main
```

Count how many commits were brought in by comparing the log before and after
(or inspect the merge output). Note the count for your final report.

If there are merge conflicts, resolve them with the fork invariants in mind:
- If upstream re-adds openclaw content, discard it during resolution
- For everything else, take the upstream version

### 1c. Remove any openclaw traces

After merging, scan the entire working tree:

```bash
grep -ril "openclaw" . --exclude-dir=.git
```

For every file returned:

**If it's the `.openclaw/` directory or any file inside it:** delete it entirely.
```bash
rm -rf .openclaw
```

**If it's a Markdown file with an openclaw section**, surgically remove these patterns:
- The openclaw badge: any `<a href="#openclaw">` anchor or `img.shields.io/badge/OpenClaw` img tag (and its surrounding `<a>` wrapper if it's standalone)
- The `### OpenClaw` section heading through the end of that section (typically 3-4 lines: heading, instruction text, the fetch URL line)
- The table row `| OpenClaw | ...` in the platform compatibility table

The README.md and READMEs/ translated files all follow the same structure, so the same patterns apply across all of them.

**If it's a docs file** (e.g., `docs/superpowers/`), remove the openclaw-specific lines (references to openclaw paths, openclaw install instructions) while preserving surrounding context.

After editing, confirm no openclaw references remain:
```bash
grep -ril "openclaw" . --exclude-dir=.git
```

If anything is still found, fix it before continuing.

### 1d. Commit openclaw removals (if any)

If you made any changes in step 1c:
```bash
git add -A
git commit -m "remove openclaw references"
```

---

## Step 2 — Enforce README fork notice and install URLs

### 2a. Check for the fork notice

Open `README.md` and look for this exact `[!NOTE]` callout block near the top
(it should appear just before the `[!TIP]` community thank-you note):

```
> [!NOTE]
> **This is a fork of [Lum1104/Understand-Anything](https://github.com/Lum1104/Understand-Anything).** It exists to make Understand-Anything accessible to organizations whose security policies block installation of software that contains files referencing "openclaw". All openclaw references and files have been removed; everything else is identical to the upstream project. Installation instructions below reference this fork (`benstroud/Understand-Anything`).
```

If it's missing, insert it immediately before the `> [!TIP]` block.

### 2b. Check install URLs

Scan README.md for any occurrence of `Lum1104/Understand-Anything` in a
context that is an install command or INSTALL.md fetch URL. Specifically look for:

- `/plugin marketplace add Lum1104/Understand-Anything` → change to `benstroud/Understand-Anything`
- `raw.githubusercontent.com/Lum1104/Understand-Anything/` in fetch instructions → change to `raw.githubusercontent.com/benstroud/Understand-Anything/`
- `copilot plugin install Lum1104/Understand-Anything:` → change to `benstroud/Understand-Anything:`

Do **not** change `Lum1104` references that are:
- Star history badge/chart URLs
- The license badge URL
- The upstream link inside the fork notice itself
- Links to example repos or other projects by Lum1104

A good heuristic: only update `Lum1104/Understand-Anything` occurrences that
appear inside `` ``` `` code blocks or inline `backtick` spans that are
installation instructions.

### 2c. Commit README fixes (if any)

If you changed anything in steps 2a or 2b:
```bash
git add README.md
git commit -m "chore: maintain fork notice and install URLs"
```

---

## Step 3 — Report

Summarize what happened:

```
## Fork sync complete

**Upstream commits merged:** N (or "already up to date")
**Openclaw cleanup:**
  - Files deleted: [list or "none"]
  - Files edited: [list or "none"]
**README:**
  - Fork notice: present / added
  - Install URLs: correct / N URLs updated
**Commits created:** [list commit hashes + messages, or "none"]
```
