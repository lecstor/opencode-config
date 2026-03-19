---
description: commit changes to Git
---

Commit changes to Git in organised, scoped commits with clear descriptions of what and why changes were made.

Review all staged and unstaged changes, then group files by logical relatedness — one commit per logical change. Never bundle unrelated changes into a single commit for convenience; if changes touch separate concerns (e.g. a bug fix and a refactor, or a new feature and a documentation update), split them into separate commits.

Use scoped conventional commit messages in the format `type(scope): subject`, choosing the type that best describes the nature of the change:

- `feat` — a new feature or capability
- `fix` — a bug fix
- `refactor` — restructuring code without changing behaviour
- `chore` — maintenance tasks, dependency updates, tooling changes
- `docs` — documentation only
- `test` — adding or updating tests
- `security` — security-related fixes or hardening

The scope should identify the area of the codebase affected (e.g. `feat(auth): add OAuth2 login flow`). The subject should be a concise imperative description of what changed.

When the motivation behind a change is non-obvious, include a commit body that briefly explains *why* the change was made — not just what changed. Use `git commit -m "subject" -m "body"` to provide both. If the reasoning is self-evident from the subject line alone, the body can be omitted.
