---
name: branch-naming
description: Choose a conventional, descriptive Git branch name for the current task.
---

# Branch naming

When asked to name or rename a branch, describe the primary change with a short, lowercase kebab-case name. Use a conventional change type such as `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, or `revert`.

For regular Git branches, use `<type>/<description>`, for example `feat/contact-form` or `fix/language-redirect`. Include an issue number when one is provided and useful.

For the Copilot app's branch-renaming feature, use `<type>-<description>` instead: the app accepts a kebab-case name and applies its configured prefix automatically. Do not include that prefix yourself. Respect the app's length limit.

Honor an explicitly requested branch name. Do not rename a branch merely because this skill is available, or rename a shared or protected branch without authorization.
