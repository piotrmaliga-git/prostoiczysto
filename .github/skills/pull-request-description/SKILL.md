---
name: pull-request-description
description: Prepare a conventional pull request title and a template-based description with verified evidence.
---

# Pull request title and description

When asked to create or update a pull request:

1. Inspect the relevant changes and read `.github/PULL_REQUEST_TEMPLATE.md` and any applicable named templates. Use the appropriate template, preserving its headings, comments, checklist items, and order.
2. Choose a conventional title describing the primary change: `<type>(<optional-scope>): <description>`. Use the commit types and header rules in `orchestration/docs/technical/technical-requirements.md`; for example, `docs: add implementation roadmap`. Keep the title concise and reflect the whole pull request rather than just one commit.
3. Explain why the change is needed and summarize the approach. Mark changed areas accurately and check acceptance criteria only when verified; leave non-applicable items unchecked and explain why.
4. Follow the template's validation order. Inspect the documented commands, run applicable checks when feasible, and record each exact command and its observed result. State clearly what was not run and why; never claim an unverified check passed.
5. Mark testing notes only when verified. Include relevant evidence for visual or browser changes, and describe remaining limitations and follow-up work.
6. Keep the description concise and accurate. Do not invent business facts, test results, or source issue references.
