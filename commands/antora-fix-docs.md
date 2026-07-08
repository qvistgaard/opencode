---
description: Fix Antora documentation to match the antora-docs skill and application behavior.
agent: build
---

Use the `antora-docs` skill.

Fix the current workspace documentation so it follows the Antora documentation rules in that skill and matches how the application actually works.

Scope:

- Find Antora docs roots, including `antora.yml`, `modules/*/nav.adoc`, and `modules/*/pages/**/*.adoc`.
- Inspect the existing documentation structure before changing it.
- Treat `Documentation Information Architecture` in the skill as the source of truth.
- Update `nav.adoc` to follow that information architecture while preserving valid existing project-specific pages.
- Fix page structure, AsciiDoc style, code block conventions, admonition use, and rewrite checklist issues.
- Organize feature and reference material under the required topics from the skill.
- Analyze the project source code, configuration, tests, and build files enough to verify that documented behavior matches how the application actually works.
- Do not rely on documentation claims without checking implementation when the workspace contains relevant source.

Editing rules:

- Keep changes small and coherent.
- Preserve existing project terminology when it is correct.
- Do not invent product behavior.
- Do not add broad placeholder documentation.
- If behavior cannot be verified from code, use conditional wording only when appropriate or stop and ask one focused question.
- Prefer fixing existing Antora pages over creating new pages unless the information architecture clearly requires a missing page.
- When creating pages, add them to `nav.adoc`.

After editing:

- Summarize changed files.
- List behavior claims verified against source code, configuration, tests, or build files.
- List remaining assumptions or questions.
- Mention any docs build or validation command run, or say why none was run.
