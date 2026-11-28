---
name: commit-changes
description: Create Git commits using this repository's conventional commit message rules.
---

# Commit changes

When asked to commit changes:

1. Inspect the workspace status and staged and unstaged changes. Stage the files requested by the user; stage everything only if they ask for all changes.
2. Look for coherent groups by purpose, change type, and affected files. When appropriate, stage and commit groups separately to create a clear history. Keep dependent changes together, including their tests and documentation; preserve existing staged changes and follow any explicit user instructions about grouping.
3. Write each commit message conforming to `@commitlint/config-conventional`. Use one of these types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, or `revert`. Keep the header at most 200 characters.
4. Include any required commit trailers and let the local `commit-msg` hook validate each message when configured. Do not bypass the hook.
5. Verify the commits and workspace status. Do not push unless explicitly requested.

The Commitlint dependencies and local hook are to be added when the application and package manifest are scaffolded, as described in `orchestration/docs/technical/technical-requirements.md`.
