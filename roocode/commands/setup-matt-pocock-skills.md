---
description: Configure this repo for the engineering skills — set up its issue tracker, triage label vocabulary, domain doc layout, and workflow file paths. Run once before first use of the other engineering skills.
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

Then write the config files to the agent config path chosen in Section D, using the seed templates below as a starting point:

- **Issue Tracker: GitHub** — GitHub issue tracker
- **Issue Tracker: GitLab** — GitLab issue tracker
- **Issue Tracker: Local Markdown** — local-markdown issue tracker
- **Triage Labels** — label mapping (only if `triage` is installed)
- **Domain Docs** — domain doc consumer rules + layout

For "other" issue trackers, write `issue-tracker.md` from scratch using the user's description.

### 5. Done

Tell the user the setup is complete and which engineering skills will now read from these files. Mention they can edit the config files directly later — re-running this skill is only necessary if they want to switch issue trackers, change the file layout, or restart from scratch.

---

# Seed Templates

## Domain Docs

How the engineering skills should consume this repo's domain documentation when exploring the codebase.

### Before exploring, read these

The paths below are placeholders — read the actual on-disk paths from the **Workflow paths** table in this repo's `CLAUDE.md`/`AGENTS.md`.

- **Domain glossary** (`<glossary>`): a single `CONTEXT.md`, or
- **Context map** (`<context-map>`): a `CONTEXT-MAP.md` if it exists — it points at one `CONTEXT.md` per context. Read each one relevant to the topic.
- **ADRs** (`<adrs>`): read ADRs that touch the area you're about to work in. In multi-context repos, also check `src/<context>/<adrs>` for context-scoped decisions.

If any of these files don't exist, **proceed silently**. Don't flag their absence; don't suggest creating them upfront. The domain-modeling skill (reached via `/grill-with-docs` and `/improve-codebase-architecture`) creates them lazily when terms or decisions actually get resolved.

### File structure

The on-disk layout depends on the workflow file layout chosen during setup (Centralized, Hybrid, or Distributed). The examples below show the **Distributed** layout, where the glossary and context map sit at the repo root; in other layouts they live under `matt-workflow/` — the Workflow paths table holds the exact paths.

Single-context repo (most repos):

```
/
├── CONTEXT.md
├── docs/adr/
│   ├── 0001-event-sourced-orders.md
│   └── 0002-postgres-for-write-model.md
└── src/
```

Multi-context repo (presence of a context map at the root):

```
/
├── CONTEXT-MAP.md
├── docs/adr/                          ← system-wide decisions
└── src/
    ├── ordering/
    │   ├── CONTEXT.md
    │   └── docs/adr/                  ← context-specific decisions
    └── billing/
        ├── CONTEXT.md
        └── docs/adr/
```

### Use the glossary's vocabulary

When your output names a domain concept (in an issue title, a refactor proposal, a hypothesis, a test name), use the term as defined in the domain glossary. Don't drift to synonyms the glossary explicitly avoids.

If the concept you need isn't in the glossary yet, that's a signal — either you're inventing language the project doesn't use (reconsider) or there's a real gap (note it for the domain-modeling skill).

### Flag ADR conflicts

If your output contradicts an existing ADR, surface it explicitly rather than silently overriding:

> _Contradicts ADR-0007 (event-sourced orders) — but worth reopening because…_

## Triage Labels

The skills speak in terms of five canonical triage roles. This file maps those roles to the actual label strings used in this repo's issue tracker.

| Label in mattpocock/skills | Label in our tracker | Meaning                                  |
| -------------------------- | -------------------- | ---------------------------------------- |
| `needs-triage`             | `needs-triage`       | Maintainer needs to evaluate this issue  |
| `needs-info`               | `needs-info`         | Waiting on reporter for more information |
| `ready-for-agent`          | `ready-for-agent`    | Fully specified, ready for an AFK agent  |
| `ready-for-human`          | `ready-for-human`    | Requires human implementation            |
| `wontfix`                  | `wontfix`            | Will not be actioned                     |

When a skill mentions a role (e.g. "apply the AFK-ready triage label"), use the corresponding label string from this table.

Edit the right-hand column to match whatever vocabulary you actually use.

## Issue Tracker: GitHub

Issues and PRDs for this repo live as GitHub issues. Use the `gh` CLI for all operations.

### Conventions

- **Create an issue**: `gh issue create --title "..." --body "..."`. Use a heredoc for multi-line bodies.
- **Read an issue**: `gh issue view <number> --comments`, filtering comments by `jq` and also fetching labels.
- **List issues**: `gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'` with appropriate `--label` and `--state` filters.
- **Comment on an issue**: `gh issue comment <number> --body "..."`
- **Apply / remove labels**: `gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **Close**: `gh issue close <number> --comment "..."`

Infer the repo from `git remote -v` — `gh` does this automatically when run inside a clone.

### Pull requests as a triage surface

**PRs as a request surface: no.** _(Set to `yes` if this repo treats external PRs as feature requests; `/triage` reads this flag.)_

When set to `yes`, PRs run through the same labels and states as issues, using the `gh pr` equivalents:

- **Read a PR**: `gh pr view <number> --comments` and `gh pr diff <number>` for the diff.
- **List external PRs for triage**: `gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments` then keep only `authorAssociation` of `CONTRIBUTOR`, `FIRST_TIME_CONTRIBUTOR`, or `NONE` (drop `OWNER`/`MEMBER`/`COLLABORATOR`).
- **Comment / label / close**: `gh pr comment`, `gh pr edit --add-label`/`--remove-label`, `gh pr close`.

GitHub shares one number space across issues and PRs, so a bare `#42` may be either — resolve with `gh pr view 42` and fall back to `gh issue view 42`.

### When a skill says "publish to the issue tracker"

Create a GitHub issue.

### When a skill says "fetch the relevant ticket"

Run `gh issue view <number> --comments`.

### Wayfinding operations

Used by `/wayfinder`. The **map** is a single issue with **child** issues as tickets.

- **Map**: a single issue labelled `wayfinder:map`, holding the Notes / Decisions-so-far / Fog body. `gh issue create --label wayfinder:map`.
- **Child ticket**: an issue linked to the map as a GitHub sub-issue (`gh api` on the sub-issues endpoint). Where sub-issues aren't enabled, add the child to a task list in the map body and put `Part of #<map>` at the top of the child body. Labels: `wayfinder:<type>` (`research`/`prototype`/`grilling`/`task`). Once claimed, the ticket is assigned to the driving dev.
- **Blocking**: GitHub's **native issue dependencies** — the canonical, UI-visible representation. Add an edge with `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`, where `<blocker-db-id>` is the blocker's numeric **database id** (`gh api repos/<owner>/<repo>/issues/<n> --jq .id`, _not_ the `#number` or `node_id`). GitHub reports `issue_dependencies_summary.blocked_by` (open blockers only — the live gate). Where dependencies aren't available, fall back to a `Blocked by: #<n>, #<n>` line at the top of the child body. A ticket is unblocked when every blocker is closed.
- **Frontier query**: list the map's open children (`gh issue list --state open`, scoped to the map's sub-issues / task list), drop any with an open blocker (`issue_dependencies_summary.blocked_by > 0`, or an open issue in the `Blocked by` line) or an assignee; first in map order wins.
- **Claim**: `gh issue edit <n> --add-assignee @me` — the session's first write.
- **Resolve**: `gh issue comment <n> --body "<answer>"`, then `gh issue close <n>`, then append a context pointer (gist + link) to the map's Decisions-so-far.

## Issue Tracker: GitLab

Issues and PRDs for this repo live as GitLab issues. Use the [`glab`](https://gitlab.com/gitlab-org/cli) CLI for all operations.

### Conventions

- **Create an issue**: `glab issue create --title "..." --description "..."`. Use a heredoc for multi-line descriptions. Pass `--description -` to open an editor.
- **Read an issue**: `glab issue view <number> --comments`. Use `-F json` for machine-readable output.
- **List issues**: `glab issue list -F json` with appropriate `--label` filters.
- **Comment on an issue**: `glab issue note <number> --message "..."`. GitLab calls comments "notes".
- **Apply / remove labels**: `glab issue update <number> --label "..."` / `--unlabel "..."`. Multiple labels can be comma-separated or by repeating the flag.
- **Close**: `glab issue close <number>`. `glab issue close` does not accept a closing comment, so post the explanation first with `glab issue note <number> --message "..."`, then close.
- **Merge requests**: GitLab calls PRs "merge requests". Use `glab mr create`, `glab mr view`, `glab mr note`, etc. — the same shape as `gh pr ...` with `mr` in place of `pr` and `note`/`--message` in place of `comment`/`--body`.

Infer the repo from `git remote -v` — `glab` does this automatically when run inside a clone.

### Merge requests as a triage surface

**MRs as a request surface: no.** _(Set to `yes` if this repo treats external merge requests as feature requests; `/triage` reads this flag.)_

When set to `yes`, MRs run through the same labels and states as issues, using the `glab mr` equivalents:

- **Read an MR**: `glab mr view <number> --comments` and `glab mr diff <number>` for the diff.
- **List external MRs for triage**: `glab mr list -F json`, then keep only MRs whose author is not a project member/owner (a contributor's MR, not a maintainer's in-flight work).
- **Comment / label / close**: `glab mr note`, `glab mr update --label`/`--unlabel`, `glab mr close`.

Unlike GitHub, GitLab numbers issues and MRs separately, so `#42` is unambiguous once you know which surface the maintainer means.

### When a skill says "publish to the issue tracker"

Create a GitLab issue.

### When a skill says "fetch the relevant ticket"

Run `glab issue view <number> --comments`.

### Wayfinding operations

Used by `/wayfinder`. The **map** is a single issue with **child** issues as tickets.

- **Map**: a single issue labelled `wayfinder:map`, holding the Notes / Decisions-so-far / Fog body. `glab issue create --label wayfinder:map`. (On GitLab tiers with native epics, an epic may hold the map instead; a labelled issue works everywhere.)
- **Child ticket**: an issue carrying `Part of #<map>` at the top of its description and labels `wayfinder:<type>` (`research`/`prototype`/`grilling`/`task`). Once claimed, the ticket is assigned to the driving dev.
- **Blocking**: GitLab's **native blocking link** — the canonical, UI-visible representation. Add it with the `/blocked_by #<n>` quick action, posted as a note (`glab issue note <child> --message "/blocked_by #<blocker>"`). Native blocking links are a Premium/Ultimate feature; on the free tier (or where unavailable) fall back to a `Blocked by: #<n>, #<n>` line at the top of the description. A ticket is unblocked when every blocker is closed.
- **Frontier query**: `glab issue list -F json` scoped to the map's children, drop any with an open blocker — a native `blocked_by` link to an open issue (`glab api projects/:id/issues/:iid/links`), or an open issue in the `Blocked by` line — or an assignee; first in map order wins.
- **Claim**: `glab issue update <n> --assignee @me` — the session's first write.
- **Resolve**: `glab issue note <n> --message "<answer>"`, then `glab issue close <n>`, then append a context pointer (gist + link) to the map's Decisions-so-far.

## Issue Tracker: Local Markdown

Issues and specs (you may know a spec as a PRD) for this repo live as markdown files in the **local issues/PRDs** directory — the path recorded in the Workflow paths table in this repo's `CLAUDE.md`/`AGENTS.md`. This file uses `<scratch>` as a placeholder; substitute the real path when reading or writing.

### Conventions

- One feature per directory: `<scratch>/<feature-slug>/`
- The spec is `<scratch>/<feature-slug>/spec.md`
- Implementation issues are one file per ticket at `<scratch>/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01` — never a single combined tickets file
- Triage state is recorded as a `Status:` line near the top of each issue file (see `triage-labels.md` for the role strings)
- Comments and conversation history append to the bottom of the file under a `## Comments` heading

### When a skill says "publish to the issue tracker"

Create a new file under `<scratch>/<feature-slug>/` (creating the directory if needed).

### When a skill says "fetch the relevant ticket"

Read the file at the referenced path. The user will normally pass the path or the issue number directly.

### Wayfinding operations

Used by `/wayfinder`. The **map** is a file with one **child** file per ticket.

- **Map**: `<scratch>/<effort>/map.md` — the Notes / Decisions-so-far / Fog body.
- **Child ticket**: `<scratch>/<effort>/issues/NN-<slug>.md`, numbered from `01`, with the question in the body. A `Type:` line records the ticket type (`research`/`prototype`/`grilling`/`task`); a `Status:` line records `claimed`/`resolved`.
- **Blocking**: a `Blocked by: NN, NN` line near the top. A ticket is unblocked when every file it lists is `resolved`.
- **Frontier**: scan `<scratch>/<effort>/issues/` for files that are open, unblocked, and unclaimed; first by number wins.
- **Claim**: set `Status: claimed` and save before any work.
- **Resolve**: append the answer under an `## Answer` heading, set `Status: resolved`, then append a context pointer (gist + link) to the map's Decisions-so-far in `map.md`.
