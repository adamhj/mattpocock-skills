---
name: setup-matt-pocock-skills
description: Configure this repo for the engineering skills — set up its issue tracker, triage label vocabulary, domain doc layout, and workflow file paths. Run once before first use of the other engineering skills.
disable-model-invocation: true
---

# Setup Matt Pocock's Skills

Scaffold the per-repo configuration that the engineering skills assume:

- **Issue tracker** — where issues live (GitHub by default; local markdown is also supported out of the box)
- **Triage labels** — the strings used for the five canonical triage roles
- **Domain docs** — single-context or multi-context layout
- **Workflow paths** — where all workflow artifacts live on disk

This is a prompt-driven skill, not a deterministic script. Explore, present what you found, confirm with the user, then write.

## Process

### 1. Explore

Look at the current repo to understand its starting state. Read whatever exists; don't assume:

- `git remote -v` and `.git/config` — is this a GitHub repo? Which one?
- `AGENTS.md` and `CLAUDE.md` at the repo root — does either exist? Is there already an `## Agent skills` section in either?
- `CONTEXT.md` and `CONTEXT-MAP.md` at the repo root, and `matt-workflow/domain/` — either may hold the domain glossary
- `docs/adr/` and any `src/*/docs/adr/` directories, and `matt-workflow/domain/adr/` — either may hold ADRs
- Any prior setup output — check both `docs/agents/` and `matt-workflow/config/` for existing config files
- `.scratch/` or `matt-workflow/scratch/` — sign that a local-markdown issue tracker convention is already in use
- Is the `triage` skill installed? (a `triage` skill folder alongside this one, or `triage` in your available skills.) This decides whether Section B runs at all.
- Monorepo signals — a `pnpm-workspace.yaml`, a `workspaces` field in `package.json`, or a populated `packages/*` with its own `src/`. Present only in a genuinely large multi-package repo; their absence means single-context, which is almost every repo.

### 2. Present findings and ask

Summarise what's present and what's missing. Then walk the user through the four decisions **one at a time** — present a section, get the user's answer, then move to the next. Don't dump all four at once.

Lead each section with the recommended answer so the user can accept it in a word. Give a one-line explainer only when the choice genuinely branches; skip the section entirely when exploration already settled it (Section B when `triage` isn't installed, Section C when there's no monorepo).

**Section A — Issue tracker.**

> Explainer: The "issue tracker" is where issues live for this repo. Skills like `to-tickets`, `triage`, `to-spec`, and `qa` read from and write to it — they need to know whether to call `gh issue create`, write a markdown file locally, or follow some other workflow you describe. Pick the place you actually track work for this repo.

Default posture: these skills were designed for GitHub. If a `git remote` points at GitHub, propose that. If a `git remote` points at GitLab (`gitlab.com` or a self-hosted host), propose GitLab. Otherwise (or if the user prefers), offer:

- **GitHub** — issues live in the repo's GitHub Issues (uses the `gh` CLI)
- **GitLab** — issues live in the repo's GitLab Issues (uses the [`glab`](https://gitlab.com/gitlab-org/cli) CLI)
- **Local markdown** — issues live as files in this repo (good for solo projects or repos without a remote). The exact on-disk location is settled in Section D.
- **Other** (Jira, Linear, etc.) — ask the user to describe the workflow in one paragraph; the skill will record it as freeform prose

Record the choice in the issue tracker config file (written to the agent config path chosen in Section D). The GitHub and GitLab templates carry a "PRs as a request surface" flag, defaulted **off** — leave it off and don't raise it; a user who wants external PRs in the triage queue can flip the flag in the file later.

**Section B — Triage label vocabulary.** Skip this section entirely if the `triage` skill isn't installed (exploration told you) — an uninstalled skill needs no labels.

If it is installed, ask exactly one question:

> Do you want to keep the default triage labels? (recommended: **yes**)

The defaults are the five canonical roles, each label string equal to its name: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. On **yes**, write them as-is. Only if the user says no — usually because their tracker already uses other names (e.g. `bug:triage` for `needs-triage`) — collect the overrides so `triage` applies existing labels instead of creating duplicates.

**Section C — Domain docs.** Default to **single-context** — one domain glossary plus an ADR directory. This fits almost every repo; write it without asking.

Offer **multi-context** — a context map pointing to one domain glossary per context — only when exploration found monorepo signals. Then confirm which layout they want.

The exact on-disk paths for the glossary, context map, and ADRs are settled in Section D, which applies to all workflow artifacts at once.

**Section D — Workflow file layout.** All engineering skills read artifact paths from a **Workflow paths** table in `CLAUDE.md`/`AGENTS.md` (written in step 4), instead of hardcoding them. This section decides where every workflow artifact lives on disk.

> Explainer: The engineering skills produce and consume several artifacts — the domain glossary, context map, ADRs, the out-of-scope knowledge base, agent config files, handoff documents, and local issues/specs. Where they live on disk is a per-repo choice. Picking a layout once, here, means every skill can read the same paths from one table instead of each guessing.

Ask which layout they want. Default posture: **Centralized**, unless exploration found existing artifacts in conventional locations worth preserving — in that case recommend the layout that keeps them where they are.

- **Centralized** — all workflow artifacts live under `matt-workflow/`:

  ```
  matt-workflow/
  ├── domain/
  │   ├── CONTEXT.md
  │   ├── CONTEXT-MAP.md
  │   └── adr/
  ├── out-of-scope/
  ├── config/
  ├── handoffs/
  └── scratch/
  ```

- **Hybrid** — `CONTEXT.md` and `CONTEXT-MAP.md` stay at the repo root (community convention), everything else under `matt-workflow/`:

  ```
  /
  ├── CONTEXT.md
  ├── CONTEXT-MAP.md
  └── matt-workflow/
      ├── adr/
      ├── out-of-scope/
      ├── config/
      ├── handoffs/
      └── scratch/
  ```

- **Distributed** — files stay in their conventional locations; only handoff docs and local issues move under `matt-workflow/`:

  ```
  /
  ├── CONTEXT.md
  ├── CONTEXT-MAP.md
  ├── .out-of-scope/
  ├── docs/
  │   ├── adr/
  │   └── agents/
  └── matt-workflow/
      ├── handoffs/
      └── scratch/
  ```

### 3. Confirm and edit

Show the user a draft of:

- The `## Agent skills` block to add to whichever of `CLAUDE.md` / `AGENTS.md` is being edited (see step 4 for selection rules)
- The contents of the config files (issue-tracker, triage-labels, domain) that will be written to the agent config path chosen in Section D

Let them edit before writing.

### 4. Write

**Pick the file to edit:**

- If `CLAUDE.md` exists, edit it.
- Else if `AGENTS.md` exists, edit it.
- If neither exists, ask the user which one to create — don't pick for them.

Never create `AGENTS.md` when `CLAUDE.md` already exists (or vice versa) — always edit the one that's already there.

If an `## Agent skills` block already exists in the chosen file, update its contents in-place rather than appending a duplicate. Don't overwrite user edits to the surrounding sections.

The block:

```markdown
## Agent skills

### Workflow paths

All engineering skills read artifact paths from this table. Do not hardcode paths elsewhere.

| Artifact | Path |
|---|---|
| Domain glossary | `<path>` |
| Context map | `<path>` |
| ADRs | `<path>` |
| Out-of-scope KB | `<path>` |
| Agent config | `<path>` |
| Handoff docs | `<path>` |
| Local issues/PRDs | `<path>` |

### Issue tracker

[one-line summary]. See `<agent-config-path>/issue-tracker.md`.

### Triage labels

[one-line summary]. See `<agent-config-path>/triage-labels.md`.

### Domain docs

[one-line summary of layout — "single-context" or "multi-context"]. See `<agent-config-path>/domain.md`.
```

Include the `### Triage labels` sub-block, and write `triage-labels.md` to the agent config path, only when `triage` is installed and Section B ran. When it isn't, both are omitted.

Then write the config files to the agent config path chosen in Section D, using the seed templates in this skill folder as a starting point:

- [issue-tracker-github.md](./issue-tracker-github.md) — GitHub issue tracker
- [issue-tracker-gitlab.md](./issue-tracker-gitlab.md) — GitLab issue tracker
- [issue-tracker-local.md](./issue-tracker-local.md) — local-markdown issue tracker
- [triage-labels.md](./triage-labels.md) — label mapping (only if `triage` is installed)
- [domain.md](./domain.md) — domain doc consumer rules + layout

For "other" issue trackers, write `issue-tracker.md` from scratch using the user's description.

### 5. Done

Tell the user the setup is complete and which engineering skills will now read from these files. Mention they can edit the config files directly later — re-running this skill is only necessary if they want to switch issue trackers, change the file layout, or restart from scratch.
