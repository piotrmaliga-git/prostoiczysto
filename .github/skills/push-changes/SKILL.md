---
name: push-changes
description: Push Git changes only after the application build succeeds.
---

# Push changes

When asked to push changes:

1. Run `npm run build` in the repository-root `prostoiczysto/` application directory before pushing.
2. Push only if the build succeeds. If the application or build command is unavailable, or the build fails, report the blocker and do not push.
3. Use the requested remote and branch when specified; otherwise follow the current branch's configured upstream.
