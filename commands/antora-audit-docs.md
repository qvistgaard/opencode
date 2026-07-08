---
description: Audit Antora documentation against the antora-docs skill and application behavior.
agent: build
---

Use the `antora-docs` skill.

Audit the current workspace documentation against the Antora documentation rules in that skill.

Scope:

- Find Antora docs roots, including `antora.yml`, `modules/*/nav.adoc`, and `modules/*/pages/**/*.adoc`.
- Inspect the existing documentation structure before judging it.
- Treat `Documentation Information Architecture` in the skill as the source of truth.
- Verify that `nav.adoc` follows that information architecture.
- Verify page structure, AsciiDoc style, code block conventions, admonition use, and the rewrite checklist.
- Verify that feature and reference material is organized under the required topics from the skill.
- Analyze the project source code, configuration, tests, and build files enough to verify that documented behavior matches how the application actually works.
- Do not rely on documentation claims without checking implementation when the workspace contains relevant source.

Output:

- Findings ordered by severity.
- Each finding must include file path and line number when possible.
- Explain which `antora-docs` rule is violated.
- Explain any mismatch between documentation and application behavior.
- Suggest an exact fix for each finding.
- Include missing pages, misplaced pages, and required `nav.adoc` changes.
- Include assumptions and focused questions when behavior cannot be verified from code.

Do not edit files unless explicitly asked.
Do not invent product behavior. If source behavior is unclear, say what code or owner knowledge is needed.
