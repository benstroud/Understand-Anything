---
name: sync-fork
description: >
  Syncs this fork (benstroud/Understand-Anything) with the upstream repo
  (Lum1104/Understand-Anything), then enforces the fork's invariants:
  no openclaw files or references, all install/fetch URLs point to the fork,
  and README.md contains the correct fork notice.

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
and consistent in pointing users to the fork rather than the upstream.

## Context

- Upstream: `https://github.com/Lum1104/Understand-Anything.git`
- Fork: `https://github.com/benstroud/Understand-Anything`
- Fork invariants:
  1. No openclaw files, directories, or text references anywhere
  2. README.md contains the fork notice callout (see Step 2)
  3. All install commands, INSTALL.md clone/fetch URLs, plugin.json homepage/repository fields, and homepage source files reference `benstroud/Understand-Anything`

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

**If it's a docs file** (e.g., `docs/superpowers/`), remove the openclaw-specific lines while preserving surrounding context.

After editing, confirm no openclaw references remain:
```bash
grep -ril "openclaw" . --exclude-dir=.git
```

Expected survivors (these are intentional and should NOT be removed):
- `README.md` — the fork notice text that explains the fork's purpose
- `.claude/skills/sync-fork/SKILL.md` — this file's own instructions

If anything else is found, fix it before continuing.

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

### 2b. Check install URLs across the full codebase

After merging upstream, scan for any `Lum1104/Understand-Anything` references that
should point to the fork instead. Run:

```bash
grep -rn "Lum1104/Understand-Anything" . --exclude-dir=.git \
  --include="*.md" --include="*.json" --include="*.ts" --include="*.tsx" \
  --include="*.astro" --include="*.html"
```

**Update to `benstroud/Understand-Anything`** wherever found in:
- `/plugin marketplace add ...` commands (README.md, READMEs/*.md, docs/)
- `raw.githubusercontent.com/Lum1104/...` fetch URLs in install instructions
- `copilot plugin install Lum1104/...` commands
- `git clone https://github.com/Lum1104/Understand-Anything.git` in INSTALL.md files (`.codex/`, `.opencode/`, `.gemini/`, `.pi/`, `.vscode/`, `.antigravity/`)
- `homepage` and `repository` fields in plugin.json files (`.claude-plugin/`, `.copilot-plugin/`, `.cursor-plugin/`, `understand-anything-plugin/.claude-plugin/`)
- `githubUrl` variable assignments in homepage Astro components (`homepage/src/components/`)
- Issue/bug report URL in `understand-anything-plugin/packages/dashboard/src/components/WarningBanner.tsx`

**Leave as `Lum1104/Understand-Anything`**:
- `api.star-history.com` badge and chart URLs (they track the upstream's star count)
- The upstream link inside the fork notice itself (`README.md` line with `[!NOTE]`)
- Issue/PR number links in `docs/superpowers/` (e.g. `issues/61`) — they reference specific upstream issues

### 2c. Commit all fixes (if any)

If you changed anything in steps 2a or 2b:
```bash
git add -A
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
**URL/install fixes:**
  - Files updated: [list or "none"]
**README:**
  - Fork notice: present / added
**Commits created:** [list commit hashes + messages, or "none"]
```
