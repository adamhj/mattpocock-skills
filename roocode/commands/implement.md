---
description: "Implement a piece of work based on a spec or set of tickets."
---

Implement the work described by the user in the spec or tickets.

Use the 'tdd' skill where possible, at pre-agreed seams.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Before committing, record `git rev-parse HEAD` as `<review-base>`.

Commit your work to the current branch with a descriptive commit message, then inform the user:

```
Implementation complete and committed to `<branch-name>` at `<commit-sha>`.

To review, run in a **fresh session**: `/code-review <review-base> <issue-reference>`
(e.g., `/code-review abc123 #42` or `/code-review abc123 path/to/issue.md`)
```

**Do not update documentation (specs, issues, ADRs) at this stage.** That belongs to the review's close-out, once the work has passed.
