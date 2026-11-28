---
name: push-changes
description: Push Git changes after the application build succeeds, when the application exists.
---

# Push changes

When asked to push changes:

1. If the application has not been scaffolded, report that the build is unavailable and proceed without it.
2. Once the application exists, run `npm run build` from the repository root before pushing. If the build command is unavailable or the build fails, report the blocker and do not push.
3. Use the requested remote and branch when specified; otherwise follow the current branch's configured upstream.
