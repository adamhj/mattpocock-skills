# Domain Docs

How the engineering skills should consume this repo's domain documentation when exploring the codebase.

## Before exploring, read these

The paths below are placeholders — read the actual on-disk paths from the **Workflow paths** table in this repo's `CLAUDE.md`/`AGENTS.md`.

- **Domain glossary** (`<glossary>`): a single `CONTEXT.md`, or
- **Context map** (`<context-map>`): a `CONTEXT-MAP.md` if it exists — it points at one `CONTEXT.md` per context. Read each one relevant to the topic.
- **ADRs** (`<adrs>`): read ADRs that touch the area you're about to work in. In multi-context repos, also check `src/<context>/<adrs>` for context-scoped decisions.

If any of these files don't exist, **proceed silently**. Don't flag their absence; don't suggest creating them upfront. The `/domain-modeling` skill (reached via `/grill-with-docs` and `/improve-codebase-architecture`) creates them lazily when terms or decisions actually get resolved.

## File structure

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

## Use the glossary's vocabulary

When your output names a domain concept (in an issue title, a refactor proposal, a hypothesis, a test name), use the term as defined in the domain glossary. Don't drift to synonyms the glossary explicitly avoids.

If the concept you need isn't in the glossary yet, that's a signal — either you're inventing language the project doesn't use (reconsider) or there's a real gap (note it for `/domain-modeling`).

## Flag ADR conflicts

If your output contradicts an existing ADR, surface it explicitly rather than silently overriding:

> _Contradicts ADR-0007 (event-sourced orders) — but worth reopening because…_
