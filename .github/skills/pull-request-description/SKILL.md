---
name: pull-request-description
description: Prepare a pull request description using the repository template and verified evidence.
---

# Pull request description

When asked to create or update a pull request description:

1. Inspect the relevant changes and read `.github/PULL_REQUEST_TEMPLATE.md` and any applicable named templates. Use the appropriate template, preserving its headings, comments, checklist items, and order.
2. Explain why the change is needed and summarize the approach. Mark changed areas accurately and check acceptance criteria only when verified; leave non-applicable items unchecked and explain why.
3. Follow the template's validation order. Inspect the documented commands, run applicable checks when feasible, and record each exact command and its observed result. State clearly what was not run and why; never claim an unverified check passed.
4. Mark testing notes only when verified. Include relevant evidence for visual or browser changes, and describe remaining limitations and follow-up work.
5. Keep the description concise and accurate. Do not invent business facts, test results, or source issue references.
